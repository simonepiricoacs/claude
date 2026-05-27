# Water Framework REST API — Identity Module

Covers: **User**, **Role**, **Permission**

---

## USER API — `/water/users`

### Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "entityCreateDate": "2025-01-15T10:30:00.000Z",
  "entityModifyDate": "2025-01-15T10:30:00.000Z",
  "name": "Mario",
  "lastname": "Rossi",
  "username": "mario.rossi",
  "email": "mario.rossi@example.com",
  "admin": false,
  "imagePath": "/images/avatar.png"
}
```

Write-only (sent in requests, never returned): `password`, `passwordConfirm`  
Never returned: `salt`, `active`, `deleted`, `activateCode`, `deletionCode`, `passwordResetCode`, `roles`

### Field Validation

| Field | Constraints |
|-------|-------------|
| `name` | NotEmpty, max 500 chars |
| `lastname` | NotEmpty, max 500 chars |
| `username` | NotEmpty, max 400 chars, pattern `^[A-Za-z0-9\-\.\_]+$`, unique |
| `email` | Valid email, unique |
| `password` | `@ValidPassword` (8+ chars, upper, digit, special char), write-only |
| `passwordConfirm` | Must match `password`, transient |

### Endpoints

**POST /water/users/register** *(public)*
```http
POST /water/users/register
Content-Type: application/json

{ "name":"Mario","lastname":"Rossi","username":"mario.rossi","email":"mario.rossi@example.com","password":"Admin1234!","passwordConfirm":"Admin1234!" }
```
Response 200: created user. Side effect: activation email sent.

**PUT /water/users/activate** *(public)*
```http
PUT /water/users/activate?email=mario.rossi%40example.com&activationCode=ABC123
```

**PUT /water/users/resetPasswordRequest** *(public)*
```http
PUT /water/users/resetPasswordRequest?email=mario.rossi%40example.com
```

**POST /water/users** *(JWT required)*
```http
POST /water/users
Authorization: Bearer <token>
Content-Type: application/json

{ "name":"Luigi","lastname":"Verdi","username":"luigi.verdi","email":"luigi.verdi@example.com","password":"Admin1234!","passwordConfirm":"Admin1234!" }
```

**PUT /water/users** *(JWT required)* — include `id` + `entityVersion`
```http
PUT /water/users
Authorization: Bearer <token>
Content-Type: application/json

{ "id":5,"entityVersion":2,"name":"Luigi Updated","lastname":"Verdi","username":"luigi.verdi","email":"luigi.verdi@example.com" }
```

**GET /water/users/{id}** *(JWT required)*
```http
GET /water/users/5
Authorization: Bearer <token>
```

**GET /water/users** *(JWT required)*
```http
GET /water/users?page=0&pageSize=20&orderMainField=username&orderDirection=ASC
Authorization: Bearer <token>
```

**DELETE /water/users/{id}** *(JWT required)*
```http
DELETE /water/users/5
Authorization: Bearer <token>
```

**PUT /water/users/{id}/activate** *(JWT required, admin)*
```http
PUT /water/users/5/activate
Authorization: Bearer <token>
```

**PUT /water/users/{id}/deactivate** *(JWT required, admin)*
```http
PUT /water/users/5/deactivate
Authorization: Bearer <token>
```

**POST /water/users/unregisterRequest** *(JWT required)* — sends self-deletion confirmation email
```http
POST /water/users/unregisterRequest
Authorization: Bearer <token>
```

**DELETE /water/users/unregister** *(JWT required)* — confirm self-deletion via email code
```http
DELETE /water/users/unregister?email=mario.rossi%40example.com&deletionCode=XYZ789
Authorization: Bearer <token>
```

### Access Control

| Role | Allowed |
|------|---------|
| `userManager` | Full CRUD + activate/deactivate |
| `userViewer` | FIND, FIND_ALL |
| `userEditor` | SAVE, UPDATE, FIND, FIND_ALL |

---

## ROLE API — `/water/roles`

### Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "entityCreateDate": "2025-01-01T00:00:00.000Z",
  "entityModifyDate": "2025-01-01T00:00:00.000Z",
  "name": "ADMIN",
  "description": "Full administrative access"
}
```

Validation: `name` NotEmpty max 255 unique; `description` max 3000.

### Endpoints (all JWT required)

**POST /water/roles**
```http
POST /water/roles
Authorization: Bearer <token>
Content-Type: application/json

{ "name":"EDITOR","description":"Can edit content" }
```

**PUT /water/roles** — include `id` + `entityVersion`
```http
PUT /water/roles
Authorization: Bearer <token>
Content-Type: application/json

{ "id":3,"entityVersion":1,"name":"EDITOR","description":"Updated description" }
```

**GET /water/roles/{id}** / **GET /water/roles** / **DELETE /water/roles/{id}** — standard pattern.

**POST /water/roles/assign** — form params
```http
POST /water/roles/assign
Authorization: Bearer <token>
Content-Type: application/x-www-form-urlencoded

userId=5&roleId=3
```

**POST /water/roles/unassign** — form params
```http
POST /water/roles/unassign
Authorization: Bearer <token>
Content-Type: application/x-www-form-urlencoded

userId=5&roleId=3
```

### Access Control

| Role | Allowed |
|------|---------|
| `roleManager` | Full CRUD + assign/unassign |
| `roleViewer` | FIND, FIND_ALL |
| `roleEditor` | SAVE, UPDATE, FIND, FIND_ALL, ASSIGN, UNASSIGN |

---

## PERMISSION API — `/water/permissions`

### Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "name": "View Documents",
  "actionIds": 4,
  "entityResourceName": "it.water.documents.manager.model.Document",
  "resourceId": 0,
  "roleId": 3,
  "userId": 0
}
```

Key fields:
- `actionIds` — bitmask: 1=SAVE, 2=UPDATE, 4=FIND, 8=FIND_ALL, 16=REMOVE, 32+=custom
- `entityResourceName` — fully-qualified entity class name
- `resourceId` — `0` = all instances; `>0` = specific instance ID
- `roleId` / `userId` — permission applies to role OR user (not both)

Unique constraint: `(roleId, userId, entityResourceName, resourceId)`

### Endpoints (all JWT required)

**POST /water/permissions**
```http
POST /water/permissions
Authorization: Bearer <token>
Content-Type: application/json

{ "name":"Edit Documents","actionIds":3,"entityResourceName":"it.water.documents.manager.model.Document","resourceId":0,"roleId":3,"userId":0 }
```

**PUT /water/permissions** — include `id` + `entityVersion`

**GET /water/permissions/{id}** / **GET /water/permissions** / **DELETE /water/permissions/{id}** — standard pattern.

**POST /water/permissions/map** — check what a user can do on a set of entity instances
```http
POST /water/permissions/map
Authorization: Bearer <token>
Content-Type: application/json

{ "entityPks": "it.water.documents.manager.model.Document:1,2,3" }
```
Response: map of `entityPk → actionIds` bitmask for the current authenticated user.

### Frontend: check action from bitmask

```js
const ACTION = { SAVE:1, UPDATE:2, FIND:4, FIND_ALL:8, REMOVE:16 };
const can = (actionIds, action) => (actionIds & action) !== 0;

// Example: show Delete button only if permitted
{can(permission.actionIds, ACTION.REMOVE) && <DeleteButton />}
```

### Access Control

| Role | Allowed |
|------|---------|
| `permissionManager` | Full CRUD + permission map |
| `permissionViewer` | FIND, FIND_ALL |
| `permissionEditor` | SAVE, UPDATE, FIND, FIND_ALL |
