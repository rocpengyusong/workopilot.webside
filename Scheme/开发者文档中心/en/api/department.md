<script setup>
const createDeptRequestParams = [
  { name: 'name', type: 'string', required: true, description: 'Department name', default: '-' },
  { name: 'parentId', type: 'number', required: false, description: 'Parent department ID (null if no parent)', default: 'null' },
  { name: 'description', type: 'string', required: false, description: 'Department description', default: 'null' },
  { name: 'sort', type: 'number', required: false, description: 'Sort order', default: '0' },
  { name: 'status', type: 'number', required: false, description: 'Status (1-Enabled, 0-Disabled)', default: '1' }
]

const deptResponseParams = [
  { name: 'id', type: 'number', description: 'Department ID' },
  { name: 'name', type: 'string', description: 'Department name' },
  { name: 'parentId', type: 'number', description: 'Parent department ID' },
  { name: 'description', type: 'string', description: 'Department description' },
  { name: 'sort', type: 'number', description: 'Sort order' },
  { name: 'status', type: 'number', description: 'Status (1-Enabled, 0-Disabled)' },
  { name: 'createdAt', type: 'string', description: 'Creation time' },
  { name: 'updatedAt', type: 'string', description: 'Update time' }
]

const queryDeptRequestParams = [
  { name: 'name', type: 'string', required: false, description: 'Department name (fuzzy search)', default: '-' },
  { name: 'parentId', type: 'number', required: false, description: 'Parent department ID (search sub-departments)', default: '-' },
  { name: 'status', type: 'number', required: false, description: 'Status (1-Enabled, 0-Disabled)', default: '-' },
  { name: 'page', type: 'number', required: false, description: 'Page number', default: '1' },
  { name: 'pageSize', type: 'number', required: false, description: 'Page size', default: '20' }
]

const queryDeptResponseParams = [
  { name: 'total', type: 'number', description: 'Total records' },
  { name: 'page', type: 'number', description: 'Current page number' },
  { name: 'pageSize', type: 'number', description: 'Page size' },
  { name: 'list', type: 'object[]', description: 'Department list' }
]

const deptTreeResponseParams = [
  { name: 'id', type: 'number', description: 'Department ID' },
  { name: 'name', type: 'string', description: 'Department name' },
  { name: 'parentId', type: 'number', description: 'Parent department ID' },
  { name: 'children', type: 'object[]', description: 'Sub-department list', subProps: [
    { name: 'id', type: 'number', description: 'Department ID' },
    { name: 'name', type: 'string', description: 'Department name' },
    { name: 'parentId', type: 'number', description: 'Parent department ID' },
    { name: 'children', type: 'object[]', description: 'Sub-department list' }
  ]}
]

const columns = {
  name: 'Parameter',
  type: 'Type',
  default: 'Default',
  description: 'Description'
}
</script>

# Department Management

This module includes interfaces for department creation, query, and management.

## 1. Create Department

Create a new department.

- **Endpoint**: `/api/v1/department/create`
- **Method**: `POST`
- **Content-Type**: `application/json`
- **Authentication**: Bearer Token

### Request Parameters

<ApiTable :data="createDeptRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="deptResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```json [Request]
{
  "name": "R&D Dept",
  "parentId": null,
  "description": "Responsible for product R&D",
  "sort": 1,
  "status": 1
}
```

```json [Response]
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "name": "R&D Dept",
    "parentId": null,
    "description": "Responsible for product R&D",
    "sort": 1,
    "status": 1,
    "createdAt": "2024-01-01 10:00:00",
    "updatedAt": "2024-01-01 10:00:00"
  }
}
```

:::

## 2. Query Department List

Query department list with pagination, supports filtering by name, parent department, status, etc.

- **Endpoint**: `/api/v1/department/list`
- **Method**: `GET`
- **Authentication**: Bearer Token

### Request Parameters

<ApiTable :data="queryDeptRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="queryDeptResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/api/v1/department/list?name=R&D&status=1&page=1&pageSize=20" \
  -H "Authorization: Bearer <token>"
```

```json [Response]
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 50,
    "page": 1,
    "pageSize": 20,
    "list": [
      {
        "id": 1,
        "name": "R&D Dept",
        "parentId": null,
        "description": "Responsible for product R&D",
        "sort": 1,
        "status": 1,
        "createdAt": "2024-01-01 10:00:00",
        "updatedAt": "2024-01-01 10:00:00"
      }
    ]
  }
}
```

:::

## 3. Get Department Tree

Get the full department hierarchy tree.

- **Endpoint**: `/api/v1/department/tree`
- **Method**: `GET`
- **Authentication**: Bearer Token

### Response Parameters

<ApiTable :data="deptTreeResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/api/v1/department/tree" \
  -H "Authorization: Bearer <token>"
```

```json [Response]
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": 1,
      "name": "Headquarters",
      "parentId": null,
      "children": [
        {
          "id": 2,
          "name": "R&D Dept",
          "parentId": 1,
          "children": [
            {
              "id": 5,
              "name": "Frontend Team",
              "parentId": 2,
              "children": []
            },
            {
              "id": 6,
              "name": "Backend Team",
              "parentId": 2,
              "children": []
            }
          ]
        },
        {
          "id": 3,
          "name": "Marketing Dept",
          "parentId": 1,
          "children": []
        },
        {
          "id": 4,
          "name": "HR Dept",
          "parentId": 1,
          "children": []
        }
      ]
    }
  ]
}
```

:::
