---
feature_area: agentforce-agent-script
status: duplicate-of-catalog
catalog_id: P1
catalog_ref: patterns/canonical/agent-script-library.md
note: "This whole folder (README + 13 chapter files) is already cataloged as P1 (Agent Script knowledge library). Kept here for local browsing — do not re-catalog."
---

# Agentforce Agent Script — Research Documentation

> **Scraped & compiled:** 2026-04-20  
> **Source:** Salesforce Developer Documentation (developer.salesforce.com)

---

## About This Research

This folder contains detailed documentation scraped from Salesforce's official Agentforce Developer Guide, covering **Agent Script** — the language for building Agentforce agents. Includes information on subagents, actions, topics, variables, tools, patterns, Salesforce DX tooling, VS Code integration, CLI commands, metadata types, and the Agent API.

> **April 2026 Naming Change**: Agent **topics** are now called **subagents**. No functionality changes. Both terms may appear in docs during the transition period.

---

## Document Index

| # | File | Topic |
|---|------|-------|
| 01 | [01-overview.md](./01-overview.md) | Overview — what is Agent Script, the three authoring modes, DX tools |
| 02 | [02-language-characteristics.md](./02-language-characteristics.md) | Language syntax, `->` logic vs `\|` prompt, indentation, `@` references |
| 03 | [03-blocks.md](./03-blocks.md) | All block types: system, config, variables, language, connection, start_agent, subagent |
| 04 | [04-flow-of-control.md](./04-flow-of-control.md) | Execution order, how prompts are built, transitions, before/after reasoning |
| 05 | [05-agentforce-dx.md](./05-agentforce-dx.md) | **Salesforce DX, VS Code, CLI** — full developer workflow, setup, generate, validate, publish, preview, test |
| 06 | [06-subagents.md](./06-subagents.md) | Subagents (formerly topics) — structure, agent router, transitions, `available when`, model config, system overrides |
| 07 | [07-actions.md](./07-actions.md) | Actions — defining, calling, action chaining, output handling, targets (apex/flow/prompt) |
| 08 | [08-variables.md](./08-variables.md) | Variables — regular, linked, system, slot filling, sharing between subagents |
| 09 | [09-tools-utils.md](./09-tools-utils.md) | Tools (reasoning actions), utils (transition, setVariables, escalate), reasoning instructions, operators |
| 10 | [10-patterns.md](./10-patterns.md) | All common patterns: action chaining, agent router, conditionals, fetch data, filtering, required flows, transitions, variables |
| 11 | [11-metadata.md](./11-metadata.md) | Metadata types: AiAuthoringBundle, Bot, BotVersion, GenAiPlannerBundle, GenAiPlugin, GenAiFunction |
| 12 | [12-agent-api.md](./12-agent-api.md) | Agent REST API — setup, authentication, sessions, sending messages |
| 13 | [13-reference.md](./13-reference.md) | Quick reference — all symbols, keywords, operators, complete file skeleton |

---

## Key Concepts at a Glance

### Agent Script File Structure
```
.agent file
├── system          → Agent-level instructions & messages
├── config          → Agent name, type, user, description
├── variables       → Global variable definitions
├── language        → Supported languages
├── connection      → External connections (Omni-Channel)
├── start_agent     → Agent Router (entry point for every utterance)
└── subagent(s)     → Each job/topic the agent can handle
      ├── description
      ├── system.instructions  (optional override)
      ├── model_config         (optional LLM selection)
      ├── reasoning
      │     ├── instructions   (-> logic | prompt)
      │     └── actions        (tools the LLM can call)
      ├── actions              (action definitions)
      └── after_reasoning      (post-reasoning logic)
```

### Agentforce DX CLI Commands
```bash
sf agent generate agent-spec         # Generate agent spec YAML
sf agent generate authoring-bundle   # Generate .agent file
sf agent generate test-spec          # Generate test spec YAML
sf agent validate authoring-bundle   # Validate Agent Script
sf agent publish authoring-bundle    # Publish to org
sf agent preview                     # Interactive agent preview
sf agent preview start/send/end      # Programmatic preview
sf agent test create                 # Create test in org
sf agent test run                    # Run tests
sf org create agent-user             # Create default agent user
```

### Action Targets
```
target: "flow://Flow_API_Name"      Salesforce Flow
target: "apex://ClassName"          Apex class
target: "prompt://TemplateName"     Prompt template
```

### Agent Metadata Hierarchy
```
AiAuthoringBundle (.agent file)
  → Bot
    → BotVersion
      → GenAiPlannerBundle (reasoning engine)
        → GenAiPlugin (one per subagent)
          → GenAiFunction (one per action)
```

---

## Official Documentation Home

- **Agent Script Guide**: https://developer.salesforce.com/docs/ai/agentforce/guide/agent-script.html
- **Agentforce DX Guide**: https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx.html
- **Agent API Guide**: https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api.html
- **Agent Script Recipes**: https://developer.salesforce.com/sample-apps/agent-script-recipes
- **CLI Reference**: https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_agent_commands_unified.htm
- **VS Code Extensions**: https://developer.salesforce.com/docs/platform/sfvscode-extensions/guide
