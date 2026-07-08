# Agentforce DX — Salesforce DX, CLI, and VS Code

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-set-up-env.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-script.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-authbundle.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-publish.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-preview.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-generate-agent-spec.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-test.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-test-spec.html  
> **Scraped:** 2026-04-20

---

## What is Agentforce DX?

**Agentforce DX** extends Salesforce Developer Experience (DX) tools to work with agents. It provides pro-code tools to:

- Create, preview, and test agents **outside** Agentforce Studio
- Move agent metadata between your Salesforce DX project and scratch orgs, sandboxes, and production orgs
- Incorporate agents into a **modern DevOps process**

Agentforce DX can be used:
- At the **command line** (Salesforce CLI)
- With a **VS Code extension** (Agentforce DX Extension)

---

## Iterative Development Workflow

You can switch back and forth between low-code (Agentforce Builder UI) and pro-code (VS Code, CLI) tools:

1. **Author locally** — Generate an authoring bundle in your DX project, code the `.agent` script file in VS Code
2. **Publish to org** — Push authoring bundle to org, open in Agentforce Builder UI
3. **Retrieve changes** — Pull any UI-made changes back to your local DX project
4. **Build new actions** — Create Apex classes that implement custom actions, update the Agent Script file
5. **Deploy to org** — Push local Apex + agent changes
6. **Preview & debug** — Use VS Code preview or `agent preview` CLI command
7. **Commit to VCS** — Check metadata into source control (GitHub, etc.)

---

## Setting Up the Development Environment

### Required Tools

| Tool | Description |
|------|-------------|
| Visual Studio Code | IDE with integrated terminal, code editor, and debugger |
| Salesforce Extension Pack | Bundle of extensions installed together |
| Salesforce CLI | Command-line interface with `agent` commands |

### VS Code Extensions in Salesforce Extension Pack

| Extension | Purpose |
|-----------|---------|
| [Agentforce DX](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode-agents) | Core DX panel, preview pane, publish/validate commands |
| [Agent Script Language Server](https://marketplace.visualstudio.com/items?itemName=salesforce.agent-script-language-client) | Syntax highlighting, autocompletion, validation for `.agent` files |
| [Apex Language Server](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode-apex) | Apex code support |
| [Apex Replay Debugger](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode-apex-replay-debugger) | Debug Apex classes during agent preview |
| [Agentforce Vibes](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-einstein-gpt) | AI-powered vibe coding — generates code from natural language prompts |

### Installation Steps

1. [Download and install VS Code](https://code.visualstudio.com/)
2. Install the [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode) from VS Code Marketplace (or [Open VSX Registry](https://open-vsx.org/extension/salesforce/salesforcedx-vscode))
3. [Install Salesforce CLI](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm)
4. In VS Code integrated terminal, explore agent commands: `sf search` → enter `agent`

### Using Other AI Coding Tools (Cursor, Claude Code, etc.)

You can use non-Salesforce AI tools with Agentforce DX:

1. Install Salesforce CLI on your computer
2. Download these three Agentforce skills and store them for your AI tool:
   - [Agentforce Development Skill](https://github.com/forcedotcom/afv-library/tree/main/skills/developing-agentforce)
   - [Agentforce Testing Skill](https://github.com/forcedotcom/afv-library/blob/main/skills/testing-agentforce/SKILL.md)
   - [Agentforce Observability Skill](https://github.com/forcedotcom/afv-library/blob/main/skills/observing-agentforce/SKILL.md)

---

## Development Org Selection

### Sandboxes vs. Scratch Orgs

| Feature | Sandboxes | Scratch Orgs |
|---------|-----------|--------------|
| Contains production metadata | Yes (copy at creation time) | No |
| Best for | Integration testing, agents with Data 360 | Source-driven development, fast iteration |
| Data 360 support | Yes | Requires manual setup |
| Creation | `org create sandbox` CLI or Setup UI | `org create scratch` CLI |

### Recommendation
Use a **sandbox** (Developer or Developer Pro) when using Agentforce DX, especially if agents require Data 360.

### Enabling Agentforce in Your Org

1. (If required) Enable Data 360 from **Setup → Data Cloud Setup Home**
2. From Setup, find **Einstein Setup** → ensure Einstein is enabled
3. From Setup, find **Agentforce Agents** → ensure Agentforce is enabled

### Required Permissions

| Task | Required Permissions |
|------|---------------------|
| Publish an authoring bundle | `Modify All Data` + `Manage AI Agents` |
| Preview an agent | `Agent Platform Builder` |
| Validate/generate | No special permissions needed |

---

## Creating a Salesforce DX Project

### Using VS Code
1. `View → Command Palette → SFDX: Create Project`
2. Select **Agent** project template (includes sample agent and scratch org config)
3. Enter project name, select location

### Using CLI
```bash
sf project generate --name agentforcedx --template agent
```

The `agent` template generates:
- Sample agent called `Local Info Agent`
- Scratch org config file (`config/project-scratch-def.json`) with required Agentforce settings

---

## Authorizing Your Org

### VS Code
1. `View → Command Palette → SFDX: Authorize an Org`
2. Click **Project Default**
3. Enter an alias (e.g., `agentforce`)
4. Log in via the browser window that opens

### CLI
```bash
sf org login web --alias agentforce --set-default
```

---

## Creating the Default Agent User

The default agent user is the Salesforce user that runs agent actions in your org.

### CLI (Recommended)
```bash
sf org create agent-user --target-org agentforce
```

This command:
- Creates a user called `Agent User` with a globally unique username
- Assigns the `Einstein Agent User` profile
- Assigns permission sets: `AgentforceServiceAgentBase`, `AgentforceServiceAgentUser`, `EinsteinGPTPromptTemplateUser`

Custom name example:
```bash
sf org create agent-user --target-org agentforce --name "Service Agent"
```

---

## Generating an Agent Spec File (Optional)

An agent spec is a YAML file that provides a customized starting point for your authoring bundle.

```bash
sf agent generate agent-spec --target-org agentforce
```

The command prompts for:
- Agent type (customer-facing or internal)
- Agent role (natural language description)
- Company description

Flags for non-interactive use:
```bash
sf agent generate agent-spec \
  --type customer \
  --company-name "Acme Corp" \
  --company-description "A leading global retailer" \
  --tone professional \
  --target-org agentforce
# (will still prompt for --role)
```

Iterate on the spec by passing back the same file:
```bash
sf agent generate agent-spec \
  --spec specs/agentSpec.yaml \
  --role "Help customers find products and resolve order issues" \
  --target-org agentforce
```

Generated file location: `specs/agentSpec.yaml`

### Agent Spec File Structure

```yaml
agentType: customer
companyName: Acme Corp
companyDescription: A leading global retailer
role: Help customers find the perfect product
tone: professional
maxTopics: 5

topics:
  - name: Order_Management
    description: Helps customers track and manage their orders.
  - name: Returns
    description: Assists customers with product returns and refunds.
  - name: Product_Discovery
    description: Helps customers find products that match their needs.
```

---

## Generating an Authoring Bundle

An authoring bundle is the blueprint of an agent, containing the `.agent` script file.

### VS Code
1. Open Command Palette → `AFDX: Create Agent`
   (or right-click `aiAuthoringBundles` directory in explorer)
2. Select template:
   - **Default template** (Recommended) — basic boilerplate Agent Script
   - **From an agent spec YAML file** (Advanced) — more customized
3. Enter agent label and API name

### CLI
```bash
sf agent generate authoring-bundle --target-org agentforce
```

With flags (non-interactive):
```bash
sf agent generate authoring-bundle \
  --name "My Agent" \
  --api-name My_Agent \
  --no-spec \
  --target-org agentforce
```

Or using an agent spec:
```bash
sf agent generate authoring-bundle \
  --spec specs/agentSpec.yaml \
  --name "Shopping Agent" \
  --api-name Shopping_Agent \
  --target-org agentforce
```

### Generated File Structure
```
force-app/
  main/
    default/
      aiAuthoringBundles/
        My_Bundle/
          My_Bundle.agent          ← Agent Script file (edit this)
          My_Bundle.bundle-meta.xml ← Standard Metadata API XML
```

---

## Coding the Agent Script File

### Opening and Editing
1. Open `.agent` file in VS Code — full language support:
   - Syntax highlighting
   - Red squiggles for syntax errors
   - Internal validation
   - VS Code **Outline view** shows symbol tree (blocks, subagents, etc.)

### Vibe Coding with Agentforce Vibes
1. Open Agentforce Vibes panel (click icon in Activity Bar)
2. Go to **Manage Agentforce Rules, Workflows, Hooks & Skills** → **Skills** tab
3. Enable these three skills:
   - `developing-agentforce`
   - `testing-agentforce`
   - `observing-agentforce`
4. Enter a natural language prompt in the chat box

Sample vibe coding prompts:
- `"Change the display name to Local Info Agent."`
- `"Add a new subagent for updating customer information."`
- `"Add an action to identify_customer for identifying a customer based on their cell phone."`
- `"Review all the agent instructions and improve them for focus and conciseness."`

### Agent Script File Location
```
force-app/main/default/aiAuthoringBundles/<API-name>/<API-name>.agent
```

Example:
```
force-app/main/default/aiAuthoringBundles/Local_Info_Agent/Local_Info_Agent.agent
```

---

## Validating an Agent Script File

Validation ensures the script compiles successfully before publishing.

### VS Code
1. Right-click inside the `.agent` file → `AFDX: Validate This Agent`
2. Check **Problems** tab for errors (shows error description + line number)

### CLI
```bash
sf agent validate authoring-bundle --target-org agentforce
# Prompts for bundle API name, or:
sf agent validate authoring-bundle --api-name My_Agent --target-org agentforce
```

---

## Publishing an Authoring Bundle

Publishing creates or updates the agent metadata in your org (`Bot`, `BotVersion`, `GenAi*`).

What happens when you publish:
1. Agent Script file is **validated** (fails if compilation errors)
2. Script is published → generates `Bot`, `BotVersion`, `GenAiPlugin`, `GenAiFunction` metadata
3. New/changed agent metadata is **retrieved** back to your DX project (skippable)
4. `*.bundle-meta.xml` `<target>` element is updated with agent info
5. A new **versioned** authoring bundle is created in your org

**Only draft (unversioned) authoring bundles can be published.**

### VS Code
1. Ensure local Apex/Flow changes are deployed: `SFDX: Deploy This Source to Org`
2. Right-click inside `.agent` file → `AFDX: Publish this Agent`

### CLI
```bash
# Deploy any Apex changes first
sf project deploy start --source-dir force-app/main/default/classes --target-org agentforce

# Publish the authoring bundle
sf agent publish authoring-bundle --target-org agentforce
# With skip-retrieve:
sf agent publish authoring-bundle --skip-retrieve --target-org agentforce

# Retrieve all versions of a bundle
sf project retrieve start --metadata "AiAuthoringBundle:My_Bundle*" --target-org agentforce
```

---

## Previewing and Debugging an Agent

Preview allows you to have an interactive conversation with your agent to test its behavior.

### Preview Modes

| Mode | Description |
|------|-------------|
| **Simulated mode** | Uses only the Agent Script file; mocks all actions (no org data at risk). Useful when Apex/flows aren't yet built. |
| **Live mode** | Uses actual Apex classes, flows, prompt templates in your dev org. Most accurate preview. Requires deployment of local changes. |
| **Published agent** | Always live — uses actual org data. |

### VS Code Agent Preview

1. Open Agentforce DX panel (click icon in Activity Bar)
2. Select agent from **Select agent...** drop-down (Agent Script or Published)
3. Choose mode: **Simulation** or **Live Test**
4. Click **Start Simulation** or **Start Live Test**
5. Chat in the window
6. View **Agent Tracer** tab for JSON messages (request/response details)
7. Save chat history with the **Save Chat History** icon

#### Apex Replay Debugger (Live Mode)
1. Click **Start Debug Mode**
2. Set a breakpoint in your Apex class
3. Start chatting — Apex Replay Debugger starts automatically when the class is invoked

### CLI Preview (Interactive)
```bash
sf agent preview --target-org agentforce
# Live mode:
sf agent preview --use-live-actions --target-org agentforce
# Save transcripts:
sf agent preview --output-dir ./preview-results --target-org agentforce
# Generate Apex debug logs:
sf agent preview --apex-debug --output-dir ./preview-results --target-org agentforce
```

### CLI Preview (Programmatic — for use by other agents or CI)

```bash
# Start session
sf agent preview start --authoring-bundle My_Bundle --target-org agentforce
# → Returns session ID

# Send a message
sf agent preview send \
  --authoring-bundle My_Bundle \
  --session-id <SESSION_ID> \
  --utterance "What is my order status?" \
  --target-org agentforce

# List sessions
sf agent preview sessions --target-org agentforce

# End session
sf agent preview end --session-id <SESSION_ID> --target-org agentforce
```

### Preview Transcript Files

Saved sessions produce these files:
- `transcript.json` — Record of the conversation (timestamp, sessionID, role, text)
- `traces/<sessionID>.json` — Full API messages including intent, invoked subagent, execution plan, tools, reasoning

---

## Testing an Agent with Agentforce DX

### Testing Workflow

1. **Generate test spec** — `sf agent generate test-spec`
2. **Customize YAML** — Add test cases, context variables, metrics
3. **Create test in org** — `sf agent test create`
4. **Run tests** — `sf agent test run`
5. **Debug failures** — Use VS Code Agent Preview or `agent preview` CLI
6. **Iterate** until all tests pass
7. **Add to CI/CD** — Use `agent test run` in CI pipeline

### Generate a Test Spec

```bash
sf agent generate test-spec --target-org agentforce
```

The command prompts for:
- Test type (currently only `AGENT`)
- Agent API name
- Test name and description
- Test cases (utterances, expected subagents, expected actions, expected outcomes)

### Test Spec YAML Structure

```yaml
agentApiName: Resort_Manager
name: Resort Manager Tests
description: Tests for the Resort Manager agent
metrics:
  - completedWithoutError
  - utteranceClassifiedCorrectly
  - actionSequenceMatched
  - botResponseRatedOnTopic
testCases:
  - utterance: "Who is working the front desk today at noon?"
    expectedSubagent: staffing_information
    expectedActions:
      - get_staff_schedule
    expectedOutcome: The staff member working the front desk is provided.
    customEvaluations: []
    conversationHistory: []
```

### Optional: Convert from AiEvaluationDefinition XML
```bash
sf agent generate test-spec \
  --from-definition force-app/main/default/aiEvaluationDefinitions/My_Test.xml \
  --output-file specs/My_Test-testSpec.yaml
```

### Create Agent Test in Org
```bash
sf agent test create --spec specs/Resort_Manager-testSpec.yaml --target-org agentforce
```

### Run Agent Tests
```bash
sf agent test run --target-org agentforce
```

---

## Salesforce CLI — `agent` Commands Summary

```bash
# Explore all agent commands
sf search  # type "agent" in the search

# Generate commands
sf agent generate agent-spec          # Generate agent spec YAML
sf agent generate authoring-bundle    # Generate authoring bundle (agent script)
sf agent generate test-spec           # Generate test spec YAML

# Validate & Publish
sf agent validate authoring-bundle    # Validate Agent Script file
sf agent publish authoring-bundle     # Publish agent to org

# Preview commands
sf agent preview                      # Interactive preview
sf agent preview start                # Start programmatic session
sf agent preview send                 # Send message to agent
sf agent preview sessions             # List active sessions
sf agent preview end                  # End session

# Test commands
sf agent test create                  # Create agent test in org
sf agent test run                     # Run agent tests

# Org commands
sf org create agent-user              # Create default agent user
sf org create sandbox                 # Create a sandbox
sf org create scratch                 # Create a scratch org
sf org login web                      # Authorize an org
```

Full CLI reference: [Salesforce CLI: agent Commands](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_agent_commands_unified.htm)

---

## Official References

- [Build Agents with Agentforce DX](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx.html)
- [Set Up Development Environment](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-set-up-env.html)
- [Code Your Agent Using Its Script File](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-script.html)
- [Generate an Authoring Bundle](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-authbundle.html)
- [Publish an Authoring Bundle](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-publish.html)
- [Preview and Debug an Agent](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-nga-preview.html)
- [Generate an Agent Spec File](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-generate-agent-spec.html)
- [Test an Agent with Agentforce DX](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-test.html)
- [Generate a Test Spec File](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-dx-test-spec.html)
- [Salesforce Extensions for Visual Studio Code](https://developer.salesforce.com/docs/platform/sfvscode-extensions/guide)
- [Agentforce Vibes Extension](https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/einstein-overview.html)
- [Trailhead: Quick Start: VS Code for Salesforce Development](https://trailhead.salesforce.com/content/learn/projects/quickstart-vscode-salesforce)
