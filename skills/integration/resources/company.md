# Water Framework REST API — Company Module

Base path: `/water/companies` — all endpoints require JWT.

---

## Company Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "entityCreateDate": "2025-03-10T09:00:00.000Z",
  "entityModifyDate": "2025-03-10T09:00:00.000Z",
  "businessName": "Acme Corporation S.r.l.",
  "invoiceAddress": "Via Roma 1",
  "city": "Milan",
  "postalCode": "20121",
  "nation": "Italy",
  "vatNumber": "IT01234567890"
}
```

Never returned: `ownerUserId`.  
Extends `AbstractJpaExpandableEntity` — supports dynamic extension fields.  
Implements `SharedEntity` — records can be shared with other users via SharedEntity API.

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `businessName` | Yes | NotEmpty, max 255 |
| `invoiceAddress` | Yes | NotEmpty, max 255 |
| `city` | Yes | NotEmpty, max 255 |
| `postalCode` | Yes | NotEmpty, max 255 |
| `nation` | Yes | NotEmpty, max 255 |
| `vatNumber` | Yes | NotEmpty, max 255, **unique** |

---

## Endpoints

**POST /water/companies**
```http
POST /water/companies
Authorization: Bearer <token>
Content-Type: application/json

{ "businessName":"Acme S.r.l.","invoiceAddress":"Via Roma 1","city":"Milan","postalCode":"20121","nation":"Italy","vatNumber":"IT01234567890" }
```
Error 409 if `vatNumber` already exists.

**PUT /water/companies** — include `id` + `entityVersion`
```http
PUT /water/companies
Authorization: Bearer <token>
Content-Type: application/json

{ "id":1,"entityVersion":1,"businessName":"Acme S.r.l.","invoiceAddress":"Via Garibaldi 5","city":"Rome","postalCode":"00100","nation":"Italy","vatNumber":"IT01234567890" }
```

**GET /water/companies/{id}**
```http
GET /water/companies/1
Authorization: Bearer <token>
```

**GET /water/companies** — with pagination
```http
GET /water/companies?page=0&pageSize=10&orderMainField=businessName&orderDirection=ASC
Authorization: Bearer <token>
```

Filter by nation:
```
GET /water/companies?filters=[{"field":"nation","op":"EQ","value":"Italy"}]
```

**DELETE /water/companies/{id}**
```http
DELETE /water/companies/1
Authorization: Bearer <token>
```

---

## Access Control

| Role | Allowed |
|------|---------|
| `companyManager` | Full CRUD |
| `companyViewer` | FIND, FIND_ALL |
| `companyEditor` | SAVE, UPDATE, FIND, FIND_ALL |
