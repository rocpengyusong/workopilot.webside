<script setup>
const billCollectRequestParams = [
  { name: 'RequestId', type: 'number', required: true, description: 'Unique identifier for the request, can be null. If null, the system will automatically generate one and return it in the response header X-C-RequestID.', default: '-' },
  { name: 'IsAudit', type: 'boolean', required: true, description: 'Whether to audit synchronously', default: '-' },
  { name: 'Method', type: 'string', required: true, description: 'Collection method: API, RPA', default: '-' },
  { name: 'BillHeader', type: 'object', required: true, description: 'Bill header information', default: '-' },
  { name: 'BillDetails', type: 'object[]', required: true, description: 'Bill detail information', default: '-' },
  { name: 'BillAttachments', type: 'object[]', required: true, description: 'Bill attachment information', default: '-' }
]

const billHeaderParams = [
  { name: 'BillId', type: 'string', required: true, description: 'Bill ID from the business system' },
  { name: 'BillCode', type: 'string', required: true, description: 'Bill code from the business system' },
  { name: 'AppCode', type: 'string', required: true, description: 'Business system code in the audit platform (corresponding to the bill source system)' },
  { name: 'SourceType', type: 'integer', required: true, description: 'Source: 1 for RPA, 2 for API' },
  { name: 'BillTypeCode', type: 'string', required: true, description: 'Bill type code' },
  { name: 'TotalAmount', type: 'number', required: true, description: 'Reimbursement amount' },
  { name: 'RequestUserId', type: 'string', required: true, description: 'Bill requester' },
  { name: 'CompanyId', type: 'string', required: true, description: 'Company ID' },
  { name: 'SubmitTime', type: 'string', required: true, description: 'Submission time in the business system' },
  { name: 'ExtendJsonData', type: 'string', required: true, description: 'Extended JSON data for the bill' }
]

const billDetailParams = [
  { name: 'LineNo', type: 'string', required: false, description: 'Line number' },
  { name: 'ItemCode', type: 'string', required: false, description: 'Expense item code' },
  { name: 'Amount', type: 'number', required: false, description: 'Reimbursement amount' },
  { name: 'ExtendJsonData', type: 'string', required: false, description: 'Extended JSON data for the detail line' }
]

const billAttachmentParams = [
  { name: 'ItemLineNo', type: 'string', required: false, description: 'Detail line number. If null, it represents a header attachment.' },
  { name: 'FileUrl', type: 'string', required: false, description: 'File download/preview URL' },
  { name: 'FileType', type: 'string', required: false, description: 'File type: pdf, jpg, png' },
  { name: 'FileName', type: 'string', required: false, description: 'File name' }
]

const billCollectResponseParams = [
  { name: 'code', type: 'integer', description: 'Response code, 200 for success' },
  { name: 'msg', type: 'string | null', description: 'Description message' },
  { name: 'data', type: 'string', description: 'If synchronous audit is requested, an audit batch number is returned.' },
  { name: 'total', type: 'integer', description: 'Total count' },
  { name: 'rows', type: 'null', description: 'Row data' }
]

const billAuditRequestParams = [
  { name: 'BillNo', type: 'string', required: true, description: 'Bill number. Choose either BillNo or BillId.', default: '-' },
  { name: 'BillId', type: 'string', required: true, description: 'Bill ID. Choose either BillNo or BillId.', default: '-' }
]

const billAuditResponseParams = [
  { name: 'code', type: 'integer', description: 'Response code, 200 for success' },
  { name: 'msg', type: 'string | null', description: 'Description message' },
  { name: 'data', type: 'string', description: 'Audit batch number' },
  { name: 'total', type: 'integer', description: 'Total count' },
  { name: 'rows', type: 'null', description: 'Row data' }
]

const getBillAuditResultRequestParams = [
  { name: 'BillNo', type: 'string', required: true, description: 'Bill number', default: '-' }
]

const auditStatisticsParams = [
  { name: 'AiSummary', type: 'string', description: 'AI audit summary in Markdown format' },
  { name: 'TotalCount', type: 'integer', description: 'Total number of audit items' },
  { name: 'PassedCount', type: 'integer', description: 'Number of passed audit items' },
  { name: 'WarningCount', type: 'integer', description: 'Number of warning audit items' },
  { name: 'BlockCount', type: 'integer', description: 'Number of blocking audit items' }
]

const headerAuditItemParams = [
  { name: 'PointName', type: 'string', description: 'Audit item name' },
  { name: 'IsPassed', type: 'boolean', description: 'Whether the audit item passed' },
  { name: 'RejectionReason', type: 'string', description: 'Rejection reason' },
  { name: 'RiskLevel', type: 'string', description: 'Risk level: WARNING / BLOCK' }
]

const reviewPointParams = [
  { name: 'PointName', type: 'string', description: 'Audit item name' },
  { name: 'IsPassed', type: 'boolean', description: 'Whether the audit point passed' },
  { name: 'RejectionReason', type: 'string', description: 'Rejection reason' },
  { name: 'RiskLevel', type: 'string', description: 'Risk level: WARNING / BLOCK' }
]

const detailAuditItemParams = [
  { name: 'CategoryName', type: 'string', description: 'Detail category name' },
  { name: 'DeclaredAmount', type: 'integer', description: 'Detail amount' },
  { name: 'IsPassed', type: 'boolean', description: 'Whether passed' },
  { name: 'ReviewPoints', type: 'object[]', description: 'List of audit points' },
  { name: 'WarningCount', type: 'integer', description: 'Number of warning items' },
  { name: 'BlockCount', type: 'integer', description: 'Number of blocking items' }
]

const getBillAuditResultResponseParams = [
  { name: 'code', type: 'integer', description: 'Response code, 200 for success' },
  { name: 'msg', type: 'string', description: 'Description message' },
  {
    name: 'data',
    type: 'object',
    description: 'Audit result data',
    subProps: [
      { name: 'BillId', type: 'string', description: 'Bill ID' },
      { name: 'BillNo', type: 'string', description: 'Bill number' },
      { name: 'ReportId', type: 'string', description: 'Audit report ID' },
      { name: 'ReportTime', type: 'string', description: 'Audit report generation time' },
      {
        name: 'Statistics',
        type: 'object',
        description: 'Audit statistics',
        subProps: auditStatisticsParams
      },
      {
        name: 'HeaderAuditItems',
        type: 'object[]',
        description: 'Bill-level audit',
        subProps: headerAuditItemParams
      },
      {
        name: 'DetailAuditItems',
        type: 'object[]',
        description: 'Line-level audit',
        subProps: detailAuditItemParams
      }
    ]
  },
  { name: 'total', type: 'integer', description: 'Total count' },
  { name: 'rows', type: 'null', description: 'Row data' }
]

const columns = {
  name: 'Parameter',
  type: 'Type',
  default: 'Default',
  description: 'Description'
}
</script>

# Bill Audit

This module includes interfaces for bill collection, audit initiation, and result query.

## 1. Bill Collection

Interface for bill information collection in the intelligent audit module.

- **Endpoint**: `/api/BillAudit/BillCollect`
- **Method**: `POST`
- **Content-Type**: `application/json`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="billCollectRequestParams" :columns="columns" />

#### BillHeader Object Structure

<ApiTable :data="billHeaderParams" :columns="columns" :show-default="false" />

#### BillDetails Object Structure

<ApiTable :data="billDetailParams" :columns="columns" :show-default="false" />

#### BillAttachments Object Structure

<ApiTable :data="billAttachmentParams" :columns="columns" :show-default="false" />

### Response Parameters

<ApiTable :data="billCollectResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```json [Request]
{
  "RequestId": 2031933932215341000,
  "IsAudit": true,
  "Method": "API",
  "BillHeader": {
    "BillId": "22f753c0-0dea-4605-b932-031c46bbd8d8",
    "BillCode": "GYJ_CLBX2026031200002",
    "AppCode": "APright",
    "SourceType": 2,
    "BillTypeCode": "CLBX",
    "TotalAmount": 11,
    "RequestUserId": "55e2af99-fc61-eb11-a82d-8b16757882a9",
    "CompanyId": "wise2021",
    "SubmitTime": "2026-03-12T11:22:20.88",
    "ExtendJsonData": "{\"TenantID\":\"wise2021\",\"CompanyGroupID\":\"afe7af99-fc61-eb11-a82d-8b16757882a9\",\"CompanyCode\":\"wise2021\",\"CompanyCode_Name\":\"Jinan Qicaitong Software Co., Ltd.\",\"RequestCode\":\"GYJ_CLBX2026031200002\",\"RequestName\":\"AP Division Qicaitong Super Admin. 2026-03-12 Travel Reimbursement Bill\",\"RequestAmt\":11.0,\"RequestDate\":\"2026-03-12T11:22:20.88\",\"RequstUserID\":\"55e2af99-fc61-eb11-a82d-8b16757882a9\",\"RequstUserID_Name\":\"Qicaitong Super Admin.\",\"OriginatorID\":\"55e2af99-fc61-eb11-a82d-8b16757882a9\",\"OriginatorID_Name\":\"Qicaitong Super Admin.\",\"RequestStatusID_Name\":\"Draft\",\"StatusResourceKey\":\"RB_UnSubmit\",\"StatusLabelClass\":\"default\",\"BillConfigID\":2740,\"BillConfigID_Name\":\"Travel Reimbursement Bill\",\"RequestDeptID\":\"f8e5c1a9-4736-453c-ab41-b3df00a48359\",\"RequestDeptCode\":\"2026012701\",\"RequestDeptID_Name\":\"Account Manager Team\",\"CurrencyId\":10417,\"CurrencyId_Name\":\"CNY\",\"PaymentAmt\":11.0,\"BankKeyID\":11032,\"WriteOffAmt\":0.0,\"SupplierName\":\"\",\"BudgetOwnerCode\":\"\",\"BudgetOwnerName\":\"\",\"TpCode\":\"\",\"TpName\":\"\",\"DefaultEditUrl\":\"personClaimCf2?isShare=true&EnablePayment=true&EnableBudget&EnableRepay=true&theme=tm_sia&NeedBill=true&isHiddenReplace=true\",\"DefaultViewUrl\":\"personClaimCf2?isShare=true&EnablePayment=true&EnableBudget&EnableRepay=true&theme=tm_sia&NeedBill=true&ShowApplyBill=false&isHiddenReplace=true\",\"DeleteUrl\":\"\",\"DefaultApprovalUrl\":\"\",\"DefaultMobileUrl\":\"personClaim?EnableCategory=false&isShare=true&isHiddenReplace=true&ifNewRelation=true&NeedBill=true\",\"RevokedActionUrl\":\"\",\"RequestBillCode\":\"CLBX\",\"BillTypeID\":530,\"BillTypeID_Name\":\"Reimbursement Class\",\"EnableDept\":false,\"EnableBrand\":false,\"EnableCompany\":false,\"EnableSupplier\":false,\"EnableDistributor\":false,\"BoolColumn1\":false,\"BoolColumn2\":false,\"BoolColumn3\":false,\"BoolColumn4\":false,\"BoolColumn5\":false,\"FloatColumn5\":11.0,\"IntColumn4\":2,\"StrColumn10_Name\":\"1\",\"RequestRemark\":\" \",\"OriginalAmt\":0.0,\"LanKey\":\"A1BB3D77-EDDF-ED11-A82F-A4CE4374B2F3\",\"LanguageID\":\"zh-CN\",\"EnableBudget\":true,\"EnableContract\":false,\"EnablePolicy\":true,\"EnableControl\":false,\"AutoIndexID\":203972,\"PaymentCompanyID\":\"\",\"PaymentStatusID\":0,\"PaymentStatusID_Name\":\"\",\"ID\":\"22f753c0-0dea-4605-b932-031c46bbd8d8\"}"
  },
  "BillDetails": [
    {
      "LineNo": "1",
      "ItemCode": "0301",
      "Amount": 11,
      "ExtendJsonData": "{\"BillHeadId\":\"22f753c0-0dea-4605-b932-031c46bbd8d8\",\"BillDetailId\":\"1355992b-83e7-413e-9c6d-db452cd75533\",\"ClaimAmt\":11.0,\"ClaimDate\":\"2026-03-12T11:22:37\",\"BudId\":\"cf24f7d0-5e7a-4a02-978c-96982a772cce\",\"PaymentAmt\":11.0,\"CommonExpenseItemId\":9112,\"ExcluAmt\":11.0,\"VatAmt\":0.0,\"ExpenseStandandType\":1,\"CutDownEnum\":3,\"ExItemBool2\":false,\"DetailSummary\":\"\",\"HasAllocated\":false,\"ID\":\"1355992b-83e7-413e-9c6d-db452cd75533\"}"
    }
  ],
  "BillAttachments": [
    {
      "ItemLineNo": "1",
      "FileUrl": "",
      "FileType": "",
      "FileName": ""
    }
  ]
}
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "f49494e65dec47f4b015ab18be697dee",
  "total": 0,
  "rows": null
}
```

:::

## 2. Start Audit

Initiate the bill audit operation.

- **Endpoint**: `/net-api/api/BillAudit/BillAudit`
- **Method**: `POST`
- **Content-Type**: `application/json`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="billAuditRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="billAuditResponseParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```json [Request]
{
  "BillNo": "GYJ_CLBX2026031200001",
  "BillId": ""
}
```

```json [Response]
{
  "code": 200,
  "msg": null,
  "data": "833ada762f6f4c47bdd0a270e3f02f38",
  "total": 0,
  "rows": null
}
```

:::

## 3. Get Audit Result

Get the result of the bill audit.

- **Endpoint**: `/net-api/api/BillAudit/GetBillAuditResult`
- **Method**: `GET`

### Request Headers

| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| API-KEY | No | string | API key |

### Request Parameters

<ApiTable :data="getBillAuditResultRequestParams" :columns="columns" />

### Response Parameters

<ApiTable :data="getBillAuditResultResponseParams" :columns="columns" :show-default="false" />

#### Statistics Object Structure

<ApiTable :data="auditStatisticsParams" :columns="columns" :show-default="false" />

#### HeaderAuditItem Object Structure

<ApiTable :data="headerAuditItemParams" :columns="columns" :show-default="false" />

#### ReviewPoint Object Structure

<ApiTable :data="reviewPointParams" :columns="columns" :show-default="false" />

#### DetailAuditItem Object Structure

<ApiTable :data="detailAuditItemParams" :columns="columns" :show-default="false" />

### Example

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/net-api/api/BillAudit/GetBillAuditResult?BillNo=BX202601210001" \
  -H "API-KEY: your-api-key"
```

```json [Response]
{
  "code": 200,
  "msg": "Query successful",
  "data": {
    "BillId": "123456",
    "BillNo": "BX202601210001",
    "ReportId": "2016713808189460480",
    "ReportTime": "2026-01-29 11:23:04",
    "Statistics": {
      "AiSummary": "The compliance score for this bill is **<span style=\"color:red\">45 points</span>**. Based on the <u style=\"text-decoration-style: dashed;\">《Travel Expense Management Policy》</u>, **2 blocking items** and **2 warning items** were detected. It is recommended that the requester immediately supplement: ① _Full flight ticket order certificate matching the detail amount of 1500.00 yuan (including passenger, flight segment, date, total tax-inclusive price)_；② _Official screenshot or search result of high-speed rail second-class seat price within 3 hours before and after the flight date (November 17, 2025) from departure to destination_；③ _Compliant VAT invoice for this accommodation (header is full company name, correct tax number, content is 'accommodation fee') and corresponding hotel official seal water bill (including check-in/check-out date, room type, amount)_；④ _If the flight price is higher than the high-speed rail fare, a screenshot of the chat record approved by the direct supervisor must be provided simultaneously_. The reimbursement will not be approved until all materials are completed.",
      "TotalCount": 7,
      "PassedCount": 3,
      "WarningCount": 2,
      "BlockCount": 2
    },
    "HeaderAuditItems": [
      {
        "PointName": "Total reimbursement amount per bill cannot exceed 5000",
        "IsPassed": true,
        "RejectionReason": "",
        "RiskLevel": "BLOCK"
      }
    ],
    "DetailAuditItems": [
      {
        "CategoryName": "Flight Ticket",
        "DeclaredAmount": 1500,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "Flight price compliance check (need to compare with Tongcheng high-speed rail second-class seat price)",
            "IsPassed": false,
            "RejectionReason": "The airline itinerary attachment shows a total flight amount of 670.00 yuan, which is seriously inconsistent with the detail application amount of 1500.00 yuan; and high-speed rail second-class seat price information for the corresponding date and departure/destination is not provided, making it impossible to verify whether it meets Article 1.2.1.③ of the 《Travel Expense Management Policy》 regarding flight and high-speed rail price comparison requirements.",
            "RiskLevel": "WARNING"
          }
        ],
        "WarningCount": 1,
        "BlockCount": 0
      },
      {
        "CategoryName": "Accommodation Fee",
        "DeclaredAmount": 100,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "Audit if property fees must provide a water bill",
            "IsPassed": true,
            "RejectionReason": "",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "Must provide audit evidence attachment",
            "IsPassed": false,
            "RejectionReason": "Current detail has no associated attachment. Accommodation fee reimbursement must provide compliant invoices and corresponding water bills (e.g., hotel water bill) as evidence.",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "Amount on payment voucher attachment must be greater than or equal to the expense detail data",
            "IsPassed": false,
            "RejectionReason": "No associated attachment, cannot verify if the payment voucher amount covers the applied amount.",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "Reimbursement amount cannot exceed 300",
            "IsPassed": true,
            "RejectionReason": "",
            "RiskLevel": "BLOCK"
          }
        ],
        "WarningCount": 0,
        "BlockCount": 2
      },
      {
        "CategoryName": "Flight Ticket",
        "DeclaredAmount": 800,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "Flight price compliance check (need to compare with high-speed rail second-class seat price of the same period)",
            "IsPassed": false,
            "RejectionReason": "No associated attachments (such as flight itineraries, high-speed rail fare screenshots, etc.), making it impossible to verify if the flight price is lower than the high-speed rail second-class seat price of the same period; and key information such as flight date, departure, and destination are not provided, preventing the tool from retrieving corresponding high-speed rail fares, failing to meet policy 1.2.1.③ requirements.",
            "RiskLevel": "WARNING"
          }
        ],
        "WarningCount": 1,
        "BlockCount": 0
      }
    ]
  },
  "total": 0,
  "rows": null
}
```

:::
