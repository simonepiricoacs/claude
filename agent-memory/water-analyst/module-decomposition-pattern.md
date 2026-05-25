---
name: module-decomposition-pattern
description: Standard Water Framework module decomposition into model/api/service (+ optional application) for any new domain module
metadata:
  type: project
---

For a new standalone Water Framework domain module, decompose into:
- `<Name>-model` — entities, enums, DTOs. NO dependency on Water service layer.
- `<Name>-api` — `*Api`, `*SystemApi`, `*RepositoryApi`, `*RestApi` interfaces. Depends on model + Water-api.
- `<Name>-service` — `*ServiceImpl`, `*RepositoryImpl`, `*RestApiImpl`. Depends on api + Water-service + Water-repository-jpa + Water-rest.
- `<Name>-application` (optional) — Spring Boot bootstrap, only when module is standalone-deployable as a microservice.

**Why:** This separation is the de-facto Water Framework convention (see ApiGateway split into ApiGateway-model/-api/-service). It enforces ISP/DIP: consumers depend on `-api` only and implementations are swappable.

**How to apply:** Always propose this 3-module (or 4-module) layout in §4 of the functional analysis. Use `yo water:new-project` then `yo water:add-entity` per entity — generator produces the layout automatically. Verify the module names use PascalCase and groupId follows `it.water.<lowercase-name>` pattern.
