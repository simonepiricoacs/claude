# Water Framework REST API — Platform Modules

Covers: **SharedEntity**, **ServiceDiscovery**

All endpoints require JWT.

---

## SHAREDENTITY API — `/water/entities/shared`

### What it is

`WaterSharedEntity` represents a sharing relationship: entity instance `(entityResourceName + entityId)` is visible to a specific `userId`. Used by modules that implement `SharedEntity` (Company, Document, AssetCategory).

### Model

```json
{
  "entityResourceName": "it.water.company.model.Company",
  "entityId": 42,
  "userId": 7
}
```

**Composite PK**: `(entityResourceName, entityId, userId)` — no auto-generated `id`.

Write-only (never returned): `userId`, `userEmail`, `username`.  
Server resolves `userEmail` or `username` to `userId` if provided instead.

### Known entity resource names

| Entity | resourceName |
|--------|-------------|
| Company | `it.water.company.model.Company` |
| Document | `it.water.documents.manager.model.Document` |
| AssetCategory | `it.water.assetcategory.model.AssetCategory` |

### Endpoints

**POST /water/entities/shared** — share with user
```http
POST /water/entities/shared
Authorization: Bearer <token>
Content-Type: application/json

{ "entityResourceName":"it.water.company.model.Company","entityId":42,"userId":7 }
```
Alternative: use `"userEmail":"luigi@example.com"` instead of `userId`.

**GET /water/entities/shared/{id}** — by surrogate ID

**GET /water/entities/shared** — paginated list

**DELETE /water/entities/shared** — unshare by composite key
```http
DELETE /water/entities/shared
Authorization: Bearer <token>
Content-Type: application/json

{ "entityResourceName":"it.water.company.model.Company","entityId":42,"userId":7 }
```

**GET /water/entities/shared/findByPK** — check if sharing exists
```http
GET /water/entities/shared/findByPK
Authorization: Bearer <token>
Content-Type: application/json

{ "entityResourceName":"it.water.company.model.Company","entityId":42,"userId":7 }
```

**GET /water/entities/shared/findByEntity** — all shares for an entity instance
```http
GET /water/entities/shared/findByEntity?entityResourceName=it.water.company.model.Company&entityId=42
Authorization: Bearer <token>
```

**GET /water/entities/shared/findByUser/{userId}** — all entities shared with a user
```http
GET /water/entities/shared/findByUser/7
Authorization: Bearer <token>
```

**GET /water/entities/shared/sharingUsers** — all users who can access an entity
```http
GET /water/entities/shared/sharingUsers?entityResourceName=it.water.company.model.Company&entityId=42
Authorization: Bearer <token>
```

### Access Control

| Role | Allowed |
|------|---------|
| `sharedentityManager` | Full CRUD |
| `sharedentityViewer` | FIND, FIND_ALL |
| `sharedentityEditor` | SAVE, UPDATE, FIND, FIND_ALL |

---

## SERVICEDISCOVERY API — `/water/api/serviceregistration`

### ServiceRegistration Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "serviceName": "UserService",
  "serviceVersion": "1.0.0",
  "instanceId": "user-service-instance-001",
  "endpoint": "http://user-service:8080",
  "protocol": "HTTP",
  "description": "Manages user accounts",
  "tags": ["auth", "users"],
  "metadata": { "region": "eu-west-1" },
  "status": "RUNNING",
  "lastHeartbeat": "2025-01-10T09:29:55.000Z",
  "healthCheckInterval": 30,
  "healthCheckEndpoint": "/actuator/health"
}
```

Never returned: `ownerUserId`, `configuration`.

### `status` enum (ServiceStatus)

`STARTING` (default on register) | `RUNNING` | `STOPPING` | `STOPPED` | `FAILED` | `UNKNOWN`

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `serviceName` | Yes | NotNull, 1-255 chars |
| `serviceVersion` | Yes | NotNull, 1-50 chars |
| `instanceId` | Yes | NotNull, 1-255 chars, unique with `serviceName` |
| `endpoint` | Yes | NotNull, 1-500 chars |
| `protocol` | Yes | NotNull, 1-50 chars |
| `status` | Yes | Enum ServiceStatus |
| `description` | No | max 2000 chars |
| `tags` | No | Set<String> |
| `metadata` | No | Map<String,String> |
| `healthCheckInterval` | No | seconds |
| `healthCheckEndpoint` | No | max 500 chars |

Unique constraint: `(serviceName, instanceId)`.

### Endpoints

**POST /water/api/serviceregistration** — register
```http
POST /water/api/serviceregistration
Authorization: Bearer <token>
Content-Type: application/json

{ "serviceName":"OrderService","serviceVersion":"2.1.0","instanceId":"order-001","endpoint":"http://order-service:8081","protocol":"HTTP","status":"STARTING","healthCheckEndpoint":"/health","healthCheckInterval":30 }
```

**PUT /water/api/serviceregistration** — update / heartbeat (include `id` + `entityVersion`)
```http
PUT /water/api/serviceregistration
Authorization: Bearer <token>
Content-Type: application/json

{ "id":1,"entityVersion":3,"serviceName":"OrderService","serviceVersion":"2.1.0","instanceId":"order-001","endpoint":"http://order-service:8081","protocol":"HTTP","status":"RUNNING","lastHeartbeat":"2025-01-10T10:00:00.000Z" }
```

**GET /water/api/serviceregistration/{id}** / **DELETE /water/api/serviceregistration/{id}** — standard pattern.

**GET /water/api/serviceregistration** — paginated list

Filter running services only:
```
GET /water/api/serviceregistration?filters=[{"field":"status","op":"EQ","value":"RUNNING"}]
```

### Access Control

| Role | Allowed |
|------|---------|
| `serviceDiscoveryManager` | Full CRUD |
| `serviceDiscoveryViewer` | FIND, FIND_ALL |
| `serviceDiscoveryOperator` | FIND, FIND_ALL, UPDATE (heartbeat) |
