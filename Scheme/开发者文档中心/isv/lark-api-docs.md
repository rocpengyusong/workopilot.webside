# LARK 飞书服务接口文档

## 1. 接口概述

Workopilot LARK 飞书服务提供完整的飞书消息收发和资源管理能力，所有接口均通过 RESTful API 方式提供。

### 接口基础信息

| 项目 | 说明 |
|------|------|
| 接口地址 | `https://api.workopilot.com/openapi/app/lark` |
| 认证方式 | API Key (通过 `x-api-key` 请求头) |
| 数据格式 | JSON |
| 字符编码 | UTF-8 |

### 支持的消息类型

| 消息类型 | 说明 |
|----------|------|
| text | 文本消息 |
| image | 图片消息 |
| post | 富文本消息 |
| interactive | 交互式卡片消息 |

---

## 2. 消息发送接口

### 2.1 发送文本消息

**接口地址**：`POST /openapi/app/lark/message/text`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | String | 是 | 用户ID，支持多种ID类型 |
| idType | String | 是 | ID类型：OPEN_ID/USER_ID/CHAT_ID/UNION_ID/PHONE/EMAIL |
| text | String | 是 | 文本内容，最大长度2048 |

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/message/text \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "+8613800138000",
    "idType": "PHONE",
    "text": "您的订单已发货，预计3天内送达"
  }'
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": "om_xxxxxxxxxxxxxxxxx",
  "timestamp": 1736058900000
}
```

**响应参数说明**：

| 参数名 | 类型 | 说明 |
|--------|------|------|
| code | Integer | 响应码，0表示成功 |
| message | String | 响应消息 |
| data | String | 消息ID |
| timestamp | Long | 响应时间戳 |

---

### 2.2 发送图片消息

**接口地址**：`POST /openapi/app/lark/message/image`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| idType | String | 是 | ID类型：OPEN_ID/USER_ID/CHAT_ID/UNION_ID/PHONE/EMAIL |
| imageKey | String | 是 | 图片key，通过上传图片接口获取 |

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/message/image \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "ou_xxx",
    "idType": "OPEN_ID",
    "imageKey": "img_v2_xxx-xxx"
  }'
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": "om_xxxxxxxxxxxxxxxxx",
  "timestamp": 1736058900000
}
```

---

### 2.3 发送富文本消息

**接口地址**：`POST /openapi/app/lark/message/post`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| idType | String | 是 | ID类型 |
| content | Object | 是 | 富文本内容，参考飞书富文本格式 |

**富文本内容示例**：

```json
{
  "post": {
    "zh_cn": {
      "title": "订单通知",
      "content": [
        {
          "tag": "text",
          "text": "您的订单"
        },
        {
          "tag": "a",
          "text": "ORD20250204001",
          "href": "https://example.com/order/123"
        },
        {
          "tag": "text",
          "text": "已发货"
        }
      ]
    }
  }
}
```

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/message/post \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "ou_xxx",
    "idType": "OPEN_ID",
    "content": {
      "post": {
        "zh_cn": {
          "title": "订单通知",
          "content": [
            {"tag": "text", "text": "您的订单已发货"}
          ]
        }
      }
    }
  }'
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": "om_xxxxxxxxxxxxxxxxx",
  "timestamp": 1736058900000
}
```

---

### 2.4 发送卡片消息

**接口地址**：`POST /openapi/app/lark/message/card`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| idType | String | 是 | ID类型 |
| card | Object | 是 | 卡片内容，参考飞书卡片格式 |

**卡片内容示例**：

```json
{
  "config": {
    "wide_screen_mode": true
  },
  "header": {
    "title": {
      "content": "订单通知",
      "tag": "plain_text"
    }
  },
  "elements": [
    {
      "tag": "div",
      "text": {
        "content": "您的订单已发货",
        "tag": "lark_md"
      }
    },
    {
      "tag": "action",
      "actions": [
        {
          "tag": "button",
          "text": {
            "content": "查看详情",
            "tag": "plain_text"
          },
          "type": "default",
          "url": "https://example.com/order/123"
        }
      ]
    }
  ]
}
```

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/message/card \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "ou_xxx",
    "idType": "OPEN_ID",
    "card": {
      "config": {
        "wide_screen_mode": true
      },
      "header": {
        "title": {
          "content": "订单通知",
          "tag": "plain_text"
        }
      },
      "elements": [
        {
          "tag": "div",
          "text": {
            "content": "您的订单已发货",
            "tag": "lark_md"
          }
        }
      ]
    }
  }'
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": "om_xxxxxxxxxxxxxxxxx",
  "timestamp": 1736058900000
}
```

---

### 2.5 发送卡片消息（使用卡片模板）

**接口地址**：`POST /openapi/app/lark/message/cardById`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| idType | String | 是 | ID类型 |
| cardId | String | 是 | 卡片模板ID |

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/message/cardById \
  -H "x-api-key: sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "ou_xxx",
    "idType": "OPEN_ID",
    "cardId": "custom_xxx"
  }'
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": "om_xxxxxxxxxxxxxxxxx",
  "timestamp": 1736058900000
}
```

---

## 3. 资源上传接口

### 3.1 上传图片

**接口地址**：`POST /openapi/app/lark/resource/image/upload`

**请求类型**：`multipart/form-data`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | File | 是 | 图片文件，最大10MB |
| imageType | String | 否 | 图片类型：message(默认)/avatar |

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/resource/image/upload \
  -H "x-api-key: sk_live_xxx" \
  -F "file=@/path/to/image.jpg" \
  -F "imageType=message"
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "imageKey": "img_v2_xxx-xxx",
    "resourceType": "IMAGE"
  },
  "timestamp": 1736058900000
}
```

**支持的图片格式**：JPG、JPEG、PNG、GIF、WEBP

---

### 3.2 上传文件

**接口地址**：`POST /openapi/app/lark/resource/file/upload`

**请求类型**：`multipart/form-data`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | File | 是 | 文件，最大1GB |
| fileType | String | 否 | 文件类型：source/file/audio/video/media |
| duration | Integer | 否 | 文件时长（毫秒），视频/音频需要 |

**请求示例**：

```bash
curl -X POST https://api.workopilot.com/openapi/app/lark/resource/file/upload \
  -H "x-api-key: sk_live_xxx" \
  -F "file=@/path/to/document.pdf" \
  -F "fileType=file"
```

**响应示例**：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "fileKey": "file_v2_xxx-xxx",
    "resourceType": "FILE"
  },
  "timestamp": 1736058900000
}
```

**支持的文件类型**：PDF、DOC、DOCX、XLS、XLSX、PPT、PPTX、ZIP、RAR、MP3、MP4、MOV 等

---

## 4. 资源下载接口

### 4.1 下载图片

**接口地址**：`GET /openapi/app/lark/resource/image/download`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| imageKey | String | 是 | 图片key |

**请求示例**：

```bash
curl -X GET "https://api.workopilot.com/openapi/app/lark/resource/image/download?imageKey=img_v2_xxx-xxx" \
  -H "x-api-key: sk_live_xxx" \
  -o image.jpg
```

**响应**：直接返回图片二进制数据，Content-Type 根据图片类型自动设置

---

### 4.2 下载文件

**接口地址**：`GET /openapi/app/lark/resource/file/download`

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| fileKey | String | 是 | 文件key |

**请求示例**：

```bash
curl -X GET "https://api.workopilot.com/openapi/app/lark/resource/file/download?fileKey=file_v2_xxx-xxx" \
  -H "x-api-key: sk_live_xxx" \
  -o document.pdf
```

**响应**：直接返回文件二进制数据，Content-Type 根据文件类型自动设置

---

## 5. 用户标识类型说明

### 5.1 ID类型枚举

| idType值 | 说明 | 示例 |
|----------|------|------|
| OPEN_ID | 飞书用户open_id | ou_xxxxxxxxxxxxxxxx |
| USER_ID | 飞书用户user_id | xxxxxxxxxxxxxxxxxx |
| CHAT_ID | 飞书群聊ID | oc_xxxxxxxxxxxxxxxx |
| UNION_ID | 飞书用户union_id | on_xxxxxxxxxxxxxxxx |
| PHONE | 手机号（自动转换） | +8613800138000 |
| EMAIL | 邮箱（自动转换） | user@example.com |

### 5.2 使用建议

- **PHONE**：推荐使用，系统会自动查询用户open_id
- **EMAIL**：推荐使用，系统会自动查询用户open_id
- **OPEN_ID**：直接使用，性能最优
- **CHAT_ID**：用于向群聊发送消息

---

## 6. 错误码说明

### 6.1 平台错误码

| 错误码 | 错误信息 | 说明 | 处理建议 |
|--------|----------|------|----------|
| 10002 | API Key 无效 | API Key 不存在或已过期 | 检查 API Key 是否正确 |
| 30002 | 服务未配置 | 未配置飞书服务 | 在应用服务中配置飞书服务 |
| 99991008 | App ID 无效 | 飞书App ID错误 | 检查飞书服务配置 |
| 99991015 | App Secret 无效 | 飞书App Secret错误 | 检查飞书服务配置 |
| 99991401 | 无接口权限 | 飞书应用缺少必要权限 | 在飞书开放平台申请权限 |
| 99990320 | 用户不存在 | 指定用户不存在 | 检查 userId 是否正确 |
| 99991404 | 文件不存在 | imageKey或fileKey无效 | 检查资源key是否正确 |

### 6.2 业务错误码

| 错误码 | 错误信息 | 说明 |
|--------|----------|------|
| 20001 | 参数校验失败 | 请求参数格式错误 |
| 30001 | 业务错误 | 业务逻辑处理失败 |

---

## 7. 完整调用示例

### 7.1 发送订单通知（卡片消息）

```python
import requests

API_KEY = "sk_live_xxx"
BASE_URL = "https://api.workopilot.com/openapi/app/lark"
headers = {"x-api-key": API_KEY}

def sendOrderNotification(phone, orderId):
    """发送订单通知"""
    url = f"{BASE_URL}/message/card"
    data = {
        "userId": phone,
        "idType": "PHONE",
        "card": {
            "config": {"wide_screen_mode": True},
            "header": {
                "title": {"content": "订单发货通知", "tag": "plain_text"},
                "template": "blue"
            },
            "elements": [
                {
                    "tag": "div",
                    "text": {
                        "content": f"您的订单 **{orderId}** 已发货，预计3天内送达。",
                        "tag": "lark_md"
                    }
                },
                {
                    "tag": "action",
                    "actions": [
                        {
                            "tag": "button",
                            "text": {"content": "查看物流", "tag": "plain_text"},
                            "type": "primary",
                            "url": f"https://example.com/order/{orderId}"
                        }
                    ]
                }
            ]
        }
    }
    response = requests.post(url, json=data, headers=headers)
    return response.json()

# 使用示例
result = sendOrderNotification("+8613800138000", "ORD20250204001")
if result["code"] == 0:
    print(f"发送成功，消息ID: {result['data']}")
else:
    print(f"发送失败: {result['message']}")
```

### 7.2 上传并发送图片消息

```python
import requests

API_KEY = "sk_live_xxx"
BASE_URL = "https://api.workopilot.com/openapi/app/lark"
headers = {"x-api-key": API_KEY}

def uploadAndSendImage(phone, image_path):
    """上传图片并发送"""
    # 1. 上传图片
    upload_url = f"{BASE_URL}/resource/image/upload"
    with open(image_path, "rb") as f:
        files = {"file": f}
        upload_resp = requests.post(upload_url, files=files, headers=headers)

    if upload_resp.json()["code"] != 0:
        print(f"上传失败: {upload_resp.json()['message']}")
        return

    image_key = upload_resp.json()["data"]["imageKey"]

    # 2. 发送图片消息
    send_url = f"{BASE_URL}/message/image"
    data = {
        "userId": phone,
        "idType": "PHONE",
        "imageKey": image_key
    }
    send_resp = requests.post(send_url, json=data, headers=headers)

    return send_resp.json()

# 使用示例
result = uploadAndSendImage("+8613800138000", "/path/to/product.jpg")
print(result)
```

---

## 8. 参考文档

- [Development Guide](./development-guide.md) - 接入准备、认证方式、错误码等
- [Platform Services](./platform-services.md) - 平台服务功能介绍
- [LARK Config Guide](./lark-config-guide.md) - 飞书服务配置说明
- [飞书开放平台文档](https://open.feishu.cn/document/server-docs/api-reference)

---

**© 2025 Workopilot. All rights reserved.**
