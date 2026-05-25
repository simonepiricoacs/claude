---
name: rest-path-convention
description: CXF auto-prepends /water to all REST endpoints — @Path must never include /water prefix
metadata:
  type: feedback
---

In any `@Path` annotation on a `*RestApi` interface, NEVER include the `/water` prefix. CXF's Water REST server adds it automatically as the base context.

**Why:** A confirmed regression in the ApiGateway module: paths like `/water/api/gateway/routes` were doubled to `/water/water/api/gateway/routes` and caused HTTP 404 across all endpoints. Fix applied Feb 2026 by stripping the prefix.

**How to apply:** In §3.5 of the functional analysis, the table column "declared @Path" always omits `/water`. The "full URL" column shows it. Flag any incoming requirement that includes `/water` in path specs as an anti-pattern and correct it before handoff to backend-architect.
