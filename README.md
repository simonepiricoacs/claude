# Water Framework — Claude Code Plugin

A Claude Code plugin that brings a full multi-agent development team and an extensive knowledge base for the [Water Framework](https://www.water-framework.com/) directly into your IDE.

The plugin ships **5 specialized agents** and **11 skills** that cover every phase of Water Framework development: requirements analysis, solution architecture, code scaffolding, backend implementation, and test generation — all without needing to read the framework source code.

---

## Installation

> Requires Claude Code with `/plugin` support.

```bash
# 1. Add this repo as a marketplace
/plugin marketplace add Water-Framework/claude

# 2. Install the plugin
/plugin install water-framework@Water-Framework/claude
```

Alternatively, add it as a git submodule inside an existing project:

```bash
git submodule add https://github.com/Water-Framework/claude.git .claude
```

---

## Agents

Agents are autonomous specialists invoked automatically by Claude Code based on the task at hand, or called directly by name.

### `water-analyst`
**Functional analysis** — Decomposes complex or ambiguous requirements into structured, actionable specifications aligned with Water Framework architectural principles. Invoke before any code is written when requirements span multiple bounded contexts or involve cross-aggregate state.

### `water-solution-architect`
**Technical architecture** — Produces Technical Architecture Analyses (TAA): which modules/microservices are needed, how they communicate, which runtime each targets, and integration blueprints. Does not write code — it defines what gets built and how the pieces connect.

### `water-scaffolder`
**Code generation** — Executes all `yo water:*` generator commands: new projects, add entities, add REST services, build, publish. Enforces the generator-first protocol — no manual file creation where the generator can do it.

### `water-backend-architect`
**Backend implementation** — Implements code inside existing generated module structures: entity fields, business logic, custom methods, security annotations, custom queries, and custom REST endpoints. Works only within what the generator has already produced.

### `water-test-engineer`
**Test generation** — Writes JUnit 5 unit tests and Karate REST integration tests targeting ≥80% instruction coverage for SonarQube compliance. Identifies coverage gaps and fills them with targeted tests.

---

## Skills

Skills are knowledge bases that agents load on demand. They make every agent self-sufficient without requiring access to the Water Framework source code.

| Skill | Purpose |
|-------|---------|
| `framework-core-knowledge` | Core philosophy, all base interfaces with full Java signatures, entity hierarchy, Api/SystemApi distinction, QueryBuilder, REST layer, permission system, component lifecycle, anti-patterns |
| `architecture-knowledge` | DDD patterns, microservice design, module construction, bounded contexts |
| `authentication-knowledge` | JWT flows, `AuthenticationProvider`, login/registration, REST security filters |
| `authorization-knowledge` | `@AllowPermissions`, `@AllowRoles`, security interceptors, permission actions, RBAC patterns |
| `persistence-knowledge` | Repository pattern, JPA implementation, `QueryBuilder` fluent API, pagination, transactions |
| `properties-knowledge` | `Options` pattern, `ApplicationProperties`, runtime property loading, environment variables |
| `rest-knowledge` | `RestApi` hierarchy, JAX-RS / Spring MVC dual support, JWT filters, JSON views, Swagger, exception mapping |
| `runtime-knowledge` | `ComponentRegistry`, `@FrameworkComponent`, `@OnActivate`/`@OnDeactivate`, `@Inject`, OSGi/Spring differences |
| `karate-testing` | Karate feature file patterns, `karate-config.js`, test runner configuration, Water test runtime integration |
| `water-generate` | All `yo water:*` commands with options, generator-first protocol, `.yo-rc.json` rules |
| `test-generation` | Unit test scaffolding, coverage enforcement, SonarQube compliance patterns |

---

## Multi-Agent Workflow

Claude Code coordinates agents automatically. The standard decision tree:

```
New module / microservice from scratch
  └─> water-analyst → water-solution-architect → water-scaffolder → water-backend-architect → water-test-engineer

Add entity to existing project
  └─> water-scaffolder (yo water:add-entity) → water-backend-architect → water-test-engineer

Fix bug / implement feature in existing code
  └─> water-backend-architect → water-test-engineer

Architecture question / design review
  └─> water-solution-architect  or  water-analyst

Generate/run tests only
  └─> water-test-engineer
```

Each phase validates its output before handing off to the next agent. Sequential phases run one at a time; independent tasks (e.g., scaffolding unrelated entities) run in parallel.

---

## What the Plugin Does NOT Include

- **`agent-memory/`** — Per-project agent memory is generated locally at runtime and is not distributed. Each project starts with a clean slate.
- **`settings.local.json`** — Local IDE settings, excluded via `.gitignore`.
- **Framework source code** — All framework knowledge is embedded in skills; no source access is required.

---

## Requirements

- [Claude Code](https://claude.ai/code) with `/plugin` marketplace support
- Water Framework projects using `yo water` generator
- Java 17+, Gradle

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

## Links

- [Water Framework](https://www.water-framework.com/)
- [Water Framework GitHub](https://github.com/Water-Framework)
- [Claude Code](https://claude.ai/code)