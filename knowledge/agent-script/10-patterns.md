# Agent Script — Common Patterns

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-action-chaining.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-topic-selector.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-conditionals.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-fetch-data.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-filtering.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-required-flow.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-resource-references.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-system-overrides.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-transitions.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-variables.html  
> **Scraped:** 2026-04-20

---

## All Patterns Overview

| Pattern | Description |
|---------|-------------|
| [Action Chaining & Sequencing](#1-action-chaining--sequencing) | Run multiple actions in a guaranteed sequence |
| [Agent Router](#2-agent-router) | Set up effective subagent routing in the start_agent block |
| [Conditionals](#3-conditionals) | Use if/else logic to control instructions, actions, and transitions |
| [Fetch Data Before Reasoning](#4-fetch-data-before-reasoning) | Run actions to retrieve data before the LLM begins reasoning |
| [Filtering with Available When](#5-filtering-with-available-when) | Control when subagents and actions are visible to the reasoning engine |
| [Required Subagent Workflow](#6-required-subagent-workflow) | Guarantee users pass through required steps before proceeding |
| [Resource References](#7-resource-references) | Reference variables and actions directly in reasoning instructions |
| [System Overrides](#8-system-overrides) | Override global system instructions per subagent |
| [Transitions](#9-transitions) | Move execution between subagents |
| [Variables](#10-variables) | Store and use state effectively across subagents |

---

## General Principles

- **Start simple** — Use the fewest instructions necessary. Add complexity based on testing.
- **Use good names and descriptions** — Clear, distinct, specific names help the agent make better decisions.
- **Add determinism strategically** — Balance LLM reasoning with programmatic logic for business-critical workflows.
- **Reference resources directly** — Use `@` mentions to give the LLM explicit guidance toward resources.
- **Use plain language** — Not technical terms — so the LLM matches user requests correctly.

---

## 1. Action Chaining & Sequencing

**Goal**: Run multiple actions in a guaranteed sequence.

### Pattern A: Sequential Actions in Reasoning Instructions

Both actions execute **deterministically** before the prompt is sent to the LLM.

```yaml
subagent order_return:
  reasoning:
    instructions:
      # Step 1: Get the order
      -> run @actions.get_order
         with order_id = @variables.order_id
      -> set @variables.order_date = @outputs.order_date
      -> set @variables.order_status = @outputs.status
      
      # Step 2: Check return eligibility using data from Step 1
      -> run @actions.check_return_eligibility
         with order_id = @variables.order_id
         with order_date = @variables.order_date
      -> set @variables.is_eligible = @outputs.is_eligible
      
      # Prompt based on results
      -> if @variables.is_eligible == True:
        | Your order is eligible for return. Would you like to proceed?
      -> else:
        | Unfortunately, your order is no longer within the return window.
```

### Pattern B: Follow-up Action After LLM Tool Call

Define an action that runs **automatically** after the LLM calls a specific tool:

```yaml
reasoning:
  actions:
    get_order_by_number:
      description: Look up an order by its order number.
      with order_number = ...
      then: check_return_eligibility  # Auto-runs after get_order_by_number
    
    check_return_eligibility:
      description: Check if the order is eligible for return.
      with order_id = @outputs.order_id
```

### Pattern C: Conditional Action Chaining

```yaml
reasoning:
  instructions:
    -> run @actions.check_inventory
       with product_id = @variables.product_id
    -> set @variables.in_stock = @outputs.in_stock
    -> if @variables.in_stock == True:
      -> run @actions.place_order
         with product_id = @variables.product_id
      -> set @variables.order_id = @outputs.order_id
      -> transition to @subagent.order_confirmation
    -> else:
      | This item is out of stock. Would you like to be notified when it's available?
```

### Pattern D: Action → Transition

```yaml
reasoning:
  actions:
    complete_payment_and_confirm:
      description: Process payment and transition to order confirmation.
      with amount = @variables.total
      set @variables.payment_confirmed = @outputs.confirmed
      @utils.transition to @subagent.order_confirmation
```

### Best Practices for Action Chaining
- Use **sequential instructions** for deterministic flows — when you always want actions in a specific order
- Use **variables** to pass data between chained actions
- Store action outputs in variables before using them as inputs to the next action

---

## 2. Agent Router

**Goal**: Set up effective subagent routing in the `start_agent` block.

```yaml
start_agent agent_router:
  description: Routes users to the correct subagent based on their request.
  
  reasoning:
    instructions:
      # Critical routing: enforce identity verification first
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      
      | Welcome! How can I help you today?
    
    actions:
      go_to_general_info:
        description: Answer general questions about our products and services. Available to all users.
        @utils.transition to @subagent.general_info
      
      go_to_order_management:
        description: Help with order status, tracking, and delivery. Requires verification.
        @utils.transition to @subagent.order_management
        available when: @variables.is_verified == True
      
      go_to_returns:
        description: Process returns and refunds for recent purchases. Requires verification.
        @utils.transition to @subagent.returns
        available when: @variables.is_verified == True
      
      escalate:
        description: Escalate to human support for complex or sensitive issues.
        @utils.escalate
        available when: @variables.is_verified == True and @variables.business_hours == True
```

### Agent Router Best Practices
- **Limit subagents** — Start with essential ones; add more gradually
- **`go_to_` prefix** — Name transition tools `go_to_<destination>` for clarity
- **Write detailed descriptions** — Especially for similar subagents
- **Hide subagents based on context** — Use `available when`
- **Place conditional transitions first** — Guarantee required routing before any LLM processing

---

## 3. Conditionals

**Goal**: Use if/else logic to control instructions, actions, and transitions deterministically.

### Conditional Prompt Customization

```yaml
reasoning:
  instructions:
    -> if @variables.membership_tier == "vip":
      | Thank you for being a Platinum VIP member, {!@variables.user_name}!
      | You have {!@variables.points_balance} reward points. You're eligible for exclusive perks.
    -> else:
      | Thank you for being a Gold member, {!@variables.user_name}!
      | You have {!@variables.points_balance} reward points.
```

### Conditional Action Execution

```yaml
reasoning:
  instructions:
    # Only look up order if we don't already have one
    -> if @variables.order_summary == "":
      -> run @actions.get_order_summary
         with customer_id = @variables.customer_id
      -> set @variables.order_summary = @outputs.summary
    | Here is your order summary: {!@variables.order_summary}
```

### Conditional Routing

```yaml
reasoning:
  instructions:
    -> if @variables.is_verified == False:
      -> transition to @subagent.identity_verification
    -> if @variables.has_pending_order == True:
      -> transition to @subagent.order_management
    | How can I help you today?
```

### Mutually Exclusive Conditions

```yaml
reasoning:
  instructions:
    -> if @variables.payment_status == "paid":
      -> transition to @subagent.shipping_info
    -> else:
      -> run @actions.process_payment
         with amount = @variables.order_total
      -> set @variables.payment_confirmed = @outputs.confirmed
```

### Combining Conditions

```yaml
-> if @variables.is_verified == True and @variables.is_premium == True:
  | You have access to our premium support team.
```

### Conditional Tips
- **Initialize variables with defaults** — ensures conditional checks work correctly
- **Use `is None` for null checks** — `@variables.value is None` vs. `== ""` (empty string)
- **Keep conditions simple** — avoid complex nested logic; break into separate variables or subagents

---

## 4. Fetch Data Before Reasoning

**Goal**: Run actions to retrieve data before the LLM begins reasoning.

```yaml
subagent customer_greeting:
  reasoning:
    instructions:
      # Check if data already fetched to avoid redundant calls
      -> if @variables.customer_profile == "":
        -> run @actions.get_customer_profile
           with customer_id = @variables.customer_id
        -> set @variables.customer_profile = @outputs.profile
        -> set @variables.recent_orders = @outputs.recent_orders
      
      # Fetch order status if needed
      -> if @variables.order_status == "" and @variables.recent_order_id is not None:
        -> run @actions.get_order_status
           with order_id = @variables.recent_order_id
        -> set @variables.order_status = @outputs.status
      
      # Use fetched data in prompt
      | Hello {!@variables.user_name}! I can see your recent order is {!@variables.order_status}.
      | How can I help you today?
```

**Why this works**: Actions inside reasoning instructions execute **before the prompt is sent to the LLM**. This ensures the LLM has accurate, current data when generating responses.

**Key tip**: Always check if data already exists before fetching to avoid redundant calls.

---

## 5. Filtering with Available When

**Goal**: Control which subagents or actions are available to the LLM using `available when`.

When conditions aren't met, the LLM **cannot see or access** the subagent/action at all.

```yaml
start_agent agent_router:
  reasoning:
    actions:
      go_to_general_info:
        description: Answer general questions. Available to all users.
        @utils.transition to @subagent.general_info
      
      go_to_order_management:
        description: Help with orders. Verified users only.
        @utils.transition to @subagent.order_management
        available when: @variables.is_verified == True
      
      escalate:
        description: Escalate to human support. Requires verification and business hours.
        @utils.escalate
        available when: @variables.is_verified == True and @variables.business_hours == True
```

**Action filtering example**:

```yaml
subagent returns:
  reasoning:
    actions:
      create_return:
        description: Create a return request.
        with order_id = @variables.order_id
        available when: |
          @variables.is_verified == True and
          @variables.return_eligible == True and
          @variables.days_since_purchase <= 30
```

### Filtering vs. Required Flow

| Approach | When to Use |
|----------|-------------|
| `available when` | Control which reasoning actions are available; LLM chooses among available ones |
| Conditional transition | Require users to complete a step; no LLM choice |
| Step variables pattern | Enforce step-by-step sequencing through multiple conversation turns |

---

## 6. Required Subagent Workflow

**Goal**: Guarantee users pass through required steps before accessing other features.

Use **conditional transitions at the top of instructions** — the transition happens immediately, before any LLM processing.

```yaml
start_agent agent_router:
  reasoning:
    instructions:
      # Force unverified users through identity verification first
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      
      # After verification, proceed normally
      | Welcome! How can I help you today?
    
    actions:
      go_to_orders:
        @utils.transition to @subagent.order_management
      go_to_returns:
        @utils.transition to @subagent.returns
```

After identity_verification completes, the next utterance returns to `start_agent`. Because `is_verified` is now `True`, the conditional is satisfied and the user can proceed.

**This pattern works in any subagent, not just the agent router**:

```yaml
subagent returns:
  reasoning:
    instructions:
      # Require payment method before processing return
      -> if @variables.payment_method is None:
        -> transition to @subagent.capture_payment_method
      | Let me process your return.
```

---

## 7. Resource References

**Goal**: Reference subagents, actions, and variables directly in prompt text for stronger LLM guidance.

### Syntax

| Resource Type | In Logic Instructions | In Prompt Text |
|---------------|----------------------|----------------|
| Variable | `@variables.name` | `{!@variables.name}` |
| Action | `@actions.name` | `{!@actions.name}` |
| Subagent | `@subagent.name` | `{!@subagents.name}` |

### Reference Variable in Prompt

```yaml
| Hello {!@variables.user_name}!
| Your order #{!@variables.order_id} is expected to arrive by {!@variables.delivery_date}.
```

### Reference Action in Prompt (Direct Guidance)

```yaml
| Please provide your order number. I'll use {!@actions.lookup_order} to find the details.
```

### Reference Both

```yaml
reasoning:
  instructions:
    -> if @variables.order_id is None:
      | I need your order number to help you. Use {!@actions.capture_order_id} to provide it.
    -> else:
      | Looking up order {!@variables.order_id} using {!@actions.get_order_status}.
```

**Best practice**: Reference resources when the agent has many similar actions, adding a reference helps the agent pick the right one. Direct references are a **stronger signal** to the LLM.

---

## 8. System Overrides

**Goal**: Override agent-level system instructions within specific subagents.

### Priority Hierarchy

1. **Subagent-level** `system.instructions` (highest priority)
2. **Agent-level** `system.instructions` (fallback)

```yaml
# Agent-level system instructions
system:
  instructions: |
    Always maintain a professional, conservative tone.
    Never suggest alcoholic beverages.
    Focus on family-friendly recommendations.

# Subagent that overrides with different instructions
subagent adult_event_planning:
  description: Plans exclusive adult-only events and celebrations.
  
  system.instructions: |
    You are an upscale event planning specialist for adult-only events.
    You may recommend fine wines, cocktails, and premium spirits.
    Maintain an elegant, sophisticated tone.
  
  reasoning:
    instructions:
      | Let's plan an unforgettable adult celebration!
```

### When to Use Overrides

- **Resolving instruction conflicts** — Agent-level instructions contradict subagent-level needs
- **Different tones per subagent** — Casual FAQ vs. formal compliance, billing specialist vs. technical support
- **Different personas** — Technical expert mode vs. general user mode

---

## 9. Transitions

**Goal**: Move execution from one subagent to another.

### LLM-Driven Transitions (Tools)

Expose as reasoning actions so the LLM can choose when to transition:

```yaml
reasoning:
  actions:
    go_to_general_faq:
      description: Go to general FAQ for product or policy questions.
      @utils.transition to @subagent.general_faq
    
    go_to_returns:
      description: Go to returns when the user wants to return or exchange a product.
      @utils.transition to @subagent.returns
      available when: @variables.is_verified == True
```

Reference transitions in prompt for extra guidance:

```yaml
| If the user asks about a return, use {!@actions.go_to_returns} to help them.
```

### Deterministic Transitions (Logic Instructions)

```yaml
reasoning:
  instructions:
    # Transitions in instructions use 'transition to' (not '@utils.')
    -> if @variables.is_verified == False:
      -> transition to @subagent.identity_verification
    -> if @variables.step == 3:
      -> transition to @subagent.confirmation
    | How can I help you today?
```

**Key rule**: When using conditional transitions, **put them at the top** of instructions. If the agent transitions before reasoning, no prompt is sent to the LLM.

### Chaining: Action Then Transition

```yaml
reasoning:
  actions:
    complete_order:
      description: Place the order and move to confirmation.
      with cart_id = @variables.cart_id
      set @variables.order_confirmed = @outputs.confirmed
      @utils.transition to @subagent.order_confirmation
```

### Transition Anti-patterns to Avoid

- **Infinite loops** — Don't create circular transitions (A → B → A → B...)
- **Unnecessary pre-transition actions** — Place transitions before actions unless the actions are needed for the transition decision (pre-transition actions incur cost but results are discarded)

### Transition Best Practices

| Practice | Description |
|----------|-------------|
| Use `go_to_` prefix | Name transition tools `go_to_<destination>` so the LLM understands their purpose |
| Provide clear descriptions | Help the LLM understand when to use each transition |
| Use deterministic transitions sparingly | Only when you need guaranteed routing; otherwise let the LLM choose |
| Place transitions first | Put required conditional transitions at the top of instructions |
| Reference transitions in prompt | When exposing as reasoning actions, reference them: `{!@actions.go_to_escalation}` |

---

## 10. Variables

**Goal**: Store and use state effectively across subagents.

### Initialize with Defaults

```yaml
variables:
  is_verified: mutable boolean = False
  order_id: mutable string = ""
  order_status: mutable string = ""
  customer_tier: mutable string = "standard"
```

### Store Action Outputs

```yaml
reasoning:
  instructions:
    -> run @actions.get_order
       with order_id = @variables.order_id
    -> set @variables.status = @outputs.status
    -> set @variables.delivery_date = @outputs.delivery_date
```

### Share State Between Subagents

```yaml
variables:
  current_temperature: mutable number = 0.0

subagent weather_info:
  reasoning:
    instructions:
      -> run @actions.get_weather
         with location = @variables.user_location
      -> set @variables.current_temperature = @outputs.temperature

subagent activity_recommendations:
  reasoning:
    instructions:
      -> if @variables.current_temperature > 75:
        | Great weather for outdoor activities!
```

### Slot Filling (LLM Sets Variables)

```yaml
reasoning:
  actions:
    capture_user_info:
      description: Collect the user's name and email.
      with first_name = ...
      with last_name = ...
      with email = ...
      set @variables.first_name = @outputs.first_name
      set @variables.last_name = @outputs.last_name
      set @variables.email = @outputs.email
```

### Variable Best Practices

| Practice | Details |
|----------|---------|
| Name clearly | `order_return_eligible` not `flag1` |
| In reasoning instructions, assign variables to action inputs/outputs | Actions run before reasoning — must manually set variables |
| In reasoning actions, use variables as inputs sparingly | Over-specifying can cause inconsistent action selection |
| Store outputs when needed | For conditional expressions, required inputs, or deterministic workflows |

---

## Official References

- [Patterns Overview](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns.html)
- [Action Chaining](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-action-chaining.html)
- [Agent Router](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-topic-selector.html)
- [Conditionals](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-conditionals.html)
- [Fetch Data](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-fetch-data.html)
- [Filtering](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-filtering.html)
- [Required Subagent Workflow](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-required-flow.html)
- [Resource References](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-resource-references.html)
- [System Overrides](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-system-overrides.html)
- [Transitions](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-transitions.html)
- [Variables](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-variables.html)
- [Agent Script Recipes](https://developer.salesforce.com/sample-apps/agent-script-recipes)
