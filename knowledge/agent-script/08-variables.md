# Agent Script — Variables

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-variables.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-variables.html  
> **Scraped:** 2026-04-20

---

## What are Variables?

Variables let agents **deterministically remember information** across conversation turns, track progress, and maintain context throughout a session.

- All variables are defined in the **`variables` block** (top-level)
- All subagents in the agent can **access** the variables

---

## Variable Types

| Type | Description |
|------|-------------|
| **Regular variable** | Can have a default value; agent can change the value if `mutable` |
| **Linked variable** | Value tied to an external source (e.g., action output); no default value |
| **System variable** | Predefined, prepopulated variables (e.g., `@system_variables.user_input`) |

---

## Variable Naming Rules

- Begin with a letter (not underscore)
- Contain only alphanumeric characters and underscores
- Cannot end with underscore
- Cannot contain consecutive underscores (`__`)
- Maximum length: 80 characters

---

## Referencing Variables

| Context | Syntax |
|---------|--------|
| In logic instructions | `@variables.<variable_name>` |
| In prompt instructions (inline in text) | `{!@variables.<variable_name>}` |

---

## 1. Regular Variables

### Properties

| Property | Description |
|----------|-------------|
| `mutable` | Allows the agent to change the variable's value. Variables without `mutable` cannot be changed. |
| `description` | Describes the variable. If you want the LLM to use reasoning to set the value (slot filling), include a description. |
| `label` | Optional display name in the UI. Auto-generated from name if not specified. |

### Supported Types

| Type | Notes | Example |
|------|-------|---------|
| `string` | Any alphanumeric string without special characters | `name: mutable string = "John Doe"` |
| `number` | Integers and decimals — IEEE 754 double-precision float | `price: mutable number = 99.99` |
| `boolean` | `True` or `False` (case-sensitive — capitalize first letter) | `is_active: mutable boolean = True` |
| `object` | Complex JSON object `{"key": "value"}` | `order_line: mutable object = {"SKU": "abc123"}` |
| `date` | Any valid date format | `start_date: mutable date = 2025-01-15` |
| `id` | Salesforce record ID | `record_id: mutable id` |
| `list[type]` | List of values of the specified type. All primitive types supported. | `flags: mutable list[boolean] = [True, False]` |

### Examples

```yaml
variables:
  # Immutable (set once, never changed by agent)
  session_start_time: string = "2026-04-20T09:00:00Z"
  
  # Mutable basic types
  user_name: mutable string = ""
  is_verified: mutable boolean = False
  order_total: mutable number = 0.0
  num_turns: mutable integer = 0
  
  # Mutable with description (enables LLM slot filling)
  appointment_type:
    mutable string = ""
    description: The type of appointment the user wants to schedule (e.g., "in-person", "video").
  
  # Complex object
  customer_info: mutable object = {}
  
  # List types
  product_ids: mutable list[string] = []
  scores: list[number] = [95, 87.5, 92]
```

---

## 2. Linked Variables

A linked variable's value is **tied to a source** (typically an action's output).

### Restrictions
- Cannot have a default value
- Cannot be set by the agent
- Cannot be an object or a list

### Supported Types
- `string`
- `number`
- `boolean`
- `date`
- `id`

### Syntax

```yaml
variables:
  session_id: linked string
  customer_tier: linked string
  account_balance: linked number
```

Linked variables are used when you want to ensure a variable **always reflects the latest value** from an external source.

---

## 3. System Variables

Predefined, prepopulated variables available to all agents.

| Variable | Description |
|----------|-------------|
| `@system_variables.user_input` | The customer's **most recent utterance** (not the entire conversation history) |

### Notes on System Variables
- **Read-only** — cannot be changed
- **Predefined** — not defined in the `variables` block
- Used in the same places as regular or linked variables

### Typical Use Case for `user_input`

Pass the last customer utterance into an action (e.g., sentiment analysis):

```yaml
reasoning:
  instructions:
    -> run @actions.analyze_sentiment
       with text = @system_variables.user_input
    -> set @variables.sentiment_score = @outputs.sentiment
    -> if @variables.sentiment_score < 0.3:
      | I'm sorry to hear you're having a frustrating experience.
```

> The LLM remembers the entire conversation history, so you typically only need `user_input` when passing the last utterance into an action.

---

## Setting Variables

### Set from Action Output

```yaml
-> run @actions.get_order_status
   with order_id = @variables.order_id
-> set @variables.status = @outputs.status
-> set @variables.delivery_date = @outputs.delivery_date
```

### Set Conditionally

```yaml
-> if @variables.is_premium == True:
  -> set @variables.discount_rate = 0.20
-> else:
  -> set @variables.discount_rate = 0.05
```

### Set via LLM Slot Filling

Use `...` token to instruct the LLM to set the variable value based on user input:

```yaml
reasoning:
  actions:
    capture_user_info:
      description: Capture the user's name and email from their message.
      with first_name = ...
      with last_name = ...
      with email = ...
      set @variables.first_name = @outputs.first_name
      set @variables.last_name = @outputs.last_name
      set @variables.email = @outputs.email
```

Or using `@utils.setVariables`:

```yaml
reasoning:
  actions:
    set_appointment_type:
      @utils.setVariables
      description: Set the appointment type based on what the user wants.
```

---

## Sharing Variables Between Subagents

Variables are global — all subagents can read and write them:

```yaml
variables:
  current_temperature: mutable number = 0.0
  verified: mutable boolean = False

subagent weather_info:
  reasoning:
    instructions:
      -> run @actions.Get_Current_Weather_Data
         with location = @variables.user_location
      -> set @variables.current_temperature = @outputs.temperature
      | The current temperature is {!@variables.current_temperature}°F.

subagent activity_recommendations:
  reasoning:
    instructions:
      -> if @variables.current_temperature > 75:
        | Great weather for outdoor activities!
      -> else:
        | It's a bit cool today — here are some indoor options.
```

---

## Variables in Conditionals (`available when`)

Use variables to control which subagents and actions are visible to the LLM:

```yaml
start_agent agent_router:
  reasoning:
    actions:
      go_to_premium_support:
        @utils.transition to @subagent.premium_support
        available when: @variables.is_premium == True and @variables.is_verified == True
```

---

## Best Practices

| Tip | Details |
|-----|---------|
| **Initialize with sensible defaults** | Give variables default values (e.g., `= ""` or `= False`) so conditional checks work correctly |
| **Name variables clearly** | Use descriptive names like `order_return_eligible` not `flag1` |
| **Use `is None` for null checks** | `@variables.value is None` checks for unassigned value; different from `== ""` (empty string check) |
| **Store action outputs when needed** | Store outputs in variables if you need them in conditionals, as required inputs to another action, or for deterministic workflows |
| **Use variables for `available when`** | Conditions to determine which actions and subagents are available |
| **Be careful with mutable inputs** | When specifying variables as reasoning action inputs, use sparingly — too many can cause inconsistent action selection |

---

## Variable Reference in Prompt Text

When embedding variable values in prompt instructions, use curly braces:

```yaml
| Hello {!@variables.user_name}! Your order #{!@variables.order_id} is {!@variables.status}.
```

---

## Official References

- [Variables Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-variables.html)
- [Pattern: Using Variables Effectively](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-variables.html)
- [Utils: setVariables](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-utils.html#utilssetvariables)
