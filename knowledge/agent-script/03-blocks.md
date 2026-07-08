# Agent Script — Blocks

> **Source:** https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-blocks.html  
> **Scraped:** 2026-04-20

---

## Overview

A script consists of **blocks** where each block contains a set of properties. These properties can describe data or procedures.

```
[system block]
[config block]
[variables block]
[language block]
[connection block]
[start_agent block]   ← The Agent Router (entry point)
[subagent blocks]     ← One per job/topic area
```

---

## 1. System Block

Contains **general instructions for the agent**, including message prompts for specific scenarios.

### Required Messages
- `welcome` — displayed when the agent starts a conversation
- `error` — displayed when an error occurs

```yaml
system:
  messages:
    welcome: |
      Hi {!@variables.userPreferredName}! I'm your personal shopping assistant.
    error: |
      I'm having trouble right now. Please try again in a moment.
  instructions: |
    You are a helpful shopping assistant. Always be friendly and concise.
```

### Notes
- Use the pipe symbol `|` for multiline messages
- To personalize messages, use [linked variables](./08-variables.md#linked-variables): `{!@variables.userPreferredName}`

---

## 2. Config Block

Contains **configuration parameters** that define the agent.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `developer_name` | Yes | Salesforce API name of the agent (max 80 chars). Must start with a letter, alphanumeric + underscores, no trailing/consecutive underscores. Must be unique in org. |
| `default_agent_user` | Conditional | API name or ID of the default Salesforce user running the agent. Required for `AgentforceServiceAgent`, ignored for `AgentforceEmployeeAgent`. |
| `agent_label` | No | Display label shown in the UI. Auto-generated from `developer_name` if not provided. |
| `description` | No | Description of the agent's goals and purpose. |
| `company` | No | Information about your company. |
| `role` | No | The agent's role (e.g., "Help the customer select the perfect gift.") |
| `agent_version` | Auto | Set automatically when you create a new version. |
| `agent_type` | No | `AgentforceServiceAgent` (default) or `AgentforceEmployeeAgent`. Set automatically when creating from a template. |
| `enable_enhanced_event_logs` | No | `True`/`False` — Enable conversation logging for debugging. Default: `False`. |
| `user_locale` | No | User locale setting. |

```yaml
config:
  developer_name: My_Shopping_Agent
  agent_label: My Shopping Agent
  description: Helps customers find and purchase products.
  company: Acme Corp — a leading retailer.
  role: Help the customer select the perfect gift.
  default_agent_user: agent_user@myorg.com
  agent_type: AgentforceServiceAgent
  enable_enhanced_event_logs: True
```

---

## 3. Variables Block

Contains the list of **global variables** that the agent and all subagents can use.

```yaml
variables:
  user_name: mutable string = ""
  is_verified: mutable boolean = False
  order_id: mutable string = ""
  session_id: linked string
```

Reference variables throughout the script using: `@variables.<variable_name>`

See [Variables Reference](./08-variables.md) for full details.

---

## 4. Language Block

Defines which **languages the agent supports**.

```yaml
language:
  - en_US
  - es_MX
```

---

## 5. Connection Block

Describes how the agent **interacts with outside connections** (e.g., Enhanced Chat, Omni-Channel).

```yaml
connection messaging:
  outbound_route_type: omni_flow
  outbound_route_name: My_Escalation_Flow
```

Used alongside the `@utils.escalate` command for transferring conversations to human agents.

See [Salesforce Help: Transfer Conversations from an Agent](https://help.salesforce.com/s/articleView?id=ai.service_agent_escalation.htm)

---

## 6. Subagent Block

The main functional block. Specifies the **instructions, logic, and actions** for a subagent.

### Properties

| Property | Description |
|----------|-------------|
| `subagent name` | Name in `snake_case` that describes the scope/purpose. No spaces allowed. |
| `description` | Helps the agent determine when to use this subagent based on user intent. |
| `system.instructions` | (Optional) Override system-level instructions for this subagent only. |
| `reasoning` | Contains `instructions` and `actions` — information sent to the reasoning engine. |
| `reasoning.instructions` | Guidance for the reasoning engine (combo of logic + prompt instructions). |
| `reasoning.actions` | List of tools the LLM can choose to use (actions + utilities). |
| `actions` | Definitions of actions available from this subagent. |
| `model_config` | (Optional) Customize which LLM model the subagent uses. |

```yaml
subagent order_management:
  description: Helps customers check order status and track deliveries.
  
  reasoning:
    instructions:
      -> run @actions.get_delivery_date
         with order_id = @variables.order_id
      -> set @variables.delivery_date = @outputs.delivery_date
      | Tell the customer their order {!@variables.order_id} is expected on {!@variables.delivery_date}.
    
    actions:
      lookup_order:
        description: Look up order details for the customer.
        with order_id = @variables.order_id
  
  actions:
    get_delivery_date:
      description: Retrieves the delivery date for an order.
      inputs:
        order_id:
          type: string
          required: true
      target: "flow://Get_Delivery_Date"
      outputs:
        delivery_date:
          developer_name: delivery_date
          type: string
```

---

## 7. Start Agent Block (Agent Router)

A **special subagent** using the `start_agent` prefix instead of `subagent`. Called the **"Agent Router"** in Canvas view.

- **Every customer utterance** begins execution at this block
- Used to initiate the conversation and determine which subagent to switch to
- Handles **subagent classification, filtering, and routing**

```yaml
start_agent agent_router:
  description: Routes users to the correct subagent based on their request.
  
  model_config:
    model: EinsteinHyperClassifier  # optional — fast classifier
  
  reasoning:
    instructions:
      -> if @variables.is_verified == False:
        -> transition to @subagent.identity_verification
      | Welcome! I can help with orders, returns, or general questions.
    
    actions:
      go_to_orders:
        description: Route to order management.
        @utils.transition to @subagent.order_management
      
      go_to_returns:
        description: Route to returns and refunds.
        @utils.transition to @subagent.returns
```

### EinsteinHyperClassifier Model

Used for subagent classification in the agent router. Advantages:
- Significantly **faster** subagent classification
- **Increased classification accuracy**, especially for specialized classification constraints and negative instructions

Limitations when using EinsteinHyperClassifier:
- **Cannot** use `before_reasoning` or `after_reasoning`
- **Can only** use the `@utils.transition` tool (no other tools)
- **Can** deterministically call an action

---

## Block Summary

| Block | Keyword | Description |
|-------|---------|-------------|
| System | `system:` | Agent-level instructions and messages |
| Config | `config:` | Agent configuration |
| Variables | `variables:` | Global variable definitions |
| Language | `language:` | Supported languages |
| Connection | `connection messaging:` | External connection config |
| Agent Router | `start_agent <name>:` | Entry point subagent |
| Subagent | `subagent <name>:` | A topic/job the agent can handle |

---

## Official References

- [Agent Script Blocks](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-blocks.html)
- [Subagent Classification and Routing (Salesforce Help)](https://help.salesforce.com/s/articleView?id=ai.agent_topics_routing.htm)
