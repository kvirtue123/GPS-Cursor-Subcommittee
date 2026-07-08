---
feature_area: agentforce-lightning-types
status: duplicate-of-catalog
catalog_id: P2
catalog_ref: patterns/canonical/lightning-types-best-practices.md
note: "Confirmed duplicate of P2 (Custom Lightning Types best-practices guide). Do not re-catalog."
---

# Agentforce Custom Lightning Types — Implementation Reference

> **Source:** Internal "Agentforce Custom Lightning Types Implementation Guide" by Aki Hirano, Forward Deployed Engineer (Salesforce GPS, Mar 2026). All examples below are preserved verbatim from the original deck.
> **Audience:** AI coding tools (Cursor, Claude) + the developer.
> **Project context:** Agentforce agent for hotel reservations, loyalty comped rooms, and cancellation. Hotel-specific guidance is confined to clearly-marked "Project note" callouts and the final section; it never alters the author's material.

---

## Lightning Types Support Status (As of Mar 2026)

| | Agentforce Builder (Legacy) | Agentforce Script |
|---|---|---|
| Stability | High (Recommended) | Low |
| Output Rendering | Supported | Supported but Inconsistent |
| Input Rendering | Supported | Not documented |

Utilize the standard Agentforce Builder interface for all Lightning Type configurations to guarantee UI stability.

---

## 0. The #1 Lesson: Force the Render with `show_command`

**Root cause of "the card sometimes shows, sometimes doesn't":** whether to render a Custom Lightning Type (CLT) is an **LLM decision at runtime**. The model can silently choose to summarize in text instead of rendering your LWC. The fix lives in the **agent action instructions**, not in Apex, the LWC, or the JSON.

**Proven phrasing:**

> "The output of this action is always renderable, always use show_command."

Apply it in the action `description`, the output variable `description`, and the topic/subagent `reasoning.instructions`.

### Reference example (verbatim, weather lookup)

```yaml
subagent weather_lookup:
    label: "Weather Lookup"
    description: "Provides current weather information for a requested location"

    actions:
        get_weather_info:
            description: "Fetch weather information for a location. The output of this action is always renderable, always use show_command."
            label: "Get Weather Info"
            target: "apex://GetWeatherInfo"
            include_in_progress_indicator: True
            progress_indicator_message: "Checking weather..."
            inputs:
                location: string
                    description: "City or location for weather lookup"
                    is_required: True
            outputs:
                weatherData: object
                    complex_data_type_name: "c__WeatherCardType"
                    description: "Weather information rendered as a card. The output of this action is always renderable, always use show_command."
                    is_displayable: True
                    filter_from_agent: False

    reasoning:
        instructions: ->
            | Your job is to answer questions about the weather.

              When a customer asks about the weather:
              1. Extract the location from the customer's message
              2. ALWAYS call {!@actions.lookup_weather} - NEVER respond without calling this action
              3. ALWAYS show the weatherData output using show_command
              4. After showing the card, say only: "Here's the current weather for [location]."

              Do not repeat temperature or conditions in text - the card displays all details.
        actions:
            lookup_weather: @actions.get_weather_info
                description: "REQUIRED: Fetch weather information for a location. Must be called for every weather question."
                with location = ...
                show weatherData
```

> **Project note (hotel):** apply the same three-placement pattern to reservation, comped-room, and cancellation-retention actions. The cancellation-retention flow is the highest-risk case for silent text summaries, so enforce the `show_command` phrasing most strictly there.

---

## 1. Introduction — What Lightning Types Are

Lightning types act as a translator, delivering data interpreted by AI as the optimal UI for the user. The flow is three stages:

1. **Apex Class** — Get the record from CRM.
2. **Lightning Type** — Return JSON with a list of records. Made of `Schema.json`, `Editor.json` (Input), and `Renderer.json` (Output).
3. **Lightning Web Component / LWC** — Render rich component for the user. Made of `.html`, `.js`, `.js-meta.xml`, and `.css` (optional).

### Standard vs. Custom Lightning Types

**Standard Lightning Types**
- Data Shape: Fixed to system objects (Case, Lead, etc.)
- UI Control: Standard system-defined layouts only.
- Flexibility: No support for custom carousels or CSS.
- Setup: Out-of-the-box, No-code.
- *Suitable for basic record summaries within Agent chat.*

**Custom Lightning Types**
- Data Shape: Dynamic. Supports Apex Wrapper classes.
- UI Control: Full LWC capability. High-fidelity layouts.
- Flexibility: Supports Carousels, Forms, and complex logic.
- Setup: Requires metadata (CLT) & LWC development.
- *Required for brand-consistent, rich, and interactive UIs.*

### Standard Lightning Types Examples

Automatically selects and displays the standard Lightning type in chat based on the Agent action's data type (text, number, object, etc.). Standard Lightning Types use predefined field sets. For custom field selection, define a Custom Lightning Type.

---

## 2. Agentforce UI Customization Architecture

Based on the rendering type, Agentforce selects a Lightning type and renders the data using the corresponding LWC component.

```
User → Chat Component (Web Site)
     → Embedded Service Deployment
     → Message Channel
     → Agentforce Orchestrator (Agent)
     → Apex → Object
     ← Lightning Types → LWC
     → Response back to Chat Component
```

Components covered in this doc: the LWC, Lightning Types, and Apex.

### Development Process

*(Prerequisite) Prepare the host web site.*
1. Develop an Apex Class
2. Develop Lightning Web Components (LWC)
3. Define Lightning Type definitions
4. Deploy Apex, LWC and LT
5. Configure an Agent
6. Establish a Messaging Channel and Set up Routing
7. Publish and deploy to the Website

---

## 3. Develop an Apex Class

**Purpose:** Implement an Apex class that is invoked by an Agent Action and returns a structured response to the Agent.

### 3.1 The Response Wrapper Pattern

**Best Practice:** Always return `List<Response Class>` for Invocable Actions used by Agents.

- **Structure:** `List<Response Class>` ➔ containing `List<Wrapper Data Class>`
- **Reason (System):** The Invocable framework is designed for bulk processing; the outer List is mandatory to handle execution context.
- **Reason (UI):** Wrapping data in a Response Class ensures the Agentforce Output Renderer can correctly identify the "Data Shape" for your LWC.

```apex
public with sharing class ProductController {
    public class ProductInfo {
        @AuraEnabled public String name;
        @AuraEnabled public String id;
        @AuraEnabled public String description;

        public ProductInfo (Product2 p) {
            this.name = p.name;
            this.id = p.id;
            this.description = p.description;
        }
    }

    @JsonAccess(serializable='always' deserializable='always')
    public class SearchProductResponse {
        @InvocableVariable
        public List<ProductInfo> products;
    }

    @AuraEnabled(cacheable=true)
    @InvocableMethod(label='Find Products' description='Find Available products')
    public static List<SearchProductResponse> getProductList(List<SearchProductRequest> req) {
        // ロジック...
        return new List<SearchProductResponse>();
    }
}
```

### 3.2 Annotation Practices

**`@JsonAccess`**
Controls JSON serialization and deserialization visibility. The root-level Request/Response classes must have this annotation to be accessible from the engine. Without it, serialization fails and the object results in a null value.

**`@AuraEnabled`**
Bridges Apex with LWC / Aura UI. Applied to static methods (to allow LWC to call them) and member variables in data wrapper classes (to allow LWC to access and display the data).

**`@InvocableVariable`**
Exposes variables to the logic engine (Einstein Bots, Agents, and Flows). Applied to member variables inside response classes. This allows Bots/Flows to pass input data to or receive output data from your Apex class.

**`@InvocableMethod`**
Exposes a method as a custom action for Bots, Agents, and Flows. Applied to a Single Static Method. This acts as the entry point for the logic engine.

---

## 4. Develop LWCs

**Purpose:** Develop custom LWC to tailor the visual presentation of Agent outputs, delivering a more intuitive UX within the Enhanced Chat window.

### 4.1 Structure of Lightning Web Components

**Directory Structure**

```
force-app/main/default/lwc/
└── myCustomRenderer/          /* LWC Folder */
    ├── myCustomRenderer.html
    ├── myCustomRenderer.js
    ├── myCustomRenderer.js-meta.xml
    └── myCustomRenderer.css
```

**HTML (Template)** — Defines the structure and layout. It uses standard HTML with data binding `{property}` and directives to control dynamic rendering.

**CSS (Style) *Optional*** — Defines the visual styling. While standard CSS is used, developers leverage SLDS classes to ensure a consistent Look & Feel with the Salesforce platform.

**JavaScript (Logic)** — Handles business logic, event processing, and data fetching. Built on ES6+, it uses `@api` for public properties and `@wire` to interact with Salesforce data.

**XML (Metadata)** — Configures platform behavior. It determines where the component is visible (Record Pages, App Builder, Messaging, Agentforce, etc.) and defines properties accessible to Admins.

### 4.2 Development tips for HTML and JavaScript files

Determine the component layout based on SLDS 2.0 design patterns and implement the solution using code samples from the Lightning Component Reference.

### 4.3 Mastering js-meta.xml configuration

**1. Versioning & Labels**
- **Keep apiVersion at 62.0 or higher.** Newer targets like `lightning__AgentforceOutput` may not be recognized in older versions. Always use 62.0+ for full AI compatibility.
- **Set a clear masterLabel.** This is the display name in the Agent Builder. Use descriptive names like "Product List Renderer" for Admins.

**2. Core Attributes & Descriptions**
- **`isExposed = "true"`** — Mandatory. Without this, Agentforce cannot "see" the component during the rendering phase.
- **Ensure `<target>` includes `lightning__AgentforceOutput`.**
- **Utilize the `<description>` tag** — Define the component's purpose (e.g., "Displays flight status details"). Crucial for multi-developer collaboration.

**3. Consistency in sourceType**
The `js-meta.xml` must be configured differently depending on whether your LWC handles a single record or a collection (list).

**Pattern A: Single Response.** The `name` attribute in `sourceType` must exactly match your Lightning Type API Name.
- Format: `c__yourLightningTypeName`
- Note: This links the LWC to a specific data schema.

```xml
<targetConfig targets="lightning__AgentforceOutput">
    <sourceType name="c__yourLightningTypeName" />
</targetConfig>
```

**Pattern B: Collection Response.** Use `property` with `type="lightning__listType"` instead of `sourceType`.
- Mechanism: This allows the engine to inject an array of data into the LWC's `value` property.
- Note: The specific Lightning Type is mapped via `renderer.json` in the Lightning Type metadata.

```xml
<targetConfig targets="lightning__AgentforceOutput">
    <property name="value" type="lightning__listType" label="value"/>
</targetConfig>
```

---

## 5. Develop Lightning Types

**Purpose:** Map structured data from Agentforce actions to custom LWC rendered, ensuring seamless visual integration within the messaging interface.

> ※ Lightning Types can be configured directly through the Salesforce Setup UI, where you specify the target Apex class.

### 5.1 Schema.json

**Role:** Defines the Data Model and fields for AI reasoning.

**Pattern A: Single Response.** Specify the data wrapper class by appending its name to the Apex class with a '$' separator, prefixed with 'c__'.

```json
{
  "title": "My Object Details Response",
  "description": "My Object Details Response",
  "lightningType": "@apexClassType/c__ApexClass$DataWrapperClass"
}
```

**Pattern B: Collection Response.** Enter the data wrapper class name prefixed with 'c__'.

```json
{
  "title": "My Object Details Response",
  "description": "My Object Details Response",
  "lightningType": "@apexClassType/c__DataWrapperClass"
}
```

### 5.2 Renderer.json

**Role:** Maps the Data model to its LWC visual representation.

**Pattern A: Single Response.** In the 'definition' field, specify the custom component name using the `c/yourLWCName` format.

```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/yourLWCName"
      }
    }
  }
}
```

**Pattern B: Collection Response.** Collections require an extra nested level (`collection`) in the JSON structure compared to single records.

```json
{
  "collection": {
    "renderer": {
      "componentOverrides": {
        "$": {
          "definition": "c/yourLWCName"
        }
      }
    }
  }
}
```

### 5.3 Editor.json

**Role:** Defines the input-only UI, enabling data validation and rich previews during the interaction in the channel. In the 'definition' field, specify the LWC name using the `c/yourLWCName` format.

```json
{
  "editor": {
    "componentOverrides": {
      "$": {
        "definition": "c/yourLWCName"
      }
    }
  }
}
```

---

## 6. Deployment

### Component Deployment via Manifest (package.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>customcomponentNamel</members>
        <name>Lightning Component Bundle</name>
    </types>
    <types>
        <members>customLightningTypeName</members>
        <name>LightningTypeBundle</name>
    </types>
    <types>
        <members>CustomApexClassname</members>
        <name>ApexClass</name>
    </types>
    <version>65.0</version>
</Package>
```

Replace with your custom component names.

---

## 7. Configure an Agent

### Integrate Custom Lightning Type into Agent

Map a Lightning Type to the Agent Action to enable custom LWC visualization.

- In the action's Output → Advanced Settings, set **Data Type** to e.g. `@apexClassType/c__AvailableFlight`.
- Keep **Show in conversation** checked.
- In the **Output Rendering** "Select an Option" dropdown, choose your response renderer (e.g. `FlightResponse`).

**Note on Warning:**
You may see an **"Unsupported Data Type"** message in the *Map to Variable* parameter. This is a known behavior for `@apexClassType` or custom types and **can be safely ignored.**

---

## 8. Additional Configuration

### 8.1 Messaging Channel and Site Deployment

Referring to the relevant material, configure the messaging channel, set up routing to the agent, and then publish and deploy the site.

### 8.2 Tips for Displaying Product Images via CMS

Leverage Salesforce CMS for asset hosting and link the Workspace to your site to ensure secure, high-performance image rendering in Messaging.

1. **Centralize Assets in CMS Workspace**
   - Action: Upload product images to a dedicated CMS Workspace.
2. **Link CMS to your Messaging Site**
   - Action: Enable the CMS workspaces as a content source for your Experience Cloud site.
   - Benefit: This allows the messaging channel to securely pull and display images from the CMS.
3. **Map Image URLs to Product Records**
   - Action: Store the CMS Image URL in the `ImageURL` field of your Product records.
   - Tip: Your Apex action can then fetch this URL to pass to the LWC for instant rendering. Image URLs stored in Product records must be full paths, including the Experience Cloud domain (e.g., `https://[YourDomain]/[sitepath]/sfsites/c/cms/delivery/media/[Content Key]`).
4. **Configure Security (Trusted Sites)**
   - Action: Add your image domain to Trusted Sites for Scripts in your Experience Builder Security setting.
   - Why: This prevents browser-side blocking and ensures images render correctly in the chat window.

---

## 9. Security & Final Configuration

### 9.1 Essential Security Settings

Register your Site Domain URL in these key locations to ensure LWC image rendering.

| Location | Setting | Purpose / Action |
|---|---|---|
| Salesforce Setup | **Trusted URLs** | Prevents browser blocks on CMS images by enforcing Content Security Policy (CSP). |
| Experience Builder | **Trusted Sites for Scripts** | Action: Add Site Domain to security settings. Purpose: Authorizes LWCs to fetch and display internal assets. |
| Salesforce Setup | **CORS Settings** | Enables secure cross-origin resource sharing between the site and Salesforce. |

### 9.2 Reminder: Always Publish!

Changes are not live until you take these final steps.

- **The Publish Button** — Always "Publish" the Embedded Service Deployment Settings after any LWC or LT change to ensure the site cache is cleared.
- **Final Validation** — Test in a Private/Incognito window to simulate a true Guest user experience and confirm all images load correctly.

---

## Appendix — Hotel Project Application Notes

> This appendix is the only project-specific material. It applies the author's patterns above to this project's use cases without modifying any of the original examples.

This project is an Agentforce agent for **hotel reservations, loyalty comped rooms, and cancellation**. Each use case follows the same Apex → Lightning Type → LWC → agent-action chain documented above.

| Use case | Pattern | Notes |
|---|---|---|
| Single reservation lookup | A (single) | One record → `sourceType name="c__..."` + single-form schema/renderer. |
| Loyalty comped-room offers | B (collection) | Multiple options → `property type="lightning__listType"` + collection-wrapped renderer.json. |
| Cancellation retention flow | B (collection) | Carousel of alternatives. Highest risk of the agent describing offers in text instead of rendering — enforce the `show_command` instruction phrasing (Section 0) most strictly here. |

**Reusable instruction snippet** (Section 0) — place in each renderable action's `description`, its output variable's `description`, and the topic instructions:
> "The output of this action is always renderable, always use show_command."

**Pre-ship checklist**
- [ ] Apex root response class has `@JsonAccess(serializable='always' deserializable='always')`.
- [ ] All displayed fields have `@AuraEnabled`; invocable returns `List<ResponseClass>`.
- [ ] `js-meta.xml`: `apiVersion` ≥ 62.0, `isExposed="true"`, target `lightning__AgentforceOutput`.
- [ ] Single → `sourceType name="c__Type"`; Collection → `property type="lightning__listType"`.
- [ ] `schema.json` / `renderer.json` use correct single vs. collection shape (collection adds the `collection` wrapper).
- [ ] Agent action Data Type set; "Unsupported Data Type" warning ignored.
- [ ] `show_command` phrasing present in action desc + output desc + topic instructions.
- [ ] Site domain registered in Trusted URLs, Trusted Sites for Scripts, CORS.
- [ ] Embedded Service Deployment **published**; tested in incognito.
