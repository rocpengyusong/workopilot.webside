<script setup>
const classifyFormRequestParams = [
  { name: 'GroupId', type: 'integer', required: false, description: 'Group ID. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'GroupCode', type: 'string', required: false, description: 'Group Code. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'CategoryIds', type: 'string[]', required: false, description: 'Set of Category IDs. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'CategoryCodes', type: 'string[]', required: false, description: 'Set of Category Codes. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'File', type: 'string(binary)', required: false, description: 'File stream, supports multiple files. Choose one of File or FileUrls. Using this field requires form-data parameters.', default: '-' },
  { name: 'BusinessId', type: 'integer', required: false, description: 'Business ID, can be null.', default: '-' },
  { name: 'ExtractFields', type: 'boolean', required: false, description: 'Whether to extract data. Can be null, default is false.', default: 'false' }
]

const classifyJsonRequestParams = [
  { name: 'GroupId', type: 'integer', required: false, description: 'Group ID. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'GroupCode', type: 'string', required: false, description: 'Group Code. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'CategoryIds', type: 'integer[]', required: false, description: 'Set of Category IDs. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'CategoryCodes', type: 'string[]', required: false, description: 'Set of Category Codes. Choose one of GroupId, GroupCode, CategoryIds, CategoryCodes.', default: '-' },
  { name: 'FileUrls', type: 'object[]', required: false, description: 'File objects. Choose one of File or FileUrls.', default: '-' },
  { name: 'BusinessId', type: 'integer', required: false, description: 'Business ID, can be null.', default: '-' },
  { name: 'ExtractFields', type: 'boolean', required: false, description: 'Whether to extract data. Can be null, default is false.', default: 'false' }
]

const classifyResponseParams = [
  { name: 'code', type: 'integer', description: 'Response code, 200 for success.' },
  { name: 'msg', type: 'string | null', description: 'Description message.' },
  { name: 'data', type: 'string', description: 'The returned batch number. Call the result query interface with this batch number to get classification results.' },
  { name: 'total', type: 'integer', description: 'Total count.' },
  { name: 'rows', type: 'null', description: 'Row data.' }
]

const resultQueryRequestParams = [
  { name: 'batchNo', type: 'string', required: true, description: 'Batch number returned by the classification interface.', default: '-' }
]

const resultResponseParams = [
  { name: 'code', type: 'integer', description: 'Response code, 200 for success.' },
  { name: 'msg', type: 'string | null', description: 'Description message.' },
  {
    name: 'data',
    type: 'object',
    description: 'Data',
    subProps: [
      { name: 'Code', type: 'integer', description: '0 for classifying, 1 for classification completed.' },
      { name: 'Progress', type: 'integer', description: 'Progress 0~100.' },
      {
        name: 'Result',
        type: 'object[]',
        description: 'Classification result',
        subProps: [
          { name: 'FileName', type: 'string', description: 'File name.' },
          { name: 'TotalPages', type: 'integer', description: 'Total pages, 1 for images.' },
          { name: 'ElapsedMs', type: 'string', description: 'Time taken for the entire file classification, in milliseconds.' },
          { name: 'Status', type: 'string', description: 'Status description: Success|Failure.' },
          { name: 'Message', type: 'string', description: 'Description message, exception description when failed.' },
          {
            name: 'ClassificationResults',
            type: 'object[]',
            description: 'Page-level classification results.',
            subProps: [
              { name: 'FileUrl', type: 'string', description: 'Single-page file link.' },
              { name: 'FileName', type: 'string', description: 'Single-page file name.' },
              { name: 'CategoryId', type: 'integer | null', description: 'Attachment type ID.' },
              { name: 'CategoryCode', type: 'string | null', description: 'Attachment type code.' },
              { name: 'CategoryName', type: 'string', description: 'Attachment type name.' },
              { name: 'IsCover', type: 'boolean', description: 'Whether it is a cover.' },
              { name: 'OcrText', type: 'string', description: 'OCR results.' },
              { name: 'ExtractDataJson', type: 'object | null', description: 'Extracted data.' }
            ]
          },
          {
            name: 'Segments',
            type: 'object[]',
            description: 'Segmentation information.',
            subProps: [
              { name: 'CategoryId', type: 'integer | null', description: 'Attachment type ID.' },
              { name: 'CategoryCode', type: 'string', description: 'Attachment type code.' },
              { name: 'CategoryName', type: 'string', description: 'Attachment type name.' },
              { name: 'StartPage', type: 'integer', description: 'Start page number.' },
              { name: 'EndPage', type: 'integer', description: 'End page number.' },
              { name: 'Confidence', type: 'number', description: 'Average confidence score.' }
            ]
          }
        ]
      }
    ]
  },
  { name: 'total', type: 'integer', description: 'Total count.' },
  { name: 'rows', type: 'null', description: 'Row data.' }
]

const categoryDataResponseParams = [
  { name: 'code', type: 'integer', description: 'Status code, 200 for success.' },
  { name: 'msg', type: 'string | null', description: 'Description message.' },
  {
    name: 'data',
    type: 'object[]',
    description: 'Category array',
    subProps: [
      { name: 'Id', type: 'string', description: 'Category ID.' },
      { name: 'Code', type: 'string', description: 'Category code.' },
      { name: 'Name', type: 'string', description: 'Category name.' }
    ]
  },
  { name: 'total', type: 'integer', description: 'Total count.' },
  { name: 'rows', type: 'null', description: 'Row data.' }
]

// Attachment Information Extraction Interface - FORM Version Parameters
const extractFileFormRequestParams = [
  { name: 'CategoryCode', type: 'string', required: false, description: 'Category code.', default: '-' },
  { name: 'ExtractMode', type: 'string', required: false, description: 'Extraction mode: PAGE (by page), DOCUMENT (by document).', default: '-' },
  { name: 'File', type: 'string(binary)', required: false, description: 'File stream, supports multiple files. Choose one of File or FileUrls. Using this field requires form-data parameters.', default: '-' }
]

// Attachment Information Extraction Interface - JSON Version Parameters
const extractFileJsonRequestParams = [
  { name: 'CategoryCode', type: 'string', required: true, description: 'Category code.' },
  { name: 'ExtractMode', type: 'string', required: true, description: 'Extraction mode: PAGE (by page), DOCUMENT (by document).' },
  { name: 'FileUrls', type: 'object[]', required: true, description: 'File objects. Choose one of File or FileUrls.' }
]

const extractFileUrlParams = [
  { name: 'Url', type: 'string', required: false, description: 'File link URL.' },
  { name: 'FileName', type: 'string', required: false, description: 'File name.' }
]

const columns = {
  name: 'Parameter',
  type: 'Type',
  default: 'Default',
  description: 'Description'
}
</script>

# Attachment Classification

This module includes interfaces for attachment classification, result query, and category data retrieval.

## 1. Attachment Classification Interface (Form-Data)

Submit attachments for classification using Form-Data.

- **Endpoint**: `/api/Classfication/ClassifyFile`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="classifyFormRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```yaml [Request Body]
GroupId: 0
GroupCode: ""
CategoryIds:
  - "24"
  - "25"
CategoryCodes: ""
File: file://C:\Users\Administrator\Downloads\Test_Invoices\dzfp_25417000000231741186_Beijing_Guangyingjia_Tech_500.00_20250919112011.pdf
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

## 2. Attachment Classification Interface (JSON)

Submit attachment URLs for classification using JSON.

- **Endpoint**: `/net-api/api/Classfication/ClassifyFile`
- **Method**: `POST`
- **Content-Type**: `application/json`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="classifyJsonRequestParams" :columns="columns" />

#### FileUrls Object Structure

| Parameter | Type | Description |
| :--- | :--- | :--- |
| Url | string | File link URL |
| FileName | string | File name |

### Response Parameters

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### Example

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

## 3. Query Classification Data Result

Query the results of attachment classification based on the batch number.

- **Endpoint**: `/api/Classfication/GetClassificationResult`
- **Method**: `GET`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="resultQueryRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="resultResponseParams" :columns="columns" :show-default="false" />

### Example

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
        "Status": "Success",
        "Message": "",
        "ClassificationResults": [
          {
            "FileUrl": "https://ap-core-oss.obs.cn-north-1.myhuaweicloud.com/6da4fe0c95a345b99762a2ac21a9209c.png",
            "FileName": "6da4fe0c95a345b99762a2ac21a9209c.png",
            "CategoryId": null,
            "CategoryCode": null,
            "CategoryName": "Other",
            "IsCover": false,
            "OcrText": "Password Type: 0-A Password\nBlocks (0-63):\n1\nKey:\nWait Timeout (s)\n10\nCard Move Mode: 0x00-Move to magnetic stripe card operation position\nIC Card (Business) Output\nRead device ID success, ID is: 00010601202512000735",
            "ExtractDataJson": null
          }
        ],
        "Segments": [
          {
            "CategoryId": null,
            "CategoryName": "Other",
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

## 4. Query Attachment Classification Category Data

Retrieve the attachment classification category data enabled for the current tenant.

- **Endpoint**: `/api/Classfication/GetCategoryData`
- **Method**: `GET`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Response Parameters

<ApiTable :data="categoryDataResponseParams" :columns="columns" :show-default="false" />

### Example

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
      "Name": "Electronic Invoice"
    },
    {
      "Id": "23",
      "Code": "PunchinRecord",
      "Name": "Punch-in Record"
    },
    {
      "Id": "26",
      "Code": "INVOICE_FLIGHT",
      "Name": "Airline Itinerary"
    },
    {
      "Id": "27",
      "Code": "attach_flight_apply",
      "Name": "Special Case Flight Application Attachment"
    },
    {
      "Id": "29",
      "Code": "TaxiReceipt",
      "Name": "Taxi Itinerary"
    },
    {
      "Id": "30",
      "Code": "AirReceipt",
      "Name": "Airline Itinerary"
    },
    {
      "Id": "31",
      "Code": "TripLog",
      "Name": "Navigation Record"
    },
    {
      "Id": "32",
      "Code": "TradeRecord",
      "Name": "Transaction Record"
    },
    {
      "Id": "36",
      "Code": "contract",
      "Name": "Contract"
    },
    {
      "Id": "39",
      "Code": "TrainTicket",
      "Name": "Train Ticket"
    },
    {
      "Id": "40",
      "Code": "agree",
      "Name": "Leader Approval Record"
    },
    {
      "Id": "42",
      "Code": "lease",
      "Name": "Lease Contract"
    }
  ],
  "total": 0,
  "rows": null
}
```

:::

## 5. Attachment Information Extraction Interface (Form-Data)

Submit attachments for information extraction using Form-Data.

- **Endpoint**: `/api/Classfication/ExtractFile`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="extractFileFormRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### Description

- **ExtractMode**: Extraction mode
  - `PAGE`: Extract by page
  - `DOCUMENT`: Extract by document
- The returned `data` field is the batch number. You need to call the [Query Classification Result](#3-query-classification-data-result) interface with this batch number to get the extraction results.

### Example

::: code-group

```yaml [Request Body]
CategoryCode: CG_CONT
ExtractMode: DOCUMENT
File: file://C:\Users\Administrator\Downloads\Purchase_Contract_Example_-_zdjz20250303(1).pdf
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

## 6. Attachment Information Extraction Interface (JSON)

Submit attachment URLs for information extraction using JSON.

- **Endpoint**: `/api/Classfication/ExtractFile`
- **Method**: `POST`
- **Content-Type**: `application/json`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="extractFileJsonRequestParams" :columns="columns" :show-default="false" />

#### FileUrls Object Structure

<ApiTable :data="extractFileUrlParams" :columns="columns" :show-default="false" :show-required="false" />

### Response Parameters

<ApiTable :data="classifyResponseParams" :columns="columns" :show-default="false" />

### Description

- **ExtractMode**: Extraction mode
  - `PAGE`: Extract by page
  - `DOCUMENT`: Extract by document
- The returned `data` field is the batch number. You need to call the [Query Classification Result](#3-query-classification-data-result) interface with this batch number to get the extraction results.

### Example

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
