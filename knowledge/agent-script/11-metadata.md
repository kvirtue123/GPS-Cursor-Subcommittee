# Agent Script — Metadata Types

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-metadata.html  
> **Scraped:** 2026-04-20

---

## Overview

Agents are made of **metadata**, just like any other Salesforce customization. A single agent is a collection of multiple metadata components linked together.

---

## Authoring Bundle (`AiAuthoringBundle`)

The **authoring bundle** is the blueprint of an agent. It contains the Agent Script file.

### Metadata Type: `AiAuthoringBundle`

Similar in structure to `ApexClass`:

| File | Description |
|------|-------------|
| `<bundle-api-name>.bundle-meta.xml` | Standard Metadata API XML file |
| `<bundle-api-name>.agent` | The Agent Script file (`.agent` extension) — defines the agent's predictable, context-aware workflows |

### File Location in DX Project

```
force-app/
  main/
    default/
      aiAuthoringBundles/
        My_Bundle/
          My_Bundle.agent
          My_Bundle.bundle-meta.xml
```

### Versioned Bundles

When you publish an authoring bundle, a **versioned copy** is created in your org:
- Versioned bundles have a version number appended: `My_Bundle_1`
- Their `*-bundle-meta.xml` files contain a `<target>` element
- **Only draft (unversioned) bundles can be published** — publishing a versioned bundle results in an error

---

## Agent Metadata Hierarchy

When you publish an authoring bundle, the Agent Script file is compiled into these metadata types:

```
AiAuthoringBundle (blueprint)
       ↓ publishes to →
Bot
  └── BotVersion
        └── GenAiPlannerBundle (reasoning engine)
              └── GenAiPlugin (subagent — one per topic)
                    └── GenAiFunction (action — one per action)
```

---

## Core Metadata Types

### `Bot`
- **Top-level** representation of an Einstein Bot (chatbot)
- Without a `GenAiPlannerBundle`, it's a basic chatbot
- With a `GenAiPlannerBundle`, it becomes an **Agentforce agent**

**Reference**: [Bot Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_bot.htm)

---

### `BotVersion`
- Represents the **configuration for a specific Einstein Bot version**
- A single Bot can have many BotVersions
- **Only one BotVersion can be active** at a time

**Reference**: [BotVersion Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_botversion.htm)

---

### `GenAiPlannerBundle`
- The **reasoning engine** for an agent
- Uses an LLM and a reasoning strategy to:
  - Break down tasks into smaller parts
  - Find the best actions for each part
  - Execute those actions
- **An agent has a single `GenAiPlannerBundle` component**

**Reference**: [GenAiPlannerBundle Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaiplannerbundle.htm)

---

### `GenAiPlugin`
- Represents a **subagent** — a category of actions related to a particular job the agent can do
- **An agent can have multiple `GenAiPlugin` components**
- Each `GenAiPlugin` maps to a `subagent` block in Agent Script

**Reference**: [GenAiPlugin Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaiplugin.htm)

---

### `GenAiFunction`
- Represents an **agent action**
- **An agent can have multiple `GenAiFunction` components**
- Each `GenAiFunction` maps to an `actions` entry in Agent Script

**Reference**: [GenAiFunction Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaifunction.htm)

---

### `GenAiPromptTemplate` (Additional)
- Represents a **prompt template** that can be used as an action target (`prompt://`)
- Associated with agent, but not part of the core hierarchy

**Reference**: [GenAiPromptTemplate Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaiprompttemplate.htm)

---

## Metadata in Your DX Project

Agent metadata is stored in your package directory:

```
force-app/
  main/
    default/
      aiAuthoringBundles/       ← Authoring bundles (.agent files)
        My_Agent/
          My_Agent.agent
          My_Agent.bundle-meta.xml
      bots/                     ← Bot metadata
        My_Agent/
          My_Agent.bot-meta.xml
          My_Agent.botVersion-meta.xml
      genAiPlannerBundles/      ← Reasoning engine
        My_Agent_Planner/
          My_Agent_Planner.genAiPlannerBundle-meta.xml
      genAiPlugins/             ← Subagents
        Order_Management/
          Order_Management.genAiPlugin-meta.xml
      genAiFunctions/           ← Actions
        Get_Delivery_Date/
          Get_Delivery_Date.genAiFunction-meta.xml
```

---

## Retrieving Agent Metadata

```bash
# Retrieve a specific agent's metadata
sf project retrieve start --metadata "Bot:My_Agent" --target-org agentforce

# Retrieve all authoring bundle versions
sf project retrieve start --metadata "AiAuthoringBundle:My_Bundle*" --target-org agentforce

# Retrieve all agent-related metadata
sf project retrieve start \
  --metadata "Bot,BotVersion,GenAiPlannerBundle,GenAiPlugin,GenAiFunction,AiAuthoringBundle" \
  --target-org agentforce
```

---

## Testing Metadata: `AiEvaluationDefinition`

Test cases for agents are stored as `AiEvaluationDefinition` metadata:

```
force-app/
  main/
    default/
      aiEvaluationDefinitions/
        My_Agent_Test/
          My_Agent_Test.aiEvaluationDefinition-meta.xml
```

The `agent test create` CLI command automatically creates this metadata from a test spec YAML file and retrieves it back to your DX project.

---

## Official References

- [Agent Metadata: A Shallow Dive](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-metadata.html)
- [AiAuthoringBundle Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_aiauthoringbundle.htm)
- [Bot Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_bot.htm)
- [BotVersion Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_botversion.htm)
- [GenAiPlannerBundle Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaiplannerbundle.htm)
- [GenAiPlugin Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaiplugin.htm)
- [GenAiFunction Metadata Type](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_genaifunction.htm)
- [Metadata API Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_intro.htm)
- [Agents Metadata Reference](https://developer.salesforce.com/docs/ai/agentforce/references/agents-metadata-tooling/agents-metadata.html)
