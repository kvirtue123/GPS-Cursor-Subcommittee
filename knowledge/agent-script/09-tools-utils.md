# Agent Script — Tools (Reasoning Actions) and Utils

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-tools.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-utils.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-instructions.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-expressions.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-operators.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-before-after-reasoning.html  
> **Scraped:** 2026-04-20

---

## Tools (Reasoning Actions)

### What are Tools?

**Tools** are executable functions that the **LLM can choose to call**, based on the tool's description and the current context. They are defined in the subagent's `reasoning.actions` block.

Tools can wrap:
- An **action** defined in `subagent.actions`
- A **`@utils` function** (transition, escalate, setVariables)

### Tools vs. Actions (Summary)

| Concept | Location | Caller | Call Timing |
|---------|----------|--------|-------------|
| Subagent actions | `subagent.actions` | You (developer) via logic instructions | Deterministically — when Agentforce parses the subagent |
| Reasoning actions (tools) | `subagent.reasoning.actions` | The LLM | Subjectively — after the LLM receives the resolved prompt |

> In Canvas view, this distinction is handled automatically, but it's important to understand when writing Agent Script directly.

---

### Defining a Tool

```yaml
reasoning:
  actions:
    lookup_order:
      description: |
        Look up order details for the customer. Use this when the user asks
        about their order status, delivery date, or tracking number.
      with order_id = @variables.order_id
      set @variables.order_status = @outputs.status
    
    go_to_returns:
      description: Go to the returns subagent when the customer wants to return or exchange a product.
      @utils.transition to @subagent.returns
    
    escalate_to_human:
      description: Escalate to a human agent when the customer is frustrated or has a complex issue.
      @utils.escalate
```

---

### Making Tools Conditional: `available when`

Control when the LLM can access a tool using `available when`:

```yaml
reasoning:
  actions:
    process_return:
      description: Process a product return.
      with order_id = @variables.order_id
      available when: @variables.is_verified == True and @variables.return_eligible == True
    
    escalate:
      description: Escalate to human support.
      @utils.escalate
      available when: @variables.business_hours == True
```

Benefits:
- Hides the tool from the LLM entirely when conditions aren't met
- Prevents customers from manipulating the LLM to call disallowed tools
- Enforces business rules deterministically

---

### Referencing Tools in Prompts

The LLM normally decides which tool to call based on names and descriptions. You can provide a **stronger signal** by referencing the tool directly in the prompt:

**Weak (relying on LLM to figure it out):**
```yaml
| Please help the customer with their order.
```

**Strong (explicit reference):**
```yaml
| Use {!@actions.capture_order_info} to collect the customer's order details.
| Once captured, use {!@actions.lookup_order} to retrieve their order status.
```

---

### Referencing a Subagent as a Tool

You can reference another subagent directly in `reasoning.actions` using `@subagent.<name>`:

```yaml
reasoning:
  actions:
    consult_specialist:
      description: Consult the billing specialist subagent for complex billing questions.
      consult: @subagent.billing_specialist
```

**Key difference from transitions:**
- Direct `@subagent.<name>` reference: Delegates to the subagent. After it runs, **flow returns** to the original subagent.
- `@utils.transition to @subagent.<name>`: **One-way** — no return to the original.

---

## Utils

Utils are **utility functions** that can be used as tools for common agent patterns.

### `@utils.transition to`

Tells the agent to **move to a different subagent**. Transitions are one-way — execution never returns to the calling subagent.

#### As a Reasoning Action (LLM-driven)

```yaml
reasoning:
  actions:
    go_to_order_management:
      description: Go to order management when the user asks about their order.
      @utils.transition to @subagent.order_management
    
    go_to_returns:
      description: Go to returns when the user wants to return an item.
      @utils.transition to @subagent.returns
      available when: @variables.is_verified == True
```

#### As Logic Instructions (Deterministic)

In reasoning instructions, use `transition to` (without `@utils.`):

```yaml
reasoning:
  instructions:
    -> if @variables.is_verified == False:
      -> transition to @subagent.identity_verification
    -> if @variables.step == "complete":
      -> transition to @subagent.wrap_up
```

#### In `after_reasoning` (also use without `@utils.`)

```yaml
after_reasoning:
  -> if @variables.appointment_confirmed == True:
    -> transition to @subagent.confirmation
```

#### Important Transition Behaviors

- Transitions in reasoning instructions **discard the current prompt** — no instructions from the original subagent are sent to the LLM
- Transitions in `after_reasoning` are executed **after** the reasoning loop exits
- If a transition occurs **mid-subagent**, the original `after_reasoning` block is **NOT run**
- After the second subagent completes, flow **does not return** to the original; instead, the agent waits for the next customer utterance and then returns to `start_agent`

---

### `@utils.setVariables`

Instructs the LLM to **set variable values** based on the customer's utterance (slot filling).

```yaml
reasoning:
  actions:
    capture_appointment_info:
      @utils.setVariables
      description: |
        Capture the appointment type and preferred date from the user's message.
        Set appointment_type and preferred_date variables accordingly.
```

The `...` token in action inputs also instructs the LLM to set a variable:

```yaml
reasoning:
  actions:
    capture_user_info:
      with first_name = ...
      with last_name = ...
      set @variables.first_name = @outputs.first_name
      set @variables.last_name = @outputs.last_name
```

---

### `@utils.escalate`

Tells the agent to **escalate to a human service representative**.

#### Requirements
- An active **Omni-Channel connection** must be configured
- Defined in a `connection messaging` block with `outbound_route_type` and `outbound_route_name`

```yaml
connection messaging:
  outbound_route_type: omni_flow
  outbound_route_name: My_Escalation_Flow
```

#### Usage as a Tool

```yaml
reasoning:
  actions:
    escalate_to_human:
      description: Escalate to a human agent when the customer is frustrated or has a complex issue.
      @utils.escalate
      available when: @variables.business_hours == True
```

> `escalate` is a **reserved keyword** and cannot be used for subagent or action names.

---

## Reasoning Instructions

### What are Reasoning Instructions?

The `reasoning.instructions` block contains instructions that Agentforce **resolves into a prompt** for the LLM. The resolved prompt instructs the LLM to perform the subagent's purpose.

**General rule**: Shorter reasoning instructions result in more accurate and reliable results.

### Two Types of Instructions

| Type | Symbol | Description |
|------|--------|-------------|
| Logic instructions | `->` | Deterministic — run programmatic logic, actions, variable setting, conditional branching |
| Prompt instructions | `\|` | Natural language — passed as text to the LLM |

### Combining Logic and Prompt Instructions

```yaml
reasoning:
  instructions:
    # Logic: deterministically check and fetch data
    -> if @variables.email is not None:
      -> run @actions.send_verification_code
         with email = @variables.email
      -> set @variables.code_sent = @outputs.code_sent
    
    # Prompt: tell the LLM what to say
    | Ask the user to enter the verification code they received.
    | If they say they didn't receive it, use {!@actions.send_verification_code_tool} to resend.
```

### Using Variables in Prompt Instructions

```yaml
| Your preferred name is {!@variables.user_name}.
| The total amount due is ${!@variables.order_total}.
```

### `before_reasoning` and `after_reasoning` Blocks

| Block | When It Runs | Can Use `\|` Pipe? |
|-------|-------------|-------------------|
| `before_reasoning` | Before reasoning instructions (functionally same as adding logic to the top) | No |
| `after_reasoning` | After the reasoning loop exits, on every request | No |

Typical uses for `after_reasoning`:
- Set customer-entered information into a variable
- Transition to a different subagent
- Run cleanup or logging action

```yaml
after_reasoning:
  -> if @variables.urgency == "high":
    -> set @variables.appointment_duration = 60
  -> else:
    -> set @variables.appointment_duration = 30
  -> if @variables.confirmed == True:
    -> transition to @subagent.confirmation
```

---

## Conditional Expressions

Use `if` and `else` to **deterministically control** what actions take or which prompts to include.

### Syntax

```yaml
-> if <condition>:
  -> # action or logic
-> else:
  -> # action or logic
```

> Note: Agent Script supports `if` and `else` but **not `else if`** after an `if` statement.

### Run an Action Conditionally

```yaml
-> if @variables.order_summary == "":
  -> run @actions.get_order_summary
     with customer_id = @variables.customer_id
  -> set @variables.order_summary = @outputs.summary
```

### Conditional Prompt Inclusion

```yaml
-> if @variables.is_vip == True:
  | Thank you for being a Platinum VIP member, {!@variables.user_name}!
  | You have {!@variables.points_balance} reward points available.
-> else:
  | Thank you for being a Gold member, {!@variables.user_name}!
```

### Conditional Transition

```yaml
-> if @variables.is_verified == False:
  -> transition to @subagent.identity_verification
```

### Conditional Variable Setting

```yaml
-> if @variables.order_count > 5:
  -> set @variables.customer_tier = "premium"
-> else:
  -> set @variables.customer_tier = "standard"
```

---

## Supported Operators

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equal to | `@variables.count == 10` |
| `!=` | Not equal to | `@variables.status != "done"` |
| `<` | Less than | `@variables.age < 18` |
| `<=` | Less than or equal | `@variables.score <= 100` |
| `>` | Greater than | `@variables.count > 0` |
| `>=` | Greater than or equal | `@variables.total >= 50` |
| `is` | Identity check (null check) | `@variables.value is None` |
| `is not` | Negated identity check | `@variables.data is not None` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `and` | Logical AND | `@variables.a and @variables.b` |
| `or` | Logical OR | `@variables.x or @variables.y` |
| `not` | Logical NOT | `not @variables.flag` |

### Arithmetic Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Addition | `@variables.count + 1` |
| `-` | Subtraction | `@variables.total - 5` |

---

## Official References

- [Tools (Reasoning Actions) Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-tools.html)
- [Utils Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-utils.html)
- [Reasoning Instructions Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-instructions.html)
- [After Reasoning Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-before-after-reasoning.html)
- [Conditional Expressions Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-expressions.html)
- [Supported Operators Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-operators.html)
