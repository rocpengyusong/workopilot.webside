# 通用技能卡片开发规范

## 1. 适用范围

本文档基于当前最新代码实现编写，用于指导“聊天消息里的技能卡片 + 右侧抽屉”能力的开发、注册和配置。

当前系统里有两条卡片链路：

- 旧链路：`pharma_sku_filter` 独占 `ai_chat_temp_selection`，通过写临时结果后自动出卡。
- 新链路：除 `pharma_sku_filter` 外的新技能，统一通过 `showCard` 工具显式触发卡片。

如果你开发的是新技能卡片，默认应走 **`showCard` 通用链路**，不要再复用 `ai_chat_temp_selection`。

---

## 2. 当前代码中的真实工作方式

### 2.1 后端运行时逻辑

聊天执行服务会先加载当前数字员工启用的技能配置，合并出每个技能的：

- `default_tools / override_tools`
- `default_card / override_card`
- `default_drawer / override_drawer`

然后按下面规则处理卡片：

1. 先判断该技能的工具配置里是否启用了 `showCard`
2. 再判断该技能卡片配置里是否存在有效的 `trigger_prompt`
3. 如果满足以上条件，就把触发规则注入系统提示词
4. 大模型在对话中满足条件时，显式调用 `showCard`
5. 后端记录本轮卡片意图，并在 assistant 回复完成后生成 `card_data`
6. 如果本轮没有 `showCard`，才回退到 `pharma_sku_filter` 的旧自动出卡逻辑

也就是说：

- **新技能能不能出卡，关键不在前端，而在技能工具配置和卡片配置是否完整**
- **只配置了卡片和抽屉，但没有启用 `showCard`，新技能不会自动出卡**

对应代码位置：

- `AiChatExecuteService`：负责技能配置合并、`showCard` 优先出卡、旧链路 fallback
- `AiSkillShowCardService`：负责实现 `showCard` 工具
- `RobotSystemPromptBuilderService`：负责把 `trigger_prompt` 注入系统提示词

---

## 3. 卡片支持的两种抽屉形式

当前卡片的详情抽屉支持两种模式：

### 3.1 组件模式

配置特征：

- `drawer_mode = "COMPONENT"`
- `component = "你的抽屉组件名"`

运行时行为：

- 聊天页收到卡片后，点击“查看详情”
- 前端从 `drawerComponentRegistry` 中按组件名找到本地组件
- 右侧抽屉直接渲染该 Vue 组件

适用场景：

- 详情数据需要前端自行组装
- 要在抽屉内做复杂交互
- 页面要和当前系统 UI 风格保持一致

当前示例：

- `SkuListDrawer`
- `FillCardDrawer`

### 3.2 Iframe 模式

配置特征：

- `drawer_mode = "URL"`
- `drawer_url = "https://xxx.com/page?sessionId={{session_id}}&userId={{user_id}}"`

运行时行为：

- 点击卡片后，不渲染本地 Vue 组件
- 右侧抽屉直接渲染 iframe 页面
- 后端会把 `drawer_url` 和 `sessionId / userId` 等上下文写入卡片数据

适用场景：

- 详情页已经在其他系统中存在
- 想快速对接外部页面，不重新开发本地抽屉

注意：

- `drawer_mode` 只有 `COMPONENT` 和 `URL` 两种有效值
- 组件模式忽略 `drawer_url`
- URL 模式不依赖 `drawerComponentRegistry`

---

## 4. 组件抽屉开发规范

### 4.1 组件放置位置

聊天抽屉组件统一放在前端聊天页目录下，建议按业务分类建子目录：

```text
wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/components/
```

例如：

- 药品筛选抽屉：`components/pharma/SkuListDrawer.vue`
- 智能填报抽屉：`components/fill/FillCardDrawer.vue`

建议规范：

- 一个技能对应一个独立抽屉组件
- 文件名使用 PascalCase
- 目录按业务域归类，不要全部堆在根目录

### 4.2 组件注册位置

所有聊天抽屉组件都需要注册到：

`wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/index.vue`

当前注册方式是：

```ts
const drawerComponentRegistry: Record<string, any> = {
  fillcarddrawer: defineAsyncComponent(
    () => import('./components/fill/FillCardDrawer.vue'),
  ),
  skulistdrawer: defineAsyncComponent(
    () => import('./components/pharma/SkuListDrawer.vue'),
  ),
};
```

开发新组件时要做两件事：

1. 新建 `.vue` 组件文件
2. 在 `drawerComponentRegistry` 中增加一条映射

### 4.3 组件命名规范

注册名最终会经过规范化处理，因此建议：

- `component` 配置值直接写组件名，例如 `FillCardDrawer`
- 注册表 key 使用小写无空格，例如 `fillcarddrawer`

推荐保持一致：

- Vue 文件名：`FillCardDrawer.vue`
- 配置里的 `component`：`FillCardDrawer`
- 注册表 key：`fillcarddrawer`

### 4.4 组件入参规范

聊天抽屉组件应优先依赖这些运行时入参：

- `sessionId`
- `skillCode`
- `cardData`
- `drawerData`

推荐实践：

- 抽屉组件自己根据 `sessionId + skillCode` 去查询详情
- 不要把大块业务结果塞进 `card_data`
- `card_data` 只保留最小上下文

这样可以保证：

- 历史消息刷新后仍能恢复卡片
- 抽屉逻辑独立
- 后端卡片协议保持轻量

---

## 5. 系统注册规范

### 5.1 新技能要不要注册 `showCard`

结论很明确：

- `pharma_sku_filter`：**不要改**，继续走旧链路，不需要补 `showCard`
- 所有新增的通用技能卡片：**需要注册 `showCard`**

原因是当前后端逻辑明确要求：

- 技能工具配置里启用了 `showCard`
- 技能卡片配置里填写了 `trigger_prompt`

只有同时满足这两个条件，系统提示词里才会告诉模型“什么时候调用 `showCard`”。

如果缺少 `showCard`：

- 大模型不会被允许使用通用出卡链路
- 即使你配了抽屉和标题提示词，也不会自动出卡

### 5.2 历史技能是否要重新执行 SQL

如果历史技能是：

- 只想保持 `pharma_sku_filter` 旧逻辑：不用改
- 想升级为通用技能卡片：需要更新 `default_tools/default_card/default_drawer`，并重新执行更新脚本

例如 `20260414-add-fill-card-skill.sql` 现在就应该使用带 `showCard` 的版本。

---

## 6. 技能注册 SQL 模板

下面给出一个通用模板，适用于 **新技能走 `showCard` 链路** 的注册方式。

```sql
START TRANSACTION;

SET @skill_id := (
  SELECT COALESCE(MAX(`id`), 100000) + 1
  FROM `ai_skill_registry`
);

INSERT INTO `ai_skill_registry` (
  `id`,
  `skill_name`,
  `skill_code`,
  `skill_category`,
  `skill_desc`,
  `skill_icon`,
  `default_tools`,
  `default_card`,
  `default_drawer`,
  `is_active`,
  `sort_index`,
  `create_time`,
  `update_time`
)
SELECT
  @skill_id,
  '技能名称',
  'skill_code_here',
  '业务分类',
  '技能说明',
  'file-text',
  '{
    "tools":[
      {
        "code":"showCard",
        "name":"showCard",
        "is_active":true
      }
    ]
  }',
  '{
    "card_type":"custom_result_card",
    "trigger_prompt":"当技能已经产出可供用户继续查看的结构化结果时，调用 showCard 展示结果卡片。只有真正产出结果后才触发，不要在纯解释、澄清、空结果或失败结果时触发。",
    "title_prompt":"根据本次处理结果生成一句 20 到 30 字内的卡片标题，突出结果价值或完成状态。",
    "action":"open_drawer",
    "drawer_code":"custom_result_drawer"
  }',
  '{
    "drawer_code":"custom_result_drawer",
    "drawer_mode":"COMPONENT",
    "component":"CustomResultDrawer",
    "title":"结果详情",
    "drawer_url":null
  }',
  1,
  100,
  NOW(),
  NOW()
FROM DUAL
WHERE NOT EXISTS (
  SELECT 1
  FROM `ai_skill_registry`
  WHERE `skill_code` = 'skill_code_here'
);

UPDATE `ai_skill_registry`
SET
  `skill_name` = '技能名称',
  `skill_category` = '业务分类',
  `skill_desc` = '技能说明',
  `skill_icon` = 'file-text',
  `default_tools` = '{
    "tools":[
      {
        "code":"showCard",
        "name":"showCard",
        "is_active":true
      }
    ]
  }',
  `default_card` = '{
    "card_type":"custom_result_card",
    "trigger_prompt":"当技能已经产出可供用户继续查看的结构化结果时，调用 showCard 展示结果卡片。只有真正产出结果后才触发，不要在纯解释、澄清、空结果或失败结果时触发。",
    "title_prompt":"根据本次处理结果生成一句 20 到 30 字内的卡片标题，突出结果价值或完成状态。",
    "action":"open_drawer",
    "drawer_code":"custom_result_drawer"
  }',
  `default_drawer` = '{
    "drawer_code":"custom_result_drawer",
    "drawer_mode":"COMPONENT",
    "component":"CustomResultDrawer",
    "title":"结果详情",
    "drawer_url":null
  }',
  `is_active` = 1,
  `sort_index` = 100,
  `update_time` = NOW()
WHERE `skill_code` = 'skill_code_here';

COMMIT;
```

---

## 7. 默认配置怎么写

### 7.1 `default_tools`

新技能要走通用卡片，必须至少包含：

```json
{
  "tools": [
    {
      "code": "showCard",
      "name": "showCard",
      "is_active": true
    }
  ]
}
```

如果这里没有 `showCard`：

- `trigger_prompt` 不会生效
- 模型不会被引导调用通用出卡工具

### 7.2 `default_card`

当前最关键的字段有：

- `card_type`
- `trigger_prompt`
- `title_prompt`
- `action`
- `drawer_code`

推荐最小结构：

```json
{
  "card_type": "custom_result_card",
  "trigger_prompt": "当技能已经产出可供用户继续查看的结构化结果时，调用 showCard 展示结果卡片。",
  "title_prompt": "根据本次结果生成一句简洁中文标题，突出结果价值。",
  "action": "open_drawer",
  "drawer_code": "custom_result_drawer"
}
```

### 7.3 `default_drawer`

组件模式：

```json
{
  "drawer_code": "custom_result_drawer",
  "drawer_mode": "COMPONENT",
  "component": "CustomResultDrawer",
  "title": "结果详情",
  "drawer_url": null
}
```

Iframe 模式：

```json
{
  "drawer_code": "custom_result_drawer",
  "drawer_mode": "URL",
  "component": null,
  "title": "结果详情",
  "drawer_url": "https://demo.xxx.com/page?sessionId={{session_id}}&userId={{user_id}}"
}
```

---

## 8. `trigger_prompt` 和 `title_prompt` 的区别

这两个字段最容易混淆，必须分清。

### 8.1 `trigger_prompt`

作用：

- 告诉模型 **什么时候应该出卡**
- 参与系统提示词构建
- 只有在满足条件时，模型才应调用 `showCard`

它解决的是：

- “这轮要不要出卡”

推荐写法：

- 描述明确的业务结果状态
- 描述哪些情况下可以触发
- 明确哪些情况不要触发

推荐示例：

```text
当技能已经生成可供用户继续查看、确认或处理的结构化结果时，调用 showCard 展示结果卡片。只有真正产出结果后才触发，不要在闲聊、解释、澄清、无结果或失败结果时触发。
```

### 8.2 `title_prompt`

作用：

- 告诉模型 **卡片标题怎么写**
- 在卡片已经确定要展示后，用来生成卡片标题

它解决的是：

- “这张卡片标题写什么”

推荐示例：

```text
根据本次处理结果生成一句 20 到 30 字内的中文标题，突出结果价值或完成状态。
```

### 8.3 一句话总结

- `trigger_prompt` 决定 **出不出卡**
- `title_prompt` 决定 **卡片标题怎么写**

---

## 9. 设置页面如何配置

当前数字员工技能配置页面已经支持对卡片相关配置做覆盖编辑。

页面位置：

- 前端页面：`wiseai.vue/apps/web-antd/src/views/ai-employee/agent-config/index.vue`

### 9.1 配置入口

进入数字员工编辑页后：

1. 打开技能覆盖配置弹窗
2. 找到“卡片触发与标题”区域
3. 可编辑：
   - `触发提示词`
   - `标题提示词`

同时页面还会展示：

- 当前卡片 `CardType`
- 当前卡片绑定的 `Drawer`
- 当前抽屉 `DrawerMode`
- 当前抽屉组件或 URL

### 9.2 页面上的配置关系

当前页面实际上是三层配置一起工作：

1. 工具配置
   - 决定 `showCard` 是否启用
2. 卡片配置
   - 决定 `trigger_prompt / title_prompt / drawer_code`
3. 抽屉配置
   - 决定详情是本地组件还是 iframe

因此正确配置顺序建议是：

1. 先确认工具里启用了 `showCard`
2. 再配置 `trigger_prompt`
3. 再配置 `title_prompt`
4. 最后确认 `drawer_code` 对应的抽屉配置是否完整

---

## 10. 卡片与抽屉的关系

### 10.1 关系模型

当前实现里：

- 卡片只负责展示摘要
- 抽屉负责展示详情

卡片里通常只保留这些轻量字段：

- `skillCode`
- `sessionId`
- `title`
- `subtitle`
- `highlights`
- `drawerCode`

抽屉数据里通常只保留：

- `drawer_mode`
- `component`
- `drawer_url`
- `sessionId`
- `userId`
- `skillCode`

### 10.2 设计原则

不要这样做：

- 把整份详情结果直接塞到 `card_data`
- 把卡片做成“大数据载体”

推荐这样做：

- 卡片只保存最小上下文
- 抽屉组件根据 `sessionId + skillCode` 再查详情

这样做的好处：

- 历史消息恢复稳定
- 前后端职责清晰
- 卡片协议不会越来越重

---

## 11. 开发新卡片的完整步骤

### 11.1 组件模式

1. 在前端创建抽屉组件，例如：
   - `wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/components/fill/FillCardDrawer.vue`
2. 在聊天页注册组件：
   - `drawerComponentRegistry`
3. 在 `ai_skill_registry` 中注册技能：
   - `default_tools` 包含 `showCard`
   - `default_card` 配好 `trigger_prompt/title_prompt`
   - `default_drawer` 配成 `COMPONENT + component`
4. 在数字员工里绑定该技能
5. 需要覆盖时，在数字员工技能配置页修改

### 11.2 Iframe 模式

1. 不需要开发本地 Vue 抽屉组件
2. `default_tools` 仍然要启用 `showCard`
3. `default_card` 仍然要配置 `trigger_prompt/title_prompt`
4. `default_drawer` 配成 `URL + drawer_url`
5. 验证 URL 中的上下文参数是否能正确替换

---

## 12. 推荐检查清单

发布前至少检查以下内容：

- 技能是否启用了 `showCard`
- `trigger_prompt` 是否明确说明“何时触发”和“何时不要触发”
- `title_prompt` 是否明确限制标题风格和长度
- `drawer_code` 是否和抽屉配置一致
- `drawer_mode` 是否正确
- 组件模式下，组件是否已注册到 `drawerComponentRegistry`
- 历史消息刷新后，卡片是否还能重新打开抽屉
- 非结果型对话是否不会误触发卡片

---

## 13. 常见问题

### 13.1 只配了卡片和抽屉，为什么不出卡

最常见原因：

- 没有启用 `showCard`
- 没有填写 `trigger_prompt`

### 13.2 为什么 `pharma_sku_filter` 不用加 `showCard`

因为它当前仍走旧链路：

- 通过 `ai_chat_temp_selection` 写临时数据
- assistant 回复完成后自动 fallback 出卡

这是特例，不是新技能模板。

### 13.3 一个技能能不能既走旧链路又走新链路

当前不推荐这样做。

建议：

- `pharma_sku_filter` 保持旧链路
- 新技能统一走 `showCard`

避免维护两套行为。

---

## 14. 当前参考实现

可以直接参考下面这些文件：

- 后端执行链：`wiseai.net.service/WIseAi.Services/AiChatService/Impl/AiChatExecuteService.cs`
- `showCard` 工具：`wiseai.net.service/WIseAi.Services/skills/CommonCard/AiSkillShowCardService.cs`
- 系统提示词构建：`wiseai.net.service/WIseAi.Services/AiChatService/Impl/RobotSystemPromptBuilderService.cs`
- 前端聊天页：`wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/index.vue`
- 技能配置页：`wiseai.vue/apps/web-antd/src/views/ai-employee/agent-config/index.vue`
- `fill_card` SQL 示例：`wiseai.net.service/docs/AIChat/sql/20260414-add-fill-card-skill.sql`

这份文档应作为后续技能卡片开发的基线规范使用。
