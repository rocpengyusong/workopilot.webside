<script setup>
const createDeptRequestParams = [
  { name: 'name', type: 'string', required: true, description: '部门名称', default: '-' },
  { name: 'parentId', type: 'number', required: false, description: '上级部门 ID (无上级则为 null)', default: 'null' },
  { name: 'description', type: 'string', required: false, description: '部门描述', default: 'null' },
  { name: 'sort', type: 'number', required: false, description: '排序序号', default: '0' },
  { name: 'status', type: 'number', required: false, description: '状态 (1-启用 0-禁用)', default: '1' }
]

const deptResponseParams = [
  { name: 'id', type: 'number', description: '部门 ID' },
  { name: 'name', type: 'string', description: '部门名称' },
  { name: 'parentId', type: 'number', description: '上级部门 ID' },
  { name: 'description', type: 'string', description: '部门描述' },
  { name: 'sort', type: 'number', description: '排序序号' },
  { name: 'status', type: 'number', description: '状态 (1-启用 0-禁用)' },
  { name: 'createdAt', type: 'string', description: '创建时间' },
  { name: 'updatedAt', type: 'string', description: '更新时间' }
]

const queryDeptRequestParams = [
  { name: 'name', type: 'string', required: false, description: '部门名称 (模糊查询)', default: '-' },
  { name: 'parentId', type: 'number', required: false, description: '上级部门 ID (查询子部门)', default: '-' },
  { name: 'status', type: 'number', required: false, description: '状态 (1-启用 0-禁用)', default: '-' },
  { name: 'page', type: 'number', required: false, description: '页码', default: '1' },
  { name: 'pageSize', type: 'number', required: false, description: '每页数量', default: '20' }
]

const queryDeptResponseParams = [
  { name: 'total', type: 'number', description: '总记录数' },
  { name: 'page', type: 'number', description: '当前页码' },
  { name: 'pageSize', type: 'number', description: '每页数量' },
  { name: 'list', type: 'object[]', description: '部门列表' }
]

const deptTreeResponseParams = [
  { name: 'id', type: 'number', description: '部门 ID' },
  { name: 'name', type: 'string', description: '部门名称' },
  { name: 'parentId', type: 'number', description: '上级部门 ID' },
  { name: 'children', type: 'object[]', description: '子部门列表', subProps: [
    { name: 'id', type: 'number', description: '部门 ID' },
    { name: 'name', type: 'string', description: '部门名称' },
    { name: 'parentId', type: 'number', description: '上级部门 ID' },
    { name: 'children', type: 'object[]', description: '子部门列表' }
  ]}
]

const columns = {
  name: '参数名',
  type: '类型',
  default: '默认值',
  description: '说明'
}
</script>

# 部门管理 (Department)

本模块包含部门创建、查询及管理等相关接口。

## 1. 创建部门

创建一个新的部门。

- **接口地址**: `/api/v1/department/create`
- **请求方式**: `POST`
- **Content-Type**: `application/json`
- **认证方式**: Bearer Token

### 请求参数

<ApiTable :data="createDeptRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="deptResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```json [Request]
{
  "name": "研发部",
  "parentId": null,
  "description": "负责产品研发工作",
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
    "name": "研发部",
    "parentId": null,
    "description": "负责产品研发工作",
    "sort": 1,
    "status": 1,
    "createdAt": "2024-01-01 10:00:00",
    "updatedAt": "2024-01-01 10:00:00"
  }
}
```

:::

## 2. 查询部门列表

分页查询部门列表，支持按名称、上级部门、状态等条件筛选。

- **接口地址**: `/api/v1/department/list`
- **请求方式**: `GET`
- **认证方式**: Bearer Token

### 请求参数

<ApiTable :data="queryDeptRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="queryDeptResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/api/v1/department/list?name=研发&status=1&page=1&pageSize=20" \
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
        "name": "研发部",
        "parentId": null,
        "description": "负责产品研发工作",
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

## 3. 获取部门树

获取完整的部门层级结构树。

- **接口地址**: `/api/v1/department/tree`
- **请求方式**: `GET`
- **认证方式**: Bearer Token

### 响应参数

<ApiTable :data="deptTreeResponseParams" :columns="columns" :show-default="false" />

### 示例

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
      "name": "公司总部",
      "parentId": null,
      "children": [
        {
          "id": 2,
          "name": "研发部",
          "parentId": 1,
          "children": [
            {
              "id": 5,
              "name": "前端组",
              "parentId": 2,
              "children": []
            },
            {
              "id": 6,
              "name": "后端组",
              "parentId": 2,
              "children": []
            }
          ]
        },
        {
          "id": 3,
          "name": "市场部",
          "parentId": 1,
          "children": []
        },
        {
          "id": 4,
          "name": "人事部",
          "parentId": 1,
          "children": []
        }
      ]
    }
  ]
}
```

:::
