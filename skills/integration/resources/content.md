# Water Framework REST API — Content & Taxonomy Modules

Covers: **Document**, **Folder**, **AssetCategory**, **AssetTag**

All endpoints require JWT.

---

## DOCUMENT API — `/water/documents`

### Document Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "entityCreateDate": "2025-02-01T14:30:00.000Z",
  "entityModifyDate": "2025-02-01T14:30:00.000Z",
  "path": "/reports/2025",
  "fileName": "annual-report.pdf",
  "uid": "doc-a1b2c3d4-e5f6",
  "contentType": "application/pdf"
}
```

Never returned: `ownerUserId`, `documentContentInputStream`.  
Implements `SharedEntity` — documents can be shared via SharedEntity API.  
Unique constraints: `(path, fileName)` pair; `uid` alone.

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `path` | Yes | NotNull, no malicious code |
| `fileName` | Yes | NotNull, no malicious code |
| `uid` | Yes | NotNull, unique |
| `contentType` | Yes | NotNull (MIME type) |

### Endpoints

**POST /water/documents** — multipart upload
```http
POST /water/documents
Authorization: Bearer <token>
Content-Type: multipart/form-data

Part "document" (application/json): { "path":"/reports/2025","fileName":"annual-report.pdf","uid":"doc-a1b2c3d4","contentType":"application/pdf" }
Part "file": binary file data
```

JS example:
```js
const form = new FormData();
form.append('document', new Blob([JSON.stringify(meta)], { type: 'application/json' }));
form.append('file', file, file.name);
await fetch('/water/documents', { method:'POST', headers:{'Authorization':`Bearer ${token}`}, body:form });
```

**PUT /water/documents** — same multipart format, include `id` + `entityVersion` in the document part.

**GET /water/documents/{id}** — metadata only (no file content)

**GET /water/documents** — paginated list of document metadata

**DELETE /water/documents/{id}**

**GET /water/documents/content** — download by path + filename
```http
GET /water/documents/content?path=%2Freports%2F2025&fileName=annual-report.pdf
Authorization: Bearer <token>
```
Response: `application/octet-stream`

**GET /water/documents/content/id/{documentId}** — download by database ID
```http
GET /water/documents/content/id/1
Authorization: Bearer <token>
```

**GET /water/documents/content/uid/{documentUID}** — download by UID
```http
GET /water/documents/content/uid/doc-a1b2c3d4-e5f6
Authorization: Bearer <token>
```

JS download helper:
```js
async function downloadFile(url, filename) {
  const res = await fetch(url, { headers: { Authorization: `Bearer ${token}` } });
  const blob = await res.blob();
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = filename;
  a.click();
}
```

### Access Control

| Role | Allowed |
|------|---------|
| `documentsManager` | Full CRUD |
| `documentsViewer` | FIND, FIND_ALL |
| `documentsEditor` | SAVE, UPDATE, FIND, FIND_ALL |

---

## FOLDER API — `/water/documents/folders`

### Folder Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "path": "/reports",
  "name": "2025"
}
```

Never returned: `ownerUserId`, `parent` (lazy @JsonIgnore), `children` (@JsonIgnore).  
Unique constraint: `(path, name)`. DELETE cascades to child folders.

### Endpoints

**POST /water/documents/folders**
```http
POST /water/documents/folders
Authorization: Bearer <token>
Content-Type: application/json

{ "path":"/reports","name":"2025" }
```

**PUT /water/documents/folders** — include `id` + `entityVersion`

**GET /water/documents/folders/{id}** / **GET /water/documents/folders** / **DELETE /water/documents/folders/{id}** — standard pattern.

---

## ASSETCATEGORY API — `/water/assetcategories`

### AssetCategory Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "name": "Electronics",
  "parent": { "id": 0, "name": "Root" },
  "innerAssets": [
    { "id": 2, "name": "Smartphones", "innerAssets": [] }
  ]
}
```

Never returned: `ownerUserId`, `resources`.  
Unique constraint: `(name, parent_id)` — same name allowed under different parents.  
`innerAssets` = immediate children (eager loaded). DELETE cascades via CASCADE_REMOVE.

### Endpoints

**POST /water/assetcategories** — for root category omit `parent`
```http
POST /water/assetcategories
Authorization: Bearer <token>
Content-Type: application/json

{ "name":"Smartphones","parent":{"id":1} }
```

**PUT /water/assetcategories** — include `id` + `entityVersion`

**GET /water/assetcategories/{id}** — returns category with `innerAssets` populated

**GET /water/assetcategories** / **DELETE /water/assetcategories/{id}** — standard pattern.

Tree rendering:
```jsx
function CategoryNode({ node }) {
  return (
    <li>{node.name}
      {node.innerAssets?.length > 0 && (
        <ul>{node.innerAssets.map(c => <CategoryNode key={c.id} node={c} />)}</ul>
      )}
    </li>
  );
}
```

### Access Control

| Role | Allowed |
|------|---------|
| `assetCategoryManager` | Full CRUD |
| `assetCategoryViewer` | FIND, FIND_ALL |
| `assetCategoryEditor` | SAVE, UPDATE, FIND, FIND_ALL |

---

## ASSETTAG API — `/water/assettags`

### AssetTag Model

```json
{
  "id": 1,
  "entityVersion": 1,
  "name": "urgent",
  "description": "High priority items",
  "color": "#FF5733"
}
```

Never returned: `ownerUserId`, `resources`.  
Unique constraint: `(name, ownerUserId)`.

### Field Validation

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | NotEmpty, max 255 |
| `description` | No | max 255 |
| `color` | No | 3-7 chars hex (`#RGB` or `#RRGGBB`) |

### Endpoints

**POST /water/assettags**
```http
POST /water/assettags
Authorization: Bearer <token>
Content-Type: application/json

{ "name":"urgent","description":"High priority","color":"#FF5733" }
```

**PUT /water/assettags** — include `id` + `entityVersion`

**GET /water/assettags/{id}** / **GET /water/assettags** / **DELETE /water/assettags/{id}** — standard pattern.

### Access Control

| Role | Allowed |
|------|---------|
| `AssetTagManager` | Full CRUD |
| `AssetTagViewer` | FIND, FIND_ALL |
| `AssetTagEditor` | SAVE, UPDATE, FIND, FIND_ALL |
