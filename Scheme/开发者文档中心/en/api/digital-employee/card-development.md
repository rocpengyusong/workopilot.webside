# Universal Skill Card Development Specification

## 1. Scope

This document is based on the latest code implementation and provides guidance for developing, registering, and configuring the "Skill Card in Chat Message + Right Drawer" capability.

There are currently two card pathways in the system:

- **Legacy Pathway**: `pharma_sku_filter` exclusively uses `ai_chat_temp_selection`. Cards are automatically generated after writing temporary results.
- **Universal Pathway**: New skills (except `pharma_sku_filter`) explicitly trigger cards via the `showCard` tool.

If you are developing a new skill card, you should default to the **`showCard` universal pathway** instead of reusing `ai_chat_temp_selection`.

---

## 2. Real Working Mechanism in Current Code

### 2.1 Backend Runtime Logic

The chat execution service first loads the enabled skill configurations for the current Digital Employee and merges the following for each skill:

- `default_tools / override_tools`
- `default_card / override_card`
- `default_drawer / override_drawer`

Cards are then handled according to these rules:

1. Determine if `showCard` is enabled in the skill's tool configuration.
2. Determine if a valid `trigger_prompt` exists in the skill's card configuration.
3. If both conditions are met, the trigger rules are injected into the system prompt.
4. When conditions are met during a conversation, the LLM explicitly calls `showCard`.
5. The backend records the card intent for this round and generates `card_data` after the assistant's reply is completed.
6. If `showCard` is not called in this round, the system falls back to the legacy automatic card logic of `pharma_sku_filter`.

In other words:

- **Whether a new skill can generate a card depends on the completeness of its tool and card configurations, not on the frontend.**
- **If card and drawer are configured but `showCard` is not enabled, the card will not be automatically generated.**

Code Locations:

- `AiChatExecuteService`: Responsible for skill configuration merging, `showCard` priority card generation, and legacy fallback.
- `AiSkillShowCardService`: Implements the `showCard` tool.
- `RobotSystemPromptBuilderService`: Injects `trigger_prompt` into the system prompt.

---

## 3. Two Supported Drawer Modes

The detail drawer for current cards supports two modes:

### 3.1 Component Mode

Configuration Characteristics:

- `drawer_mode = "COMPONENT"`
- `component = "YourDrawerComponentName"`

Runtime Behavior:

- When the chat page receives a card, the user clicks "View Details".
- The frontend looks up the local component by name from the `drawerComponentRegistry`.
- The right drawer renders the Vue component directly.

Use Cases:

- Detail data needs to be assembled by the frontend.
- Complex interactions are required within the drawer.
- The page needs to maintain a consistent UI style with the current system.

Current Examples:

- `SkuListDrawer`
- `FillCardDrawer`

### 3.2 Iframe Mode

Configuration Characteristics:

- `drawer_mode = "URL"`
- `drawer_url = "https://xxx.com/page?sessionId={{session_id}}&userId={{user_id}}"`

Runtime Behavior:

- When the card is clicked, no local Vue component is rendered.
- The right drawer renders the iframe page directly.
- The backend writes context like `drawer_url`, `sessionId`, and `userId` into the card data.

Use Cases:

- The detail page already exists in another system.
- Quick integration with external pages without developing local drawers.

Notes:

- `drawer_mode` only has two valid values: `COMPONENT` and `URL`.
- Component mode ignores `drawer_url`.
- URL mode does not depend on `drawerComponentRegistry`.

---

## 4. Component Drawer Development Specification

### 4.1 Component Location

Chat drawer components are placed in the frontend chat page directory, categorized by business subdirectories:

```text
wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/components/
```

Examples:

- Medicine Filter Drawer: `components/pharma/SkuListDrawer.vue`
- Intelligent Filling Drawer: `components/fill/FillCardDrawer.vue`

Guidelines:

- One skill corresponds to one independent drawer component.
- Use `PascalCase` for filenames.
- Organize by business domain directories; do not pile everything in the root.

### 4.2 Component Registration

All chat drawer components must be registered in:

`wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/index.vue`

Current registration method:

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

To develop a new component:

1. Create a new `.vue` component file.
2. Add a mapping entry in `drawerComponentRegistry`.

### 4.3 Component Naming Specification

The registered name will eventually undergo normalization, so we recommend:

- Use the component name directly for the `component` configuration value, e.g., `FillCardDrawer`.
- Use lowercase without spaces for the registry key, e.g., `fillcarddrawer`.

Consistency Recommendation:

- Vue filename: `FillCardDrawer.vue`
- `component` in config: `FillCardDrawer`
- Registry key: `fillcarddrawer`

### 4.4 Component Input Specification

Chat drawer components should primarily rely on these runtime inputs:

- `sessionId`
- `skillCode`
- `cardData`
- `drawerData`

Best Practices:

- The drawer component should query details itself using `sessionId + skillCode`.
- Do not stuff large amounts of business results into `card_data`.
- Keep `card_data` for minimal context only.

Benefits:

- Cards can be restored after historical messages are refreshed.
- Drawer logic remains independent.
- Backend card protocol stays lightweight.

---

## 5. System Registration Specification

### 5.1 Should New Skills Register `showCard`?

The conclusion is clear:

- `pharma_sku_filter`: **Do not change**. Continue using the legacy pathway; no need for `showCard`.
- All newly added universal skill cards: **Must register `showCard`**.

This is because the current backend logic explicitly requires:

- `showCard` enabled in the skill tool configuration.
- `trigger_prompt` filled in the skill card configuration.

Only when both conditions are met will the system prompt tell the model "when to call `showCard`".

If `showCard` is missing:

- The LLM will not be allowed to use the universal card generation pathway.
- Even if you configure the drawer and title prompts, the card will not be automatically generated.

### 5.2 Should Historical Skills Re-execute SQL?

If a historical skill is:

- Intended to keep the `pharma_sku_filter` legacy logic: No changes needed.
- To be upgraded to a universal skill card: Update `default_tools/default_card/default_drawer` and re-run the update scripts.

For example, `20260414-add-fill-card-skill.sql` should now use the version with `showCard`.

---

## 6. Skill Registration SQL Template

Below is a universal template for registering a **new skill using the `showCard` pathway**.

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
  'Skill Name',
  'skill_code_here',
  'Business Category',
  'Skill Description',
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
    "trigger_prompt":"Call showCard to display the result card when the skill has produced structured results for the user to continue viewing. Trigger only after results are actually produced; do not trigger for pure explanations, clarifications, empty results, or failed results.",
    "title_prompt":"Generate a card title within 20 to 30 characters based on the processing result, highlighting the value or completion status.",
    "action":"open_drawer",
    "drawer_code":"custom_result_drawer"
  }',
  '{
    "drawer_code":"custom_result_drawer",
    "drawer_mode":"COMPONENT",
    "component":"CustomResultDrawer",
    "title":"Result Details",
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
  `skill_name` = 'Skill Name',
  `skill_category` = 'Business Category',
  `skill_desc` = 'Skill Description',
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
    "trigger_prompt":"Call showCard to display the result card when the skill has produced structured results for the user to continue viewing. Trigger only after results are actually produced; do not trigger for pure explanations, clarifications, empty results, or failed results.",
    "title_prompt":"Generate a card title within 20 to 30 characters based on the processing result, highlighting the value or completion status.",
    "action":"open_drawer",
    "drawer_code":"custom_result_drawer"
  }',
  `default_drawer` = '{
    "drawer_code":"custom_result_drawer",
    "drawer_mode":"COMPONENT",
    "component":"CustomResultDrawer",
    "title":"Result Details",
    "drawer_url":null
  }',
  `is_active` = 1,
  `sort_index` = 100,
  `update_time` = NOW()
WHERE `skill_code` = 'skill_code_here';

COMMIT;
```

---

## 7. Default Configuration Guidelines

### 7.1 `default_tools`

New skills intended for universal cards must at least include:

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

If `showCard` is missing:

- `trigger_prompt` will not take effect.
- The model will not be guided to call the universal card tool.

### 7.2 `default_card`

The most critical fields are:

- `card_type`
- `trigger_prompt`
- `title_prompt`
- `action`
- `drawer_code`

Recommended minimal structure:

```json
{
  "card_type": "custom_result_card",
  "trigger_prompt": "Call showCard to display the result card when the skill has produced structured results for the user to continue viewing.",
  "title_prompt": "Generate a concise title based on this result, highlighting the result value.",
  "action": "open_drawer",
  "drawer_code": "custom_result_drawer"
}
```

### 7.3 `default_drawer`

Component Mode:

```json
{
  "drawer_code": "custom_result_drawer",
  "drawer_mode": "COMPONENT",
  "component": "CustomResultDrawer",
  "title": "Result Details",
  "drawer_url": null
}
```

Iframe Mode:

```json
{
  "drawer_code": "custom_result_drawer",
  "drawer_mode": "URL",
  "component": null,
  "title": "Result Details",
  "drawer_url": "https://demo.xxx.com/page?sessionId={{session_id}}&userId={{user_id}}"
}
```

---

## 8. Difference Between `trigger_prompt` and `title_prompt`

These two fields are often confused and must be clearly distinguished.

### 8.1 `trigger_prompt`

Role:

- Tells the model **when to generate a card**.
- Participates in system prompt construction.
- The model should only call `showCard` when conditions are met.

It addresses: "Should a card be generated in this round?"

Recommended Writing:

- Describe clear business result states.
- Describe conditions for triggering.
- Specify when NOT to trigger.

Example:

> "Call `showCard` to display the result card when the skill has produced structured results for the user to continue viewing, confirming, or processing. Trigger only after results are actually produced; do not trigger for small talk, explanations, clarifications, no results, or failed results."

### 8.2 `title_prompt`

Role:

- Tells the model **what to write for the card title**.
- Used to generate the card title once it's determined that a card will be shown.

It addresses: "What should the title of this card be?"

Example:

> "Generate a Chinese title within 20 to 30 characters based on the processing result, highlighting the result value or completion status."

### 8.3 Summary

- `trigger_prompt` determines **whether to generate a card**.
- `title_prompt` determines **how to write the card title**.

---

## 9. Configuration in Settings Page

The current Digital Employee skill configuration page supports overriding card-related settings.

Page Location:

- Frontend Page: `wiseai.vue/apps/web-antd/src/views/ai-employee/agent-config/index.vue`

### 9.1 Configuration Entry

In the Digital Employee edit page:

1. Open the skill override configuration modal.
2. Find the "Card Trigger and Title" area.
3. Editable fields:
   - `Trigger Prompt`
   - `Title Prompt`

The page also displays:

- Current `CardType`
- Bound `Drawer`
- `DrawerMode`
- Current component or URL

### 9.2 Relationship on the Page

Three configuration layers work together:

1. **Tool Configuration**: Determines if `showCard` is enabled.
2. **Card Configuration**: Determines `trigger_prompt / title_prompt / drawer_code`.
3. **Drawer Configuration**: Determines if the details are a local component or an iframe.

Recommended configuration order:

1. Confirm `showCard` is enabled in tools.
2. Configure `trigger_prompt`.
3. Configure `title_prompt`.
4. Confirm `drawer_code` corresponds to a complete drawer configuration.

---

## 10. Relationship Between Card and Drawer

### 10.1 Relationship Model

- **Card**: Responsible for displaying the summary.
- **Drawer**: Responsible for displaying details.

Cards typically only keep lightweight fields:

- `skillCode`, `sessionId`, `title`, `subtitle`, `highlights`, `drawerCode`

Drawers typically only keep:

- `drawer_mode`, `component`, `drawer_url`, `sessionId`, `userId`, `skillCode`

### 10.2 Design Principles

**Avoid**:

- Stuffing the entire detailed result directly into `card_data`.
- Making the card a "large data carrier".

**Recommend**:

- Keeping minimal context in the card.
- Let the drawer component query details again using `sessionId + skillCode`.

**Benefits**:

- Stable restoration of historical messages.
- Clear separation of responsibilities between frontend and backend.
- Card protocol remains lightweight.

---

## 11. Steps for Developing a New Card

### 11.1 Component Mode

1. Create a drawer component in the frontend, e.g.:
   - `wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/components/fill/FillCardDrawer.vue`
2. Register the component in the chat page:
   - `drawerComponentRegistry`
3. Register the skill in `ai_skill_registry`:
   - `default_tools` includes `showCard`.
   - `default_card` includes `trigger_prompt/title_prompt`.
   - `default_drawer` set to `COMPONENT + component`.
4. Bind the skill in Digital Employee settings.
5. Override in the Digital Employee skill configuration page if needed.

### 11.2 Iframe Mode

1. No local Vue drawer component needed.
2. Enable `showCard` in `default_tools`.
3. Configure `trigger_prompt/title_prompt` in `default_card`.
4. Set `default_drawer` to `URL + drawer_url`.
5. Verify that context parameters in the URL can be correctly replaced.

---

## 12. Recommended Checklist

Check at least the following before release:

- Is `showCard` enabled for the skill?
- Does `trigger_prompt` clearly state when to trigger and when not to?
- Does `title_prompt` clearly limit the title style and length?
- Is `drawer_code` consistent with the drawer configuration?
- Is `drawer_mode` correct?
- For component mode, is the component registered in `drawerComponentRegistry`?
- Can cards still reopen the drawer after historical messages are refreshed?
- Do non-result conversations NOT accidentally trigger cards?

---

## 13. FAQ

### 13.1 Why is no card generated even though card and drawer are configured?

Most common reasons:

- `showCard` is not enabled.
- `trigger_prompt` is empty.

### 13.2 Why doesn't `pharma_sku_filter` need `showCard`?

Because it currently uses the legacy pathway:

- Writes temporary data via `ai_chat_temp_selection`.
- Automatically falls back to generate a card after the assistant reply.

This is an exception, not a template for new skills.

### 13.3 Can a skill use both legacy and new pathways?

Currently not recommended. We suggest:

- `pharma_sku_filter` stays on the legacy pathway.
- New skills consistently use `showCard`.

This avoids maintaining two sets of behaviors.

---

## 14. Current Reference Implementations

Refer to these files directly:

- Backend Execution Chain: `wiseai.net.service/WIseAi.Services/AiChatService/Impl/AiChatExecuteService.cs`
- `showCard` Tool: `wiseai.net.service/WIseAi.Services/skills/CommonCard/AiSkillShowCardService.cs`
- System Prompt Building: `wiseai.net.service/WIseAi.Services/AiChatService/Impl/RobotSystemPromptBuilderService.cs`
- Frontend Chat Page: `wiseai.vue/apps/web-antd/src/views/ai-employee/workbench/index.vue`
- Skill Config Page: `wiseai.vue/apps/web-antd/src/views/ai-employee/agent-config/index.vue`
- `fill_card` SQL Example: `wiseai.net.service/docs/AIChat/sql/20260414-add-fill-card-skill.sql`

This document should serve as the baseline specification for future skill card development.
