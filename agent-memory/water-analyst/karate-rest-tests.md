---
name: karate-rest-tests
description: Standard Karate REST integration test setup for Water Framework modules
metadata:
  type: reference
---

Karate REST tests live in `<Module>-service/src/test/resources/karate/` as `.feature` files, driven by a single JUnit 5 runner (e.g. `HomeLibraryRestApiTest`).

Required `it.water.application.properties` for tests:
- `water.testMode=true`
- `water.rest.security.jwt.validate=false`

Naming: one feature file per resource or per high-level use case:
- `<entity>-crud.feature` — basic CRUD + pagination
- `<entity>-<usecase>.feature` — state transitions and custom actions
- `security.feature` — role-based access denial cases

**Why:** This is the convention validated in the ApiGateway module (Feb 2026): 138 tests passing, 86% REST package coverage. Without `water.testMode=true` the JWT filter rejects every request; without `water.rest.security.jwt.validate=false` tests fail at token signature validation.

**How to apply:** In §9 of the functional analysis, always list expected `.feature` files and remind the team of the two required properties. Cross-link to [[rest-path-convention]] because Karate tests will silently 404 if paths are wrong.
