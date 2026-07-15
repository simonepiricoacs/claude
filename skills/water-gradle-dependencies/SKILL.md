---
name: water-gradle-dependencies
description: Knowledge base for managing additional dependencies in Water Framework Gradle projects via the WaterWorkspaceGradlePlugin. Covers the implementationInclude / implementationIncludeTransitive plugin configurations, the manual includeInJar + ShadowJar pattern, how each bundles classes into the jar, the Atteo ClassIndex annotation merge, exposing a ShadowJar to sibling projects, and the common build gotchas (annotation-merge crash, transitive duplication, test-utils leak). Use when adding/bundling dependencies in any build.gradle of a Water module, designing a distribution/fat jar, or debugging "Could not expand ZIP" / duplicate component / leaked dependency build errors.
allowed-tools: Read, Glob, Grep
---

You are an expert on the Water Framework build system (`WaterWorkspaceGradlePlugin`, applied via `apply plugin: 'it.water.workspace'` in the root `settings.gradle`). You use this knowledge to decide HOW a dependency should be added to a Water module's `build.gradle` and to diagnose packaging problems.

> Ground truth lives in `WaterWorkspaceGradlePlugin/src/main/java/it/water/gradle/plugins/workspace/WaterWorkspaceGradlePlugin.java`. When in doubt, read it — this skill summarizes its behavior, it does not replace it.

---

# WATER GRADLE — DEPENDENCY MANAGEMENT KNOWLEDGE BASE

## Table of Contents
1. [The core question: classpath only, or bundle into the jar?](#1-the-core-question)
2. [The four ways to declare a dependency](#2-the-four-ways)
3. [`implementationInclude` (plugin-provided)](#3-implementationinclude)
4. [`implementationIncludeTransitive` (plugin-provided)](#4-implementationincludetransitive)
5. [`includeInJar` + manual ShadowJar (module-local pattern)](#5-includeinjar--shadowjar)
6. [The Atteo ClassIndex annotation merge](#6-atteo-annotation-merge)
7. [Exposing a ShadowJar to sibling projects in the same build](#7-exposing-a-shadowjar)
8. [Gotchas & failure modes](#8-gotchas)
9. [Decision tree](#9-decision-tree)
10. [Quick reference](#10-quick-reference)

---

## 1. The core question

Before adding any dependency, answer one thing: **does this dependency only need to be on MY classpath, or must its classes be physically embedded inside MY jar?**

- **Classpath only** → standard Gradle `implementation` / `api`. The consumer resolves it transitively from your POM. This is the default and correct choice for normal library modules (`*-api`, `*-model`, `*-service`).
- **Embed into my jar** (fat jar / distribution / module that must be self-contained on a runtime that does not resolve transitive deps) → use one of the *include* mechanisms below.

The Water plugin exists to make the second case ergonomic **and** to keep the Atteo `@FrameworkComponent` index correct when classes from many jars are merged into one.

---

## 2. The four ways

| Mechanism | Provided by | On compile/runtime classpath? | Embedded in jar? | Transitive? | POM declares it? |
|---|---|---|---|---|---|
| `implementation` / `api` | Gradle | yes | no | yes (`api`) / runtime-only (`implementation`) | yes |
| `implementationInclude` | **Water plugin** | **yes** (compile+runtime+test) | **yes** (into the standard `jar`) | **no** | no |
| `implementationIncludeTransitive` | **Water plugin** | yes (compile+runtime+test) | yes (into the standard `jar`) | **yes** | no |
| `includeInJar` + ShadowJar | **module-local** (hand-rolled config) | only if you also extend the classpaths yourself | yes (into the ShadowJar) | configurable on the config | depends on publication |

Key implications of "embedded in jar / POM does NOT declare it":
- A module using *include* mechanisms publishes a **self-contained** jar. Consumers do **not** get those classes as transitive POM dependencies — they get them baked inside the jar.
- This is exactly what `Distribution-spring` (Core + Implementation bundle) and the `Distribution-osgi` uber-jar do.

---

## 3. `implementationInclude`

The plugin registers this configuration on **every** subproject (`WaterWorkspaceGradlePlugin.java`, `INCLUDE_IN_JAR_CONF`). Properties:
- `canBeResolved = true`, `canBeConsumed = false`, **`transitive = false`**.
- The plugin **extends** `compileClasspath`, `runtimeClasspath`, `testCompileClasspath`, `testRuntimeClasspath` from it — so whatever you put here is available to compile and run against, exactly like `implementation`.
- The plugin customizes the standard `Jar` task to:
  - set `duplicatesStrategy = INCLUDE` and `zip64 = true`;
  - **fold the resolved dependency files into the jar** (`jar.from { config.resolve().collect { zipTree(it) } }`);
  - run the **Atteo annotation merge** (see §6).

Use it when: a module must ship a jar that physically contains a specific dependency's classes, and you want it on the compile classpath too. Typical for runtime-flavored modules that publish `from components.java` (e.g. `JpaRepository-spring`, `Rest-spring-api` bundling their `-service` submodule).

```gradle
dependencies {
    // these classes end up INSIDE this module's jar, and on its compile/runtime classpath
    implementationInclude project(":Rest-service")
    implementationInclude 'it.water.repository:Repository-entity:' + project.waterVersion
}

publishing {
    publications {
        water(MavenPublication) { from components.java }   // the fattened standard jar
    }
}
```

`transitive = false` means **only the named artifact** is folded in — not its dependencies. If the named module is itself a fat/self-contained jar, that is exactly what you want (no double-bundling).

---

## 4. `implementationIncludeTransitive`

Same as `implementationInclude` (same jar-fold, same Atteo merge, same classpath extension) **except `transitive = true`** (`INCLUDE_IN_JAR_TRANSITIVE_CONF`). The full dependency graph of what you declare gets folded into the jar.

Use it **sparingly** — it is easy to drag a huge transitive closure (and duplicate classes already bundled elsewhere) into the jar. Prefer `implementationInclude` and add the few extra coordinates you actually need explicitly.

---

## 5. `includeInJar` + ShadowJar

**`includeInJar` is NOT provided by the plugin.** It is a **hand-rolled** resolvable configuration used by modules that build their artifact with the Gradle **Shadow** plugin instead of the plugin-fattened standard jar. Found in `Distribution-spring`, `Distribution-osgi`, `JpaRepository-osgi`.

```gradle
plugins { id "com.github.johnrengelman.shadow" version "7.1.2" }

configurations {
    includeInJar { canBeResolved(true); canBeConsumed(false); transitive false }
}

dependencies {
    includeInJar group: 'it.water.core', name: 'Core-api', version: project.waterVersion
    // ...
}

task("springJar", type: ShadowJar) {
    from sourceSets.main.output
    from project.configurations.includeInJar.collect { it.isDirectory() ? it : zipTree(it) }
    duplicatesStrategy = DuplicatesStrategy.INCLUDE
    mergeServiceFiles { path = '**/META-INF/annotations' }   // Atteo merge, Shadow-native
}

jar { enabled false }            // the ShadowJar is the real artifact
jar.dependsOn(springJar)
```

When to use ShadowJar over `implementationInclude`:
- You need Shadow-only features: relocation, `mergeServiceFiles`, `META-INF/services` concatenation, Spring `spring.factories` / `AutoConfiguration.imports` merging, signature stripping, etc.
- You are building a true **uber-jar / distribution** (OSGi uber-jar, executable Spring fat jar).

Note the Atteo merge here is done by **Shadow's** `mergeServiceFiles { path = '**/META-INF/annotations' }`, NOT by the plugin's mechanism. The two are independent.

---

## 6. Atteo annotation merge

Water uses Atteo ClassIndex for **zero-reflection** `@FrameworkComponent` discovery: each module's compiled jar carries `META-INF/annotations/it.water.core.interceptors.annotations.FrameworkComponent` listing its components. When you merge many jars into one, these index files **must be concatenated**, or components get lost.

Two independent mechanisms do this:
- **Plugin** (`implementationInclude` / `implementationIncludeTransitive`): a `jar.doFirst` (`setupAnnotationMerges`) reads `META-INF/annotations/*` from each included jar and **appends** them into `build/classes/java/main/META-INF/annotations/<name>`.
- **Shadow** (`includeInJar` modules): `mergeServiceFiles { path = '**/META-INF/annotations' }`.

⚠️ **Never overwrite these files — always merge.** A dropped entry = a `@FrameworkComponent` silently missing at runtime (manifests as "ComponentRegistry bean missing" / unresolved `@Inject`).

---

## 7. Exposing a ShadowJar to sibling projects

**Problem:** a module that builds a ShadowJar sets `jar.enabled = false`, and the ShadowJar is usually attached only to the `MavenPublication`. The project's **default outgoing artifact** (`runtimeElements`/`apiElements`, derived from the `jar` task) is therefore empty. So a sibling doing `project(':ThatModule')` resolves **nothing**.

Within the **same build**, you should NOT depend on the module's *published coordinate* either (e.g. `Water-distribution-spring`): the publication may not exist yet at configuration time → build-order short-circuit (compiling against a stale/absent m2 artifact).

**Solution:** expose the ShadowJar through a dedicated **consumable** configuration on the producer, and consume it with a `project(path:, configuration:)` dependency:

```gradle
// PRODUCER (the ShadowJar module, e.g. Distribution-spring)
configurations {
    springDistributionElements { canBeResolved(false); canBeConsumed(true) }
}
artifacts {
    springDistributionElements(springJar)   // expose the ShadowJar task output
}

// CONSUMER (sibling, e.g. Distribution-spring-app)
dependencies {
    implementation project(path: ':Distribution-spring', configuration: 'springDistributionElements')
}
```

**Which configuration to consume with — `implementation` vs `implementationInclude`?**
- If the consumer builds its OWN fat/ShadowJar (so its `jar` is disabled), use plain **`implementation`**. The consumed bundle lands on `runtimeClasspath`, the consumer's ShadowJar embeds it, and the consumer's own Shadow merge handles Atteo. Do **NOT** use `implementationInclude` here — see the crash in §8.
- If the consumer publishes the plugin-fattened standard jar, use `implementationInclude`.

---

## 8. Gotchas

### 8.1 `implementationInclude` on a module that builds a ShadowJar → build crash
Symptom:
```
Could not expand ZIP '.../Water-Distribution-spring-3.0.0.jar'
Caused by: RuntimeException: Error while adding annotation: .../META-INF/annotations/it.water.core.interceptors.annotations.FrameworkComponent
Caused by: java.nio.file.NoSuchFileException: .../build/classes/java/main/META-INF/annotations/...FrameworkComponent
```
Cause: the plugin customizes **every** `Jar` task — and `ShadowJar extends Jar`. So declaring `implementationInclude` triggers the plugin's annotation-merge `doFirst` on the ShadowJar, which appends to `build/classes/java/main/META-INF/annotations/<name>` using `Files.write(..., CREATE, APPEND)`. That call **does not create parent directories**, so if the module has **no `@FrameworkComponent` of its own** (hence no `META-INF/annotations` dir generated by its annotation processor), it throws `NoSuchFileException`.
Fix: for a module whose real artifact is a ShadowJar, consume bundles with plain **`implementation`** (let Shadow do the merge), not `implementationInclude`.

### 8.2 Transitive duplication / a bundled dependency leaks back in
A fat "distribution" jar bundles Core. Other modules (e.g. `Rest-spring-api`, `JpaRepository-spring`) **also** depend on Core — either via the published distribution coordinate, or as individual `Core-*` `implementation` deps. When a consumer pulls both the fat bundle **and** those modules, Core ends up on the classpath **twice** → duplicate Atteo index entries and (if the bundle is an older build) leaked extras like `Core-testing-utils`.
- To drop a transitively-pulled fat coordinate, exclude it:
  ```gradle
  implementation('it.water.service.rest:Rest-spring-api:' + project.waterVersion) {
      exclude group: 'it.water.distribution', module: 'Water-distribution-spring'
  }
  ```
- **Residual duplication is often structural and unavoidable**: a fat bundle (Core inside) + modules that legitimately depend on individual `Core-*` jars will overlap on a single classpath. The framework tolerates duplicate Atteo entries at registration. Do not chase a zero-duplicate index with a sprawling, fragile `exclude` list — fix the leaks that matter (test-utils, wrong versions) and accept benign overlap.

### 8.3 `Core-testing-utils` in a deployable
Test utilities must **never** ship in a runtime distribution. Keep them out of `includeInJar` / `implementationInclude` of distribution modules, and `exclude` them if a transitive fat coordinate drags them in (see 8.2). Test keystores/certs follow the same rule (live under `src/test`, never `src/main`).

### 8.4 The plugin skips its setup if you redefine its configuration name
`addConfiguration(...)` only registers `implementationInclude` (and wires the jar customization + classpath extension) **if a configuration with that name does not already exist**. If a module hand-declares a configuration named `implementationInclude`, the plugin's behavior is silently skipped. Use a different name for module-local configs (the convention is `includeInJar`).

---

## 9. Decision tree

```
Need a dependency in a Water module?
│
├─ Only needs to be on the classpath (normal library module)?
│     └─> implementation (or api if it leaks into your public types)
│
├─ Must be physically embedded in THIS module's jar?
│   │
│   ├─ This module publishes the plugin-fattened STANDARD jar (from components.java)?
│   │     ├─ just the artifact, no transitives ......... implementationInclude
│   │     └─ need its whole transitive closure too ...... implementationIncludeTransitive  (use sparingly)
│   │
│   └─ This module builds a ShadowJar / uber-jar (jar.enabled=false)?
│         └─> includeInJar config + ShadowJar { from configurations.includeInJar... ; mergeServiceFiles '**/META-INF/annotations' }
│             (do NOT use implementationInclude here — §8.1)
│
└─ Need to consume a sibling ShadowJar in the SAME build (single source of truth)?
      └─> producer: consumable configuration + artifacts { thatConf(shadowTask) }
          consumer: implementation project(path: ':Sibling', configuration: 'thatConf')
          (NOT the published coordinate — build-order short-circuit; §7)
```

---

## 10. Quick reference

| I want to… | Do this |
|---|---|
| Add a normal compile dep | `implementation '<coord>'` / `api '<coord>'` |
| Embed a dep's classes into my standard jar | `implementationInclude '<coord>'` |
| Embed a dep + its full transitive graph | `implementationIncludeTransitive '<coord>'` (sparingly) |
| Build an uber/fat/distribution jar | Shadow plugin + `includeInJar` config + ShadowJar task |
| Merge Atteo index (plugin jar) | automatic with `implementationInclude*` |
| Merge Atteo index (ShadowJar) | `mergeServiceFiles { path = '**/META-INF/annotations' }` |
| Let a sibling consume my ShadowJar | consumable config + `artifacts { conf(shadowTask) }` |
| Consume a sibling's ShadowJar | `implementation project(path:, configuration:)` |
| Stop a transitive fat coordinate | `exclude group:, module:` on the offending dep |
| Keep test-utils out of a deployable | never `include*` them; `exclude` if pulled transitively |

### Real-world examples in this repo
- `JpaRepository/JpaRepository-spring/build.gradle` — `implementationInclude` of `Repository-*` + `JpaRepository-api`, publishes `from components.java`.
- `Rest/Rest-spring-api/build.gradle` — `implementationInclude project(":Rest-service")`.
- `Distribution/Distribution-spring/build.gradle` — `includeInJar` + ShadowJar (`springJar`), plus a `springDistributionElements` consumable config exposing it to siblings.
- `Distribution/Distribution-spring-app/build.gradle` — consumes the above with `implementation project(path: ':Distribution-spring', configuration: 'springDistributionElements')` and `exclude`s the transitive published `Water-distribution-spring`.
