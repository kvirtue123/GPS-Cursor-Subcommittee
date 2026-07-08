# Agent Script — Language Characteristics

> **Source:** https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-lang.html  
> **Scraped:** 2026-04-20

---

## Overview

Agent Script is a **compiled language** designed by Salesforce specifically to build Agentforce agents. When you save a version of the agent, the script compiles into lower-level metadata used by the reasoning engine.

---

## Hybrid: Deterministic + LLM Reasoning

Agent Script combines deterministic logic with LLM reasoning in a single workflow.

| Symbol | Type | Description |
|--------|------|-------------|
| `->` | Logic instructions | Run **deterministically** every time — used for business rules, actions, variable setting, conditional branching |
| `\|` | Prompt instructions | Natural language sent to the **LLM** — the LLM interprets and decides how to respond |

---

## Declarative + Procedural

Agent Script has elements of both:

- **Declarative**: You declare what you want without specifying exact step-by-step flow. The basic Agent Script Blocks resemble a declarative language.
- **Procedural**: You specify instructions in logical steps. The logic in reasoning instructions resembles a procedural language.

---

## Human-Readable Design

Agent Script is designed to be human-readable so that even non-developers can get a basic understanding of how the agent works.

---

## Syntax: Key/Value Properties

Agent Script is made up of a collection of properties in `key: value` format. Some properties are multiple lines; some contain sub-properties. The `key` is always before the colon `:` and the `value` always after.

Top-level properties are called **blocks** (e.g., `config`, `system`, `variables`, `subagent`).

---

## Whitespace Sensitivity

Agent Script is **whitespace-sensitive**, similar to Python or YAML:
- Indentation indicates structure and relationships between properties
- Indent with **at least 2 spaces or 1 tab** to indicate a value belongs to the previous line's property
- **Choose one indentation method** and use it consistently throughout the entire script
- All lines at the same nesting level must use the same indentation
- **Mixing spaces and tabs will cause parsing errors**

---

## Logic Instructions: `->` Syntax

To specify logic instructions, use the arrow symbol `->` followed by indented instructions:

```
reasoning:
  instructions:
    -> if @variables.verified:
      -> run @actions.get_order
      -> set @variables.order_id = @outputs.order_id
    -> else:
      | Please verify your identity first.
```

---

## Prompt Instructions: `|` Pipe Symbol

Use the pipe symbol `|` for:
- Multiline strings in reasoning instructions, descriptions, and system messages
- Switching to a prompt from logic-based instructions

```
instructions:
  | Help the customer with their order.
  | If the order is delayed, apologize and offer a discount.
```

---

## Resource References with `@`

Access resources (actions, subagents, variables) using the `@` symbol:

| Syntax | Usage |
|--------|-------|
| `@actions.<action_name>` | Reference an action |
| `@subagent.<subagent_name>` | Reference a subagent |
| `@variables.<variable_name>` | Reference a variable |
| `@outputs.<output_name>` | Reference an action output |

---

## Running Actions

- Use `run` command to execute an action
- Use `with` to provide inputs
- Use `set` to store outputs

```
-> run @actions.get_order
   with order_id = @variables.order_id
-> set @variables.status = @outputs.status
```

---

## Referencing Variables in Prompt Text

When referencing a variable from **within prompt text** in reasoning instructions, use brackets:

```
{!@variables.<variable_name>}
```

Example:
```
| Your preferred name is {!@variables.userPreferredName}.
```

---

## Exposing Subagents as Tools

You can specify a subagent as a tool available to the LLM. This allows the LLM to choose when and whether to switch subagents.

---

## Flow Control

Agent Script uses familiar flow control:
- `if` and `else` (no `else if` support)
- Mathematical expressions: `+`, `-`
- Comparison expressions: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Null checks: `is None` and `is not None`
- Logical: `and`, `or`, `not`

---

## Comments

Use the pound `#` symbol for single-line comments:

```
# This is a comment — content after # is ignored
config:
  developer_name: My_Agent  # The API name of the agent
```

---

## Official References

- [Language Characteristics](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-lang.html)
- [Reasoning Instructions Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-instructions.html)
- [Conditional Expressions Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-expressions.html)
- [Supported Operators Reference](https://developer.salesforce.com/docs/ai/agentforce/guide/ascript-ref-operators.html)
