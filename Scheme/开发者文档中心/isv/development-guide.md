# Workopilot OpenAPI 开发指南

## 1. 文档概述

本文档面向独立软件开发商（ISV），提供 Workopilot OpenAPI 的接入开发指南，包括认证方式、错误码、限流规则等通用信息。

### 文档版本

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0.0 | 2025-02-04 | 初始版本 |

### 技术支持

- 技术支持邮箱: support@workopilot.com
- 问题反馈: https://github.com/workopilot/workopilot-agent-api/issues

---

## 2. 接入准备

### 2.1 获取 API Key

1. 登录 Workopilot 管理后台
2. 进入「应用管理」→「外部应用」
3. 创建新应用，系统自动分配 API Key
4. 保存 API Key，用于接口调用鉴权

**安全提示**: API Key 是敏感信息，请妥善保管，不要在客户端代码中暴露。

### 2.2 配置服务

如需使用特定平台服务（如飞书），需完成以下配置：

1. 在对应平台开放平台创建应用
2. 获取应用凭证（App ID、App Secret 等）
3. 在 Workopilot 后台「应用管理」→「应用服务」中添加服务配置
4. 填写必要的配置信息和回调地址

---

## 3. 认证方式

### 3.1 请求头认证

所有 OpenAPI 接口需在请求头中携带 API Key：

```
x-api-key: sk_live_xxxxxxxxxxxxxxxxxxxx
```

### 3.2 支持的认证方式

| 优先级 | 请求头名称 | 说明 |
|--------|------------|------|
| 1 | `x-api-key` | 推荐使用 |
| 2 | `aikey` | 兼容旧版本 |
| 3 | `Authorization: Bearer {token}` | 标准 Bearer Token |

### 3.3 认证示例

```bash
# 使用 x-api-key（推荐）
curl -X POST https://api.workopilot.com/openapi/app/lark/message/text \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"userId": "+8613800138000", "idType": "PHONE", "text": "Hello"}'

# 使用 Authorization Bearer
curl -X POST https://api.workopilot.com/openapi/app/lark/message/text \
  -H "Authorization: Bearer sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"userId": "+8613800138000", "idType": "PHONE", "text": "Hello"}'
```

---

## 4. 统一响应格式

### 4.1 成功响应

```json
{
  "code": 0,
  "message": "success",
  "data": { /* 业务数据 */ },
  "timestamp": 1736058900000
}
```

### 4.2 错误响应

```json
{
  "code": 10002,
  "message": "API Key 无效",
  "data": null,
  "timestamp": 1736058900000
}
```

---

## 5. 错误码说明

### 5.1 平台错误码

| 错误码 | 错误信息 | 说明 | 处理建议 |
|--------|----------|------|----------|
| 0 | success | 成功 | - |
| 10001 | API Key 缺失 | 请求头缺少 API Key | 添加 `x-api-key` 请求头 |
| 10002 | API Key 无效 | API Key 不存在或已过期 | 检查 API Key 是否正确 |
| 10003 | 应用已禁用 | 应用已被管理员禁用 | 联系管理员启用应用 |
| 20001 | 参数校验失败 | 请求参数格式错误 | 检查请求参数格式和必填项 |
| 30001 | 业务错误 | 业务逻辑处理失败 | 查看具体错误信息 |
| 30002 | 服务未配置 | 未配置必需的平台服务 | 在应用服务中配置对应服务 |
| 40001 | 未授权 | 无权限访问该接口 | 检查 API Key 权限 |
| 42900 | 请求过于频繁 | 超过频率限制 | 降低调用频率 |
| 50000 | 系统内部错误 | 服务器内部错误 | 稍后重试或联系技术支持 |

### 5.2 HTTP 状态码

| HTTP 状态码 | 说明 |
|-------------|------|
| 200 | 请求成功 |
| 400 | 请求参数错误 |
| 401 | 认证失败 |
| 403 | 权限不足 |
| 404 | 接口不存在 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |
| 503 | 服务暂时不可用 |

---

## 6. 接口限流

### 6.1 频率限制

| 限制项 | 限制值 |
|--------|--------|
| 默认频率 | 100 次/分钟 |
| 峰值频率 | 10 次/秒 |

超过频率限制将返回错误码 `42900`。

### 6.2 文件大小限制

| 接口类型 | 限制值 |
|----------|--------|
| 图片上传 | ≤ 10 MB |
| 文件上传 | ≤ 1 GB |

---

## 7. 开发示例

### 7.1 Python 示例

```python
import requests

API_KEY = "sk_live_xxx"
BASE_URL = "https://api.workopilot.com/openapi/app"
headers = {"x-api-key": API_KEY}

def send_message(phone, text):
    """发送消息"""
    url = f"{BASE_URL}/lark/message/text"
    data = {
        "userId": phone,
        "idType": "PHONE",
        "text": text
    }
    response = requests.post(url, json=data, headers=headers)
    return response.json()

# 使用示例
result = send_message("+8613800138000", "您的订单已发货")
if result["code"] == 0:
    print(f"发送成功，消息ID: {result['data']}")
else:
    print(f"发送失败: {result['message']}")
```

### 7.2 Node.js 示例

```javascript
const axios = require('axios');

const API_KEY = 'sk_live_xxx';
const BASE_URL = 'https://api.workopilot.com/openapi/app';

async function sendLarkMessage(phone, text) {
  try {
    const response = await axios.post(`${BASE_URL}/lark/message/text`, {
      userId: phone,
      idType: 'PHONE',
      text: text
    }, {
      headers: {
        'x-api-key': API_KEY
      }
    });

    if (response.data.code === 0) {
      console.log('发送成功，消息ID:', response.data.data);
      return response.data.data;
    } else {
      console.error('发送失败:', response.data.message);
      throw new Error(response.data.message);
    }
  } catch (error) {
    console.error('请求异常:', error.message);
    throw error;
  }
}

// 使用示例
sendLarkMessage('+8613800138000', '您的订单已发货');
```

### 7.3 Java 示例

```java
import org.springframework.web.client.RestTemplate;
import org.springframework.http.*;

public class WorkopilotClient {
    private static final String API_KEY = "sk_live_xxx";
    private static final String BASE_URL = "https://api.workopilot.com/openapi/app";
    private final RestTemplate restTemplate = new RestTemplate();

    public String sendText(String phone, String text) {
        HttpHeaders headers = new HttpHeaders();
        headers.set("x-api-key", API_KEY);
        headers.setContentType(MediaType.APPLICATION_JSON);

        Map<String, Object> body = Map.of(
            "userId", phone,
            "idType", "PHONE",
            "text", text
        );

        HttpEntity<Map<String, Object>> request = new HttpEntity<>(body, headers);
        ResponseEntity<Map> response = restTemplate.postForEntity(
            BASE_URL + "/lark/message/text",
            request,
            Map.class
        );

        Map result = response.getBody();
        if (result != null && Integer.valueOf(0).equals(result.get("code"))) {
            return (String) result.get("data");
        }
        throw new RuntimeException(result != null ? (String) result.get("message") : "请求失败");
    }
}
```

---

## 8. 最佳实践

### 8.1 安全建议

1. **API Key 管理**
   - 不要将 API Key 硬编码在代码中
   - 使用环境变量或密钥管理服务存储
   - 定期轮换 API Key
   - 为不同环境使用不同的 API Key

2. **错误处理**
   - 实现指数退避重试机制
   - 记录完整的错误日志用于排查
   - 对用户错误进行友好提示

### 8.2 性能优化

1. **连接池管理**
   - 复用 HTTP 连接
   - 设置合理的超时时间（推荐 30 秒）
   - 使用异步请求提高并发性能

2. **缓存策略**
   - 缓存用户 ID 映射关系
   - 缓存应用配置信息
   - 设置合理的缓存过期时间

### 8.3 监控告警

建议监控以下指标：

- **可用性**: 接口成功率、响应时间
- **业务**: 消息发送量、失败率
- **错误**: 错误码分布、异常频率
- **性能**: P50/P95/P99 响应时间

---

## 9. 常见问题

### Q1: 如何获取用户的 open_id？

A: 系统支持通过手机号或邮箱自动查询用户 open_id。只需将 `idType` 设置为 `PHONE` 或 `EMAIL`，传入对应的手机号或邮箱即可。

### Q2: 接口调用超时怎么办？

A: 建议设置合理的超时时间（推荐 30 秒），并实现重试机制。如果持续超时，请联系技术支持。

### Q3: 如何提高接口调用成功率？

A:
1. 确保已配置相应的平台服务
2. 检查 API Key 是否有效
3. 控制调用频率避免触发限流
4. 实现适当的错误处理和重试机制

---

## 10. 参考文档

- [Platform Services](./platform-services.md) - 了解平台支持的各种服务
- [LARK Config Guide](./lark-config-guide.md) - 飞书服务配置说明
- [LARK API Docs](./lark-api-docs.md) - 飞书服务 API 详情

---

**© 2025 Workopilot. All rights reserved.**
