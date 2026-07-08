---
feature_area: agentforce-lightning-types
status: folded-into-catalog
catalog_id: P2
catalog_ref: patterns/canonical/lightning-types-best-practices.md
note: "Folded into P2 §14 (chip-menu gotchas, show_command force-render, LEX-vs-ECv2 click pattern) plus salesforce-deploy-gotchas.mdc. Do not re-catalog as a new asset."
---

# Agentforce Chip Menu — Custom Lightning Type Renderer

A reference for building clickable chip menus in Agentforce chat using a Custom Lightning Type (CLT) backed by an Apex `@InvocableMethod` and rendered by an LWC.

**Default surface:** Enhanced Web Chat v2 (ECv2), via `embeddedservice_bootstrap.utilAPI.sendTextMessage()`.
**Alternate surface:** Lightning Agentforce Panel (LEX), via `lightning/accApi` `execute()`. See [§5](#5-lwc-bundle) for the swap.

---

## ⚠️ Read First — Load-Bearing Gotchas

These are the failure points. Get them wrong and you get clean deploys that break at runtime or become immutable.

| # | Rule | Why it matters |
|---|------|----------------|
| 1 | **`c__` prefix** on `@apexClassType/c__ChipMenuData` in `schema.json` | Omitting it deploys fine but breaks `sf agent preview --use-live-actions` (HTTP 400) and becomes **immutable** once an active BotVersion references the LT. Get it right the first time. |
| 2 | **Only** `lightningDesktopGenAi/renderer.json` | Do **NOT** create an `enhancedWebChat/renderer.json` override. |
| 3 | **No `attributes` block** in `renderer.json` | Fails deploy with `unevaluatedProperties` error. Only `definition` is allowed. |
| 4 | **`is_displayable: True` + `filter_from_agent: False`** on the chipMenu output | This combination triggers `ShowCommand` and hides the raw JSON from the LLM context. |
| 5 | **ShowCommand is non-deterministic** | If chips don't render, strengthen `reasoning.instructions` with directive language ("MUST present", "Do NOT rewrite as plain text"). |
| 6 | **CLTs do NOT render in Agent Builder Preview** | Always test on the live ECv2 site or Lightning panel. |
| 7 | **`sendTextMessage` is ECv2-only** | For the LEX Agentforce panel, use `lightning/accApi` `execute()` instead. |
| 8 | **`sendTextMessage` must target the parent window** | Required for the chip on-click action to work. |
| 9 | **Re-publish the ESD regularly** when publishing the CLT | Needed to make it renderable in the ECv2 deployment. |
| 10 | **Action must be stand-alone** | You cannot build the action into an existing flow as an Apex action with a CLT output. It must be a stand-alone action called by the agent. |

---

## File Structure

```
force-app/main/default/
├── classes/
│   ├── ChipMenuData.cls               ← Apex Renderer DTO
│   └── ChipMenuService.cls            ← @InvocableMethod service
├── lightningTypes/
│   └── ChipMenuOutput/
│       ├── ChipMenuOutput.lightningTypeBundle-meta.xml
│       ├── schema.json
│       └── lightningDesktopGenAi/
│           └── renderer.json
└── lwc/
    └── chipMenuRenderer/
        ├── chipMenuRenderer.html
        ├── chipMenuRenderer.js
        ├── chipMenuRenderer.css
        └── chipMenuRenderer.js-meta.xml
```

---

## 1. Apex Renderer DTO — `ChipMenuData.cls`

```apex
@JsonAccess(serializable='always' deserializable='always')
global class ChipMenuData {

    @AuraEnabled
    global String menuJSON;   // ← LWC reads this.value.menuJSON

    global ChipMenuData(String menuJSON) {
        this.menuJSON = menuJSON;
    }

    global ChipMenuData() {
        this.menuJSON = '';
    }
}
```

**Rules:**
- Must be `global` — LightningType `@apexClassType` resolution refuses non-global classes.
- `@JsonAccess` with both `serializable` and `deserializable` set to `always`.
- `@AuraEnabled` on the field — **not** `@InvocableVariable` (that belongs on the `Response` class).
- One `String` field holding the entire JSON payload.

---

## 2. Apex Service — `ChipMenuService.cls`

The action receives a list of `ChipItem` objects (`label` + `utterance`) from the agent and packages them for the LWC.

```apex
public with sharing class ChipMenuService {

    public class ChipItem {
        public String label;
        public String utterance;
    }

    public class Request {
        @InvocableVariable(label='Chips JSON' description='JSON array of chip objects with label and utterance fields')
        public String chips;
    }

    public class Response {
        @InvocableVariable
        public Boolean success;

        @InvocableVariable(label='Chip Menu' description='Structured payload for ChipMenuRenderer LWC')
        public ChipMenuData chipMenu;

        @InvocableVariable
        public String errorMessage;
    }

    @InvocableMethod(label='Show Chip Menu' description='Renders clickable chips.')
    public static List<Response> execute(List<Request> requests) {
        List<Response> responses = new List<Response>();
        for (Request req : requests) {
            Response res = new Response();
            try {
                List<Map<String, String>> chipList = new List<Map<String, String>>();
                if (String.isNotBlank(req.chips)) {
                    List<ChipItem> parsedChips =
                        (List<ChipItem>) JSON.deserialize(req.chips, List<ChipItem>.class);
                    for (ChipItem c : parsedChips) {
                        Map<String, String> m = new Map<String, String>();
                        m.put('label', c.label);
                        m.put('utterance', c.utterance);
                        chipList.add(m);
                    }
                }
                Map<String, Object> payload = new Map<String, Object>{
                    'chips' => chipList
                };
                res.chipMenu = new ChipMenuData(JSON.serialize(payload));
                res.success = true;
            } catch (Exception e) {
                res.success = false;
                res.errorMessage = e.getMessage();
                res.chipMenu = new ChipMenuData('{"error":"' + e.getMessage().replace('"', '\\"') + '"}');
            }
            responses.add(res);
        }
        return responses;
    }
}
```

**Pattern notes:**
- `@InvocableVariable` goes on the `Request`/`Response` fields (not the DTO).
- Errors are caught and packaged back into a `ChipMenuData` with an `error` key so the LWC can surface them.

---

## 3. LightningTypeBundle

### `ChipMenuOutput.lightningTypeBundle-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningTypeBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>63.0</apiVersion>
    <description>Clickable chip menu renderer for Agentforce chat.</description>
    <isExposed>true</isExposed>
    <masterLabel>Chip Menu Output</masterLabel>
</LightningTypeBundle>
```

### `schema.json`

```json
{
  "title": "Chip Menu Output",
  "description": "Renders a welcome message and clickable chips in Agentforce chat.",
  "lightning:type": "@apexClassType/c__ChipMenuData"
}
```

> **Critical:** The `c__` prefix on the Apex class reference is required. The unprefixed form deploys cleanly but breaks `sf agent preview --use-live-actions` with HTTP 400 and becomes **immutable** once referenced by an active BotVersion.

### `lightningDesktopGenAi/renderer.json`

```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/chipMenuRenderer"
      }
    }
  }
}
```

**Rules:**
- Only `lightningDesktopGenAi/renderer.json` — do **NOT** create `enhancedWebChat/renderer.json`.
- Only `definition` in the renderer — **no `attributes` block** (fails with `unevaluatedProperties`).

---

## 4. LWC Bundle

### `chipMenuRenderer.js-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>63.0</apiVersion>
    <isExposed>true</isExposed>
    <masterLabel>Chip Menu Renderer</masterLabel>
    <description>Renders a welcome message and clickable chip buttons in Agentforce chat.</description>
    <targets>
        <target>lightning__AgentforceOutput</target>
        <target>lightning__AgentforceInput</target>
    </targets>
</LightningComponentBundle>
```

### `chipMenuRenderer.html`

```html
<template>
    <!-- Error state -->
    <template if:true={errorMessage}>
        <div class="error-state">{errorMessage}</div>
    </template>

    <!-- Happy path -->
    <template if:true={hasData}>
        <div class="chip-menu-container">
            <!-- Chip buttons -->
            <div class="chip-grid">
                <template for:each={chips} for:item="chip">
                    <button
                        key={chip.label}
                        class="chip-btn"
                        data-utterance={chip.utterance}
                        onclick={handleChipClick}
                    >
                        {chip.label}
                    </button>
                </template>
            </div>
        </div>
    </template>
</template>
```

### `chipMenuRenderer.js`

The renderer reads `this.value.menuJSON`, parses it, and renders chips. Clicks send the chip's `utterance` back to the agent — **targeting the parent window** — with a retry loop because `sendTextMessage` may not be ready immediately.

```javascript
import { LightningElement, api, track } from 'lwc';

export default class ChipMenuRenderer extends LightningElement {

    @api value;
    @track chips = [];
    @track errorMessage = '';

    connectedCallback() {
        console.log('#### ChipMenuRenderer MOUNTED, value:', JSON.stringify(this.value));

        try {
            if (!this.value || !this.value.menuJSON) {
                this.errorMessage = 'No menu data provided.';
                return;
            }

            const raw = this.value.menuJSON;
            const data = typeof raw === 'string' ? JSON.parse(raw) : raw;

            if (data.error) {
                this.errorMessage = data.error;
                return;
            }

            this.chips = (data.chips || []).map((c, i) => ({
                label: c.label,
                utterance: c.utterance,
                key: c.label + '_' + i
            }));

        } catch (e) {
            console.error('ChipMenuRenderer parse error:', e);
            this.errorMessage = 'Error loading chip menu.';
        }
    }

    get hasData() {
        return !this.errorMessage && this.chips.length > 0;
    }

    handleChipClick(event) {
        const utterance = event.currentTarget.dataset.utterance;
        if (!utterance) return;

        this._sendWithRetry(utterance, 0);
    }

    _sendWithRetry(utterance, attempts) {
        const MAX_ATTEMPTS = 10;
        const RETRY_DELAY_MS = 300;

        const parentWindow = window.parent || window;
        const api = parentWindow.embeddedservice_bootstrap &&
                parentWindow.embeddedservice_bootstrap.utilAPI &&
                typeof parentWindow.embeddedservice_bootstrap.utilAPI.sendTextMessage === 'function'
                ? parentWindow.embeddedservice_bootstrap.utilAPI
                : null;

        if (api) {
            api.sendTextMessage(utterance)
                .then(() => {
                    console.log('ChipMenuRenderer: utterance sent:', utterance);
                })
                .catch((e) => {
                    console.error('ChipMenuRenderer: sendTextMessage error:', e);
                });
            return;
        }

        if (attempts < MAX_ATTEMPTS) {
            console.warn(`ChipMenuRenderer: sendTextMessage not ready, retrying (${attempts + 1}/${MAX_ATTEMPTS})...`);
            // eslint-disable-next-line @lwc/lwc/no-async-operation
            setTimeout(() => this._sendWithRetry(utterance, attempts + 1), RETRY_DELAY_MS);
        } else {
            console.error('ChipMenuRenderer: sendTextMessage not available after max retries. Surface may not support it.');
        }
    }
}
```

> **For the Lightning Agentforce Panel (LEX):** Replace the `sendTextMessage` block with the `lightning/accApi` approach:
>
> ```javascript
> import { execute } from 'lightning/accApi';
>
> // Inside handleChipClick:
> await this.copilotConnection.execute({
>     type: 'utterance',
>     value: {
>         utterance: utterance,   // what the agent processes
>         label: displayLabel     // what appears in the chat bubble
>     }
> });
> ```
>
> See the ACC API docs for connection setup.

### `chipMenuRenderer.css`

```css
.chip-menu-container {
   padding: 12px 4px;
}

.chip-grid {
   display: flex;
   flex-direction: column;
   gap: 16px;
}

.chip-btn {
   align-items: center;
   justify-content: center;
   color: #512EAB;
   cursor: pointer;
   transition: background-color 0.15s ease, color 0.15s ease;
   white-space: nowrap;
   border-radius: 99px;
   opacity: 1;
   padding-top: 4px;
   padding-right: 8px;
   padding-bottom: 4px;
   padding-left: 8px;
   background: var(--color--ui-background-light, #FFFFFF);
   border: 1px solid var(--color--ui-border-01, #C1BFFF);
   font-weight: 400;
   line-height: 160%;
   letter-spacing: 0%;
}

.chip-btn:hover {
   background: var(--color--ui-background-brand-04, #EDECF6);
   border: 1px solid var(--color--ui-border-03, #512EAB);
   color: #060609;
}

.chip-btn:active {
   background-color: #EDECF6;
}

.error-state {
   font-size: 13px;
   color: #c23934;
   padding: 8px 0;
}
```

---

## 5. Agent Script (Path B) — `.agent` File Action

```
subagent WelcomeMenu:
    label: "Welcome Menu"
    description: "Shows the welcome chip menu when the user first engages or asks what the agent can do."
    reasoning:
        instructions: ->
            | When this topic is invoked, call the @actions.promptChipsOutput action immediately without asking any clarifying questions.
              The chipMenu output is always renderable, always use show_command.
        actions:
            show_chip_menu: @actions.show_chip_menu
                with welcomeMessage = "How can I help you today?"
                with chips = ...

    actions:
        show_chip_menu:
            description: "Renders a welcome message with clickable option chips."
            label: "Show Chip Menu"
            require_user_confirmation: False
            include_in_progress_indicator: False
            target: "apex://ChipMenuService"
            inputs:
                chips: string
                    description: "JSON array of chip objects with label and utterance fields"
                    label: "Chips"
                    is_required: True
            outputs:
                chipMenu: object
                    description: "Structured payload rendered as the chip menu chat card."
                    label: "Chip Menu"
                    complex_data_type_name: "c__ChipMenuOutput"
                    filter_from_agent: False
                    is_displayable: True
                success: boolean
                    label: "Success"
                    complex_data_type_name: "lightning__booleanType"
                    filter_from_agent: False
                    is_displayable: False
                message: string
                    label: "Message"
                    complex_data_type_name: "lightning__textType"
                    filter_from_agent: False
                    is_displayable: False
```

The `chipMenu` output carries `is_displayable: True` + `filter_from_agent: False` — the combination that triggers ShowCommand and keeps the raw JSON out of the LLM context. If chips intermittently fail to render, make the `reasoning.instructions` more directive.

---

## 6. Permission Set

Create a permission set with `classAccesses` for **both** `ChipMenuData` and `ChipMenuService`, then assign it:

- **Service Agent** → assign to `<botUser>` via `sf org assign permset --on-behalf-of <botUser>`
- **Employee Agent** → assign to end users / Permission Set Group

---

## 7. Deploy Order

Order matters — the DTO is a dependency for both the LightningType and the service, and the LT + LWC must land before the agent publish.

```bash
# 1. DTO class first (dependency for both LT and service)
sf project deploy start -m "ApexClass:ChipMenuData"

# 2. Service class
sf project deploy start -m "ApexClass:ChipMenuService"

# 3. LightningTypeBundle + LWC together (before agent publish)
sf project deploy start \
  -m "LightningTypeBundle:ChipMenuOutput" \
  -m "LightningComponentBundle:chipMenuRenderer"

# 4. Permission set
sf project deploy start -m "PermissionSet:ChipMenu_Access"

# 5. Validate, publish, and activate agent
sf agent validate authoring-bundle --api-name YourAgent
sf agent publish authoring-bundle --api-name YourAgent
sf agent activate --api-name YourAgent --version <N>

# 6. Assign permset to run-as user (agent-type-dependent)
sf org assign permset -n ChipMenu_Access --on-behalf-of <user>

# 7. Re-publish the Embedded Service Deployment (ESD)
```

> **Reminder:** Re-publish the ESD whenever you publish the CLT, or it won't be renderable in the ECv2 deployment.

---

## Quick Troubleshooting Map

| Symptom | Likely cause |
|---------|--------------|
| `--use-live-actions` returns HTTP 400 | Missing `c__` prefix in `schema.json` |
| Deploy fails with `unevaluatedProperties` | `attributes` block present in `renderer.json` |
| Chips render as plain text / raw JSON | ShowCommand didn't fire — strengthen `reasoning.instructions`; verify `is_displayable: True` + `filter_from_agent: False` |
| Nothing renders in Agent Builder Preview | Expected — CLTs only render on live ECv2 / Lightning panel |
| Chip clicks do nothing | `sendTextMessage` not targeting parent window, or wrong surface (LEX needs `accApi`) |
| Renders in old session but not new | ESD not re-published after CLT publish |
| LT change rejected as immutable | An active BotVersion already references it — was deployed without `c__` prefix |
