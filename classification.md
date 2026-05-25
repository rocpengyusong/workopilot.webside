<script setup>
const classifyFormRequestParams = [
  { name: 'GroupId', type: 'integer', required: false, description: '分组ID，与GroupCode、CategoryIds、CategoryCodes四选一', default: '-' },
  { name: 'GroupCode', type: 'string', required: false, description: '分组编码，与GroupId、CategoryIds、CategoryCodes四选一', default: '-' },
  { name: 'CategoryIds', type: 'string[]', required: false, description: '分类ID集合，与GroupId、GroupCode、CategoryCodes四选一', default: '-' },
  { name: 'CategoryCodes', type: 'string[]', required: false, description: '分类编码集合，与GroupId、GroupCode、CategoryIds四选一', default: '-' },
  { name: 'File', type: 'string(binary)', required: false, description: '文件流，支持多文件，与FileUrls二选一，使用此字段需要参数为form-data', default: '-' },
  { name: 'BusinessId', type: 'integer', required: false, description: '业务ID，可空', default: '-' },
  { name: 'ExtractFields', type: 'boolean', required: false, description: '是否提取数据，可空，默认否', default: 'false' }
]

const classifyJsonRequestParams = [
  { name: 'GroupId', type: 'integer', required: false, description: '分组ID，与GroupCode、CategoryIds、CategoryCodes四选一', default: '-' },
  { name: 'GroupCode', type: 'string', required: false, description: '分组编码，与GroupId、CategoryIds、CategoryCodes四选一', default: '-' },
  { name: 'CategoryIds', type: 'integer[]', required: false, description: '分类ID集合，与GroupId、GroupCode、CategoryCodes四选一', default: '-' },
  { name: 'CategoryCodes', type: 'string[]', required: false, description: '分类编码集合，与GroupId、GroupCode、CategoryIds四选一', default: '-' },
  { name: 'FileUrls', type: 'object[]', required: false, description: '文件对象，与File二选一', default: '-' },
  { name: 'BusinessId', type: 'integer', required: false, description: '业务ID，可空', default: '-' },
  { name: 'ExtractFields', type: 'boolean', required: false, description: '是否提取数据，可空，默认否', default: 'false' }
]

const classifyResponseParams = [
  { name: 'code', type: 'integer', description: '编码，200为成功' },
  { name: 'msg', type: 'string | null', description: '描述信息' },
  { name: 'data', type: 'string', description: '返回的批次号，根据批次号调用结果查询接口获取分类结果' },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

const resultQueryRequestParams = [
  { name: 'batchNo', type: 'string', required: true, description: '分类接口返回的批次号', default: '-' }
]

const resultResponseParams = [
  { name: 'code', type: 'integer', description: '编码，200为成功' },
  { name: 'msg', type: 'string | null', description: '描述信息' },
  {
    name: 'data',
    type: 'object',
    description: '数据',
    subProps: [
      { name: 'Code', type: 'integer', description: '0分类中  1分类完成' },
      { name: 'Progress', type: 'integer', description: '进度 0~100' },
      {
        name: 'Result',
        type: 'object[]',
        description: '分类结果',
        subProps: [
          { name: 'FileName', type: 'string', description: '文件名称' },
          { name: 'TotalPages', type: 'integer', description: '总页数，图片类型时为1' },
          { name: 'ElapsedMs', type: 'string', description: '整个文件分类耗时，毫秒' },
          { name: 'Status', type: 'string', description: '状态描述：成功|失败' },
          { name: 'Message', type: 'string', description: '描述信息，失败时为异常描述' },
          {
            name: 'ClassificationResults',
            type: 'object[]',
            description: '文件page分类结果',
            subProps: [
              { name: 'FileUrl', type: 'string', description: '单页文件链接地址' },
              { name: 'FileName', type: 'string', description: '单页文件名称' },
              { name: 'CategoryId', type: 'integer | null', description: '附件类型ID' },
              { name: 'CategoryCode', type: 'string | null', description: '附件类型编码' },
              { name: 'CategoryName', type: 'string', description: '附件类型名称' },
              { name: 'IsCover', type: 'boolean', description: '是否封面' },
              { name: 'OcrText', type: 'string', description: 'OCR结果' },
              { name: 'ExtractDataJson', type: 'object | null', description: '提取数据' }
            ]
          },
          {
            name: 'Segments',
            type: 'object[]',
            description: '分段信息',
            subProps: [
              { name: 'CategoryId', type: 'integer | null', description: '附件类型ID' },
              { name: 'CategoryCode', type: 'string', description: '附件类型编码' },
              { name: 'CategoryName', type: 'string', description: '附件类型名称' },
              { name: 'StartPage', type: 'integer', description: '开始页码' },
              { name: 'EndPage', type: 'integer', description: '结束页码' },
              { name: 'Confidence', type: 'number', description: '平均置信度' }
            ]
          }
        ]
      }
    ]
  },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

const categoryDataResponseParams = [
  { name: 'code', type: 'integer', description: '状态码，200成功' },
  { name: 'msg', type: 'string | null', description: '描述信息' },
  {
    name: 'data',
    type: 'object[]',
    description: '类别数组',
    subProps: [
      { name: 'Id', type: 'string', description: '类别ID' },
      { name: 'Code', type: 'string', description: '类别编码' },
      { name: 'Name', type: 'string', description: '类别名称' }
    ]
  },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

// 附件信息提取接口 - FORM版本参数
const extractFileFormRequestParams = [
  { name: 'CategoryCode', type: 'string', required: false, description: '分类编码', default: '-' },
  { name: 'ExtractMode', type: 'string', required: false, description: '取数模式：PAGE=按页, DOCUMENT=按文档', default: '-' },
  { name: 'File', type: 'string(binary)', required: false, description: '文件流，支持多文件，与FileUrls二选一，使用此字段需要参数为form-data', default: '-' }
]

// 附件信息提取接口 - JSON版本参数
const extractFileJsonRequestParams = [
  { name: 'CategoryCode', type: 'string', required: true, description: '分类编码' },
  { name: 'ExtractMode', type: 'string', required: true, description: '取数模式：PAGE=按页, DOCUMENT=按文档' },
  { name: 'FileUrls', type: 'object[]', required: true, description: '文件对象，与File二选一' }
]

const extractFileUrlParams = [
  { name: 'Url', type: 'string', required: false, description: '文件链接地址' },
  { name: 'FileName', type: 'string', required: false, description: '文件名称' }
]

const columns = {
  name: '参数名',
  type: '类型',
  default: '默认值',
  description: '说明'
}
</script>

# 附件分类 (Attachment Classification)

本模块包含附件分类、结果查询及类别数据获取等相关接口。

## 1. 附件分类接口 (Form-Data)

使用 Form-Data 方式提交附件进行分类。

- **接口地址**: `/api/Classfication/ClassifyFile`
- **请求方式**: `POST`
- **Content-Type**: `multipart/form-data`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="classifyFormRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```yaml [Request Body]
GroupId: 0
GroupCode: ""
CategoryIds:
  - "24"
  - "25"
CategoryCodes: ""
File: file://C:\Users\Administrator\Downloads\测试发票\dzfp_25417000000231741186_北京广赢佳科技有限公司_500.00_20250919112011.pdf
BusinessId: 0
ExtractFields: "true"
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "0af20d0f4a644104832118187dadf0cc",
  "total": 0,
  "rows": null
}
```

:::

## 2. 附件分类接口 (JSON)

使用 JSON 方式提交附件URL进行分类。

- **接口地址**: `/net-api/api/Classfication/ClassifyFile`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="classifyJsonRequestParams" :columns="columns" />

#### FileUrls 对象结构

| 参数名 | 类型 | 说明 |
| :--- | :--- | :--- |
| Url | string | 文件链接地址 |
| FileName | string | 文件名称 |

### 响应参数

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```json [Request]
{
  "GroupId": null,
  "GroupCode": null,
  "CategoryIds": null,
  "CategoryCodes": [
    "INVOICE_NOMAL"
  ],
  "BusinessId": null,
  "ExtractFields": true,
  "FileUrls": [
    {
      "Url": "https://wiseai-prod.obs.cn-north-1.myhuaweicloud.com/2026/02/04/7a06df317f4d4b5b953cc56d5013253a.jpg",
      "FileName": "90cd567e59854ca1a292c9579726899c.jpg"
    }
  ]
}
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "0af20d0f4a644104832118187dadf0cc",
  "total": 0,
  "rows": null
}
```

:::

## 3. 查询分类取数结果

根据批次号查询附件分类的结果。

- **接口地址**: `/api/Classfication/GetClassificationResult`
- **请求方式**: `GET`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="resultQueryRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="resultResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/api/Classfication/GetClassificationResult?batchNo=0af20d0f4a644104832118187dadf0cc" \
  -H "API-KEY: your-api-key"
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": {
    "Code": 1,
    "Progress": 100,
    "Result": [
      {
        "FileName": "ScreenShot_2026-03-12_083230_614.png",
        "TotalPages": 1,
        "ElapsedMs": "4249",
        "Status": "成功",
        "Message": "",
        "ClassificationResults": [
          {
            "FileUrl": "https://ap-core-oss.obs.cn-north-1.myhuaweicloud.com/6da4fe0c95a345b99762a2ac21a9209c.png",
            "FileName": "6da4fe0c95a345b99762a2ac21a9209c.png",
            "CategoryId": null,
            "CategoryCode": null,
            "CategoryName": "其他",
            "IsCover": false,
            "OcrText": "密码类型：0-A密码\n块数（0-63）：\n1\n密钥：\n等待超时时间（s）\n10\n移卡模式：0x00-移动到磁条卡操作位置\nIC卡（业务）输出\n读取机具识别号成功，识别号是：00010601202512000735",
            "ExtractDataJson": null
          }
        ],
        "Segments": [
          {
            "CategoryId": null,
            "CategoryName": "其他",
            "StartPage": 0,
            "EndPage": 0,
            "Confidence": 0.95
          }
        ]
      }
    ]
  },
  "total": 0,
  "rows": null
}
```

:::

## 4. 查询附件分类类别数据

获取当前租户下启用状态的附件分类类别数据。

- **接口地址**: `/api/Classfication/GetCategoryData`
- **请求方式**: `GET`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 响应参数

<ApiTable :data="categoryDataResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/api/Classfication/GetCategoryData" \
  -H "API-KEY: your-api-key"
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": [
    {
      "Id": "21",
      "Code": "INVOICE_NOMAL",
      "Name": "电子发票"
    },
    {
      "Id": "23",
      "Code": "PunchinRecord",
      "Name": "打卡记录"
    },
    {
      "Id": "26",
      "Code": "INVOICE_FLIGHT",
      "Name": "航空行程单"
    },
    {
      "Id": "27",
      "Code": "attach_flight_apply",
      "Name": "特殊情况机票申请附件"
    },
    {
      "Id": "29",
      "Code": "TaxiReceipt",
      "Name": "打车行程单"
    },
    {
      "Id": "30",
      "Code": "AirReceipt",
      "Name": "航空行程单"
    },
    {
      "Id": "31",
      "Code": "TripLog",
      "Name": "导航记录"
    },
    {
      "Id": "32",
      "Code": "TradeRecord",
      "Name": "交易记录"
    },
    {
      "Id": "36",
      "Code": "contract",
      "Name": "合同"
    },
    {
      "Id": "39",
      "Code": "TrainTicket",
      "Name": "火车票"
    },
    {
      "Id": "40",
      "Code": "agree",
      "Name": "领导审批记录"
    },
    {
      "Id": "42",
      "Code": "lease",
      "Name": "租赁合同"
    }
  ],
  "total": 0,
  "rows": null
}
```

:::

## 5. 附件信息提取接口 (Form-Data)

使用 Form-Data 方式提交附件进行信息提取。

- **接口地址**: `/api/Classfication/ExtractFile`
- **请求方式**: `POST`
- **Content-Type**: `multipart/form-data`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="extractFileFormRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### 说明

- **ExtractMode**: 取数模式
  - `PAGE`: 按页提取
  - `DOCUMENT`: 按文档提取
- 返回的 `data` 字段为批次号，需使用该批次号调用[查询分类结果](#3-查询分类结果)接口获取提取结果

### 示例

::: code-group

```yaml [Request Body]
CategoryCode: CG_CONT
ExtractMode: DOCUMENT
File: file://C:\Users\Administrator\Downloads\采购合同样例 - zdjz20250303(1).pdf
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "0af20d0f4a644104832118187dadf0cc",
  "total": 0,
  "rows": null
}
```

:::

## 6. 附件信息提取接口 (JSON)

使用 JSON 方式提交附件URL进行信息提取。

- **接口地址**: `/api/Classfication/ExtractFile`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="extractFileJsonRequestParams" :columns="columns" :show-default="false" />

#### FileUrls 对象结构

<ApiTable :data="extractFileUrlParams" :columns="columns" :show-default="false" :show-required="false" />

### 响应参数

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### 说明

- **ExtractMode**: 取数模式
  - `PAGE`: 按页提取
  - `DOCUMENT`: 按文档提取
- 返回的 `data` 字段为批次号，需使用该批次号调用[查询分类结果](#3-查询分类结果)接口获取提取结果

### 示例

::: code-group

```json [Request]
{
  "CategoryCode": "CG_CONT",
  "ExtractMode": "DOCUMENT",
  "FileUrls": [
    {
      "Url": "https://wiseai-prod.obs.cn-north-1.myhuaweicloud.com/2026/02/04/7a06df317f4d4b5b953cc56d5013253a.jpg",
      "FileName": "90cd567e59854ca1a292c9579726899c.jpg"
    }
  ]
}
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "0af20d0f4a644104832118187dadf0cc",
  "total": 0,
  "rows": null
}
```

:::
