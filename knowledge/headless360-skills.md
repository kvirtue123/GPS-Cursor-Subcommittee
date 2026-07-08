---
feature_area: headless360
status: reference-only
catalog_id: null
note: "Overlaps P1/P5 'what is a skill / tooling inventory' commentary per catalog audit. Kept as supplementary reference, not independently cataloged."
---

# Headless 360 — Skills (Cursor Build Mode Prompt)

> Cursor: When the user asks you to "add a skill," they mean an on-demand, modular instruction bundle (Agentforce Vibes / `agentskills.io` spec) — *not* an Apex class or an Agentforce subagent. Skills are how you teach Cursor (and other agentic IDEs) to behave correctly inside this repo: they load only when the description matches the user's request.

## Description (2 sentences)

Skills are modular, progressively-loaded instruction packs that extend an agentic IDE (Agentforce Vibes, Cursor, etc.) for specific tasks — each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions) plus optional `references/`, `assets/`, and `scripts/` subdirectories. Salesforce ships out-of-the-box "Salesforce Skills" for org metadata + Apex/LWC patterns, and the [Agentforce Vibes Library](https://github.com/forcedotcom/afv-library) is the canonical home for community + Salesforce-authored skills (incl. the Agentforce Development Skill for building Agent Script agents).

## Capabilities

### Skill Anatomy
- **Directory** named in `kebab-case` (e.g., `apex-create-class`, `data360-sql-runner`).
- **`SKILL.md`** — YAML frontmatter (`name` must match dir, `description` ≤1024 chars) + instruction body.
- **`references/`** — long-form reference material the skill `read_file`s on demand.
- **`assets/`** — boilerplate / scaffolds the skill writes out.
- **`scripts/`** — deterministic helpers (validation, formatting, API calls); only stdout enters context.

### Progressive Loading (token economics)
- **Metadata** — always loaded at startup, ~100 tokens per skill (name + description).
- **Instructions** — loaded only when triggered, target <5k tokens for `SKILL.md` body.
- **Resources** — loaded on demand via `read_file` or script execution; effectively unlimited.

### Activation Model
- The IDE matches user intent against skill `description`s; specific verb-led, trigger-phrase-rich descriptions win.
- Skills load **on demand** — opposite of Rules, which are always active.
- Toggle skills individually from the Skills panel (`⚖️` icon) or `Settings → Agentforce Vibes → Enable Skills`.
- `Agentforce: Refresh Salesforce Skills` keeps OOTB org-aware skills current; enable **Auto Update Skills**.

### Storage Locations
- **Workspace (recommended, version-controlled):** `.a4drules/skills/`
- **Global (per-OS):**
  - macOS: `$HOME/Library/Application Support/Code/User/globalStorage`
  - Linux: `$HOME/.config/Code/User/globalStorage`
  - Windows: `%APPDATA%\Code\User\globalStorage`
- Conflict rule: global skill **wins** when names collide; Salesforce Skills take priority over custom.

### Authoring Workflow
- UI: Skills tab → **New skill…** → name + template `SKILL.md`.
- Slash command: `/newrule` (rules) — for skills, scaffold a directory.
- Manual: drop the directory into `.a4drules/skills/`; auto-detected.
- Commit `.a4drules/skills/` to source control so the team shares them.

### Notable Skills for Headless 360
- **Agentforce Development Skill** (`afv-library/skills/developing-agentforce`) — authoritative skill for building, debugging, deploying Agent Script agents.
- **Salesforce Skills (built-in)** — org-metadata-aware Apex/LWC/SFDX skills; pair with the Metadata Experts MCP server.
- **Custom project skills** (this repo) — candidates: `data360-sql-runner`, `gnn-score-fetch`, `phase3-case-creator`, `dc-describe-helper`.

### Related (not the same)
- **Rules** (`.a4drules/`) — *always-on* persistent behavior (e.g., `a4d-apex-rules-no-edit.md`); honor `agents.md` standard.
- **Workflows** — multi-step structured sequences the agent follows.
- **Hooks** — pre/post-tool injections (linters, guardrails).
- **Abilities** — gate which MCP tools light up per domain.

### Reference URLs
- https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/skills.html
- https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/devagent-overview.html
- https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/devagent-rules.html
- https://github.com/forcedotcom/afv-library
- https://agentskills.io/specification

---

## What Cursor Should Know (Headless 360 forward-look) — ≤500 chars

Skills are how this repo scales agentic IDE behavior without bloating context. Write narrow, action-verb descriptions; one skill per domain (Data 360 SQL, GNN scoring, Phase 3 Cases). Keep `SKILL.md` <5k tokens; push detail into `references/` and deterministic ops into `scripts/`. Commit `.a4drules/skills/` so every contributor's Cursor/Vibes session inherits the same playbook — same skills the headless Agentforce agent will mirror in production.
