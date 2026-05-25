# 数字员工 Iframe 接入说明

## 1. 接入准备

在开始接入前，请确保已在管理后台开启嵌入功能：

`管理平台 > 数字员工 > 数字员工管理 > 开启嵌入`

嵌入页地址一般形如：

```text
https://your-domain/embed/chat/{robotId}?token=xxx&externalUserId=USER001
```

推荐宿主在 iframe 触发 `ready` 后立即发送一次 `configure`。

## 2. 消息 Envelope

宿主页面和 iframe 之间统一通过 `window.postMessage` 通信。

```json
{
  "version": "1.0.0",
  "source": "wiseai-host",
  "type": "command",
  "action": "configure",
  "requestId": "cfg-001",
  "timestamp": "2026-05-14T10:00:00.000Z",
  "payload": {}
}
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `version` | 协议版本，当前为 `1.0.0` |
| `source` | 消息来源，宿主为 `wiseai-host`，iframe 为 `wiseai-embed` |
| `type` | 消息类型：`command`、`event`、`host-action`、`action-result` |
| `action` | 命令或事件名称 |
| `requestId` | 请求唯一标识，建议每次调用都传 |
| `timestamp` | ISO 时间字符串 |
| `payload` | 业务数据 |

## 3. 宿主发送给 iframe

### 3.1 `configure`

用于注入宿主能力、上下文、前端动作、语音命令和 UI 配置。

推荐结构：

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "configure",
  "requestId": "cfg-001",
  "payload": {
    "contextData": {
      "hostApp": "demo-html",
      "currentMeetingId": "MEETING-001",
      "currentUserName": "外部宿主用户"
    },
    "capabilities": {
      "contextSync": true,
      "filePicker": false,
      "hostActions": true,
      "microphone": false,
      "speaker": false
    },
    "forceNewSession": false,
    "theme": "deepBlue",//支持 default，deepBlue
    "themeColors": {
      "--voice-overlay-bg-start": "#485c6e"
    },
    "uiConfig": {
      "showHangupButton": true
    },
    "frontendActions": [],
    "voiceCommands": []
  }
}
```

`configure.payload` 配置项：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `contextData` | `object` | 宿主上下文，会合并到数字员工运行上下文 |
| `capabilities.contextSync` | `boolean` | 是否支持上下文同步 |
| `capabilities.filePicker` | `boolean` | 是否允许 iframe 请求宿主选择文件 |
| `capabilities.hostActions` | `boolean` | 是否允许数字员工调用宿主动作 |
| `capabilities.microphone` | `boolean` | 宿主是否提供麦克风权限协作 |
| `capabilities.speaker` | `boolean` | 宿主是否提供扬声器/播报协作 |
| `forceNewSession` | `boolean` | 是否强制创建新会话。`ready` 后立即配置也会生效 |
| `theme` | `string` | 主题名。当前支持 `default`、`deepBlue` |
| `themeName` | `string` | `theme` 的别名，兼容使用 |
| `themeColors` | `object` | 主题 CSS 变量覆盖，放在 `configure.payload` 顶层 |
| `uiConfig.showHangupButton` | `boolean` | 语音面板是否显示挂断按钮 |
| `frontendActions` | `array` | 宿主动作白名单 |
| `voiceCommands` | `array` | 语音短语到动作的映射 |

说明：

- 主题配置请放在 `configure.payload.theme` 和 `configure.payload.themeColors`。
- 旧版 `uiConfig.theme` / `uiConfig.themeColors` 仍兼容，但不推荐继续使用。
- `configure` 不会触发新的 `ready`，它只会刷新配置并触发 `state-change`。

### 3.2 主题配置

切换深蓝主题：

```json
{
  "type": "command",
  "action": "configure",
  "requestId": "cfg-theme-001",
  "source": "wiseai-host",
  "payload": {
    "theme": "deepBlue"
  }
}
```

恢复默认主题：

```json
{
  "type": "command",
  "action": "configure",
  "requestId": "cfg-theme-002",
  "source": "wiseai-host",
  "payload": {
    "theme": "default"
  }
}
```

覆盖语音背景变量：

```json
{
  "type": "command",
  "action": "configure",
  "requestId": "cfg-theme-003",
  "source": "wiseai-host",
  "payload": {
    "theme": "deepBlue",
    "themeColors": {
      "--voice-overlay-bg-start": "#485c6e",
      "--voice-overlay-bg-end": "#314557"
    }
  }
}
```

### 3.3 `updateContext`

仅更新上下文，不覆盖动作、主题等配置。

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "updateContext",
  "requestId": "ctx-001",
  "payload": {
    "contextData": {
      "customerId": "C1001",
      "orderNo": "SO20260514"
    }
  }
}
```

### 3.4 `sendToBot`

以用户身份向数字员工发送一条消息。

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "sendToBot",
  "requestId": "txt-001",
  "payload": {
    "text": "帮我总结这个客户的最近沟通记录"
  }
}
```

### 3.5 `sendToUser`

以 AI 身份向聊天区注入一条消息，可选择同时语音播报。

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "sendToUser",
  "requestId": "stu-001",
  "payload": {
    "requestId": "stu-001",
    "content": "在呢，有什么可以帮助？",
    "playAudio": true
  }
}
```

`sendToUser.payload` 入参：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `content` | `string` | 是 | 要展示或播报的 AI 消息 |
| `playAudio` | `boolean` | 否 | 是否触发语音播报，默认 `false` |
| `requestId` | `string` | 否 | 可放在 payload 内兜底；优先使用 envelope 的 `requestId` |

行为说明：

- 每次 `sendToUser` 都会在聊天区新增一条 AI 消息，不会合并到上一条。
- 如果 `playAudio=true`，语音面板必须已打开并稳定可用。
- 如果上一条 `sendToUser(playAudio=true)` 仍在播放，新请求会打断上一条。
- 播放完成、失败或被打断都会通过 `action-result` 回传同一个 `requestId`。

成功回调：

```json
{
  "source": "wiseai-embed",
  "type": "action-result",
  "action": "action-result",
  "requestId": "stu-001",
  "payload": {
    "success": true,
    "state": "granted",
    "message": "语音播报完成。",
    "errorCode": null,
    "payload": null
  }
}
```

常见失败码：

| errorCode | 说明 |
| --- | --- |
| `EMBED_INVALID_PAYLOAD` | `content` 为空 |
| `EMBED_SESSION_NOT_READY` | 会话尚未初始化完成 |
| `EMBED_VOICE_NOT_READY` | 语音面板未打开或未准备好 |
| `EMBED_VOICE_UNAVAILABLE` | 当前数字员工未启用语音能力 |
| `EMBED_TTS_FAILED` | TTS 合成或播放失败 |
| `EMBED_TTS_INTERRUPTED` | 被新的 `sendToUser(playAudio=true)` 打断 |

### 3.6 `openVoice` / `closeVoice`

打开或关闭语音面板。

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "openVoice",
  "requestId": "voice-open-001",
  "payload": {}
}
```

建议流程：

1. 发送 `openVoice`
2. 等待 `state-change.payload.voiceOverlayVisible === true`
3. 再发送 `sendToUser({ playAudio: true })`

## 4. iframe 发送给宿主的事件

### 4.1 `ready`

iframe 初始化完成，可接收 `configure`。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "ready",
  "requestId": "evt-001",
  "payload": {
    "robotId": "2036327843637628928",
    "robotCode": "demo_robot",
    "sessionId": "S001",
    "initialMode": "text",
    "contextData": {},
    "capabilities": {
      "contextSync": true,
      "filePicker": false,
      "hostActions": false,
      "microphone": false,
      "speaker": false
    }
  }
}
```

注意：

- `ready` 只在 iframe 初始化完成时发送。
- `configure` 不会再次触发 `ready`。
- 如果 iframe 使用 `v-show` 隐藏，iframe 仍可能已经加载并发送过 `ready`。

### 4.2 `state-change`

会话、配置、语音状态变化。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "state-change",
  "requestId": "cfg-001",
  "payload": {
    "sessionId": "S001",
    "voiceOverlayVisible": true,
    "voiceState": "listening",
    "configuredActionCount": 2,
    "capabilities": {
      "contextSync": true,
      "filePicker": false,
      "hostActions": true,
      "microphone": false,
      "speaker": false
    }
  }
}
```

`voiceState` 常见值：

- `idle`
- `listening`
- `recognizing`
- `thinking`
- `tool_calling`
- `speaking`

### 4.3 `voice-exit`

语音面板真正关闭时发送。未开启、初始化清理、刚打开瞬间被内部流程关闭，不会发送该事件。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "voice-exit",
  "payload": {
    "reason": "host",
    "sessionId": "S001"
  }
}
```

### 4.4 `session-created`

创建新会话时发送，例如 `forceNewSession=true` 或用户主动新建会话。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "session-created",
  "payload": {
    "robotId": "2036327843637628928",
    "sessionId": "S002"
  }
}
```

### 4.5 `assistant-message-finished`

AI 文本回复流式输出完成。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "assistant-message-finished",
  "payload": {
    "content": "最终回复文本"
  }
}
```

### 4.6 `assistant-audio-finished`

AI 语音队列播放完毕并进入空闲。

```json
{
  "source": "wiseai-embed",
  "type": "event",
  "action": "assistant-audio-finished",
  "payload": {}
}
```

### 4.7 `business-open`

业务抽屉或业务详情被打开，可用于宿主埋点。

## 5. iframe 请求宿主动作

当配置了 `frontendActions`，数字员工可以向宿主发送 `host-action`。

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "joinMeeting",
  "requestId": "host-join-001",
  "payload": {
    "meetingId": "MEETING-001",
    "meetingTitle": "产品方案评审会",
    "sessionId": "S001"
  }
}
```

宿主收到后应：

1. 校验 `origin`
2. 校验 `source/type/action`
3. 根据 `action` 查宿主白名单
4. 执行业务逻辑
5. 如需反馈，发送 `action-result`

## 6. action-result

`action-result` 用于回传异步动作结果。既可由宿主回给 iframe，也可由 iframe 回给宿主，例如 `sendToUser` 播放完成。

```json
{
  "source": "wiseai-host",
  "type": "action-result",
  "action": "action-result",
  "requestId": "host-join-001",
  "payload": {
    "success": true,
    "state": "granted",
    "message": "已加入会议",
    "errorCode": null,
    "payload": {
      "meetingId": "MEETING-001"
    }
  }
}
```

`state` 可选值：

- `granted`
- `denied`
- `blocked`
- `unsupported`

## 7. SDK API

### 7.1 `create(options)`

创建 SDK 实例。

```js
const sdk = WiseAIEmbedSDK.create({
  src: 'https://your-domain/embed/chat/robot-id?token=xxx&externalUserId=USER001',
  container: '#app',
  allowedOrigin: 'https://your-domain',
  capabilities: {
    contextSync: true,
    hostActions: true
  },
  contextData: {
    hostApp: 'demo'
  },
  onEvent(message) {
    console.log(message);
  },
  onHostAction(message, sdkInstance) {
    sdkInstance.replyActionResult(message.requestId, {
      success: true,
      state: 'granted',
      message: 'handled'
    });
  }
});
```

### 7.2 常用方法

| 方法 | 说明 |
| --- | --- |
| `mount()` | 挂载 iframe |
| `destroy()` | 销毁 iframe 与监听 |
| `configure(payload)` | 发送 `configure` |
| `updateContext(payload)` | 更新上下文 |
| `sendToBot(text)` | 发送用户消息给数字员工 |
| `sendToUser(payload, requestId?)` | 注入 AI 消息，可播报 |
| `openVoice()` | 打开语音面板 |
| `closeVoice()` | 关闭语音面板 |
| `on(eventName, handler)` | 监听事件，支持 `*` |
| `off(eventName, handler?)` | 移除监听 |
| `replyActionResult(requestId, result)` | 回传宿主动作结果 |

### 7.3 SDK configure 示例

```js
sdk.configure({
  contextData: {
    hostApp: 'demo-html',
    currentMeetingId: 'MEETING-001'
  },
  capabilities: {
    contextSync: true,
    filePicker: false,
    hostActions: true,
    microphone: false,
    speaker: false
  },
  forceNewSession: false,
  theme: 'deepBlue',
  themeColors: {
    '--voice-overlay-bg-start': '#485c6e'
  },
  uiConfig: {
    showHangupButton: true
  },
  frontendActions: [
    {
      code: 'joinMeeting',
      name: '加入会议',
      description: '当用户要求加入会议时通知宿主执行',
      target: 'parent',
      awaitResult: true,
      inputSchema: {
        type: 'object',
        properties: {
          meetingId: { type: 'string' },
          meetingTitle: { type: 'string' }
        },
        required: ['meetingId']
      }
    }
  ],
  voiceCommands: [
    {
      phrase: '加入会议',
      actionCode: 'joinMeeting'
    }
  ]
});
```

### 7.4 SDK sendToUser 示例

```js
const requestId = `stu-${Date.now()}`;

sdk.sendToUser(
  {
    content: '在呢，有什么可以帮助？',
    playAudio: true,
    requestId
  },
  requestId
);

sdk.on('action-result', (message) => {
  if (message.requestId === requestId) {
    console.log('sendToUser result:', message.payload);
  }
});
```

