# Agent Script — Subagents

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-blocks.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-tools.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-topic-selector.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-transitions.html  
> **Scraped:** 2026-04-20

---

## What is a Subagent?

A **subagent** (formerly called a "topic" — renamed April 2026) is a **set of instructions, actions, and reasoning** that defines a specific job an agent can do. An agent is composed of multiple subagents, each responsible for a particular task or domain.

> Note: The keyword `topic` still works but is deprecated. Use `subagent` going forward.

---

## Subagent Block Structure

```yaml
subagent <subagent_name>:
  description: |
    A detailed description of what this subagent handles and when it should be used.
  
  system.instructions: |
    (Optional) Override global system instructions for this subagent only.
  
  model_config:
    model: gpt-4o    # Optional: specify which LLM model to use
  
  reasoning:
    instructions:
      -> # Logic instructions (deterministic)
      | # Prompt instructions (sent to LLM)
    
    actions:
      # Tools the LLM can choose to call
  
  actions:
    # Action definitions for this subagent
```

---

## Subagent Properties

| Property | Required | Description |
|----------|----------|-------------|
| `subagent name` | Yes | `snake_case` name — must start with letter, only alphanumeric + underscores, no trailing/consecutive underscores, max 80 chars |
| `description` | Yes | Helps the agent/LLM determine when to use this subagent based on user intent |
| `system.instructions` | No | Override global system instructions for this subagent only |
| `model_config` | No | Specify which LLM model this subagent uses |
| `reasoning` | Yes | Contains `instructions` and `actions` |
| `reasoning.instructions` | Yes | Guidance for the reasoning engine (logic + prompt) |
| `reasoning.actions` | No | Tools the LLM can choose to call |
| `actions` | No | Action definitions used by this subagent |

---

## The Agent Router (start_agent)

The **agent router** is a special subagent defined with `start_agent` prefix instead of `subagent`. It is the entry point for every customer utterance.

```yaml
start_agent agent_router:
  description: Routes users to the correct subagent based on their request.
  
  reasoning:
    instructions:
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      | Welcome! How can I help you today?
    
    actions:
      go_to_orders:
        description: Go to order management for order status and tracking.
        @utils.transition to @subagent.order_management
      
      go_to_returns:
        description: Go to returns when the user wants to return a product.
        @utils.transition to @subagent.returns
        available when: @variables.is_verified == True
      
      go_to_escalation:
        description: Escalate to a human agent.
        @utils.escalate
        available when: @variables.is_verified == True and @variables.business_hours == True
```

### Agent Router Best Practices

- **Limit subagents** — Start with essential subagents; fewer = clearer routing decisions
- **Use `go_to_` prefix** — Name transition actions (e.g., `go_to_orders`) so the agent understands they navigate to other subagents
- **Write detailed descriptions** — Especially if you have similar subagents
- **Hide subagents based on context** — Use `available when` to control visibility
- **Use conditional logic** — Put required transitions at the top of instructions to guarantee routing before any other processing

---

## Subagent Classification and Routing

Every user utterance begins at `start_agent`. The agent router:

1. Welcomes users
2. **Classifies intent** — determines which subagent matches the user's request
3. **Routes** to the appropriate subagent
4. **Controls availability** based on user state (using `available when`)

### Remove Subagent from Agent Router

To make a subagent **only accessible via transitions from other subagents** (not directly from the agent router), simply don't reference it in the `start_agent` reasoning actions:

```yaml
# This subagent won't appear to the agent router — only reachable via explicit transition
subagent internal_processing:
  description: Internal workflow — not user-facing.
  ...
```

---

## Transitioning Between Subagents

### LLM-Driven Transitions (Reasoning Actions / Tools)

Expose transitions as tools that the LLM can **choose** to call:

```yaml
start_agent agent_router:
  reasoning:
    actions:
      go_to_order_management:
        description: Transition to order management for order status, tracking, and delivery questions.
        @utils.transition to @subagent.order_management
      
      go_to_returns:
        description: Transition to returns when the user wants to return or exchange a product.
        @utils.transition to @subagent.returns
        available when: @variables.is_verified == True
```

### Deterministic Transitions (Logic Instructions)

Force transitions based on conditions — the transition happens **before the LLM processes any other instructions**:

```yaml
subagent order_management:
  reasoning:
    instructions:
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      -> if @variables.order_id is None:
        -> transition to @subagent.order_lookup
      | Let me check on that order for you.
```

### Chaining a Transition After an Action

```yaml
reasoning:
  actions:
    complete_verification:
      description: Complete identity verification and move to order management.
      with user_id = @variables.user_id
      set @variables.is_verified = @outputs.verified
      @utils.transition to @subagent.order_management
```

### Direct Subagent Reference (Returns to Caller)

A direct `@subagent.<name>` reference in `reasoning.actions` **delegates** to another subagent and then **returns** to the original:

```yaml
reasoning:
  actions:
    consult_specialist:
      description: Consult the specialist subagent and return here.
      consult: @subagent.specialist
```

This is different from `@utils.transition to`, which is one-way (no return).

---

## `available when` — Filtering Subagents

Control which subagents are visible to the LLM based on variable state:

```yaml
start_agent agent_router:
  reasoning:
    actions:
      go_to_general_info:
        description: Answer general questions. Available to all users.
        @utils.transition to @subagent.general_info
      
      go_to_order_management:
        description: Help with order status and tracking.
        @utils.transition to @subagent.order_management
        available when: @variables.is_verified == True
      
      go_to_escalation:
        description: Escalate to human support.
        @utils.escalate
        available when: @variables.is_verified == True and @variables.business_hours == True
```

Benefits of `available when`:
- Simplifies LLM decision-making by hiding irrelevant subagents
- **Enforces business rules** — customers can't convince the LLM to use a hidden subagent
- Prevents reasoning errors during complex workflows (prompt noise, context drift)

---

## Model Configuration (Per-Subagent)

Use `model_config` to customize which LLM model a subagent uses:

```yaml
start_agent agent_router:
  model_config:
    model: EinsteinHyperClassifier
  ...
```

If no model is specified, the subagent uses the **default model selected in Setup**.

### EinsteinHyperClassifier

Best for the agent router (classification). Advantages:
- Significantly faster classification
- Increased accuracy for specialized classification

Limitations:
- Cannot use `before_reasoning` or `after_reasoning`
- Can only use `@utils.transition` tool
- Can deterministically call an action

---

## System Instruction Overrides

Override global `system.instructions` for a specific subagent:

```yaml
system:
  instructions: |
    Never suggest alcoholic beverages.

subagent adult_party_planning:
  system.instructions: |
    You are helping plan an exclusive adult party. You may suggest cocktails and wine pairings.
  reasoning:
    instructions:
      | Help the customer plan the perfect adult-only celebration.
```

Priority order:
1. **Subagent-level** `system.instructions` (highest priority — overrides global)
2. **Agent-level** `system.instructions` (fallback)

Use overrides when:
- Agent-level instructions **conflict** with subagent-level reasoning
- Different subagents need **different tones** (casual vs. formal, technical vs. non-technical)

---

## Complete Subagent Example

```yaml
subagent order_management:
  description: |
    Helps customers check order status, track deliveries, and get estimated arrival dates.
    Use this subagent when the user asks about their order.
  
  reasoning:
    instructions:
      -> if @variables.order_id is None:
        -> transition to @subagent.order_lookup
      -> run @actions.get_delivery_date
         with order_id = @variables.order_id
      -> set @variables.delivery_date = @outputs.delivery_date
      -> if @outputs.is_late == True:
        | I apologize — your order {!@variables.order_id} is delayed.
        | The new expected delivery date is {!@variables.delivery_date}.
      -> else:
        | Your order {!@variables.order_id} is on track to arrive by {!@variables.delivery_date}.
    
    actions:
      go_to_returns:
        description: Transition to returns if the user wants to return an item.
        @utils.transition to @subagent.returns
        available when: @variables.is_verified == True
      
      go_to_escalation:
        description: Escalate to a human agent if the customer needs more help.
        @utils.escalate
  
  actions:
    get_delivery_date:
      description: Retrieves the current delivery date and late status for an order.
      inputs:
        order_id:
          type: string
          required: true
      target: "flow://Get_Delivery_Date"
      outputs:
        delivery_date:
          developer_name: delivery_date
          type: string
        is_late:
          developer_name: is_late
          type: boolean
  
  after_reasoning:
    -> if @variables.is_satisfied == True:
      -> transition to @subagent.wrap_up
```

---

## Official References

- [Agent Script Blocks](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-blocks.html)
- [Tools (Reasoning Actions)](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-tools.html)
- [Pattern: Agent Router Strategies](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-topic-selector.html)
- [Pattern: Subagent Transitions](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-transitions.html)
- [Pattern: Filtering with Available When](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-filtering.html)
- [Pattern: System Overrides](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-patterns-system-overrides.html)
- [Subagent Classification and Routing (Salesforce Help)](https://help.salesforce.com/s/articleView?id=ai.agent_topics_routing.htm)
