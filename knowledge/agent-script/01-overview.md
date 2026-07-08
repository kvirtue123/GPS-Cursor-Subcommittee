# Agentforce Agent Script — Overview

> **Source:** https://developer.salesforce.com/docs/ai/agentforce/guide/agent-script.html  
> **Scraped:** 2026-04-20

---

## What is Agent Script?

Agent Script is the **language for building agents in Agentforce Builder**. It combines the flexibility of natural language instructions for handling conversational tasks with the reliability of programmatic expressions for handling business rules.

Beginning in **April 2026**, agent **topics** are now called **subagents**. There are no changes to functionality. During this transition, you may see a mix of the new and previous terms in documentation.

---

## Core Purpose

Agent Script gives you the ability to:

- Define **if/else conditions**, transitions, and other logic using expressions
- **Set, modify, and compare variables**
- **Select subagents and actions**
- Build **predictable, context-aware workflows** that don't rely solely on LLM interpretation
- Control when the agent transitions from one subagent to another
- Run actions in a particular sequence (called **action chaining**)

---

## Three Ways to Write Agent Script

### 1. Chat with Agentforce (No-Code)
Explain what you want your agent to do in plain language (e.g., "If the order total is over $100, then offer free shipping."). Agentforce converts your request into subagents, actions, instructions, and expressions.

### 2. Canvas View (Low-Code)
Agent Script is summarized into easily understandable blocks. You can:
- Expand blocks to view the underlying script
- Use quick action shortcuts
- Type `/` to add expressions for common patterns (e.g., if/else conditionals)
- Type `@` to add resources (subagents, actions, variables)

### 3. Script View (Pro-Code)
Advanced users can switch to Script view to write and edit script directly with:
- Syntax highlighting
- Autocompletion
- Validation

---

## Agentforce DX (Developer Tools)

Developers can use **Agentforce DX** to:
- Generate or retrieve a script file into a local Salesforce DX project
- Work with the `.agent` file in Visual Studio Code
- Use the **Agentforce DX VS Code Extension** for full Agent Script language support with standard code editing features

See [Agentforce DX docs](./05-agentforce-dx.md) for more details.

---

## What Agent Script Enables

| Feature | Description |
|---------|-------------|
| Reasoning Instructions | Define specific areas where the LLM is free to reason OR must execute deterministically |
| Variables | Reliably store information about agent state, rather than relying on LLM context memory |
| Conditional Expressions | Determine agent execution path or LLM utterances based on variable values |
| Subagent Transitions | Deterministically transition to a new subagent, or expose transitions to the LLM as a tool |

---

## Key Documentation Links

| Topic | Link |
|-------|------|
| Language Characteristics | [02-language-characteristics.md](./02-language-characteristics.md) |
| Agent Script Blocks | [03-blocks.md](./03-blocks.md) |
| Flow of Control | [04-flow-of-control.md](./04-flow-of-control.md) |
| Subagents | [06-subagents.md](./06-subagents.md) |
| Actions | [07-actions.md](./07-actions.md) |
| Variables | [08-variables.md](./08-variables.md) |
| Tools & Utils | [09-tools-utils.md](./09-tools-utils.md) |
| Patterns | [10-patterns.md](./10-patterns.md) |
| Agentforce DX (CLI, VS Code, SFDX) | [05-agentforce-dx.md](./05-agentforce-dx.md) |
| Metadata | [11-metadata.md](./11-metadata.md) |
| Agent API | [12-agent-api.md](./12-agent-api.md) |
| Reference (symbols, keywords) | [13-reference.md](./13-reference.md) |

---

## Official Resources

- [Agent Script Overview](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-script.html)
- [Agent Script Recipes (Sample Apps)](https://developer.salesforce.com/sample-apps/agent-script-recipes)
- [GitHub: Agent Script Recipes](https://github.com/trailheadapps/agent-script-recipes)
- [Agentforce Builder Help](https://help.salesforce.com/s/articleView?id=ai.agent_builder_intro.htm)
