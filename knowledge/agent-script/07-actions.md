# Agent Script — Actions

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-actions.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-action-chaining.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-fetch-data.html  
> **Scraped:** 2026-04-20

---

## What is an Action?

An **action** defines a **task** that a subagent can perform, such as:
- Calling a **Flow**
- Calling an **Apex class**
- Calling a **Prompt Template**

Actions can store output in variables, make output available to the reasoning engine, and optionally display output to customers.

---

## Key Distinctions: Actions vs. Tools

Agent Script has **two `actions` blocks**:

| Block | Location | Used By | Called |
|-------|----------|---------|--------|
| `subagent.actions` | Inside a subagent, top-level | Logic-based reasoning instructions | Deterministically by you |
| `subagent.reasoning.actions` | Inside `reasoning` | The LLM | Subjectively by the LLM as tools |

**Actions defined in `subagent.actions` must also be referenced in `subagent.reasoning.actions` for the LLM to use them.** Each subagent gets its own copy of any imported action — actions are not shared between subagents.

---

## Action Definition Properties

| Property | Required | Description |
|----------|----------|-------------|
| `action name` | Yes | Identifier in `snake_case` — same naming rules as developer names |
| `description` | No | Helps LLM decide when to call the action. Use `\|` for multiline. |
| `inputs` | No | Input parameters (object with type and required flag) |
| `include_in_progress_indicator` | No | `True`/`False` — Show progress indicator when running the action |
| `target` | Yes | Reference to executable (`apex://`, `flow://`, or `prompt://`) |
| `label` | No | Display name shown to customer. Auto-generated from action name if not specified. |
| `outputs` | No | Output parameters |
| `require_user_confirmation` | No | `True`/`False` — Customer must confirm before the action runs |

---

## Action Naming Rules

Action names must follow Salesforce developer name standards:
- Begin with a letter (not underscore)
- Contain only alphanumeric characters and underscores
- Cannot end with underscore
- Cannot contain consecutive underscores (`__`)
- Maximum length: 80 characters
- **Recommended**: use `snake_case`

---

## Parameter Types

Both input and output parameters can use these types:

| Type | Description |
|------|-------------|
| `string` | Text values |
| `number` | Numeric values (floating point / IEEE 754 double) |
| `integer` | Integer values |
| `long` | Long integer values |
| `boolean` | `True`/`False` values |
| `object` | Complex JSON objects |
| `date` | Date values (YYYY-MM-DD) |
| `datetime` | DateTime values |
| `time` | Time values |
| `currency` | Currency values |
| `id` | Salesforce ID values |
| `list[<type>]` | List of values of same type (e.g., `list[string]`, `list[number]`) |

---

## Action Targets

Use the format `{TARGET_TYPE}://{DEVELOPER_NAME}`:

| Target Type | Format | Example |
|-------------|--------|---------|
| Apex | `apex://ClassName` | `apex://GetOrderStatus` |
| Flow | `flow://Flow_API_Name` | `flow://Get_Delivery_Date` |
| Prompt Template | `prompt://TemplateName` | `prompt://SummarizeOrder` |

---

## Defining an Action

```yaml
subagent order_management:
  actions:
    get_delivery_date:
      description: |
        Retrieves the expected delivery date and late status for a given order.
        Call this action when the user asks about their delivery or tracking status.
      include_in_progress_indicator: True
      inputs:
        order_id:
          type: string
          required: true
          description: The Salesforce order ID to look up.
      target: "flow://Get_Delivery_Date"
      outputs:
        delivery_date:
          developer_name: delivery_date
          type: string
          description: The expected delivery date in YYYY-MM-DD format.
        is_late:
          developer_name: is_late
          type: boolean
          description: Whether the order is running late.
          filter_from_agent: false
        internal_notes:
          developer_name: internal_notes
          type: string
          filter_from_agent: true  # Hide from agent context
```

---

## Output Parameter Properties

| Property | Required | Description |
|----------|----------|-------------|
| `description` | No | Description of the output parameter. Auto-generated from parameter name if not specified. |
| `developer_name` | Yes | String that can override the parameter's developer name. |
| `label` | No | Human-readable label. Auto-generated from name if not specified (e.g., `delivery_date` → "Delivery Date"). |
| `complex_data_type_name` | Conditional | Required if parameter is a complex data type. Indicates the type returned by the target (e.g., `lightning__recordInfoType`). |
| `filter_from_agent` | No | `True` = exclude from agent context; `False` (default) = include in agent context. |

---

## Calling an Action Deterministically

Use `run @actions.<action_name>` in the subagent's `reasoning` block. The action runs every time the subagent runs, before the LLM reasons.

```yaml
reasoning:
  instructions:
    -> run @actions.get_delivery_date
       with order_id = @variables.order_id
    -> set @variables.delivery_date = @outputs.delivery_date
    -> set @variables.is_late = @outputs.is_late
    -> if @variables.is_late == True:
      | I apologize — your order is running late.
      | The new expected delivery date is {!@variables.delivery_date}.
    -> else:
      | Your order is on track to arrive by {!@variables.delivery_date}.
```

---

## Exposing an Action as a Tool (LLM-driven)

Define the action in `reasoning.actions` so the LLM can **choose** when to call it:

```yaml
reasoning:
  instructions:
    | Please provide your order ID so I can look up your delivery status.
  
  actions:
    lookup_order:
      description: |
        Look up the delivery date for an order.
        Use this action when the user asks about their order status or delivery date.
      with order_id = @variables.order_id
```

To explicitly reference the tool in the prompt (stronger signal to LLM):
```yaml
| Use {!@actions.lookup_order} to find the user's order details.
```

---

## Actions and Tools (Both Together)

You can define an action once and use it both deterministically AND expose it as a tool:

```yaml
subagent verify_identity:
  actions:
    send_verification_code_action:
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
  
  reasoning:
    instructions:
      -> if @variables.email is not None:
        -> run @actions.send_verification_code_action
           with email = @variables.email
        -> set @variables.code_sent = @outputs.code_sent
      | Please enter the verification code you received.
    
    actions:
      send_verification_code_tool:
        description: |
          Resend the verification code if the user didn't receive it.
        with email = @variables.email
```

Note: In the UI, action and tool have the same name. In Script view, you can differentiate them.

---

## Action Chaining

### Sequential Actions in Reasoning Instructions

Run actions one after another deterministically — both execute **before** the prompt is sent to the LLM:

```yaml
reasoning:
  instructions:
    -> run @actions.get_order
       with order_id = @variables.order_id
    -> set @variables.order_status = @outputs.status
    -> run @actions.check_return_eligibility
       with order_id = @variables.order_id
       with order_date = @outputs.order_date
    -> set @variables.is_eligible = @outputs.is_eligible
    | Here is your order status: {!@variables.order_status}.
```

### Follow-up Action After an LLM Tool Call

Define a follow-up action that automatically runs when the LLM calls a specific action:

```yaml
reasoning:
  actions:
    get_order_by_number:
      description: Look up order details by order number.
      with order_number = ...
      # Automatically runs check_return_eligibility after this action:
      then: check_eligibility_check
    
    check_eligibility_check:
      description: Check if the order is eligible for return.
      with order_id = @outputs.order_id
```

### Conditional Action Chaining

```yaml
reasoning:
  instructions:
    -> run @actions.check_inventory
       with product_id = @variables.product_id
    -> set @variables.in_stock = @outputs.in_stock
    -> if @variables.in_stock == True:
      -> run @actions.place_order
         with product_id = @variables.product_id
    -> else:
      | Unfortunately, this item is out of stock.
```

### Action → Transition Chaining

```yaml
reasoning:
  actions:
    complete_payment:
      description: Process the payment and move to confirmation.
      with amount = @variables.total
      set @variables.payment_confirmed = @outputs.confirmed
      @utils.transition to @subagent.order_confirmation
```

---

## Fetch Data Before Reasoning

Best practice: Place action calls **at the top** of reasoning instructions to fetch data before the prompt is constructed.

```yaml
subagent order_status:
  reasoning:
    instructions:
      # Step 1: Check if data already fetched
      -> if @variables.order_summary == "":
        # Step 2: Fetch from action
        -> run @actions.get_order_summary
           with customer_id = @variables.customer_id
        # Step 3: Store result
        -> set @variables.order_summary = @outputs.summary
      # Step 4: Reference variable in prompt
      | Here is your recent order summary: {!@variables.order_summary}
      | Let me know if you need help with anything specific.
```

**Key tip**: Always check if data already exists before calling a fetch action to avoid redundant calls:
```yaml
-> if @variables.data == "":
  -> run @actions.fetch_data
```

---

## Action in `after_reasoning` Block

Call actions after the reasoning loop exits:

```yaml
subagent appointment_booking:
  reasoning:
    instructions:
      | What type of appointment do you need?
  
  after_reasoning:
    -> run @actions.log_interaction
       with session_id = @system_variables.session_id
       with user_input = @system_variables.user_input
    -> set @variables.interaction_logged = @outputs.success
```

---

## Calling Action in Canvas View vs. Script View

| View | Behavior |
|------|----------|
| Canvas view | Action/tool naming is unified — no distinction between action name and tool name |
| Script view | You can define `<name>_action` in `subagent.actions` and `<name>_tool` in `reasoning.actions` |

---

## Official References

- [Actions Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-actions.html)
- [Tools (Reasoning Actions) Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-tools.html)
- [Pattern: Action Chaining and Sequencing](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-action-chaining.html)
- [Pattern: Fetch Data Before Reasoning](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-fetch-data.html)
- [Flow of Control](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-flow.html)
