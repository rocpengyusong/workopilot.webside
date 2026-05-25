# Digital Employee Open API Specification

## 1. Overview

This document describes the external interfaces of Digital Employees, allowing third-party systems to call Digital Employee capabilities via `API-KEY`.

Key features of this API set:

- Authentication using `API-KEY` in the request header
- Digital Employees identified by `robotId`
- User sessions managed by `userId`
- Business context injection via `ctx.*` query parameters
- Chat interface supporting both:
  - `stream=true`: SSE streaming response
  - `stream=false`: Standard JSON response

Base Path:

```text
/api/ai/open
```

Common Request Headers:

```http
API-KEY: your-api-key
Content-Type: application/json
```

Notes:

- `robotId` must belong to the tenant associated with the `API-KEY`.
- `userId` is the user identifier in the third-party system, which will be mapped internally to a session-bound `VisitorId`.
- `ctx.*` is passed only via query string, e.g., `?ctx.source=crm&ctx.bizOrderId=SO20260416`.

Context Priority:

1. `ctx.*` from query string is written first.
2. `ContextData` from the request body overrides keys with the same name.
3. System reserved fields override at last:
   - `external_user_id / externalUserId / user_id / userId`
   - `external_user_name / externalUserName / user_name / userName`

---

## 2. Get Digital Employee Profile

### Endpoint

```http
GET /api/ai/open/robot/profile
```

### Description

Retrieve the basic profile of a Digital Employee under the current tenant by `robotId`.

### Request Parameters

| Parameter | Location | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `robotId` | query | long | Yes | Primary ID of the Digital Employee |

### Request Example

```http
GET /api/ai/open/robot/profile?robotId=2038862861584961500
API-KEY: your-api-key
```

### Response Example

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "id": 2038862861584961500,
    "robotCode": "cxy001",
    "robotName": "Promoter",
    "avatarUrl": "https://example.com/avatar.png",
    "welcomeMessage": "Hello, I can help you organize quotes and product information.",
    "businessLine": "pharma",
    "isActive": 1
  },
  "total": 0,
  "rows": null
}
```

---

## 3. Create New Session

### Endpoint

```http
POST /api/ai/open/chat/session
```

### Description

Create a new session or resume an existing one for a user under a specific Digital Employee.

### Request Body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `RobotId` | long | Yes | Digital Employee ID |
| `UserId` | string | Yes | Third-party system User ID |
| `UserName` | string | No | Third-party system User Name |
| `SessionId` | string | No | If provided, system will attempt to resume this session |
| `ContextData` | object | No | Business context data |

### Request Example

```json
{
  "robotId": 2038862861584961500,
  "userId": "u_10001",
  "userName": "Ankang Wang",
  "sessionId": "",
  "contextData": {
    "channel": "crm",
    "region": "hk"
  }
}
```

### Response Example

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "sessionId": "af0d84f7f0f44e90a8f463f8d54fe218",
    "robotName": "Promoter",
    "avatarUrl": "https://example.com/avatar.png",
    "welcomeMessage": "Hello, I can help you organize quotes and product information.",
    "enableAsr": true,
    "enableTts": true
  },
  "total": 0,
  "rows": null
}
```

---

## 4. Get Session List

### Endpoint

```http
GET /api/ai/open/chat/sessions
```

### Description

Retrieve the list of sessions for a specific user under a designated Digital Employee.

### Request Parameters

| Parameter | Location | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `robotId` | query | long | Yes | Digital Employee ID |
| `userId` | query | string | Yes | Third-party system User ID |
| `pageNum` | query | int | No | Page number, default is 1 |
| `pageSize` | query | int | No | Items per page, default is based on system config |
| `sessionTitle` | query | string | No | Fuzzy search by session title |

### Request Example

```http
GET /api/ai/open/chat/sessions?robotId=2038862861584961500&userId=u_10001&pageNum=1&pageSize=20
API-KEY: your-api-key
```

### Response Example

```json
{
  "code": 200,
  "msg": null,
  "data": null,
  "total": 2,
  "rows": [
    {
      "id": 2039020000000000001,
      "sessionId": "af0d84f7f0f44e90a8f463f8d54fe218",
      "robotId": 2038862861584961500,
      "robotCode": "cxy001",
      "userId": null,
      "visitorId": "openapi:12:u_10001",
      "userName": null,
      "sessionTitle": "Help me pick a batch of medicines for a dental clinic",
      "isActive": 1,
      "msgCount": 6,
      "lastMsgAt": "2026-04-16 18:22:15",
      "embedSource": null,
      "createTime": "2026-04-16 18:18:01",
      "updateTime": "2026-04-16 18:22:15"
    }
  ]
}
```

---

## 5. Get Chat History of a Session

### Endpoint

```http
GET /api/ai/open/chat/history
```

### Description

Retrieve historical messages of a specific session. Users are only allowed to read their own sessions based on `userId`.

### Request Parameters

| Parameter | Location | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `robotId` | query | long | Yes | Digital Employee ID |
| `userId` | query | string | Yes | Third-party system User ID |
| `sessionId` | query | string | Yes | Session ID |
| `pageNum` | query | int | No | Page number |
| `pageSize` | query | int | No | Items per page |

### Request Example

```http
GET /api/ai/open/chat/history?robotId=2038862861584961500&userId=u_10001&sessionId=af0d84f7f0f44e90a8f463f8d54fe218&pageNum=1&pageSize=50
API-KEY: your-api-key
```

### Response Example

```json
{
  "code": 200,
  "msg": null,
  "data": null,
  "total": 2,
  "rows": [
    {
      "id": 2039020000000000101,
      "sessionId": "af0d84f7f0f44e90a8f463f8d54fe218",
      "robotId": 2038862861584961500,
      "role": "user",
      "content": "Help me recommend three common medicines for dental clinics",
      "audioUrl": null,
      "attachments": null,
      "cardData": null,
      "skillCode": null,
      "toolCalls": null,
      "toolResult": null,
      "inputTokens": null,
      "outputTokens": null,
      "msgStatus": "SUCCESS",
      "errorMsg": null,
      "seq": 1,
      "createTime": "2026-04-16 18:18:01"
    },
    {
      "id": 2039020000000000102,
      "sessionId": "af0d84f7f0f44e90a8f463f8d54fe218",
      "robotId": 2038862861584961500,
      "role": "assistant",
      "content": "You can prioritize the following three products...",
      "audioUrl": null,
      "attachments": null,
      "cardData": "{\"title\":\"Product filtering results generated\"}",
      "skillCode": "pharma_sku_filter",
      "toolCalls": "agent_tool_activated",
      "toolResult": null,
      "inputTokens": null,
      "outputTokens": null,
      "msgStatus": "SUCCESS",
      "errorMsg": null,
      "seq": 2,
      "createTime": "2026-04-16 18:18:04"
    }
  ]
}
```

---

## 6. Delete a Session

### Endpoint

```http
DELETE /api/ai/open/chat/session/{sessionId}
```

### Description

Delete a specific session. Users are only allowed to delete their own sessions based on `userId`.

### Request Parameters

| Parameter | Location | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `sessionId` | path | string | Yes | Session ID |
| `robotId` | query | long | Yes | Digital Employee ID |
| `userId` | query | string | Yes | Third-party system User ID |

### Request Example

```http
DELETE /api/ai/open/chat/session/af0d84f7f0f44e90a8f463f8d54fe218?robotId=2038862861584961500&userId=u_10001
API-KEY: your-api-key
```

### Response Example

```json
{
  "code": 200,
  "msg": "Session deleted",
  "data": true,
  "total": 0,
  "rows": null
}
```

---

## 7. Send Chat Message

### Endpoint

```http
POST /api/ai/open/chat/send
```

### Description

Send a user message, supporting both streaming and non-streaming modes.

### Request Body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `RobotId` | long | Yes | Digital Employee ID |
| `UserId` | string | Yes | Third-party system User ID |
| `UserName` | string | No | Third-party system User Name |
| `SessionId` | string | No | Session ID. If empty, a new session is created automatically. |
| `Content` | string | No | User input content, use either `Content` or `Message`. |
| `Message` | string | No | User input content, use either `Content` or `Message`. |
| `Files` | array[string] | No | List of file URLs, prioritized over `Attachments`. |
| `Attachments` | array[string] | No | List of attachment URLs. |
| `ContextData` | object | No | Business context data. |
| `Stream` | bool | No | Whether to stream, default is `true`. |

### Supported `ctx.*` Parameters

The send interface supports passing business context via query parameters:

```http
POST /api/ai/open/chat/send?ctx.source=crm&ctx.bizOrderId=SO20260416
```

### Non-streaming Request Example

```json
{
  "robotId": 2038862861584961500,
  "userId": "u_10001",
  "userName": "Ankang Wang",
  "sessionId": "",
  "content": "Recommend three common dental clinic medicines and explain their usage scenarios",
  "files": [],
  "contextData": {
    "channel": "crm"
  },
  "stream": false
}
```

### Non-streaming Response Example

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "sessionId": "af0d84f7f0f44e90a8f463f8d54fe218",
    "requestId": null,
    "message": "You can prioritize the following three products...",
    "cardData": "{\"title\":\"Product filtering results generated\"}",
    "attachments": []
  },
  "total": 0,
  "rows": null
}
```

### Streaming Request Notes

When `Stream=true`, the interface returns:

```http
Content-Type: text/event-stream
```

The SSE event format is consistent with the existing embedded chat interface. Major events include:

- `text`: Incremental text
- `tool_call`: Tool invocation
- `tool_result`: Tool output
- `frontend_command`: Frontend action
- `card`: Card event
- `done`: Conversation completed
- `error`: Execution exception

### Streaming Response Example

```text
event: text
cfdata: {"text":"You can prioritize the following three products"}
data: {"text":"You can prioritize the following three products"}

event: card
cfdata: {"title":"Product filtering results generated","skillCode":"pharma_sku_filter"}
data: {"title":"Product filtering results generated","skillCode":"pharma_sku_filter"}

event: done
cfdata: {"messageId":"2039020000000000102","sessionId":"af0d84f7f0f44e90a8f463f8d54fe218"}
data: {"messageId":"2039020000000000102","sessionId":"af0d84f7f0f44e90a8f463f8d54fe218"}
```

---

## 8. Error Response Examples

### API Key Missing

```json
{
  "code": 401,
  "msg": "API Key missing"
}
```

### API Key Unauthorized

```json
{
  "code": 403,
  "msg": "Invalid API Key or Permission Denied"
}
```

### Digital Employee Not Belonging to Current Tenant

```json
{
  "code": 500,
  "msg": "Digital Employee does not exist or does not belong to the current tenant"
}
```

### Session Not Belonging to Current User

```json
{
  "code": 500,
  "msg": "No session found for the current user"
}
```

### Streaming Error Event

```text
event: error
cfdata: {"message":"Message content cannot be empty"}
data: {"message":"Message content cannot be empty"}
```

---

## 9. Additional Field Notes

### robotId

- Primary ID of the Digital Employee.
- Must belong to the tenant associated with the current `API-KEY`.

### userId

- Unique identifier for the user in the third-party business system.
- Internally converted to `VisitorId`.
- Current mapping rule:

```text
openapi:{apiKeyId}:{userId}
```

### sessionId

- Primary key for the chat session.
- Optional when creating a session.
- Automatically generated when sending a message if not provided.
- Must be provided and belong to the current user when querying history or deleting sessions.

### ctx.*

- Passed via query string.
- Suitable for business identifiers, source channels, bill numbers, and other additional contexts.
- Example:

```http
?ctx.source=crm&ctx.bizOrderId=SO20260416&ctx.company=HuadaPharma
```

---

## 10. Usage Recommendations

- Third-party systems are recommended to always explicitly pass `robotId + userId`.
- Business context should prioritize `ContextData` in the request body; use `ctx.*` only when URL passing is required.
- Use `stream=true` for real-time typewriter effects.
- Use `stream=false` if the backend system has weak support for SSE.
- File upload interfaces are not covered in this open API document for now. Open version upload capabilities may be added later if needed.
