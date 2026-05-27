# Water Framework REST API — ApiGateway Module

Covers: **Route**, **RateLimitRule**, **Gateway Management**, **Proxy**

All endpoints require JWT.

---

## ROUTE API — `/water/api/gateway/routes`

### Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "routeId": "user-service-route",
  "pathPattern": "/api/users/**",
  "method": "ANY",
  "targetServiceName": "UserService",
  "rewritePath": "/users",
  "priority": 10,
  "enabled": true,
  "predicates": { "Host": "api.example.com" },
  "filters": { "StripPrefix": "1" }
}
```

Never returned: `ownerUserId`.

### `method` enum (HttpMethod)

`ANY` (default) | `GET` | `POST` | `PUT` | `DELETE` | `PATCH` | `HEAD` | `OPTIONS`

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `routeId` | Yes | NotNull, 1-100 chars, **unique** |
| `pathPattern` | Yes | NotNull, 1-500 chars |
| `method` | No | HttpMethod enum, default `ANY` |
| `targetServiceName` | Yes | NotNull, 1-255 chars |
| `rewritePath` | No | max 500 chars |
| `priority` | No | int, lower = higher priority |
| `enabled` | No | boolean, default `true` |
| `predicates` | No | Map<String,String> |
| `filters` | No | Map<String,String> |

### Endpoints

**POST /water/api/gateway/routes**
```http
POST /water/api/gateway/routes
Authorization: Bearer <token>
Content-Type: application/json

{ "routeId":"user-service-route","pathPattern":"/api/users/**","method":"ANY","targetServiceName":"UserService","priority":10,"enabled":true }
```
Error 409 if `routeId` exists.

**PUT /water/api/gateway/routes** — include `id` + `entityVersion`

**GET /water/api/gateway/routes/{id}** / **GET /water/api/gateway/routes** / **DELETE /water/api/gateway/routes/{id}** — standard pattern.

**POST /water/api/gateway/routes/refresh** — force reload routes into active routing table
```http
POST /water/api/gateway/routes/refresh
Authorization: Bearer <token>
```
Call after bulk changes to apply immediately without restart.

---

## RATELIMITRULE API — `/water/api/gateway/rate-limits`

### Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "ruleId": "global-api-limit",
  "keyType": "IP",
  "keyPattern": ".*",
  "maxRequests": 100,
  "windowSeconds": 60,
  "algorithm": "SLIDING_WINDOW",
  "burstCapacity": 120,
  "enabled": true
}
```

Never returned: `ownerUserId`.

### Enums

**`keyType`** (RateLimitKeyType): `IP` | `USER` | `API_KEY` | `ROUTE` | `GLOBAL`

**`algorithm`** (RateLimitAlgorithm): `FIXED_WINDOW` | `SLIDING_WINDOW` | `TOKEN_BUCKET` | `LEAKY_BUCKET`

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `ruleId` | Yes | NotNull, 1-100 chars, **unique** |
| `keyType` | Yes | Enum RateLimitKeyType |
| `keyPattern` | No | max 500 chars (regex) |
| `maxRequests` | Yes | Min 1 |
| `windowSeconds` | Yes | Min 1 |
| `algorithm` | Yes | Enum RateLimitAlgorithm |
| `burstCapacity` | No | int (TOKEN_BUCKET only) |
| `enabled` | No | boolean, default `true` |

### Endpoints

**POST /water/api/gateway/rate-limits**
```http
POST /water/api/gateway/rate-limits
Authorization: Bearer <token>
Content-Type: application/json

{ "ruleId":"per-user-limit","keyType":"USER","maxRequests":50,"windowSeconds":60,"algorithm":"SLIDING_WINDOW","enabled":true }
```

**PUT /water/api/gateway/rate-limits** — include `id` + `entityVersion`

**GET /water/api/gateway/rate-limits/{id}** / **GET /water/api/gateway/rate-limits** / **DELETE /water/api/gateway/rate-limits/{id}** — standard pattern.

---

## GATEWAY MANAGEMENT API — `/water/api/gateway/management`

**GET /water/api/gateway/management/health**
```http
GET /water/api/gateway/management/health
Authorization: Bearer <token>
```
Response: `{ "status":"UP","uptime":3600,"activeRoutes":12 }`

**GET /water/api/gateway/management/metrics**
```http
GET /water/api/gateway/management/metrics
Authorization: Bearer <token>
```
Response: `{ "totalRequests":15420,"successRequests":15100,"avgResponseTimeMs":45,"routeMetrics":{...} }`

**GET /water/api/gateway/management/circuit-breakers**
```http
GET /water/api/gateway/management/circuit-breakers
Authorization: Bearer <token>
```
Response: array of `{ "serviceName":"UserService","state":"CLOSED","failureRate":2.5,"callCount":1200 }`

States: `CLOSED` (normal) | `OPEN` (blocking) | `HALF_OPEN` (testing recovery)

**POST /water/api/gateway/management/sync** — sync with service discovery registry
```http
POST /water/api/gateway/management/sync
Authorization: Bearer <token>
```

---

## PROXY API — `/water/proxy/{path}`

Transparently forwards requests to backend services applying routing/rate-limit/circuit-breaker logic.

| Method | Path |
|--------|------|
| GET | `/water/proxy/{path}` |
| POST | `/water/proxy/{path}` |
| PUT | `/water/proxy/{path}` |
| DELETE | `/water/proxy/{path}` |
| OPTIONS | `/water/proxy/{path}` |
| HEAD | `/water/proxy/{path}` |

`{path}` is a wildcard. The gateway resolves the target service via routing rules. The JWT token is validated before proxying; hop-by-hop headers are stripped before forwarding.

```http
GET /water/proxy/api/orders/recent
Authorization: Bearer <token>
```

---

## Access Control

| Role | Route | RateLimit | Management |
|------|-------|-----------|------------|
| `gatewayManager` | Full CRUD | Full CRUD | All ops |
| `gatewayViewer` | FIND, FIND_ALL | FIND, FIND_ALL | Health, Metrics |
| `gatewayOperator` | FIND, UPDATE | — | Metrics, Sync |
