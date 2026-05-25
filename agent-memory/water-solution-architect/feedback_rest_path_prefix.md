---
name: feedback-rest-path-prefix
description: Water REST @Path annotations must NOT include /water prefix — CXF adds it automatically as base context
metadata:
  type: feedback
---

When designing REST endpoints for Water modules, the `@Path` on the `*RestApi` interface MUST NOT include the `/water` prefix.

**Why:** Apache CXF (the Water REST server) configures `/water` as the base context path. If `@Path` also starts with `/water`, the prefix gets doubled and routes return HTTP 404. This was the root cause of the ApiGateway test failures in Feb 2026 (per user auto-memory).

**How to apply:**
- In a TAA, when listing REST paths, write them WITHOUT `/water` (e.g., `/homelibrary/books`, NOT `/water/homelibrary/books`).
- Note in the document that the full external URL will be `/water/<module-path>` because CXF prepends it.
- When reviewing existing Water modules, flag any `@Path("/water/...")` as a bug.
