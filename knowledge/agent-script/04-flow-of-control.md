# Agent Script — Flow of Control

> **Source:** https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-flow.html  
> **Scraped:** 2026-04-20

---

## Overview

Understanding the order of execution and flow of control helps you design better agents. Agentforce has three main execution paths:

1. First request to an agent
2. Processing a subagent
3. Transitioning between subagents

---

## Execution Path 1: First Request to an Agent

All requests — including the first request — begin at the **agent router** (`start_agent` block).

Typical uses of `start_agent`:
- Set initial values of variables
- Perform **subagent classification** (tell the LLM which subagent to choose based on current context)

See [Start Agent Block](./03-blocks.md#7-start-agent-block-agent-router) for details.

---

## Execution Path 2: Processing a Subagent

Agentforce uses a subagent's:
- Text instructions
- Variables
- `if`/`else` conditions
- Other programmatic instructions

...to create an **LLM prompt**. 

### Key Rules
- Reasoning instructions are processed **sequentially, top-to-bottom**
- The LLM only starts reasoning **after** it receives the fully resolved prompt — not while Agentforce is still parsing
- If reasoning instructions contain a **transition command**, Agentforce **immediately transitions** to the specified subagent, discarding any existing resolved prompt

### Step-by-Step: How Agentforce Creates a Prompt

Given:
- Order ID = `1234`
- Current delivery date = `February 10, 2026`
- Package is late
- Agent has entered this subagent twice (`num_turns` = 2)

Processing steps:
1. Initialize prompt to empty
2. Increment global variable `num_turns` from 2 → 3
3. Run action `get_delivery_date`
4. Set variable `updated_delivery_date` = `@outputs.delivery_date`
5. Concatenate: `"Tell the user that the expected delivery date for order 1234 is February 10, 2026."`
6. Run action `check_if_late`
7. Set variable `is_late` = `@outputs.is_late`
8. Check if `@variables.is_late == True`
9. Concatenate: `"Apologize to the customer for the delay."`
10. Process `after_reasoning` instructions (no transition since `num_turns` = 3)
11. Send resolved prompt to LLM → return LLM response to customer

---

## Execution Path 3: Transitioning Between Subagents

Transitions can occur from:
- A **reasoning action** (the LLM chooses)
- **Reasoning instructions** (deterministic, based on conditions)
- **Before/After reasoning blocks**

### Key Rules for Transitions
- Transitions are **one-way**: no return of control to the calling subagent
- Agentforce **discards** any prompt instructions from the previous subagent
- The new subagent is read from **top to bottom** — only its instructions are sent to the LLM
- After the second subagent completes, Agentforce **waits for the next customer utterance**, then returns to `start_agent`

### Transition from a Reasoning Action (LLM-driven)

```yaml
subagent order_management:
  reasoning:
    actions:
      go_to_account_help:
        description: Transition to account help when the user asks about their account.
        @utils.transition to @subagent.account_help
```

### Transition from Reasoning Instructions (Deterministic)

```yaml
subagent verify_identity:
  reasoning:
    instructions:
      -> if @variables.is_verified == True:
        -> transition to @subagent.order_management
      | Please verify your identity to continue.
```

---

## Before Reasoning & After Reasoning

### `before_reasoning`
- Functionally equivalent to adding logic to the **beginning** of a subagent's instructions
- Same syntax as `after_reasoning`

### `after_reasoning`
- Runs **after the reasoning loop exits**, on every request
- Can contain: logic, actions, transitions, directives
- **Cannot** contain the `|` (pipe) command
- **Not executed** if a transition occurs mid-subagent

Typical use cases for `after_reasoning`:
- Set customer-entered information into a variable
- Transition to a different subagent
- Run a cleanup action

```yaml
subagent appointment_booking:
  reasoning:
    instructions:
      | Please let me know what type of appointment you need.
  
  after_reasoning:
    -> if @variables.urgency_level == "high":
      -> set @variables.appointment_duration = 60
    -> else:
      -> set @variables.appointment_duration = 30
    -> if @variables.appointment_confirmed == True:
      -> transition to @subagent.confirmation
```

### Important: Transitions in `after_reasoning`
When calling transitions in `after_reasoning`, use `transition to` (not `@utils.transition to`):

```yaml
after_reasoning:
  -> if @variables.done == True:
    -> transition to @subagent.wrap_up
```

---

## Direct Subagent Reference vs. Transition

| Method | Behavior |
|--------|----------|
| `@subagent.<name>` in reasoning.actions | **Delegates** to another subagent. After the referenced subagent runs, **flow returns** to the original subagent. |
| `@utils.transition to @subagent.<name>` | **One-way** — flow does NOT return to the original subagent. |

Note: If a directly referenced subagent includes a declarative transition, the flow follows that path until it ends, and **then returns** to the original subagent.

---

## Summary Diagram

```
Customer Utterance
       ↓
start_agent (Agent Router)
       ↓
Subagent Classification
       ↓
Selected Subagent
  ├── before_reasoning (optional)
  ├── reasoning.instructions (top-to-bottom)
  │     ├── Logic (deterministic)
  │     ├── Actions (run deterministically)
  │     └── Prompts (build LLM prompt)
  ├── reasoning.actions (LLM tools)
  ├── LLM Reasoning (receives resolved prompt)
  └── after_reasoning (optional, post-reasoning)
       ↓
Response to Customer (or transition to another subagent)
```

---

## Official References

- [Flow of Control](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-flow.html)
- [After Reasoning Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-before-after-reasoning.html)
- [Utils: transition to](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-utils.html#utilstransition-to)
