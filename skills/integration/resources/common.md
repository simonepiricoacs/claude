# Water Framework REST API — Common Patterns

## Base URL

All endpoints are under:
```
http(s)://<host>:<port>/water
```

| Module | Full path |
|--------|-----------|
| Authentication | `/water/authentication` |
| Users | `/water/users` |
| Roles | `/water/roles` |
| Permissions | `/water/permissions` |
| Company | `/water/companies` |
| ApiGateway Routes | `/water/api/gateway/routes` |
| ApiGateway RateLimit | `/water/api/gateway/rate-limits` |
| ApiGateway Management | `/water/api/gateway/management` |
| Proxy | `/water/proxy` |
| Documents | `/water/documents` |
| AssetCategory | `/water/assetcategories` |
| AssetTag | `/water/assettags` |
| SharedEntity | `/water/entities/shared` |
| ServiceDiscovery | `/water/api/serviceregistration` |

Spring Boot default port: 8080.

---

## Authentication — JWT Flow

### Step 1: Obtain token

```http
POST /water/authentication/login
Content-Type: application/x-www-form-urlencoded

username=admin&password=Admin1234%21
```

Response 200:
```json
{
  "token": "eyJhbGci...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

Response 401 (wrong credentials):
```json
{ "message": "Authentication failed", "statusCode": 401 }
```

### Step 2: Use token

Every protected request:
```http
Authorization: Bearer eyJhbGci...
```

### Public endpoints (no token required)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/water/authentication/login` | Get JWT |
| POST | `/water/users/register` | Self-registration |
| PUT | `/water/users/activate` | Email activation |
| PUT | `/water/users/resetPasswordRequest` | Password reset request |

---

## Standard Response Format

**Single entity** (GET /{id}, POST, PUT):
```json
{
  "id": 1,
  "entityVersion": 1,
  "entityCreateDate": "2025-01-01T10:00:00.000Z",
  "entityModifyDate": "2025-01-01T10:00:00.000Z",
  "... entity fields ..."
}
```

**Collection** (GET /):
```json
{
  "results": [ {...}, {...} ],
  "numPages": 3,
  "currentPage": 0,
  "delta": 10,
  "systemCount": 25
}
```

**Void** (DELETE, some POST): HTTP 200, empty body.

---

## Pagination

Query parameters for `GET /`:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | int | 0 | Zero-based page number |
| `pageSize` | int | 10 | Items per page |
| `orderMainField` | String | `id` | Sort field |
| `orderDirection` | String | `ASC` | `ASC` or `DESC` |
| `filters` | JSON string | — | Filter array (see below) |

Example:
```
GET /water/users?page=0&pageSize=20&orderMainField=username&orderDirection=ASC
```

Filter syntax (URL-encoded JSON array):
```json
[
  { "field": "email", "op": "LIKE", "value": "%@example.com" },
  { "field": "active", "op": "EQ", "value": "true" }
]
```

Operators: `EQ`, `NEQ`, `LT`, `LTE`, `GT`, `GTE`, `LIKE`, `IN`, `NOT_IN`, `IS_NULL`, `IS_NOT_NULL`.

---

## Error Handling

```json
{
  "type": "it.water.core.api.exception.ValidationException",
  "message": "Field 'email' is not valid",
  "statusCode": 422,
  "validationErrors": [
    { "field": "email", "message": "must be a valid email" }
  ]
}
```

| Status | Meaning | Frontend action |
|--------|---------|----------------|
| 400 | Bad request / malformed body | Show form error |
| 401 | Missing or expired token | Redirect to login |
| 403 | Insufficient permissions | Show access denied |
| 404 | Entity not found | Show not found state |
| 409 | Unique constraint violation | Show conflict message |
| 422 | Bean validation failure | Show field-level errors |
| 500 | Server error | Show generic error |

---

## JSON Views — field visibility

Water uses `@JsonView` to control what fields are serialized:

| View | Visible to | Fields |
|------|-----------|--------|
| `Public` | Anyone | Core identifying fields |
| `Extended` | Authenticated | All Public + detail fields |
| `Internal` | System only | Never exposed via REST |

Fields annotated `@JsonProperty(access = WRITE_ONLY)` (e.g. `password`, `userId` in SharedEntity) are accepted on write but **never returned** in responses.  
Fields annotated `@JsonIgnore` (e.g. `salt`, `ownerUserId`) are never serialized in either direction.

---

## Base Entity Fields

Every entity response includes:

| Field | Type | Notes |
|-------|------|-------|
| `id` | Long | Auto-generated PK |
| `entityVersion` | Long | Optimistic lock — **must be sent on PUT** |
| `entityCreateDate` | ISO-8601 string | Set on first persist |
| `entityModifyDate` | ISO-8601 string | Updated on every save |
| `systemCreateDate` | ISO-8601 string | System-level audit |
| `systemModifyDate` | ISO-8601 string | System-level audit |

> Always include `id` + `entityVersion` in PUT bodies to avoid optimistic lock exceptions.

---

## Request Headers Reference

```http
Authorization: Bearer <token>                        # all protected endpoints
Content-Type: application/json                       # JSON bodies
Content-Type: application/x-www-form-urlencoded      # login, role assign/unassign
Content-Type: multipart/form-data                    # file upload (Documents)
Accept: application/json
```

---

## CORS — dev proxy config

**Vite**:
```ts
server: {
  proxy: {
    '/water': { target: 'http://localhost:8080', changeOrigin: true }
  }
}
```

**Angular** (`proxy.conf.json`):
```json
{ "/water": { "target": "http://localhost:8080", "changeOrigin": true } }
```
