# Host Event Protocol and SDK API Specification

## 1. Preparation
Before you start, please ensure that the embedding feature has been enabled in the management platform:
**Management Platform > Digital Employee > Digital Employee Management > Enable Embedding**

## 2. Protocol Positioning
This protocol is used to standardize the bidirectional communication between the host page and the Digital Employee embedded page.

Design Goals:

- A single protocol covering Browser, Android WebView, Linux WebView, and Electron
- SDK is just an official wrapper; the protocol itself remains open
- All events include a `requestId` to facilitate trace analysis
- Host tools use whitelist injection to prevent the model from accidentally calling unexposed capabilities

## 3. Message Envelope
All messages uniformly use the following structure:

```json
{
  "version": "1.0.0",
  "source": "wiseai-host",
  "type": "command",
  "action": "configure",
  "requestId": "cfg-001",
  "timestamp": "2026-04-17T10:00:00.000Z",
  "payload": {}
}
```

Field Descriptions:

- `version`
  Protocol version, currently `1.0.0`
- `source`
  Source of the message
- `type`
  Message category
- `action`
  Event or command action name
- `requestId`
  Unique request identifier
- `timestamp`
  ISO time string
- `payload`
  Business payload

Source Conventions:

- host -> child: `wiseai-host`
- child -> host: `wiseai-embed`

## 4. Message Categories
### 4.1 `command`
Instructions sent from the host to the embedded page.

Common Actions:

- `configure`
- `updateContext`
- `sendText`
- `openVoice`
- `closeVoice`

### 4.2 `event`
Status events sent from the embedded page to the host.

Common Actions:

- `ready`
- `state-change`
- `voice-exit`
- `business-open`

### 4.3 `host-action`
The embedded page notifies the host to perform an asynchronous action.

Examples:

- `openCustomerDetail`
- `pickFiles`
- `requestMicrophonePermission`
- `joinMeeting`
- `stopMeeting`

### 4.4 `action-result`
The host optionally returns the execution result of an action.

## 5. Host to Embedded Page
### 5.1 `configure`
Used to initialize host capabilities, context, and available actions.

Protocol Highlights:

- When `capabilities.hostActions = true`, it means the host allows the Digital Employee to notify the host to execute actions.
- `frontendActions` is a whitelist of actions injected by the host; the Digital Employee can only call actions declared here.
- `voiceCommands` are voice phrase rules injected by the host, used to map specific phrases to action codes.

General Example:

```json
{
  "version": "1.0.0",
  "source": "wiseai-host",
  "type": "command",
  "action": "configure",
  "requestId": "cfg-001",
  "payload": {
    "contextData": {
      "dept": "sales",
      "customerId": "C1001"
    },
    "capabilities": {
      "contextSync": true,
      "filePicker": true,
      "hostActions": true,
      "microphone": true,
      "speaker": true
    },
    "frontendActions": [
      {
        "code": "openCustomerDetail",
        "name": "Open Customer Details",
        "description": "Notify host to open the customer details page when the Digital Employee needs to view customer information",
        "target": "parent",
        "fireAndForget": true,
        "inputSchema": {
          "type": "object",
          "properties": {
            "customerId": { "type": "string" }
          },
          "required": ["customerId"]
        }
      }
    ],
    "voiceCommands": [
      {
        "phrase": "Step down",
        "actionCode": "closeVoice"
      }
    ]
  }
}
```

#### 5.1.1 Injecting Meeting Commands by Host
The following is the recommended way to inject meeting commands. Here, two actions are injected:

- `joinMeeting`
  Join a meeting, requires 2 parameters:
  - `platform`: Meeting platform, e.g., `Tencent`, `DingTalk`, `Feishu`, `Teams`
  - `meetingNo`: Meeting number, numeric
- `stopMeeting`
  Stop the meeting, no parameters

Recommended configuration:

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "configure",
  "requestId": "cfg-meeting-001",
  "payload": {
    "capabilities": {
      "contextSync": true,
      "hostActions": true,
      "filePicker": false,
      "microphone": true,
      "speaker": true
    },
    "contextData": {
      "scene": "meeting-assistant"
    },
    "frontendActions": [
      {
        "code": "joinMeeting",
        "name": "Join Meeting",
        "description": "Notify the host to join a meeting using the specified platform and meeting number when requested by the user.",
        "target": "parent",
        "awaitResult": true,
        "inputSchema": {
          "type": "object",
          "properties": {
            "platform": {
              "type": "string",
              "description": "Meeting platform, e.g., Tencent, DingTalk, Feishu, Teams"
            },
            "meetingNo": {
              "type": "integer",
              "description": "Meeting number, numeric only"
            }
          },
          "required": ["platform", "meetingNo"]
        }
      },
      {
        "code": "stopMeeting",
        "name": "Stop Meeting",
        "description": "Notify the host to stop the meeting when requested by the user.",
        "target": "parent",
        "awaitResult": true,
        "inputSchema": {
          "type": "object",
          "properties": {}
        }
      }
    ],
    "voiceCommands": [
      {
        "phrase": "Join meeting",
        "actionCode": "joinMeeting"
      },
      {
        "phrase": "Stop meeting",
        "actionCode": "stopMeeting"
      }
    ]
  }
}
```

Notes:

- `target` is recommended to be fixed as `parent`, meaning the Digital Employee does not execute locally inside the iframe but notifies the host to execute.
- `awaitResult` is recommended to be `true` so that the host can return results via `action-result` after execution, allowing the Digital Employee to perceive success, failure, and failure reasons more easily.
- `meetingNo` is recommended to use the numeric type; if the host system requires a string internally, it can be converted from numeric to string on the host side.

#### 5.1.2 How Digital Employee Calls After Injecting Join Meeting Command
Once the host completes the `configure` injection above, the Digital Employee side will recognize `joinMeeting` as a callable action.

For example, when a user says:

- "Help me join Tencent Meeting 123456789"
- "Join Feishu Meeting 987654321"

After determining that a host action needs to be called, the Digital Employee will send a `host-action` to the host:

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "joinMeeting",
  "requestId": "host-join-001",
  "payload": {
    "platform": "Tencent",
    "meetingNo": 123456789,
    "sessionId": "966209b7911f4a4981d39c3d060f2b2a"
  }
}
```

Field Descriptions:

- `action = joinMeeting`
  Indicates that this request is for the host to execute "Join Meeting".
- `payload.platform`
  Meeting platform.
- `payload.meetingNo`
  Meeting number, numeric.
- `payload.sessionId`
  Identifier for the current dialogue session, useful for trace logs or idempotency control on the host side.
- `requestId`
  Unique identifier for this action request; the host needs to return it as-is in the `action-result`.

#### 4.1.3 How Digital Employee Calls After Injecting Stop Meeting Command
For example, when a user says:

- "Stop the meeting"
- "End the current meeting"
- "Exit the meeting"

The message sent from the Digital Employee to the host is as follows:

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "stopMeeting",
  "requestId": "host-stop-001",
  "payload": {
    "sessionId": "966209b7911f4a4981d39c3d060f2b2a"
  }
}
```

Notes:

- `stopMeeting` has no business parameters.
- In the current implementation, the Digital Employee side may still automatically supplement the `sessionId`, which the host can use for log correlation.
- If the host does not need any additional information, it simply executes the stop meeting logic based on `action = stopMeeting`.

#### 5.1.4 Recommended Timing for Host Command Injection
- It is recommended to send `configure` immediately after the host receives the `ready` event from the embedded page.
- If the host refreshes available actions, context, or voice commands, it can send `configure` again.
- When only updating the context, prioritize using `updateContext` to avoid redundantly overriding the entire action configuration.

### 5.2 `updateContext`
Dynamically update the context without refreshing the iframe.

Example:

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "updateContext",
  "requestId": "ctx-001",
  "payload": {
    "contextData": {
      "customerId": "C20260401",
      "orderNo": "SO20260417"
    }
  }
}
```

### 5.3 `sendText`
Proactively send a message to the current session.

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "sendText",
  "requestId": "txt-001",
  "payload": {
    "text": "Help me summarize the highlights of the last three conversations with this customer"
  }
}
```

The host can also proactively initiate a test phrase to drive the Digital Employee to trigger meeting actions, for example:

```json
{
  "source": "wiseai-host",
  "type": "command",
  "action": "sendText",
  "requestId": "txt-meeting-001",
  "payload": {
    "text": "Please join Tencent Meeting 123456789"
  }
}
```

### 5.4 `openVoice` / `closeVoice`
Control the opening and closing of the voice overlay.

## 6. Embedded Page to Host
### 6.1 `ready`
Indicates that the embedded page has completed initialization and is ready to receive configurations.

Example payload:

```json
{
  "robotId": "2036327843637628928",
  "robotCode": "pharma_cxy_01",
  "sessionId": "966209b7911f4a4981d39c3d060f2b2a",
  "initialMode": "voice",
  "contextData": {
    "dept": "sales"
  },
  "capabilities": {
    "contextSync": true,
    "filePicker": false,
    "hostActions": false,
    "microphone": false,
    "speaker": false
  }
}
```

Recommended handling:

- Host receives `ready`.
- Record the current `sessionId`.
- Immediately send `configure`.
- Send `updateContext` if additional context needs to be supplemented.

### 6.2 `state-change`
Indicates a change in session state, voice state, or host capability state.

Example payload:

```json
{
  "sessionId": "966209b7911f4a4981d39c3d060f2b2a",
  "voiceOverlayVisible": true,
  "voiceState": "listening",
  "configuredActionCount": 2,
  "capabilities": {
    "contextSync": true,
    "filePicker": true,
    "hostActions": true,
    "microphone": true,
    "speaker": true
  }
}
```

### 6.3 `voice-exit`
Sent when voice mode is actively closed.

### 6.4 `business-open`
Sent when the business drawer is opened, useful for host-side analytics/tracking.

## 7. Host Actions
Host actions are initiated by the embedded page and executed asynchronously by the host.

### 7.1 Protocol Example

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "openCustomerDetail",
  "requestId": "host-001",
  "payload": {
    "customerId": "C1001",
    "sessionId": "966209b7911f4a4981d39c3d060f2b2a"
  }
}
```

Meeting Scenario Examples:

Join Meeting:

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "joinMeeting",
  "requestId": "host-join-001",
  "payload": {
    "platform": "Feishu",
    "meetingNo": 123456789,
    "sessionId": "966209b7911f4a4981d39c3d060f2b2a"
  }
}
```

Stop Meeting:

```json
{
  "source": "wiseai-embed",
  "type": "host-action",
  "action": "stopMeeting",
  "requestId": "host-stop-001",
  "payload": {
    "sessionId": "966209b7911f4a4981d39c3d060f2b2a"
  }
}
```

### 7.2 Recommended Host Implementation

- Host receives `host-action`.
- Look up whitelist mappings based on `action`.
- Validate parameters.
- Asynchronously execute local logic.
- If necessary, return the execution result via `action-result`.

Handling recommendations for meeting actions:

#### `joinMeeting`
- Validate if `payload.platform` exists.
- Validate if `payload.meetingNo` is numeric.
- Route to the corresponding client or native capability based on the platform.
- Return success result upon successful execution.
- Return failure reason upon failure.

#### `stopMeeting`
- No business parameters required.
- Directly stop the current meeting or call the host-side capability to end the meeting.
- It is recommended to also return the result to allow the Digital Employee to confirm.

### 7.3 Common Actions

- `pickFiles`: Open host file picker
- `requestMicrophonePermission`: Request microphone permission
- `openCustomerDetail`: Open customer details
- `openOrder`: Open order page
- `joinMeeting`: Join meeting
- `stopMeeting`: Stop meeting

## 8. Action Result Return
### 8.1 General Format

```json
{
  "source": "wiseai-host",
  "type": "action-result",
  "action": "action-result",
  "requestId": "host-001",
  "payload": {
    "success": true,
    "state": "granted",
    "message": "ok",
    "errorCode": null,
    "payload": {
      "files": [
        {
          "url": "https://example.com/a.pdf",
          "name": "a.pdf"
        }
      ]
    }
  }
}
```

#### Example of Join Meeting Result Return

```json
{
  "source": "wiseai-host",
  "type": "action-result",
  "action": "action-result",
  "requestId": "host-join-001",
  "payload": {
    "success": true,
    "state": "granted",
    "message": "Join meeting initiated",
    "errorCode": null,
    "payload": {
      "platform": "Tencent",
      "meetingNo": 123456789
    }
  }
}
```

#### Example of Stop Meeting Result Return

```json
{
  "source": "wiseai-host",
  "type": "action-result",
  "action": "action-result",
  "requestId": "host-stop-001",
  "payload": {
    "success": true,
    "state": "granted",
    "message": "Meeting stopped",
    "errorCode": null,
    "payload": {}
  }
}
```

#### Example of Failure Result Return

```json
{
  "source": "wiseai-host",
  "type": "action-result",
  "action": "action-result",
  "requestId": "host-join-001",
  "payload": {
    "success": false,
    "state": "denied",
    "message": "Corresponding meeting client not installed",
    "errorCode": "MEETING_CLIENT_NOT_FOUND",
    "payload": {
      "platform": "teams",
      "meetingNo": 123456789
    }
  }
}
```

### 8.2 Permission State Values

- `granted`
- `denied`
- `blocked`
- `unsupported`

### 8.3 Notes

- If the host does not return a result, it does not block the main conversation.
- It is recommended to always return results for scenarios where feedback is needed.
- `pickFiles` is recommended to return within 60 seconds.
- `requestMicrophonePermission` is recommended to return within 15 seconds.
- `joinMeeting` and `stopMeeting` are recommended to always return a result so the Digital Employee can clearly inform the user of the execution outcome.
- `action-result.requestId` must match the original `host-action.requestId`.

## 9. Context Priority
Runtime context is recommended to be overridden in the following priority:

1. URL `ctx.*`
2. `configure.contextData`
3. `updateContext.contextData`

If the host needs strong subsequent overriding, it is recommended to issue it again via `updateContext`.

## 10. SDK API
### 10.1 `create(options)`
Create an SDK instance.

Key Parameters:

- `src`
- `container`
- `allowedOrigin`
- `capabilities`
- `contextData`
- `onEvent`
- `onHostAction`

### 10.2 `mount()`
Mount the iframe to the container.

### 10.3 `destroy()`
Destroy the iframe and message listeners.

### 10.4 `configure(payload)`
Send the `configure` instruction.

Example of meeting action injection:

```js
const sdk = WiseAIEmbedSDK.create({
  src: 'https://example.com/embed/chat/robot-id?token=xxx',
  container: '#app',
  allowedOrigin: 'https://example.com',
  capabilities: {
    contextSync: true,
    hostActions: true,
    microphone: true,
    speaker: true
  },
  onHostAction(message, sdkInstance) {
    const action = message.action;
    const payload = message.payload || {};

    if (action === 'joinMeeting') {
      const platform = payload.platform;
      const meetingNo = payload.meetingNo;

      sdkInstance.replyActionResult(message.requestId, {
        success: true,
        state: 'granted',
        message: `Join meeting handled: ${platform} ${meetingNo}`,
        payload: {
          platform,
          meetingNo
        }
      });
      return;
    }

    if (action === 'stopMeeting') {
      sdkInstance.replyActionResult(message.requestId, {
        success: true,
        state: 'granted',
        message: 'Stop meeting handled',
        payload: {}
      });
    }
  }
});

sdk.mount();

sdk.configure({
  frontendActions: [
    {
      code: 'joinMeeting',
      name: 'Join Meeting',
      description: 'Join meeting by platform and meeting number',
      target: 'parent',
      awaitResult: true,
      inputSchema: {
        type: 'object',
        properties: {
          platform: { type: 'string' },
          meetingNo: { type: 'integer' }
        },
        required: ['platform', 'meetingNo']
      }
    },
    {
      code: 'stopMeeting',
      name: 'Stop Meeting',
      description: 'Stop the current meeting',
      target: 'parent',
      awaitResult: true,
      inputSchema: {
        type: 'object',
        properties: {}
      }
    }
  ],
  voiceCommands: [
    { phrase: 'Join meeting', actionCode: 'joinMeeting' },
    { phrase: 'Stop meeting', actionCode: 'stopMeeting' }
  ]
});
```

### 10.5 `updateContext(payload)`
Send the `updateContext` instruction.

### 10.6 `sendText(text)`
Send a text message.

### 10.7 `openVoice()` / `closeVoice()`
Open or close the voice overlay.

### 10.8 `on(eventName, handler)`
Listen for specified events, also supports listening for `*`.

### 10.9 `off(eventName, handler?)`
Remove an event listener.

### 10.10 `replyActionResult(requestId, result)`
Reply with the execution result of a host action.

## 11. Security Guidelines

- Strictly validate the `origin`.
- Hosts should not trust undeclared actions.
- Actions visible to the model must come from the whitelist.
- It is recommended to log `requestId + action + timestamp`.
- If the embedded page is deployed on multiple domains, the SDK side should explicitly pass `allowedOrigin`.
