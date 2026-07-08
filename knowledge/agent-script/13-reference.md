# Agent Script — Quick Reference

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-reference.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-lang.html  
> **Scraped:** 2026-04-20

---

> **Note (April 2026)**: Agent **topics** are now called **subagents**. The keyword `topic` still works but is deprecated. Use `subagent` going forward.

---

## Symbol Quick Reference

| Symbol | Description | Example |
|--------|-------------|---------|
| `#` | Single-line comment | `# This is a comment` |
| `...` | Slot-fill token — instructs LLM to set the value | `with order_id = ...` |
| `->` | Begins logic instructions (deterministic) | `-> if @variables.verified:` |
| `\|` | Begins prompt instructions (sent to LLM) | `\| Help the customer with their order.` |
| `{!expression}` | Resolve a variable or resource in prompt text | `{!@variables.promotion_product}` |
| `==`, `!=`, `<`, `>`, etc. | Comparison operators | `@variables.count > 0` |
| `@actions.name` | Reference an action | `run @actions.get_order` |
| `@outputs.name` | Reference an action's output | `set @variables.status = @outputs.status` |
| `@subagent.name` | Delegate to another subagent | `consult: @subagent.specialist` |
| `@utils.escalate` | Escalate to human service rep | `escalate: @utils.escalate` |
| `@utils.setVariables` | Instruct LLM to set variable values | `set_name: @utils.setVariables` |
| `@utils.transition to` | Transition to a different subagent | `@utils.transition to @subagent.Order_Management` |
| `@variables.name` | Reference a variable from logic instructions | `@variables.order_id` |
| `@system_variables.user_input` | The customer's most recent utterance | Used as action input |

---

## Keyword Quick Reference

| Keyword | Description | More Info |
|---------|-------------|-----------|
| `actions` | Define agent actions or tools available from a subagent | [Actions](./07-actions.md), [Tools](./09-tools-utils.md) |
| `after_reasoning` | Run logic after the reasoning loop exits | [Flow of Control](./04-flow-of-control.md) |
| `available when` | Conditionally show/hide a tool to the LLM | [Tools](./09-tools-utils.md) |
| `config` | Top-level block for agent configuration | [Blocks](./03-blocks.md) |
| `connection` | Top-level block for external connections (e.g., Enhanced Chat) | [Blocks](./03-blocks.md) |
| `if` / `else` | Conditional branching | [Conditionals](./09-tools-utils.md) |
| `instructions` | Guidance for the LLM within system or reasoning blocks | [Reasoning Instructions](./09-tools-utils.md) |
| `language` | Top-level block for supported languages | [Blocks](./03-blocks.md) |
| `linked` | Declare a variable whose value comes from an external source | [Variables](./08-variables.md) |
| `messages` | System messages like welcome and error prompts | [Blocks](./03-blocks.md) |
| `model_config` | Configure the LLM model for a subagent | [Subagents](./06-subagents.md) |
| `mutable` | Allow a variable's value to be changed | [Variables](./08-variables.md) |
| `reasoning` | Block containing instructions and tools for the LLM | [Flow of Control](./04-flow-of-control.md) |
| `reasoning.actions` | Tools the LLM can choose to call within a subagent | [Tools](./09-tools-utils.md) |
| `reasoning.instructions` | Prompt and logic instructions sent to the reasoning engine | [Reasoning Instructions](./09-tools-utils.md) |
| `run` | Execute an action deterministically | `run @actions.get_order` |
| `set` | Store a value in a variable | `set @variables.status = @outputs.status` |
| `start_agent` | Entry point block (Agent Router) for subagent classification and routing | [Subagents](./06-subagents.md) |
| `system` | Top-level block for agent instructions and messages | [Blocks](./03-blocks.md) |
| `system.instructions` | Override system instructions for a specific subagent | [Patterns](./10-patterns.md) |
| `target` | The flow, apex, or prompt target for an agent action | `target: "flow://Get_Order"` |
| `subagent` | Top-level block defining a subagent's instructions and actions | [Subagents](./06-subagents.md) |
| `topic` | Deprecated — use `subagent` instead | — |
| `transition to` | Move to a different subagent from logic instructions | [Utils](./09-tools-utils.md) |
| `variables` | Top-level block for global agent variables | [Variables](./08-variables.md) |
| `with` | Bind an input parameter to an action | `with order_id = @variables.order_id` |

---

## Complete Symbol Cheat Sheet

### Resource References

```
@actions.<name>             Reference a subagent action
@outputs.<name>             Reference an action's output
@subagent.<name>            Reference a subagent
@variables.<name>           Reference a variable (in logic)
@system_variables.user_input  Last customer utterance
@utils.transition to        Transition to another subagent
@utils.setVariables         LLM-driven variable slot filling
@utils.escalate             Escalate to human agent
```

### Prompt Text References (Curly Braces Required)

```
{!@variables.<name>}        Variable value inline in prompt text
{!@actions.<name>}          Action reference inline in prompt text
{!@subagents.<name>}        Subagent reference inline in prompt text
```

### Logic Instructions Syntax

```
->                          Start logic instruction
-> run @actions.name        Run an action deterministically
   with input = value       Provide action input
-> set @variables.x = y     Store a value in a variable
-> if <condition>:          Conditional branch
-> else:                    Else branch
-> transition to @subagent.name  Move to subagent (in logic)
```

### Prompt Instructions Syntax

```
|                           Start prompt instruction (multiline string)
| Text with {!@variables.x} Prompt text with variable interpolation
```

### Variable Declarations

```
name: string = "default"          Immutable string
name: mutable string = ""         Mutable string
name: mutable boolean = False     Mutable boolean
name: mutable number = 0.0        Mutable number
name: mutable list[string] = []   Mutable list
name: linked string               Linked (read-only, from external source)
```

### Action Target Formats

```
target: "flow://Flow_API_Name"     Salesforce Flow
target: "apex://ClassName"         Apex class
target: "prompt://TemplateName"    Prompt template
```

---

## Supported Parameter Types

| Type | Description | Example Default |
|------|-------------|-----------------|
| `string` | Text | `= ""` |
| `number` | Float (IEEE 754 double) | `= 0.0` |
| `integer` | Integer | `= 0` |
| `long` | Long integer | `= 0` |
| `boolean` | True/False (case-sensitive) | `= False` |
| `object` | JSON object | `= {}` |
| `date` | Date (YYYY-MM-DD) | `= 2025-01-01` |
| `datetime` | DateTime | |
| `time` | Time | |
| `currency` | Currency | |
| `id` | Salesforce record ID | |
| `list[type]` | List of typed values | `= []` |

---

## Supported Operators

### Comparison
| Operator | Description |
|----------|-------------|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |
| `is` | Identity / null check (`is None`) |
| `is not` | Negated identity check (`is not None`) |

### Logical
| Operator | Description |
|----------|-------------|
| `and` | Logical AND |
| `or` | Logical OR |
| `not` | Logical NOT |

### Arithmetic
| Operator | Description |
|----------|-------------|
| `+` | Addition |
| `-` | Subtraction |

---

## Agent Script File Skeleton

```yaml
# Agent-level system configuration
system:
  messages:
    welcome: |
      Hi! I'm your assistant. How can I help?
    error: |
      Something went wrong. Please try again.
  instructions: |
    You are a helpful and professional assistant.

# Agent configuration
config:
  developer_name: My_Agent
  agent_label: My Agent
  description: Helps customers with orders and support.
  company: Acme Corp
  role: Help customers resolve their issues efficiently.
  default_agent_user: agent_user@myorg.com
  agent_type: AgentforceServiceAgent
  enable_enhanced_event_logs: False

# Global variables
variables:
  is_verified: mutable boolean = False
  user_name: mutable string = ""
  order_id: mutable string = ""
  session_id: linked string

# Language support
language:
  - en_US

# Agent router (entry point)
start_agent agent_router:
  description: Routes users to the correct subagent based on their request.
  reasoning:
    instructions:
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      | Welcome, {!@variables.user_name}! How can I help you today?
    actions:
      go_to_orders:
        description: Go to order management for order status and tracking.
        @utils.transition to @subagent.order_management
        available when: @variables.is_verified == True
      escalate:
        description: Escalate to human support.
        @utils.escalate
        available when: @variables.is_verified == True

# Subagent definition
subagent identity_verification:
  description: Verifies the user's identity before allowing access to sensitive information.
  reasoning:
    instructions:
      -> run @actions.send_code
         with email = @variables.user_email
      -> set @variables.code_sent = @outputs.code_sent
      | Please enter the verification code we sent to your email.
    actions:
      verify_code:
        description: Verify the code the user entered.
        with code = ...
        set @variables.is_verified = @outputs.verified
  
  actions:
    send_code:
      description: Sends a verification code to the user's email.
      inputs:
        email:
          type: string
          required: true
      target: "apex://SendVerificationCode"
      outputs:
        code_sent:
          developer_name: code_sent
          type: boolean
  
  after_reasoning:
    -> if @variables.is_verified == True:
      -> transition to @subagent.order_management

subagent order_management:
  description: Helps customers check order status and delivery information.
  reasoning:
    instructions:
      -> if @variables.order_id == "":
        | Please provide your order number to get started.
      -> else:
        -> run @actions.get_order_status
           with order_id = @variables.order_id
        -> set @variables.order_status = @outputs.status
        -> set @variables.delivery_date = @outputs.delivery_date
        | Your order #{!@variables.order_id} is currently {!@variables.order_status}.
        | Expected delivery: {!@variables.delivery_date}.
    actions:
      go_to_returns:
        description: Go to returns if the user wants to return their order.
        @utils.transition to @subagent.returns
        available when: @variables.is_verified == True
  
  actions:
    get_order_status:
      description: Gets the current status and delivery date for an order.
      inputs:
        order_id:
          type: string
          required: true
      target: "flow://Get_Order_Status"
      outputs:
        status:
          developer_name: status
          type: string
        delivery_date:
          developer_name: delivery_date
          type: string
```

---

## Official Reference Links

- [Agent Script Reference (Salesforce)](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-reference.html)
- [Language Characteristics](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-lang.html)
- [Agent Script Recipes](https://developer.salesforce.com/sample-apps/agent-script-recipes)
- [GitHub: Agent Script Recipes](https://github.com/trailheadapps/agent-script-recipes)
- [Salesforce CLI: agent Commands](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_agent_commands_unified.htm)
