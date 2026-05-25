<script setup>
const billCollectRequestParams = [
  { name: 'RequestId', type: 'number', required: true, description: '请求唯一标识，可空，为空时系统自动生成并在输出响应头X-C-RequestID返回', default: '-' },
  { name: 'IsAudit', type: 'boolean', required: true, description: '是否同步审核', default: '-' },
  { name: 'Method', type: 'string', required: true, description: '采集方式 API、RPA', default: '-' },
  { name: 'BillHeader', type: 'object', required: true, description: '单头信息', default: '-' },
  { name: 'BillDetails', type: 'object[]', required: true, description: '单据明细', default: '-' },
  { name: 'BillAttachments', type: 'object[]', required: true, description: '单据附件', default: '-' }
]

const billHeaderParams = [
  { name: 'BillId', type: 'string', required: true, description: '业务系统单据ID' },
  { name: 'BillCode', type: 'string', required: true, description: '业务系统单据编号' },
  { name: 'AppCode', type: 'string', required: true, description: '智审平台业务系统编号(对应单据来源系统)' },
  { name: 'SourceType', type: 'integer', required: true, description: '来源 1RPA 2API' },
  { name: 'BillTypeCode', type: 'string', required: true, description: '单据类型编码' },
  { name: 'TotalAmount', type: 'number', required: true, description: '报销金额' },
  { name: 'RequestUserId', type: 'string', required: true, description: '单据申请人' },
  { name: 'CompanyId', type: 'string', required: true, description: '公司ID' },
  { name: 'SubmitTime', type: 'string', required: true, description: '业务提交时间' },
  { name: 'ExtendJsonData', type: 'string', required: true, description: '单据扩展JSON数据' }
]

const billDetailParams = [
  { name: 'LineNo', type: 'string', required: false, description: '行号' },
  { name: 'ItemCode', type: 'string', required: false, description: '费用科目编码' },
  { name: 'Amount', type: 'number', required: false, description: '报销金额' },
  { name: 'ExtendJsonData', type: 'string', required: false, description: '明细扩展JSON数据' }
]

const billAttachmentParams = [
  { name: 'ItemLineNo', type: 'string', required: false, description: '明细行号，为空表示为单头附件' },
  { name: 'FileUrl', type: 'string', required: false, description: '文件下载/预览地址' },
  { name: 'FileType', type: 'string', required: false, description: '文件类型: pdf, jpg, png' },
  { name: 'FileName', type: 'string', required: false, description: '文件名称' }
]

const billCollectResponseParams = [
  { name: 'code', type: 'integer', description: '响应码，200成功' },
  { name: 'msg', type: 'string | null', description: '描述信息' },
  { name: 'data', type: 'string', description: '入参同步审核时，会返回一个审核批次号' },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

const billAuditRequestParams = [
  { name: 'BillNo', type: 'string', required: true, description: '单据编号，与单据ID二选一', default: '-' },
  { name: 'BillId', type: 'string', required: true, description: '单据ID，与单据编号二选一', default: '-' }
]

const billAuditResponseParams = [
  { name: 'code', type: 'integer', description: '响应码，200成功' },
  { name: 'msg', type: 'string | null', description: '描述信息' },
  { name: 'data', type: 'string', description: '审核批次号' },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

const getBillAuditResultRequestParams = [
  { name: 'BillNo', type: 'string', required: true, description: '单据编号', default: '-' }
]

const auditStatisticsParams = [
  { name: 'AiSummary', type: 'string', description: 'AI审核总结，markdown格式' },
  { name: 'TotalCount', type: 'integer', description: '总审核项条数' },
  { name: 'PassedCount', type: 'integer', description: '通过审核项条数' },
  { name: 'WarningCount', type: 'integer', description: '警告审核项条数' },
  { name: 'BlockCount', type: 'integer', description: '阻断审核项条数' }
]

const headerAuditItemParams = [
  { name: 'PointName', type: 'string', description: '审核项名称' },
  { name: 'IsPassed', type: 'boolean', description: '是否审核通过' },
  { name: 'RejectionReason', type: 'string', description: '拒绝原因' },
  { name: 'RiskLevel', type: 'string', description: '风险等级，警告(WARNING)/阻断(BLOCK)' }
]

const reviewPointParams = [
  { name: 'PointName', type: 'string', description: '审核项名称' },
  { name: 'IsPassed', type: 'boolean', description: '是否审核通过' },
  { name: 'RejectionReason', type: 'string', description: '拒绝原因' },
  { name: 'RiskLevel', type: 'string', description: '风险等级，警告(WARNING)/阻断(BLOCK)' }
]

const detailAuditItemParams = [
  { name: 'CategoryName', type: 'string', description: '明细科目名称' },
  { name: 'DeclaredAmount', type: 'integer', description: '明细金额' },
  { name: 'IsPassed', type: 'boolean', description: '是否通过' },
  { name: 'ReviewPoints', type: 'object[]', description: '审核点列表' },
  { name: 'WarningCount', type: 'integer', description: '警告项条数' },
  { name: 'BlockCount', type: 'integer', description: '阻断项条数' }
]

const getBillAuditResultResponseParams = [
  { name: 'code', type: 'integer', description: '响应码，200成功' },
  { name: 'msg', type: 'string', description: '描述信息' },
  {
    name: 'data',
    type: 'object',
    description: '审核结果数据',
    subProps: [
      { name: 'BillId', type: 'string', description: '单据ID' },
      { name: 'BillNo', type: 'string', description: '单据编号' },
      { name: 'ReportId', type: 'string', description: '审核报告ID' },
      { name: 'ReportTime', type: 'string', description: '审核报告生成时间' },
      {
        name: 'Statistics',
        type: 'object',
        description: '审核统计',
        subProps: auditStatisticsParams
      },
      {
        name: 'HeaderAuditItems',
        type: 'object[]',
        description: '整单审核',
        subProps: headerAuditItemParams
      },
      {
        name: 'DetailAuditItems',
        type: 'object[]',
        description: '明细审核',
        subProps: detailAuditItemParams
      }
    ]
  },
  { name: 'total', type: 'integer', description: '总数' },
  { name: 'rows', type: 'null', description: '行数据' }
]

const columns = {
  name: '参数名',
  type: '类型',
  default: '默认值',
  description: '说明'
}
</script>

# 单据审核 (Bill Audit)

本模块包含单据采集、审核发起及结果查询等相关接口。

## 1. 单据采集

智能审核模块单据信息采集接口。

- **接口地址**: `/api/BillAudit/BillCollect`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="billCollectRequestParams" :columns="columns" />

#### BillHeader 对象结构

<ApiTable :data="billHeaderParams" :columns="columns" :show-default="false" />

#### BillDetails 对象结构

<ApiTable :data="billDetailParams" :columns="columns" :show-default="false" />

#### BillAttachments 对象结构

<ApiTable :data="billAttachmentParams" :columns="columns" :show-default="false" />

### 响应参数

<ApiTable :data="billCollectResponseParams" :columns="columns" :show-default="false" />

### 示例

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
    "ExtendJsonData": "{\"TenantID\":\"wise2021\",\"CompanyGroupID\":\"afe7af99-fc61-eb11-a82d-8b16757882a9\",\"CompanyCode\":\"wise2021\",\"CompanyCode_Name\":\"济南企财通软件有限公司\",\"RequestCode\":\"GYJ_CLBX2026031200002\",\"RequestName\":\"AP事业部企财通超级管理员.2026-03-12差旅报销单\",\"RequestAmt\":11.0,\"RequestDate\":\"2026-03-12T11:22:20.88\",\"RequstUserID\":\"55e2af99-fc61-eb11-a82d-8b16757882a9\",\"RequstUserID_Name\":\"企财通超级管理员.\",\"OriginatorID\":\"55e2af99-fc61-eb11-a82d-8b16757882a9\",\"OriginatorID_Name\":\"企财通超级管理员.\",\"RequestStatusID_Name\":\"草稿\",\"StatusResourceKey\":\"RB_UnSubmit\",\"StatusLabelClass\":\"default\",\"BillConfigID\":2740,\"BillConfigID_Name\":\"差旅报销单\",\"RequestDeptID\":\"f8e5c1a9-4736-453c-ab41-b3df00a48359\",\"RequestDeptCode\":\"2026012701\",\"RequestDeptID_Name\":\"客户经理组\",\"CurrencyId\":10417,\"CurrencyId_Name\":\"人民币\",\"PaymentAmt\":11.0,\"BankKeyID\":11032,\"WriteOffAmt\":0.0,\"SupplierName\":\"\",\"BudgetOwnerCode\":\"\",\"BudgetOwnerName\":\"\",\"TpCode\":\"\",\"TpName\":\"\",\"DefaultEditUrl\":\"personClaimCf2?isShare=true&EnablePayment=true&EnableBudget&EnableRepay=true&theme=tm_sia&NeedBill=true&isHiddenReplace=true\",\"DefaultViewUrl\":\"personClaimCf2?isShare=true&EnablePayment=true&EnableBudget&EnableRepay=true&theme=tm_sia&NeedBill=true&ShowApplyBill=false&isHiddenReplace=true\",\"DeleteUrl\":\"\",\"DefaultApprovalUrl\":\"\",\"DefaultMobileUrl\":\"personClaim?EnableCategory=false&isShare=true&isHiddenReplace=true&ifNewRelation=true&NeedBill=true\",\"RevokedActionUrl\":\"\",\"RequestBillCode\":\"CLBX\",\"BillTypeID\":530,\"BillTypeID_Name\":\"报销类\",\"EnableDept\":false,\"EnableBrand\":false,\"EnableCompany\":false,\"EnableSupplier\":false,\"EnableDistributor\":false,\"BoolColumn1\":false,\"BoolColumn2\":false,\"BoolColumn3\":false,\"BoolColumn4\":false,\"BoolColumn5\":false,\"FloatColumn5\":11.0,\"IntColumn4\":2,\"StrColumn10_Name\":\"1\",\"RequestRemark\":\" \",\"OriginalAmt\":0.0,\"LanKey\":\"A1BB3D77-EDDF-ED11-A82F-A4CE4374B2F3\",\"LanguageID\":\"zh-CN\",\"EnableBudget\":true,\"EnableContract\":false,\"EnablePolicy\":true,\"EnableControl\":false,\"AutoIndexID\":203972,\"PaymentCompanyID\":\"\",\"PaymentStatusID\":0,\"PaymentStatusID_Name\":\"\",\"ID\":\"22f753c0-0dea-4605-b932-031c46bbd8d8\"}"
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

## 2. 开始审核

发起单据审核操作。

- **接口地址**: `/net-api/api/BillAudit/BillAudit`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="billAuditRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="billAuditResponseParams" :columns="columns" :show-default="false" />

### 示例

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

## 3. 获取审核结果

获取单据审核结果。

- **接口地址**: `/net-api/api/BillAudit/GetBillAuditResult`
- **请求方式**: `GET`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| API-KEY | 否 | string | API密钥 |

### 请求参数

<ApiTable :data="getBillAuditResultRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="getBillAuditResultResponseParams" :columns="columns" :show-default="false" />

#### Statistics 对象结构

<ApiTable :data="auditStatisticsParams" :columns="columns" :show-default="false" />

#### HeaderAuditItem 对象结构

<ApiTable :data="headerAuditItemParams" :columns="columns" :show-default="false" />

#### ReviewPoint 对象结构

<ApiTable :data="reviewPointParams" :columns="columns" :show-default="false" />

#### DetailAuditItem 对象结构

<ApiTable :data="detailAuditItemParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```bash [Request]
curl -X GET "http://api.example.com/net-api/api/BillAudit/GetBillAuditResult?BillNo=BX202601210001" \
  -H "API-KEY: your-api-key"
```

```json [Response]
{
  "code": 200,
  "msg": "查询成功",
  "data": {
    "BillId": "123456",
    "BillNo": "BX202601210001",
    "ReportId": "2016713808189460480",
    "ReportTime": "2026-01-29 11:23:04",
    "Statistics": {
      "AiSummary": "本单据合规评分 **<span style=\"color:red\">45分</span>**。基于<u style=\"text-decoration-style: dashed;\">《差旅报销管理制度》</u>，检测到 **2项阻断** 与 **2项警告**。。建议报销人立即补充：① _与明细金额1500.00元匹配的完整机票订单凭证（含乘机人、航段、日期、含税总价）_；② _出发地至到达地、乘机日（2025年11月17日）前后3小时内高铁二等座票价官方截图或查询结果_；③ _本次住宿的合规增值税发票（抬头为公司全称、税号正确、内容为'住宿费'）及对应加盖酒店公章的水单（含入住/退房日期、房型、金额）_；④ _如机票价格高于高铁票价，须同步提供直系负责人审批同意的聊天记录截图_。所有材料补全前，该报销单不予通过。",
      "TotalCount": 7,
      "PassedCount": 3,
      "WarningCount": 2,
      "BlockCount": 2
    },
    "HeaderAuditItems": [
      {
        "PointName": "每单总报销金额不能高于5000",
        "IsPassed": true,
        "RejectionReason": "",
        "RiskLevel": "BLOCK"
      }
    ],
    "DetailAuditItems": [
      {
        "CategoryName": "飞机票",
        "DeclaredAmount": 1500,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "机票价格合规性校验（需对比同程高铁二等座价格）",
            "IsPassed": false,
            "RejectionReason": "附件航空行程单显示机票总金额为670.00元，与明细申请金额1500.00元严重不符；且未提供对应日期、出发地/目的地的高铁二等座票价信息，无法验证是否符合《差旅报销管理制度》1.2.1.③条关于机票与高铁价格对比的要求。",
            "RiskLevel": "WARNING"
          }
        ],
        "WarningCount": 1,
        "BlockCount": 0
      },
      {
        "CategoryName": "住宿费",
        "DeclaredAmount": 100,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "审核如果是物业费用必须要提供了水单",
            "IsPassed": true,
            "RejectionReason": "",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "必须提供审核凭证附件",
            "IsPassed": false,
            "RejectionReason": "当前明细无关联附件，住宿费报销须提供合规发票及对应水单（如酒店水单）作为凭证依据",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "支付凭证附件上的金额要大于费用明细的数据",
            "IsPassed": false,
            "RejectionReason": "无关联附件，无法验证支付凭证金额是否覆盖申请金额",
            "RiskLevel": "BLOCK"
          },
          {
            "PointName": "报销金额不能大于300",
            "IsPassed": true,
            "RejectionReason": "",
            "RiskLevel": "BLOCK"
          }
        ],
        "WarningCount": 0,
        "BlockCount": 2
      },
      {
        "CategoryName": "飞机票",
        "DeclaredAmount": 800,
        "IsPassed": false,
        "ReviewPoints": [
          {
            "PointName": "机票价格合规性校验（需对比同期高铁二等座价格）",
            "IsPassed": false,
            "RejectionReason": "无关联附件（如机票行程单、高铁票价截图等），无法验证机票价格是否低于同程高铁二等座价格；且未提供乘机日期、出发地、目的地等关键信息，无法通过工具检索对应高铁票价，不满足制度1.2.1.③要求",
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
