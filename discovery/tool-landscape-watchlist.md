# Cursor Tool Landscape — Watch List

> Paste this into a Slack Canvas (the **Tool Landscape Watch List** workstream canvas).
> This is a **living inventory** — add a row when you discover something worth evaluating, update Status as the team forms a view.
> Full catalog (rules, skills, patterns) lives in the **Rules & Skills** canvas and the GitHub repo.

🔗 **Repo:** [GPS-Cursor-Subcommittee](https://github.com/kvirtue123/GPS-Cursor-Subcommittee)
💬 **Feed a finding?** Drop it in the channel and someone will triage it into the table.

---

## How to use this canvas

- **Watching** — on our radar; no one has tried it yet
- **Evaluating** — someone is actively testing it
- **Adopted** — we have a verdict and a best-practice card (linked to Rules & Skills canvas or repo)
- **Passed** — evaluated and decided not to adopt (reason noted)

Add a row. Keep the notes tight — 1–2 sentences. Link to a thread for longer discussion.

---

## Tab Completion

| Tool | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| Cursor native tab completion | ✅ Adopted | Default; no config needed. | — | — |
| GitHub Copilot (in Cursor) | 👀 Watching | Can run alongside Cursor's native — eval for preference overlap. | — | — |
| Supermaven | 👀 Watching | Faster completion latency claims; free tier available. | — | — |

---

## Browser Use / Web Automation

| Tool | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| Cursor Browser tool (built-in) | ✅ Adopted | Available via MCP `plugin-browser-browser`; use for scraping JS-rendered pages. | — | — |
| Playwright MCP | 🔬 Evaluating | Used in S3 (`sf-docs-mcp`) for `developer.salesforce.com` (Shadow DOM). Proven pattern — see S3. | — | — |
| Stagehand | 👀 Watching | Natural-language browser automation; could replace bespoke Playwright scripts. | — | — |

---

## Linters & Static Analysis

| Tool | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| SLDS Linter (LWC) | ✅ Adopted | Use for SLDS 2 uplift — token migration, no-hardcoded-values. See `uplifting-components-to-slds2` skill. | — | — |
| ESLint (LWC / JS) | ✅ Adopted | Standard; run in CI. | — | — |
| PMD (Apex) | 👀 Watching | Static analysis for Apex code quality; worth evaluating for CI gates. | — | — |
| Cursor ReadLints (IDE) | ✅ Adopted | Use `ReadLints` tool in agent sessions after edits — surfaces errors without running the full build. | — | — |

---

## MCP Servers

> For the full MCP allowlist pattern see **R5** in the Rules & Skills canvas.

| Server | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| Salesforce DX MCP | ✅ Adopted | Live org queries; prefer over static metadata reads (see R11). | — | — |
| sf-docs-mcp (S3) | ✅ Adopted | Salesforce docs → clean Markdown. Node 18–24 only. Needs its own GitHub repo. | — | — |
| Figma MCP | ✅ Adopted | `use_figma`, `create_new_file`, `get_metadata`. Allowlisted in R5 template. | — | — |
| FWA Data Cloud MCP | ✅ Adopted | Data Cloud queries + describe. Pairs with R10 + S4. | — | — |
| Agentforce MCP | ✅ Adopted | Agent build/test workflows. | — | — |
| plugin-browser-browser | ✅ Adopted | Playwright-backed browser MCP. | — | — |
| Context7 | 👀 Watching | Public library docs as MCP resources — could complement sf-docs-mcp for non-SF stacks. | — | — |
| Exa (web search) | 👀 Watching | Semantic web search as an MCP tool — alternative/complement to WebFetch for research tasks. | — | — |

---

## Model Routing & Cost

> For the scoped UI-polish prompt pattern (cheapest model per task) see **P26** in the Rules & Skills canvas.
> For Flex Credit estimation see the `sf-flex-estimator` skill.

| Topic | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| claude-sonnet-4-5 as default | ✅ Adopted | Good balance of quality + cost for most coding tasks. | — | — |
| Frontier models for hard tasks | ✅ Adopted | Reserve for architecture decisions, complex debugging (P26 pattern). | — | — |
| Smaller models for cheap edits | ✅ Adopted | Isolated CSS/visual tweaks, boilerplate — P26 enforces this. | — | — |
| Model routing rules in `.mdc` | 👀 Watching | Encoding model-selection heuristics in a rule — pattern not yet formalized. | — | — |

---

## Context & Memory

| Tool | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| `.cursor/rules/*.mdc` | ✅ Adopted | The core rules mechanism — see the full Rules catalog. | — | — |
| `AGENTS.md` / `CLAUDE.md` | ✅ Adopted | Project-root orientation files; auto-loaded. See R4. | — | — |
| `.cursorignore` / `.claudeignore` | ✅ Adopted | Context-scoping — remove dead code from the index. See R7. | — | — |
| Cursor Memory (experimental) | 👀 Watching | Persistent cross-session memory beyond rules files. Evaluate once GA. | — | — |
| Mem0 MCP | 👀 Watching | Explicit long-term memory store via MCP. Use case: project context that outlives a repo. | — | — |

---

## Other / Emerging

| Tool | Status | Notes | Owner | Thread |
|---|---|---|---|---|
| Cursor Automations | 👀 Watching | Trigger agent actions on file events — see `automate` skill. | — | — |
| Cursor Hooks | 👀 Watching | Run scripts around agent events — see `create-hook` skill. | — | — |
| Cursor Background Agents | 👀 Watching | Long-running agent tasks; evaluate for CI-style workflows. | — | — |
| AI Workbench / Agentforce Grid | 🔬 Evaluating | Structured worksheet-based agent evaluation — see `sf-ai-agentforce-grid` skill. | — | — |

---

## How to contribute a new row

1. **Try it** — at least one person should have run it on a real task.
2. **Add a row** with Status = 👀 Watching or 🔬 Evaluating.
3. **Post in the Feed** with a link to this canvas so others know to look.
4. **When the team reaches a verdict**, update to ✅ Adopted or ⛔ Passed and note why.
5. **If adopted**, consider whether it deserves a Rule, Skill, or Pattern card in the main catalog.

> _Emoji key: 👀 Watching · 🔬 Evaluating · ✅ Adopted · ⛔ Passed_
