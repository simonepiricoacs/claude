---
name: permission-action-naming
description: Naming convention for Water permission action identifiers (Resource.action)
metadata:
  type: project
---

Permission identifiers follow `<ResourceClassName>.<actionVerb>`:
- Standard CRUD: `find`, `save`, `update`, `remove` (always available via base entity infrastructure).
- Custom actions: lowercase verb that names the state transition: `close`, `archive`, `restore`, `lost`, `cancel`, `approve`, etc.

Example matrix for HomeLibrary:
- `Book.find`, `Book.save`, `Book.update`, `Book.remove`, `Book.lost`, `Book.archive`
- `Loan.find`, `Loan.save`, `Loan.update`, `Loan.remove`, `Loan.close`
- `Category.find`, `Category.save`, `Category.update`, `Category.remove`

**Why:** Water Framework's `@AllowPermissions(actions = {...})` annotation matches actions against role permission grants. A consistent `<Entity>.<verb>` convention keeps role-permission mapping readable and SQL-grep-able. Custom actions must be explicitly registered via the Water action manager at module activation.

**How to apply:** In §3.6 of the analysis, always produce a permission matrix table (resource.action × role). Flag any custom action so the backend-architect knows to register it during `@OnActivate`. Keep CRUD verbs lowercase, never `Save` or `READ`.
