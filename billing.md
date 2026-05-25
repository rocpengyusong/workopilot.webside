<script setup>
const recordUsageRequestParams = [
  { name: 'DigitalEmployeeId', type: 'integer', required: true, description: '数字员工ID', default: '-' },
  { name: 'Remark', type: 'string', required: false, description: '备注信息', default: '-' }
]

const recordUsageResponseParams = [
  { name: 'code', type: 'integer', description: '状态码，200表示成功，0表示失败' },
  { name: 'msg', type: 'string', description: '描述信息' },
  { name: 'data', type: 'boolean', description: '记录结果，true表示成功，false表示失败' }
]

const columns = {
  name: '参数名',
  type: '类型',
  default: '默认值',
  description: '说明'
}
</script>

# 计费模块 (Billing)

计费模块用于记录数字员工的使用情况，系统会自动扣减租户的套餐额度，并记录使用日志。

## 1. 数字员工使用记录接口

用于记录数字员工的使用情况，系统会自动扣减租户的套餐额度，并记录使用日志。

- **接口地址**: `/net-api/api/Billing/RecordUsage`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

### 请求头

| 参数名 | 必选 | 类型 | 说明 |
| :--- | :--- | :--- | :--- |
| Authorization | 是 | string | Bearer Token，从用户登录或认证接口获取 |
| Content-Type | 是 | string | application/json |

### 请求参数

<ApiTable :data="recordUsageRequestParams" :columns="columns" />

### 响应参数

<ApiTable :data="recordUsageResponseParams" :columns="columns" :show-default="false" />

### 示例

::: code-group

```json [Request]
{
  "DigitalEmployeeId": 12345,
  "Remark": "客户咨询使用"
}
```

```json [Response - 成功]
{
  "code": 200,
  "msg": "记录成功",
  "data": true
}
```

```json [Response - 无可用套餐]
{
  "code": 0,
  "msg": "无可用套餐",
  "data": false
}
```

```json [Response - 数字员工不存在或已停用]
{
  "code": 0,
  "msg": "数字员工不存在或已停用",
  "data": false
}
```

```json [Response - 租户ID不能为空]
{
  "code": 0,
  "msg": "租户ID不能为空",
  "data": false
}
```

:::

### 业务逻辑说明

1. **权限验证**：接口需要用户认证，系统会从 Token 中提取租户ID（TenantId）和用户ID（UserId）
2. **数字员工验证**：验证指定的数字员工是否存在且处于激活状态
3. **套餐扣减**：
   - 自动查找最早到期且仍有剩余额度的套餐
   - 使用乐观锁机制防止并发问题
   - 如果发生并发冲突，会自动重试一次
4. **额度更新**：同步更新租户总额度表
5. **使用记录**：保存详细的使用日志，包括使用时间、操作人等信息
6. **状态检查**：如果套餐额度用完，自动标记为已耗尽状态

### 错误码说明

| 错误码 | 说明 |
| :--- | :--- |
| 200 | 记录成功 |
| 0 | 记录失败，具体错误信息见 msg 字段 |

### 注意事项

1. 该接口需要用户登录认证，请确保请求头中包含有效的 Authorization Token
2. 租户必须拥有可用的套餐额度才能成功记录使用
3. 每次调用会扣减数字员工对应的价格（Price）额度
4. 系统会自动处理并发情况，无需客户端特殊处理
5. 使用记录会保存详细日志，便于后续查询和统计

### 使用场景

- 数字员工被调用时记录使用情况
- 租户消费额度的实时扣减
- 使用统计和计费依据
