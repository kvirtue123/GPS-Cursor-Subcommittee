# Best Practices Catalog

Every reusable asset audited from past Cursor projects, documented with a consistent template:

- **What it is / problem it solves**
- **When to use / when NOT to use**
- **Dependencies & prerequisites**
- **Canonical example**
- **Recommended home**

Assets are grouped into **Rules**, **Skills**, and **Patterns**. Each asset has a stable ID (e.g. `R1`, `S1`, `P1`) used by the Slack Canvas discovery layer.

---

## Category: RULES

Behavioral guardrails that shape model behavior, scope context, and conserve tokens. All use the modern `.cursor/rules/*.mdc` format (no legacy `.cursorrules` exist in any project).

### R1 — Reuse Registry & Architecture rule (`alwaysApply`)

- **What / problem solved:** A single always-on rule that encodes (a) hard architectural constraints, (b) a COPY / COPY+ADAPT / SKIP registry of which org assets to reuse, and (c) a running list of deploy gotchas surfaced during the build. Stops the agent from re-litigating architecture or re-discovering the same deploy failures every session.
- **When to use:** Any project with firm tech constraints ("framework X only", "no live callouts") and a known set of assets to reuse/exclude. Especially valuable on Salesforce metadata projects where deploy order and XSD quirks bite repeatedly.
- **When NOT to use:** Exploratory/greenfield work where constraints aren't settled yet — an always-apply rule that's wrong is worse than none. Keep it small; don't let it grow into a design doc.
- **Dependencies:** Cursor rules support (`.cursor/rules/`). Token cost is paid on every request because `alwaysApply: true`.
- **Canonical example:** [`rules/canonical/melody-reuse-architecture.mdc`](rules/canonical/melody-reuse-architecture.mdc) — genericized as [`rules/templates/agent-script-architecture.mdc`](rules/templates/agent-script-architecture.mdc) and [`rules/templates/salesforce-deploy-gotchas.mdc`](rules/templates/salesforce-deploy-gotchas.mdc).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R2 — Doc-sync workflow rule (token-efficient, glob-scoped)

- **What / problem solved:** Tells the agent *when* documentation needs updating vs. when to skip, with a routing table mapping change-types to target files. Prevents two failure modes: (1) docs silently rotting, and (2) the agent burning tokens rewriting docs on every trivial edit.
- **When to use:** Projects with a maintained design/knowledge doc set where you want docs kept fresh *cheaply*. Scope it with `globs:` so it only loads when relevant files are touched.
- **When NOT to use:** Throwaway prototypes with no docs to maintain. Don't make it `alwaysApply` — the whole point is glob-scoped, on-demand loading.
- **Dependencies:** A defined doc structure to route into (the table is only as good as the file map). Pairs with R3.
- **Canonical example:** [`rules/canonical/doc-sync.mdc`](rules/canonical/doc-sync.mdc) — genericized as [`rules/templates/doc-sync.mdc`](rules/templates/doc-sync.mdc).
- **Home:** **GitHub**. Discovery card → Slack.

### R3 — Goals & Document-Map rule (`@`-reference index)

- **What / problem solved:** States the project's goal in one place and gives the agent an `@`-referenced map of where each kind of design decision lives. Turns "go read everything" into "read exactly this file for this question," scoping context and cutting wasted reads.
- **When to use:** Multi-doc projects where design knowledge is spread across files. The `@docs/...` references let the agent pull the right doc on demand.
- **When NOT to use:** Single-file or doc-light projects. Avoid duplicating the goal statement into many rules — keep it canonical here.
- **Dependencies:** The referenced docs must actually exist at the mapped paths (stale `@`-refs are worse than none). Pairs with R2.
- **Canonical example:** [`rules/canonical/melody-doc-map-goals.mdc`](rules/canonical/melody-doc-map-goals.mdc) — genericized as [`rules/templates/goals-doc-map.mdc`](rules/templates/goals-doc-map.mdc).
- **Home:** **GitHub**. Discovery card → Slack.

### R4 — `AGENTS.md` orientation file

- **What / problem solved:** A human- and agent-readable "start here" at the project root: guardrails, build-phase status table, and a key-doc index. The first thing any new agent session (or teammate) should read.
- **When to use:** Every non-trivial project. It's the lightweight front door that points to rules (R1–R3) and the knowledge library.
- **When NOT to use:** N/A — always worth having; just keep it short and link out rather than inlining detail.
- **Dependencies:** None. Cursor auto-loads `AGENTS.md`.
- **Canonical example:** [`rules/canonical/AGENTS.md`](rules/canonical/AGENTS.md) — genericized as [`rules/templates/AGENTS.md`](rules/templates/AGENTS.md).
- **Home:** **GitHub**. Discovery card → Slack.

### R5 — MCP / WebFetch permission allowlist (`settings.local.json`)

- **What / problem solved:** A tight, pre-approved allowlist of MCP tools and `WebFetch` domains in `.claude/settings.local.json`. Lets the agent work from ground truth (official docs MCP, design-system MCP) without per-call approval prompts, while keeping access *narrow* — the value is in the scoping, not breadth. Conserves tokens and reduces accidental over-broad tool use.
- **When to use:** Any project that leans on a known, stable set of MCP servers / doc domains (e.g. `sf-docs`, Figma, SLDS). Set it once per project.
- **When NOT to use:** Don't over-broaden it into a blanket allow — that defeats the purpose. Don't commit secrets; this file holds permissions only.
- **Dependencies:** The referenced MCP servers must be installed. Cursor/Claude `settings.local.json` support.
- **Canonical example:** [`rules/canonical/hrMelodyv3-mcp-allowlist.settings.local.json`](rules/canonical/hrMelodyv3-mcp-allowlist.settings.local.json) — genericized as [`rules/templates/mcp-allowlist.settings.local.json`](rules/templates/mcp-allowlist.settings.local.json).
- **Provenance:** `hrMelodyv3`; `WhitepaperGeneration` (added Figma MCP tools `mcp__figma__use_figma`/`create_new_file`/`get_metadata` + a docs MCP `mcp__mcp-adaptor__doc_search` to the template — confirms the pattern generalizes beyond Salesforce-doc MCPs to design-system tooling).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R6 — Agent runtime guardrails (anti-jailbreak / scope-lock)

- **What / problem solved:** A reusable `system.instructions` block for the **deployed** Agent Script agent (not the coding agent): refuse out-of-scope/general-knowledge questions, ignore user attempts to override system rules, never reveal prompts/config/functions, reject recap/summary requests, treat masked data as real. Hardens a customer-facing agent against prompt injection and scope creep.
- **When to use:** Every production, customer-facing agent. Pair with on-scope routing so off-topic requests are redirected, not answered.
- **When NOT to use:** Internal/developer tooling agents whose job *is* open Q&A — don't over-restrict.
- **Dependencies:** Agent Script runtime. This is distinct from the `.mdc` Cursor rules (R1–R4) — it shapes agent behavior at conversation time.
- **Canonical example:** [`rules/canonical/hrMelodyv3-agent-guardrails.agent.md`](rules/canonical/hrMelodyv3-agent-guardrails.agent.md) — genericized as [`rules/templates/agent-runtime-guardrails.agent.md`](rules/templates/agent-runtime-guardrails.agent.md).
- **Provenance:** `hrMelodyv3` (`Symphony.agent`).
- **Home:** **GitHub**. Discovery card → Slack.

### R7 — Context-scoping ignore file (`.cursorignore` / `.claudeignore`)

- **What / problem solved:** A context-scoping + token-efficiency guardrail expressed through the *ignore-file* mechanism rather than a `.mdc`: it removes dead, superseded, or known-broken code from the agent's index so it's never cited, copied, or mistaken for a reference implementation. Complements R1's COPY/SKIP registry — R1 *tells* the agent what to skip; R7 makes the skipped code physically invisible to retrieval.
- **When to use:** Any project carrying a superseded/abandoned implementation alongside the live one (legacy agent versions, a rewritten module, a deprecated framework) that the agent keeps stumbling onto. Keep `.cursorignore` and `.claudeignore` in sync.
- **When NOT to use:** To hide *active* code you simply don't want edited (use rules/instructions for that — ignoring it blinds the agent to real context). Not a secrets mechanism. Don't over-exclude — anything ignored is gone from search/grep too.
- **Dependencies:** Cursor (`.cursorignore`) and/or Claude (`.claudeignore`) ignore-file support. One comment per excluded path stating *why*.
- **Canonical example:** [`rules/canonical/sfdxMelodyv3.cursorignore`](rules/canonical/sfdxMelodyv3.cursorignore) (excluded the dead `Harmony` agent) — genericized as [`rules/templates/context-scoping-ignore.cursorignore`](rules/templates/context-scoping-ignore.cursorignore).
- **Provenance:** `sfdxMelodyv3`.
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R8 — Agentforce metadata operational rule (glob-scoped)

- **What / problem solved:** A glob-scoped `.mdc` that encodes the operational discipline for working on Agentforce subagents/topics (`GenAiPlugin`), planners, and invocable actions: deploy by named component only, *don't retrieve-to-verify* (`sortOrder` normalises to `0`), never use unscoped/wildcard retrieves, `GenAiPlugin` isn't Tooling-queryable, `GenAiPlanner` isn't retrievable, read the in-repo build doc first (don't re-derive specs from Help/MCP), and never echo org credentials. Saves tokens and stops re-discovering the same Metadata-API shape mismatches every session.
- **When to use:** Any Agentforce build that authors/deploys `GenAiPlugin`/`aiAuthoringBundle` metadata. Scope it with `globs:` so it only loads when those files are touched.
- **When NOT to use:** Non-Agentforce Salesforce work (use `salesforce-deploy-gotchas.mdc` alone) or non-SF projects. Don't make it `alwaysApply` — it's glob-scoped on purpose.
- **Dependencies:** Cursor rules support; an in-repo build doc to point `{{BUILD_DOC_PATH}}` at. Complements R1 (architecture) and the `salesforce-deploy-gotchas.mdc` template.
- **Canonical example:** [`rules/canonical/FHA-Lessons-Learned-Agent-Sessions.md`](rules/canonical/FHA-Lessons-Learned-Agent-Sessions.md) (as-shipped) — genericized as [`rules/templates/agentforce-metadata-ops.mdc`](rules/templates/agentforce-metadata-ops.mdc). Recurring deploy lines also folded into [`rules/templates/salesforce-deploy-gotchas.mdc`](rules/templates/salesforce-deploy-gotchas.mdc).
- **Provenance:** `FHA Call Center` (HUD Agentforce Service Assistant; `docs/Lessons-Learned-Agent-Sessions.md`).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R9 — Single-component architecture lock (`alwaysApply`)

- **What / problem solved:** A small always-on rule that locks the **non-obvious** architecture of one component/service (e.g. an MCP server, a worker, a library) so an agent doesn't "fix" a deliberate, counter-intuitive decision into breakage. Records the chosen approach + the *tempting-but-wrong* alternative, the most-critical files, a one-line verify command, and the load-bearing gotchas (external-dependency rotation, runtime version ceilings, forbidden actions).
- **When to use:** Any self-contained component whose correctness depends on a decision that looks wrong at first glance (e.g. "use the Aura API, NOT a headless browser, because Locker Service detects CDP"). Keep it short.
- **When NOT to use:** Multi-component repos (use R1's broader architecture/registry rule) or where constraints aren't settled. Don't let it grow into a design doc.
- **Dependencies:** Cursor rules support. Token cost paid every request (`alwaysApply`), so keep it tight. Pairs naturally with an `AGENTS.md` (R4) "why" companion.
- **Canonical example:** [`rules/canonical/sf-docs-mcp.core.mdc`](rules/canonical/sf-docs-mcp.core.mdc) (as-shipped, with its `AGENTS.md` companion) — genericized as [`rules/templates/architecture-lock.mdc`](rules/templates/architecture-lock.mdc).
- **Provenance:** `FHA Call Center` (`sf-docs-mcp/.cursor/rules/core.mdc`).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R10 — Verification & Confidence Protocol (glob-scoped)

- **What / problem solved:** A guardrail for any system whose schema/API the agent **must not guess** (Data Cloud, an external DB, an undocumented API). A HARD RULE — "never assert a field/column/API name or function as fact unless verified against live introspection, a captured reference, or a live docs source; otherwise say *'I'm not certain — let me verify'*" — plus a **query router** (one correct access path per data surface: ANSI-SQL vs SOQL vs REST vs SOSL) and a 5-step validate-before-you-write workflow. Kills the single most expensive agent failure mode: confidently writing queries against hallucinated field names.
- **When to use:** Any project touching a system with names/behavior the agent can't safely infer — Data Cloud `__dlm`/`__dll`/`__cio`, external warehouses, undocumented internal APIs. Scope with `globs:` so it loads only on relevant files.
- **When NOT to use:** Schemas the agent reliably knows (well-documented standard sObjects) or trivially cheap to introspect inline. Don't make it `alwaysApply` — glob-scope it.
- **Dependencies:** A real verification source (a live introspection command, captured docs, or a docs MCP). Pairs with skill **S4** (the introspection commands) and is the enforcement sibling of **P20**. Complementary to **R8** (Agentforce metadata "don't retrieve-to-verify") — same "verify, don't assume" spirit, different surface.
- **Canonical example:** [`rules/canonical/fwa-data-360.mdc`](rules/canonical/fwa-data-360.mdc) (Data Cloud, as-shipped) — genericized as [`rules/templates/verification-confidence-protocol.mdc`](rules/templates/verification-confidence-protocol.mdc).
- **Provenance:** `FWA Federal Fraud` (`FWA-Project/.cursor/rules/data-360.mdc`).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R11 — MCP-first org verification (token-efficient)

- **What / problem solved:** Use the Salesforce DX MCP by default (query the **live org**) instead of reading static metadata or shelling the CLI, and **verify every field/object API name against the org before writing SOQL/Apex**. Cuts tokens and kills a whole bug class caused by local source drifting from the deployed org. Distinct from R5 (which *allowlists* MCP tools) and R10 (the generic don't-guess *protocol*): R11 is the concrete "prefer the org over static files + verify-before-write" default for SFDX.
- **When to use:** Any SFDX project with the DX MCP connected, especially where local metadata is known to drift.
- **When NOT to use:** No MCP/org available, or pure-local work. Deploys & destructive changes stay on the CLI, not MCP.
- **Dependencies:** Salesforce DX MCP connected; an org alias. Same spirit as R8/R10.
- **Canonical example:** [`rules/templates/mcp-first-verification.mdc`](rules/templates/mcp-first-verification.mdc).
- **Provenance:** `ItineraryBuilder` (NEW; generalized from its AGENT_REFERENCE "Salesforce DX MCP" + verify-before-write sections).
- **Home:** **GitHub** (installable). Discovery card → Slack.

### R12 — Override-layer precedence rule

- **What / problem solved:** Designates one authoritative override doc that supersedes contradictions in all other design docs, plus a fixed reading order. Resolves cross-doc conflicts deterministically without back-editing every doc — preserving the audit trail of what changed and why. Complements R3 (doc-map says *where* knowledge lives; R12 says *which wins*).
- **When to use:** Multi-doc projects where docs written at different times now contradict each other (post-review Q&A addenda, spec corrections).
- **When NOT to use:** Single-doc or freshly-written sets with no contradictions.
- **Dependencies:** The override doc must exist and be maintained.
- **Canonical example:** [`rules/templates/override-layer-precedence.mdc`](rules/templates/override-layer-precedence.mdc).
- **Provenance:** `ItineraryBuilder` (NEW; from its QA Addendum override mechanic + "read this first" order).
- **Home:** **GitHub**. Discovery card → Slack.

### R13 — Design-language token rule (glob-scoped to LWC)

- **What / problem solved:** Forces every LWC to match a shared visual design language — `:host` CSS custom-property tokens, card structure, typography scale, semantic-pill + left-border status patterns, AI/sparkle treatment, and the shadow-DOM `setProperty` caveat for library-rendered elements. One point of change for the palette; visual consistency across components.
- **When to use:** Multi-component portals/apps with a defined brand/design system (Experience Cloud, design-system-driven UIs).
- **When NOT to use:** Single-component or non-UI work; no design system to enforce. Don't `alwaysApply` — scope with `globs` to `**/lwc/**`.
- **Dependencies:** Reference LWCs that establish the language; SLDS.
- **Canonical example:** [`rules/templates/design-language-tokens.mdc`](rules/templates/design-language-tokens.mdc).
- **Provenance:** `ItineraryBuilder` (NEW; from Phase 1.5 DesignSpec §1 + PCSPortal_UIPolish status-pill/left-border/tile-accent work).
- **Home:** **GitHub**. Discovery card → Slack.

> **VARIANTs folded (no new IDs):** `ItineraryBuilder`'s *Locked Decisions* table → folded into `rules/templates/agent-script-architecture.mdc` (R1) as an optional "Locked Decisions" section. Its LWR/LWC + compound-address deploy failures → folded into `rules/templates/salesforce-deploy-gotchas.mdc` (R1 companion). `ItineraryBuilder` added to those templates' provenance.

---

## Category: SKILLS

Reusable, task-specific agent skills you authored (distinct from the ~100 inherited Salesforce/Cursor platform skills, which are referenced — not re-homed — here).

### S1 — `agentforce-agentscript-clt`

- **What / problem solved:** End-to-end recipe for building, deploying, and reliably rendering Custom Lightning Types (CLTs) driven by Agent Script actions — including the LWC envelope-unwrap pattern and a failure-mode catalogue ("renders in Builder preview but blank in Embedded Messaging," planner skips the action, blank card, etc.).
- **When to use:** Any time an Agentforce action must render a custom card/carousel/component, or when debugging a CLT that "doesn't render."
- **When NOT to use:** Plain Apex/Flow/Prompt-Template work with no custom rendering; standard out-of-the-box agent responses.
- **Dependencies:** Salesforce DX, Agentforce with Agent Script, LWC, an org with Enhanced Chat v2 for the `enhancedWebChat` surface. Companion of pattern **P2**.
- **Canonical example:** [`skills/agentforce-agentscript-clt.md`](skills/agentforce-agentscript-clt.md).
- **Home:** **GitHub** (packaged skill). Discovery card → Slack.

### S2 — `salesforce-gps-whitepaper` (a.k.a. `wpGenerator`)

- **What / problem solved:** Converts a Markdown source into a Salesforce-GPS-branded white-paper **PDF** via a fixed pipeline (HTML build artifact → headless Chrome render → PDFKit verification → PNG visual inspection). Encodes hard-won gotchas (no `position:absolute` on pages; Chrome-only renderer; letter-spacing text-extraction trap).
- **When to use:** Turning an internal `.md` into a branded PDF deliverable using the standard GPS template.
- **When NOT to use:** Live/HTML deliverables, non-GPS branding, or decks (use Slides/PPTX tooling). Don't swap the renderer — the CSS is calibrated to Chrome's print engine.
- **Dependencies:** macOS (Chrome headless, `swift`/PDFKit), the `whitepaper.css` template + authoring guide referenced inside the skill.
- **Canonical example:** [`skills/salesforce-gps-whitepaper.md`](skills/salesforce-gps-whitepaper.md).
- **Home:** **GitHub** (packaged skill). Supporting template/guide files → **Google Drive**. Discovery card → Slack.

### S3 — `sf-docs-mcp` (Salesforce docs → Markdown MCP server)

- **What / problem solved:** A custom MCP server that makes Salesforce documentation LLM-readable, so an agent reads authoritative, current SF docs instead of hallucinating or choking on JS-heavy HTML. `help.salesforce.com` via the **Aura API + native `fetch()`** (no browser — Locker Service detects CDP); `developer.salesforce.com` via **Playwright + stealth** (Shadow DOM); 24h SQLite cache; self-heals the `fwuid` rotation (~3×/yr).
- **When to use:** Any Cursor/Claude project that needs ground-truth Salesforce docs in context (Help articles, Developer Guide, GenAI guides).
- **When NOT to use:** Offline work, or when an in-repo build doc already has the spec (see R8 — don't burn tokens re-fetching what's already specced). Not for non-SF docs.
- **Dependencies:** **Node 18–24** (25+ breaks `better-sqlite3`), `better-sqlite3`, Playwright (developer path only). Register via `.mcp.json`. Ship its architecture lock (R9) alongside.
- **Canonical example:** [`skills/sf-docs-mcp/SKILL.md`](skills/sf-docs-mcp/SKILL.md) + [`skills/sf-docs-mcp/README.md`](skills/sf-docs-mcp/README.md). (Full TS server lives in the source project; promote to its own GitHub repo.)
- **Provenance:** `FHA Call Center` (`sf-docs-mcp/`).
- **Home:** **GitHub** (server repo + packaged skill). Discovery card → Slack.

### S4 — `data-cloud-sql-runner`

- **What / problem solved:** Three deterministic Bash helpers (`dc-query.sh`, `dc-describe.sh`, `dc-list-objects.sh`) that run ANSI SQL, list columns, and list objects against **Salesforce Data Cloud (Data 360)** by wrapping `ConnectApi.CdpQuery` + `pg_catalog` via `sf apex run` — returning clean JSON, so only the result (not the Apex plumbing) enters context. The deterministic-script arm of the **R10** confidence protocol: makes "verify the field name against the live org" a one-liner.
- **When to use:** Whenever you need *verified* Data Cloud schema or data for CIOs, identity-resolution rules, data graphs, or analysis on `__dlm`/`__dll`/`__cio` objects.
- **When NOT to use:** CRM/platform sObjects (use SOQL / `sf data query`); Data Cloud *data-graph definitions* (use the REST API). Not for non-Data-Cloud systems.
- **Dependencies:** `sf` CLI authed to a Data Cloud org, `jq`, `base64`, API 62+. Scripts install to `<sfdx-root>/.cursor/skills/` and run by path (not on `PATH`). Pairs with **R10** and **P20**.
- **Canonical example:** [`skills/data-cloud-sql-runner/SKILL.md`](skills/data-cloud-sql-runner/SKILL.md) + [`README.md`](skills/data-cloud-sql-runner/README.md) + the three real scripts under [`skills/data-cloud-sql-runner/scripts/`](skills/data-cloud-sql-runner/scripts/).
- **Provenance:** `FWA Federal Fraud` (`FWA-Project/salesforce/.cursor/skills/`).
- **Home:** **GitHub** (packaged skill + scripts). Discovery card → Slack.

### S5 — `lwc-fullcalendar-integration`

- **What / problem solved:** End-to-end recipe for embedding FullCalendar v6 in an LWC as a single static-resource bundle (no npm build) with drag-and-drop, resize, external-palette drop, multiple event sources, and read-only mode — plus the five non-negotiable, hard-won patterns that make it work inside LWR's shadow DOM (init in `renderedCallback` + guard; palette in the same template; exclusive-end ±1 day; style via `info.el.style`/`setProperty`; overlay events as `editable:false` foreground tiles).
- **When to use:** Any LWC needing a drag-and-drop/resizable calendar, especially on Experience Cloud (LWR) with no build pipeline.
- **When NOT to use:** Non-LWC web apps (use FullCalendar directly); static read-only date displays; server-side scheduling.
- **Dependencies:** FullCalendar v6 global bundle as a static resource (`dayGrid`+`interaction`), `loadScript`. Pairs with R13 (design tokens) for tile theming.
- **Canonical example:** [`skills/lwc-fullcalendar-integration/SKILL.md`](skills/lwc-fullcalendar-integration/SKILL.md) + [`README.md`](skills/lwc-fullcalendar-integration/README.md) (as-shipped capabilities ref).
- **Provenance:** `ItineraryBuilder` (NEW; from `README.md` + QA Addendum §2/§8 + PCSPortal_UIPolish §4).
- **Home:** **GitHub** (packaged skill). Discovery card → Slack.

### S6 — `sf-dependent-address-picklist`

- **What / problem solved:** Reads Salesforce compound-Address State/Country **dependent picklists** in Apex (decoding the `validFor` bitVector) and drives dependent `lightning-combobox` fields in an LWC, so users pick valid ISO codes instead of typing free text that throws DML errors on save. Includes the country-first/clear-state-on-change UX and the full step-by-step.
- **When to use:** A compound `Address` field with State/Country Picklists enabled; DML errors saving a typed state/country; country→state dependent dropdowns from org metadata.
- **When NOT to use:** Plain text address fields (no picklist constraint); non-Address custom dependent picklists (same `validFor` technique, different describe).
- **Dependencies:** State/Country Picklists enabled; Apex `getPicklistValues()` + `EncodingUtil`; LWC `lightning-combobox` + cacheable `@wire`. Specific failure folded into `salesforce-deploy-gotchas.mdc`.
- **Canonical example:** [`skills/sf-dependent-address-picklist/SKILL.md`](skills/sf-dependent-address-picklist/SKILL.md) + [`README.md`](skills/sf-dependent-address-picklist/README.md) (as-shipped fix doc).
- **Provenance:** `ItineraryBuilder` (NEW; from `FIX-LOCATION-PICKLISTS.md`).
- **Home:** **GitHub** (packaged skill). Discovery card → Slack.

---

## Category: PATTERNS

Repeatable workflows, prompt structures, and architectural approaches.

### P1 — Agent Script knowledge library

- **What / problem solved:** A curated, navigable library of Agent Script DSL guidance: a patterns index (action chaining, agent router, conditionals, fetch-data, filtering, required flow, transitions, variables…), worked examples, and a reference section. Replaces scattered tribal knowledge with one playbook the agent can `@`-reference.
- **When to use:** Any Agent Script (`.agent`) authoring or review. Point rules (R3) at the overview file so the agent reads the right page on demand.
- **When NOT to use:** Non-Agentforce projects. Don't fork the whole library per project — reference it.
- **Dependencies:** Agentforce Agent Script. Mostly Salesforce-doc-derived, so re-validate against current official docs periodically.
- **Canonical example:** pointer in [`patterns/canonical/agent-script-library.md`](patterns/canonical/agent-script-library.md) (source: `hrMelody/docs/knowledge/agent-script/`).
- **Home:** **GitHub** (the curated library). Discovery card → Slack.

### P2 — Custom Lightning Types best-practices guide

- **What / problem solved:** The canonical 13-section CLT recipe: bundle structure, Apex class rules (top-level/`global`/`@AuraEnabled`/`@JsonAccess`), channel-folder matrix, collection-renderer pattern, namespace rules, the end-to-end workflow, a failure-mode table, and a copy-into-PR checklist.
- **When to use:** Designing or reviewing any CLT; onboarding someone to CLT work; as the checklist in CLT PRs.
- **When NOT to use:** Non-CLT UI work. It's a *reference*, not a per-project artifact — link to it.
- **Dependencies:** Salesforce platform v64.0+, LWC, Agentforce. Tightly coupled to skill **S1**.
- **Canonical example:** [`patterns/canonical/lightning-types-best-practices.md`](patterns/canonical/lightning-types-best-practices.md) (source: `hrMelody/demo-app/`). **§14 (Agent Script specifics: `show_command` force-render, single-object chip-menu gotchas, LEX-vs-ECv2 click pattern, ESD re-publish) folded in from `hrMelodyv3`.**
- **Provenance:** `hrMelody`, `hrMelodyv3` (§14 + chip-menu variant).
- **Home:** **GitHub**. Discovery card → Slack.

### P3 — Cross-surface agent handoff template

- **What / problem solved:** A structured "handoff packet" for passing a task from one agent/surface to another (e.g. Claude Code → Cursor for browser DevTools work): context, exact diagnostic steps, expected outcomes table, and a "report back" contract. Makes async multi-agent debugging reproducible instead of lossy.
- **When to use:** When a task needs a capability the current surface lacks (browser DevTools, a specific MCP, a different model) and must be delegated with full context.
- **When NOT to use:** Work the current agent can finish itself — a handoff packet is overhead you don't need.
- **Dependencies:** None structurally; the *content* assumes the receiving surface has the named tools.
- **Canonical example:** genericized as [`patterns/templates/cross-surface-handoff.md`](patterns/templates/cross-surface-handoff.md) (source: `sfdxMelodyv3/docs/debug/cursor-handoff-lwc-inspect.md`).
- **Home:** **GitHub** (template). Discovery card → Slack.

### P4 — Phased-delivery + design-doc-map architecture

- **What / problem solved:** A project structure that splits **design** docs (A: goals/topics, B: percepts/states, C: data model) from sequenced **build-phase** docs (phase 1→N), each phase self-contained with deploy order, COPY-vs-BUILD labels, and dependencies. Gives both humans and agents a deterministic execution path and a clean design/implementation separation.
- **When to use:** Multi-phase delivery projects (especially metadata/deploy-ordered Salesforce builds) where you want resumable, auditable progress.
- **When NOT to use:** Tiny one-shot tasks. Don't over-scaffold a 2-file change.
- **Dependencies:** Pairs with R2 (doc-sync) and R3 (doc-map) to keep the structure honest.
- **Canonical example:** [`patterns/templates/phased-delivery-structure.md`](patterns/templates/phased-delivery-structure.md) (source: `hrMelody/docs/{design,build}/`).
- **Provenance:** `hrMelody`; `NASA NSPIRES PostAwardBuild` (`docs/award-demo-guide.md` — a VARIANT: an RTM-cited, step-by-step *demo script* mapped one-to-one to build phases/artifacts, with per-step "what to click / what to say / requirement citation" and a run-time matrix. Folded as the "demo-script" flavor of phased delivery; pairs with P9. No new ID).
- **Home:** **GitHub** (template). Discovery card → Slack.

### P5 — AI tooling inventory pattern

- **What / problem solved:** A periodic snapshot of the rules, skills, subagents, MCP servers, and plugins available to the agent in a workspace — so you know what you have before deciding what to build.
- **When to use:** Kicking off an audit (like this one) or onboarding a new environment.
- **When NOT to use:** As a *documentation* substitute — a flat list isn't the same as documented best practices. This repo is the documented successor.
- **Dependencies:** Visibility into the workspace's rule/skill/MCP config.
- **Canonical example:** source `hrMelody/AI_TOOLING_INVENTORY.md` (this repo supersedes it).
- **Provenance:** `hrMelody`; `FHA Call Center` (`docs/HUD_FHA_Asset_Catalog.md` — a VARIANT that inventories *built agent assets* rather than agent *tooling*: an Asset / API-Name / What-it-does / **Integration-status** / **Gaps** table plus a gap-priority table. Folded as the "as-built asset inventory" flavor of this pattern, no new ID); `NASA NSPIRES PostAwardBuild` (`docs/oldorg-inventory.md` — a VARIANT: a Phase-0 *subagent-generated* org metadata inventory, objects/fields/record-types with COPY/SKIP labels, used as ground truth for sequenced build phases. The "metadata-ground-truth" flavor; overlaps P8 org-extraction. No new ID).
- **Home:** **GitHub** (as a template/checklist). Discovery card → Slack.

### P6 — Live-site → design-token extraction → Figma

- **What / problem solved:** A capture pipeline that reverse-engineers a **live website** into a structured **design system**: headless Playwright reads `getComputedStyle` off a representative element set, de-dupes/sorts the values into **W3C Design Token** format, then imports them into Figma (variable collections, text/effect styles) via the Figma MCP. Replaces manual eyedropper work with a repeatable, auditable token snapshot.
- **When to use:** Reproducing or reverse-engineering an existing site's look (colors, type scale, spacing, shadows, radii, transitions) as a Figma design system — e.g. rebrand/parity work where the source of truth is a shipped site, not a Figma file.
- **When NOT to use:** The design system already lives in Figma (start there); you need *semantic* tokens (this extracts raw values — semantic naming is a manual follow-up pass); auth-walled or bot-protected sites where computed styles are unreliable.
- **Dependencies:** Node + Playwright (Chromium) for extraction; Figma MCP (`use_figma`/`create_new_file`) only for the import step. Permissions allowlist: `WebFetch(domain:<target>)`, `mcp__figma__*`.
- **Canonical example:** [`patterns/canonical/wpgen-extract-design-tokens.js`](patterns/canonical/wpgen-extract-design-tokens.js) (as-shipped) — genericized as [`patterns/templates/design-token-extraction.md`](patterns/templates/design-token-extraction.md). Source project: `WhitepaperGeneration` (Hard Rock Casino Hollywood import, 2026-05-22).
- **Home:** **GitHub** (template + script). Discovery card → Slack.

### P7 — Platform data-model reference library

- **What / problem solved:** A curated, `@`-referenceable reference of an out-of-the-box (or managed-package) Salesforce data model — object index, API names, availability versions, and uniform per-object field tables — distilled from the official Developer Guide into one navigable file. Lets the agent answer "what does object X look like?" on demand instead of re-introspecting the org or re-deriving the schema every session.
- **When to use:** Projects built on a standard platform domain (PSS Grantmaking, Loans, Health Cloud, etc.) where the same objects/fields are referenced repeatedly. Point R3's doc-map at it.
- **When NOT to use:** Custom-object-light projects, or small schemas cheap to introspect live. Don't hand-maintain a reference that drifts from the org — derive it from the Developer Guide and stamp the API version.
- **Dependencies:** Access to the relevant Salesforce Developer Guide; an API version to pin. Pairs with R3 (doc-map) and R2 (doc-sync, to refresh on version bumps).
- **Canonical example:** pointer in [`patterns/canonical/grantmaking-object-reference.md`](patterns/canonical/grantmaking-object-reference.md) (source: `PostAwardExtract/docs/grantmaking-objects.md`, PSS Grantmaking API v67.0) — genericized as [`patterns/templates/platform-data-model-reference.md`](patterns/templates/platform-data-model-reference.md).
- **Home:** **GitHub** (the curated library). Discovery card → Slack.

### P8 — Org metadata extraction & audit workflow

- **What / problem solved:** A repeatable workflow for reverse-engineering an existing org feature into a local DX project, then writing a two-halved **"what was found / what was missing"** audit. The "missing/unexpected" half captures *negative findings* (things you looked for and did NOT find) — the knowledge normally lost between sessions that causes re-investigation.
- **When to use:** Brownfield / org-takeover engagements, before designing any change. Pair the output with R4 (`AGENTS.md`) so the audit is the project's ground-truth front door.
- **When NOT to use:** Greenfield builds with nothing to reverse-engineer; single known field changes.
- **Dependencies:** Salesforce CLI / metadata retrieve access to the source org; a manifest. Pairs with R4.
- **Canonical example:** pointer in [`patterns/canonical/org-metadata-extraction-audit.md`](patterns/canonical/org-metadata-extraction-audit.md) (source: `PostAwardExtract/docs/awardDetails.md`, Funding Award) — genericized as [`patterns/templates/org-metadata-extraction-audit.md`](patterns/templates/org-metadata-extraction-audit.md).
- **Home:** **GitHub** (template/checklist). Discovery card → Slack.

### P9 — Demoing a probabilistic AI product (LIVE/DEMO toggle)

- **What / problem solved:** A presenter-facing playbook for demoing an LLM-driven product without faking it or gambling on a live model: decouple the probabilistic part (language + routing) from the part that must look perfect (UI, data, critical flows), and give a **LIVE/DEMO toggle** to pick the risk level per moment. Includes the hybrid intent-detection insight, a risk-per-moment table, and a guardrails checklist. Written product-agnostic.
- **When to use:** Any SE/PM demoing a probabilistic AI product to a customer; building a demo asset that must be reliable yet authentic.
- **When NOT to use:** Deterministic products with no probabilistic surface; a static screenshot walkthrough.
- **Dependencies:** None conceptually. Realizing it cleanly needs the P10 hybrid UI architecture.
- **Canonical example:** [`patterns/canonical/probabilistic-ai-demo-playbook.md`](patterns/canonical/probabilistic-ai-demo-playbook.md) (source: `hrMelodyv3/melody-ui-Demo/DEMO-EXPLAINER.md`) — genericized as [`patterns/templates/probabilistic-ai-demo.md`](patterns/templates/probabilistic-ai-demo.md).
- **Provenance:** `hrMelodyv3`. Authored by the GPS Cursor Subcommittee. *(Note: the canonical file had been copied into the repo by a prior contribution but left un-carded — see "Orphan canonical files." This card claims it as P9.)*
- **Home:** **GitHub** (playbook + template). Discovery card → Slack. Demo recordings → Drive.

### P10 — Text-only AI API → rich-UI hybrid architecture

- **What / problem solved:** A reference architecture for a standalone chat UI on top of an AI/agent API that returns **text only**: a credential-holding proxy + two engines (live SSE + scripted state machine) feeding **one shared component layer**, with **client-side intent detection** (text phrase → injected card) to render rich UI reliably when the API and/or native rich-rendering can't. The implementation companion to P9.
- **When to use:** When the AI API gives text only and you still need branded cards/carousels, or the platform's native rich-rendering (e.g. CLT in Embedded Chat) is broken/unavailable.
- **When NOT to use:** When the API already returns structured component payloads, or native rich rendering works — render those directly.
- **Dependencies:** A frontend framework, a small proxy for OAuth/token caching, an AI API that streams text. Pairs with P9.
- **Canonical example:** genericized as [`patterns/templates/text-only-api-hybrid-ui.md`](patterns/templates/text-only-api-hybrid-ui.md) (source: `hrMelodyv3/melody-ui/` + its `HANDOFF.md`).
- **Provenance:** `hrMelodyv3`.
- **Home:** **GitHub** (template; the reference implementation stays in the project repo, not committed here). Discovery card → Slack.

### P11 — Append-only evidence log

- **What / problem solved:** A single durable Markdown file that accumulates debug evidence under dated, append-only headings (plus a small set of in-place "what's verified / not verified / smoking gun" status sections). It survives context autocompaction and session boundaries, so a fresh agent or teammate can reconstruct an entire multi-day, multi-tool investigation from one file instead of from a lost chat history.
- **When to use:** Long or cross-session/cross-tool investigations (e.g. a silent render failure debugged across Claude Code, Cursor, and Agent Builder) where the context is at risk of being compacted away.
- **When NOT to use:** A bug fixed in one sitting — just fix it. Don't let it become a design doc; it's an evidence trail, not a spec.
- **Dependencies:** None. Pairs with the cross-surface handoff (P3 — "go look"), the session brief (P12 — "new session, start here"), and layer-elimination debugging (P13 — the method that produces the evidence).
- **Canonical example:** genericized as [`patterns/templates/append-only-evidence-log.md`](patterns/templates/append-only-evidence-log.md) (source: `sfdxMelodyv3/docs/debug/symphony-carousel.md`).
- **Provenance:** `sfdxMelodyv3`.
- **Home:** **GitHub** (template). Discovery card → Slack.

### P12 — Fresh-session briefing

- **What / problem solved:** A self-contained "brief a new chat" document — what we're building / what works / what doesn't / root cause / known fix paths / HARD CONSTRAINTS / context strategy / open questions — so a brand-new session starts fully oriented with zero prior conversation. Distinct from P3 (which delegates one task to *another surface*); this re-orients a fresh session on the *same* surface after a session boundary, crash, or autocompaction.
- **When to use:** Crossing a session boundary on a non-trivial, in-flight project (next-day pickup, post-compaction, handing the project to a teammate's agent).
- **When NOT to use:** Continuing in the same session with context intact; for a tool/surface capability gap use P3 instead.
- **Dependencies:** Most valuable when it points at a durable evidence log (P11) and any memory/notes files rather than re-deriving state. Refresh/overwrite it each handoff (unlike P11, which is append-only).
- **Canonical example:** genericized as [`patterns/templates/session-brief.md`](patterns/templates/session-brief.md) (source: `sfdxMelodyv3/docs/debug/handoff-symphony-state.md`).
- **Provenance:** `sfdxMelodyv3`.
- **Home:** **GitHub** (template). Discovery card → Slack.

### P13 — Layer-elimination debugging

- **What / problem solved:** A method for debugging silent, multi-layer failures (Apex → planner schema → CLT registry → renderer.json → LWC → channel): enumerate every layer the data crosses, design **one decisive, mutually-exclusive observation** per layer, and use an outcome→diagnosis table so each test exonerates or implicates a layer instead of poking at random. The signature move is injecting an input-independent signal (a debug banner that renders on mount regardless of `value`) so a single screenshot separates "never instantiated" from "instantiated but unbound" from "bound but not rendering."
- **When to use:** Silent, "everything looks correct on paper," multi-layer failures where guessing is expensive.
- **When NOT to use:** A single-layer bug with an obvious stack trace — the ceremony isn't worth it.
- **Dependencies:** Ability to inject a minimal diagnostic scaffold and observe each layer (trace/SSE/DevTools/SOQL). Pairs with P11 (record each elimination) and P3 (delegate the observation if the current surface can't make it).
- **Canonical example:** genericized as [`patterns/templates/layer-elimination-debugging.md`](patterns/templates/layer-elimination-debugging.md) (source: `sfdxMelodyv3/docs/debug/symphony-carousel.md`).
- **Provenance:** `sfdxMelodyv3`.
- **Home:** **GitHub** (template). Discovery card → Slack.

### P14 — Flow Approval Processes knowledge library

- **What / problem solved:** A consolidated, source-cited implementation guide for Salesforce **Flow Approval Processes** (stages/steps, approval vs background steps, orchestration runs, record locking, versioning, migration from legacy approvals, and the metadata deployment artifact). Replaces re-scraping scattered Help pages every time approval automation is touched.
- **When to use:** Any project building multi-step/multi-user/multi-system approvals on Salesforce, or migrating legacy Approval Processes to Flow.
- **When NOT to use:** Simple single-approver records where the legacy Approval Process is sufficient; non-Salesforce work.
- **Dependencies:** Salesforce (Lightning; Enterprise/Unlimited/Performance/Developer). Snapshot of official docs — re-validate periodically. Authored via the **P17** workflow.
- **Canonical example:** [`patterns/canonical/nspires-postaward-flow-approval-processes-guide.md`](patterns/canonical/nspires-postaward-flow-approval-processes-guide.md) (source: NASA NSPIRES PostAwardBuild `docs/`).
- **Provenance:** `NASA NSPIRES PostAwardBuild`.
- **Home:** **GitHub** (knowledge library). Discovery card → Slack.

### P15 — OmniStudio Document Generation recipe

- **What / problem solved:** End-to-end recipe for a custom **Generate Document** feature using OmniStudio DocGen: the `fndSingleDocxLwc` Omniscript + step properties, preselecting templates via URL params, `fndmultiPDFConvertLwc` for client-side→PDF conversion, and browser-based preview configuration.
- **When to use:** Generating `.docx`/`.pdf` documents from Salesforce records via OmniStudio (GANs, letters, contracts, quotes, work orders).
- **When NOT to use:** Non-OmniStudio doc generation (the GPS white-paper pipeline is **S2**); plain Apex PDF rendering.
- **Dependencies:** OmniStudio / Industries (PSS), LWC. Snapshot of official docs. Authored via the **P17** workflow.
- **Canonical example:** [`patterns/canonical/nspires-postaward-omnistudio-docgen-guide.md`](patterns/canonical/nspires-postaward-omnistudio-docgen-guide.md) (source: NASA NSPIRES PostAwardBuild `docs/`).
- **Provenance:** `NASA NSPIRES PostAwardBuild`.
- **Home:** **GitHub** (recipe). Discovery card → Slack.

### P16 — Grantmaking / Public Sector Solutions feature reference

- **What / problem solved:** A reference guide consolidating four PSS/Grantmaking **features** used in post-award work: Budget Management, auto-creating recurring Funding Award Requirements, Compliant Data Sharing, and the Form Framework — with editions/permissions matrices. **Complements P7** (which is the *object/field schema* reference): P7 answers "what does the object look like?", P16 answers "how does the feature behave and who can use it?"
- **When to use:** Building on Public Sector Solutions or Nonprofit Cloud for Grantmaking (budgets, recurring requirements, CDS, forms).
- **When NOT to use:** Orgs without PSS/Grantmaking licensed; non-grant domains. For raw schema, use **P7** instead.
- **Dependencies:** PSS or NPC for Grantmaking license. Snapshot of official docs. Authored via the **P17** workflow.
- **Canonical example:** [`patterns/canonical/nspires-postaward-grantmaking-psc-features-guide.md`](patterns/canonical/nspires-postaward-grantmaking-psc-features-guide.md) (source: NASA NSPIRES PostAwardBuild `docs/`).
- **Provenance:** `NASA NSPIRES PostAwardBuild`.
- **Home:** **GitHub** (reference). Discovery card → Slack.

### P17 — Doc-derived implementation-guide workflow

- **What / problem solved:** The reusable *workflow + document skeleton* for turning scattered official vendor docs into one curated, source-cited, project-local implementation guide the agent `@`-references — instead of re-searching the web each session. Encodes the recurring traps (JS-rendered Metadata API pages can't be scraped → link them directly; cite every section's source URL; always include a metadata/deployment-artifact section). This is the engine behind P14/P15/P16.
- **When to use:** Any platform feature with dense, authoritative, scattered docs that you'll revisit across phases.
- **When NOT to use:** One-off features you'll never revisit; topics whose docs change weekly (a snapshot rots). For *schema* references use **P7**; this is for *behavioral/how-to* guides.
- **Dependencies:** A docs scraper/MCP; a place to register the guide (pairs with **R3** doc-map and **P4** phased delivery).
- **Canonical example:** [`patterns/templates/doc-derived-implementation-guide.md`](patterns/templates/doc-derived-implementation-guide.md) (source pattern abstracted from NASA NSPIRES PostAwardBuild's three guides).
- **Provenance:** `NASA NSPIRES PostAwardBuild`.
- **Home:** **GitHub** (template). Discovery card → Slack.

### P19 — Live-demo run-of-show + fallback matrix

- **What / problem solved:** A reusable structure for a **high-stakes live demo**: the story, a demo arc (start where the audience lives, expose the platform progressively), an off-stage pre-flight checklist, golden/hero records, verbatim on-stage prompts, a speaker talk track, and — the reusable gold — an exhaustive **fallback matrix** (per-failure symptom → likely cause → ≤60s recovery) plus global-emergency escapes and composure rules ("never debug live unless the fix is <30s"). Turns a demo from a gamble into a rehearsed, recoverable performance. **Complements P9** (LIVE/DEMO toggle for *probabilistic output*); P19 is the *whole show's* run-of-show and recovery plan regardless of product type.
- **When to use:** Any customer-facing, on-stage, or recorded demo where a live failure is expensive.
- **When NOT to use:** Async/recorded-only demos or internal smoke tests — the matrix and talk track are overhead; use a plain checklist.
- **Dependencies:** None structurally. Pairs with **P9** (probabilistic toggle), **P21** (the multi-surface story this was built to tell), and **P15/P20** for the data behind it.
- **Canonical example:** [`patterns/templates/live-demo-runbook.md`](patterns/templates/live-demo-runbook.md) — full as-shipped runbooks preserved at [`patterns/canonical/fwa-headless360-se-runbook.md`](patterns/canonical/fwa-headless360-se-runbook.md).
- **Provenance:** `FWA Federal Fraud` (`docs/headless360_se_runbook.docx.md`, `docs/phase_3/phase3-customer-demo-prompt.md`).
- **Home:** **GitHub** (template). Polished runbook/deck → **Google Drive**. Discovery card → Slack.

### P20 — Synthetic demo-data design / generate / load / teardown

- **What / problem solved:** A fill-in-the-blanks workflow for building **disposable, reproducible** synthetic demo data: discover the org schema FIRST (never design before you know real field names), design hero records around the demo beats, generate with a seeded Faker pipeline, bulk-load parent→child (or via the Data Cloud Ingestion API + IR), then tear down repeatably. Includes a runnable Python generator/loader/delete template and load/teardown checklists.
- **When to use:** Any Salesforce/Data Cloud demo needing synthetic data that must be rebuilt identically and wiped cleanly.
- **When NOT to use:** Production data migration (use a real ETL plan with validation); demos that run on existing org data.
- **Dependencies:** `sf` CLI, Python (`pandas`/`faker`/`simple-salesforce`). For Data Cloud field names, pairs with **S4**/**R10** (don't guess). Pairs with **P19** (the demo it feeds) and **P4** (phased build).
- **Canonical example:** [`patterns/templates/synthetic-demo-data-guide.md`](patterns/templates/synthetic-demo-data-guide.md) — full as-shipped SE guide (runnable Python + Ingestion-API steps) preserved at [`patterns/canonical/fwa-demo-data-guide.md`](patterns/canonical/fwa-demo-data-guide.md).
- **Provenance:** `FWA Federal Fraud` (`docs/templates/demo-data-guide.md`).
- **Home:** **GitHub** (template + canonical guide). Discovery card → Slack.

### P21 — Headless multi-surface MCP tool-plane

- **What / problem solved:** A reference architecture: publish a capability **once** as hosted MCP tools, then consume it from **N completely different surfaces** (in-platform agent, Slack, a generated web app, a script) over the same JSON-RPC + OAuth contract — "one data plane, many callers." Includes the **boundary rule** (conversational+decision → route via the agent; pure fetch → call MCP direct), least-privilege per-server split (read-only vs write, each with its own client app + permission set), keep-the-tool-surface-tight discipline (reference content as MCP *resources*, not tools), and a build checklist. **Distinct from P10** (P10 is one client's *rendering* trick for a text-only API; P21 is the *system topology* of publishing once / consuming many).
- **When to use:** Designing a multi-surface AI system where the same capability must serve an agent and one or more human/programmatic clients.
- **When NOT to use:** A single surface with no reuse horizon (a direct integration is simpler); capabilities that can't be safely exposed over per-user OAuth.
- **Dependencies:** Hosted MCP support (e.g. Salesforce Hosted MCP Servers), OAuth/External Client Apps, invocable backing actions. Pairs with **P10** (client rendering), **P19** (demo it), **R10**/**S4** (verified data access).
- **Canonical example:** genericized as [`patterns/templates/headless-mcp-tool-plane.md`](patterns/templates/headless-mcp-tool-plane.md) (source: `FWA-Project/docs/headless-360-build-plan.md`, `phase3-customer-demo-prompt.md`). The customer-facing migration prompt (`docs/templates/ReactAppContext.md`) is a **P3** variant — see findings.
- **Provenance:** `FWA Federal Fraud`.
- **Home:** **GitHub** (template). Discovery card → Slack.

### P22 — LWC wire-vs-imperative decision pattern

- **What / problem solved:** A one-table rule for deciding whether an LWC↔Apex call uses `@wire` (reads/cacheable, refreshed via `refreshApex`) or an imperative call (writes, with full error control). Removes per-method guesswork and keeps caching/error behavior consistent across a component.
- **When to use:** Any non-trivial LWC with a mix of reads and writes against Apex.
- **When NOT to use:** Trivial single-read components — the table is overkill.
- **Dependencies:** LWC + Apex (`@AuraEnabled(cacheable=true)` for the wire side; `refreshApex`).
- **Canonical example:** [`patterns/templates/lwc-wire-vs-imperative.md`](patterns/templates/lwc-wire-vs-imperative.md).
- **Provenance:** `ItineraryBuilder` (NEW).
- **Home:** **GitHub** (template). Discovery card → Slack.

### P23 — Ask → Plan → Agent co-authoring lifecycle

- **What / problem solved:** A three-stage workflow (context-gather → section-by-section refinement → reader-tested implementation) mapped onto the Cursor Ask → Plan → Agent lifecycle, with a mandated doc reading order and a plan-approval gate before any code. Forces alignment before implementation, cutting wasted edits. Distinct from P4 (which structures the *docs*); P23 structures the *act of producing and then executing* them.
- **When to use:** Co-authoring design docs that an agent will then implement, especially where requirements are still being shaped.
- **When NOT to use:** Tiny unambiguous changes where a plan-review gate is pure overhead.
- **Dependencies:** Cursor Ask/Plan/Agent modes. Pairs with R3/R12 (reading order) and P4.
- **Canonical example:** [`patterns/templates/ask-plan-agent-coauthoring.md`](patterns/templates/ask-plan-agent-coauthoring.md).
- **Provenance:** `ItineraryBuilder` (NEW; from AGENT_REFERENCE "Structured Co-Authoring Workflow").
- **Home:** **GitHub** (template). Discovery card → Slack.

### P24 — LWC concurrent-save queue + debounce

- **What / problem solved:** Handles rapid, overlapping auto-saves without races or dropped edits: debounce text inputs (~500ms) + a single-slot in-flight queue (`_isLoading` flag + one `_pendingSave`). Deliberately simple — only the latest pending save matters.
- **When to use:** Auto-saving LWC forms where field changes can fire overlapping writes.
- **When NOT to use:** Explicit save-button forms; single non-overlapping writes. If you need ordered multi-save batching, use a real queue.
- **Dependencies:** LWC; an imperative Apex save.
- **Canonical example:** [`patterns/templates/lwc-concurrent-save-queue.md`](patterns/templates/lwc-concurrent-save-queue.md).
- **Provenance:** `ItineraryBuilder` (NEW; from QA Addendum §5).
- **Home:** **GitHub** (template). Discovery card → Slack.

### P25 — Advisory validation pattern

- **What / problem solved:** Validation that informs without blocking: `save` always succeeds (never a DML gate), rule violations surface as non-blocking alerts, and only a terminal action (Submit) is gated on error-severity alerts. Lets users save partial/invalid drafts and fix issues incrementally — the right model for builders/configurators/multi-step forms.
- **When to use:** Draft-style builders where users must save in-progress, not-yet-valid work.
- **When NOT to use:** Transactional writes where an invalid record must never persist (payments, compliance) — validate before DML and reject there.
- **Dependencies:** Client-side rule engine + a gated terminal action.
- **Canonical example:** [`patterns/templates/advisory-validation.md`](patterns/templates/advisory-validation.md).
- **Provenance:** `ItineraryBuilder` (NEW; from AGENT_REFERENCE "Validation is advisory" + QA Addendum §9).
- **Home:** **GitHub** (template). Discovery card → Slack.

### P26 — Scoped UI-polish prompt pack

- **What / problem solved:** Packages several small, independent UI changes as a sequenced prompt pack — each item names its component, exact file changes, the **model to use** (reserve frontier models for hard work), expected result, a confirm-before-proceeding gate, and a per-item rollback. Makes batched visual refinement safe and cheap. Complements P4 (project-level phasing) at the micro/prompt level.
- **When to use:** A batch of isolated visual/CSS tweaks you want to run one at a time, deploy, and eyeball before continuing; delegating cheap changes to a smaller model.
- **When NOT to use:** Cross-cutting refactors where items aren't isolated (per-item rollback won't hold); a single change.
- **Dependencies:** None structural; assumes a deploy + visual-confirm loop.
- **Canonical example:** [`patterns/templates/scoped-ui-prompt-pack.md`](patterns/templates/scoped-ui-prompt-pack.md).
- **Provenance:** `ItineraryBuilder` (NEW; from `PCSPortal_UIPolish_DesignDoc`).
- **Home:** **GitHub** (template). Discovery card → Slack.

> **DUPLICATE (not re-added):** `ItineraryBuilder`'s Phase 1 / 1.5 / 2 roadmap with "do NOT build X yet" scoping + `// TODO Phase N` markers is a DUPLICATE of **P4** (phased delivery). Its two cheap refinements (explicit non-goals per phase; code-seam TODO markers) were folded into the P4 template; no new ID.

---

## Recommended-home summary

| ID | Asset | GitHub | Slack Canvas | Google Drive |
|---|---|:--:|:--:|:--:|
| R1 | Reuse/architecture rule | ● install | ● card | |
| R2 | Doc-sync rule | ● install | ● card | |
| R3 | Goals/doc-map rule | ● install | ● card | |
| R4 | AGENTS.md | ● install | ● card | |
| R5 | MCP/WebFetch allowlist | ● install | ● card | |
| R6 | Agent runtime guardrails | ● install | ● card | |
| R7 | Context-scoping ignore file | ● install | ● card | |
| R8 | Agentforce metadata ops rule | ● install | ● card | |
| R9 | Single-component architecture lock | ● install | ● card | |
| R10 | Verification & confidence protocol | ● install | ● card | |
| R11 | MCP-first org verification | ● install | ● card | |
| R12 | Override-layer precedence | ● install | ● card | |
| R13 | Design-language token rule | ● install | ● card | |
| S1 | agentforce-agentscript-clt | ● package | ● card | |
| S2 | salesforce-gps-whitepaper | ● package | ● card | ● css/guide/template |
| S3 | sf-docs-mcp | ● package+server | ● card | |
| S4 | data-cloud-sql-runner | ● package+scripts | ● card | |
| S5 | lwc-fullcalendar-integration | ● package | ● card | |
| S6 | sf-dependent-address-picklist | ● package | ● card | |
| P1 | Agent Script library | ● library | ● card | |
| P2 | CLT best-practices | ● guide | ● card | |
| P3 | Cross-surface handoff | ● template | ● card | |
| P4 | Phased-delivery structure | ● template | ● card | |
| P5 | Tooling inventory | ● template | ● card | |
| P6 | Design-token extraction | ● template+script | ● card | |
| P7 | Platform data-model reference | ● library | ● card | |
| P8 | Org metadata extraction & audit | ● template | ● card | |
| P9 | Probabilistic AI demo playbook | ● playbook+template | ● card | ● recordings |
| P10 | Text-only API hybrid UI | ● template | ● card | |
| P11 | Append-only evidence log | ● template | ● card | |
| P12 | Fresh-session briefing | ● template | ● card | |
| P13 | Layer-elimination debugging | ● template | ● card | |
| P14 | Flow Approval Processes library | ● library | ● card | |
| P15 | OmniStudio DocGen recipe | ● recipe | ● card | |
| P16 | Grantmaking/PSS feature reference | ● reference | ● card | |
| P17 | Doc-derived guide workflow | ● template | ● card | |
| P19 | Live-demo runbook + fallback matrix | ● template | ● card | ● runbook/deck |
| P20 | Synthetic demo-data guide | ● template+canonical | ● card | |
| P21 | Headless multi-surface MCP tool-plane | ● template | ● card | |
| P22 | Service Assistant quick-actions/subagents | ● reference | ● card | |
| P22 | LWC wire-vs-imperative | ● template | ● card | |
| P23 | Ask→Plan→Agent co-authoring | ● template | ● card | |
| P24 | LWC concurrent-save queue | ● template | ● card | |
| P25 | Advisory validation | ● template | ● card | |
| P26 | Scoped UI-polish prompt pack | ● template | ● card | |

---

## Gaps & duplicates (action items)

**Gaps**
1. **Rules never traveled forward.** `hrMelodyv3` has no `.cursor/rules/` — the R1–R3 rules live only in `hrMelody` and reference the older "Harmony" naming. → Install the genericized templates into v3.
2. **Skills weren't packaged.** S1/S2 had no install/README/versioning. → This repo packages them.
3. **No discovery layer existed.** → Added `discovery/slack-canvas.md`.

**Duplicates / drift (consolidate to one source of truth)**
1. `docs/design/{A,B,C}.md`, `demo-scenario-map.md`, the Windsurfer BRD, and the CLT guide are duplicated across `hrMelody` and `sfdxMelodyv3`.
2. Project naming drift: Melody → Harmony → Symphony/Sonata. Genericized templates use `PROJECT_NAME` / `ORG_ALIAS` placeholders to end the drift.
3. Versioned `.agent` files (`Harmony_v109..v112`, `Ping_Test_Agent_v1..v3`) — keep only the latest as canonical; archive the rest.

**Findings from `PostAwardExtract` (NASA NSPIRE post-award) — added 2026-06-04**
1. **No rules/skills/AGENTS.md exist in this project.** `.cursor/` is empty; no `.cursorrules`, no authored `SKILL.md`, no `AGENTS.md`. → Install the genericized R1–R4 templates here; this is the same "rules never traveled forward" gap seen in `hrMelodyv3`.
2. **Two NEW patterns contributed:** P7 (platform data-model reference, from `grantmaking-objects.md`) and P8 (org metadata extraction & audit, from `awardDetails.md`). Neither duplicates an existing asset.
3. **Naming-drift watch:** `PostAwardExtract/docs/grantmaking-objects.md` pins **API v67.0 / Summer '26**, while the source PSS guide reads "Summer '26 / API v67.0" in one place — keep the canonical reference's header version authoritative and re-stamp per release (P7 convention).
4. **Cross-project pattern overlap (not a duplicate):** P8 (org reverse-engineering) and P6 (live-site reverse-engineering) are both "capture existing reality → structured artifact" workflows in different domains (org metadata vs. web design tokens). Worth cross-linking in discovery, but they remain distinct assets.
4. **S2 skill copy drift.** `salesforce-gps-whitepaper` exists byte-identically in three places: this repo (`skills/salesforce-gps-whitepaper.md`, canonical), the source project (`WhitepaperGeneration/.claude/skills/wpGenerator/SKILL.md`), and the installed user dir (`~/.cursor/skills/salesforce-gps-whitepaper/SKILL.md`). Keep the **repo copy canonical**; treat the other two as the source and the installed instance.
5. **S2 has a hardcoded path.** The skill's "Step 0" pointer hardcodes `/Users/kvirtue/Desktop/WhitepaperGeneration/...Authoring.md` — the one non-portable line. Repoint on install (noted in `skills/salesforce-gps-whitepaper/README.md`).
6. **Orphan canonical files (wiring drift, from concurrent contributions).** Several `patterns/canonical/` files were present on disk but un-carded. **Resolved:** ~~`probabilistic-ai-demo-playbook.md`~~ → **P9** (`hrMelodyv3`); ~~`nspires-postaward-flow-approval-processes-guide.md`~~ → **P14**, ~~`nspires-postaward-omnistudio-docgen-guide.md`~~ → **P15**, ~~`nspires-postaward-grantmaking-psc-features-guide.md`~~ → **P16** (all `NASA NSPIRES PostAwardBuild`, added 2026-06-04). **Still open:** `grantmaking-object-reference.md` — contributed by `PostAwardExtract` as the **P7** canonical pointer target; verify the P7 card's link resolves (the card references it) and that the file isn't a stray duplicate of the P7 pointer.

**Findings from `hrMelodyv3` (Hard Rock Melody / Symphony) — added 2026-06-04**
1. **No `.cursor/rules/`, `.cursorrules`, `AGENTS.md`, or authored `SKILL.md` in this project.** `.cursor/` is empty; `.claude/` holds only `settings.local.json`. The reusable IP lives in `.agent` files and `docs/`. → Install genericized R1–R4 templates here (same "rules never traveled forward" gap).
2. **Two NEW rules contributed:** **R5** (MCP/WebFetch allowlist, from `.claude/settings.local.json`) and **R6** (agent runtime guardrails, from `Symphony.agent` off_topic block). Neither duplicated an existing asset.
3. **Two NEW patterns contributed:** **P9** (probabilistic AI demo playbook, from `DEMO-EXPLAINER.md` — claims the pre-existing orphan canonical file) and **P10** (text-only API hybrid UI, from `melody-ui/`).
4. **VARIANT folded, no new ID:** `docs/agentforce-chip-menu.md` + the `show_command` force-render rule + the LEX-vs-ECv2 click pattern were folded into **P2 §14** and three framework-level lines into `rules/templates/salesforce-deploy-gotchas.mdc`. `hrMelodyv3` added to P2 provenance.
5. **DUPLICATES (not re-added):** `docs/agentforce-custom-lightning-types.md` ≈ **P2**; `docs/debug/cursor-handoff-lwc-inspect.md` is already the **P3** source; `docs/design/{A,B,C}.md` + `demo-scenario-map.md` ≈ **P4** (and duplicated with `hrMelody` per existing note). The three `.mp4` recordings are already in the Drive manifest.
6. **Intra-project duplicate to fix in the source repo:** `hrMelodyv3/melody-ui/` and `hrMelodyv3/melody-ui-Demo/` have **byte-identical** `HANDOFF.md` and `DEMO-CLICKPATH.md`. Recommend keeping `melody-ui/` canonical and archiving `melody-ui-Demo/` (the playbook itself was sourced from the Demo copy, but the two are otherwise twins).
7. **Naming drift confirmed:** Melody → Harmony → Symphony/Sonata, plus versioned planners `Harmony_v109..v112`. Consistent with the existing drift note; templates use placeholders.

**Findings from `sfdxMelodyv3` (Symphony/Sonata Hard Rock booking agent — the SFDX sub-project of `hrMelodyv3`) — added 2026-06-03**
1. **Three NEW debug-workflow patterns contributed:** **P11** (append-only evidence log), **P12** (fresh-session briefing), **P13** (layer-elimination debugging) — sourced from `docs/debug/{symphony-carousel.md, handoff-symphony-state.md}`. With **P3** (cross-surface handoff, already sourced from this project) they form a coherent **multi-session/multi-tool debugging toolkit**: P3 delegates → P13 produces evidence → P11 stores it → P12 re-briefs the next session. Cross-linked in the Slack Canvas.
2. **One NEW rule contributed:** **R7** (context-scoping ignore file), from this project's `.cursorignore`/`.claudeignore` that excluded the dead `Harmony` agent. First ignore-file-based asset in the catalog; complements R1's COPY/SKIP registry.
3. **VARIANTs folded, no new IDs:** CLT-render deploy gotchas (`c__` prefix immutability, `renderer.json` `attributes` ban, `agent publish` strips UI Output-Rendering bindings, ESD re-publish requirement, Builder-preview-doesn't-render, chip-menu single-renderer) appended to `rules/templates/salesforce-deploy-gotchas.mdc` with provenance. Complementary to the `hrMelodyv3` fold of the same chip-menu doc into **P2 §14** (that captured the *recipe*; this captures the *recurring deploy failures*).
4. **DUPLICATEs confirmed (not re-added):** `docs/agentforce-custom-lightning-types.md` (Aki Hirano deck) + the collection-renderer contract ≈ **S1 / P2**; the `show_command` force-render technique is inside **S1**; `docs/design/{A,B,C}.md` ≈ **P4**; agent Global Instructions (`A_agent_goals.md` §4) ≈ **R6**; `docs/debug/cursor-handoff-lwc-inspect.md` is the **P3** source.
5. **Canonical-source note:** for the duplicated CLT material, the `hrMelody`/`sfdxMelodyv3` `docs/agentforce-custom-lightning-types.md` and the repo's `patterns/canonical/lightning-types-best-practices.md` (P2) overlap; **keep P2 canonical** — it's the doc-derived, version-stamped consolidation. The Aki Hirano deck is *supporting material* → added to the Drive manifest (Decks), not re-homed as code.
6. **Naming-drift, intra-project:** `docs/demo-scenario-map.md` carries a stale `Harmony/Melody` header while its body tracks Symphony/Sonata. Fix in-project; reinforces the `PROJECT_NAME`/`ORG_ALIAS` placeholder discipline (no shared template hardcodes these).

**Findings from `FHA Call Center` (HUD Agentforce Service Assistant) — added 2026-06-04**
1. **Two NEW rules contributed:** **R8** (Agentforce metadata operational rule, from `docs/Lessons-Learned-Agent-Sessions.md`) and **R9** (single-component architecture lock, from `sf-docs-mcp/.cursor/rules/core.mdc` + its `AGENTS.md`). Neither duplicated an existing asset.
2. **One NEW skill contributed:** **S3** (`sf-docs-mcp`, Salesforce-docs→Markdown MCP server). Packaged as `skills/sf-docs-mcp/` (SKILL.md + README). The full TS server stays in the source project and should be promoted to its own GitHub repo.
3. **One NEW pattern contributed:** **P22** (Service Assistant quick-actions/subagents grounding model). No existing Service-Assistant asset — distinct from the autonomous Agent Script library (P1). *(ID note: a concurrent `FWA Federal Fraud` contribution was landing P19–P21 at the same time; to avoid collision this asset took the next free integer, **P22**. No numbers reused. NB: the FWA cards leave a P18 gap in the card section — that's the FWA contribution's to reconcile, not renumbered here.)*
4. **VARIANTs folded, no new IDs:** (a) FHA's deploy lessons (named-component deploys, `GenAiPlugin`/`GenAiPlanner` Metadata-API gotchas, sandbox test-skip) folded into `rules/templates/salesforce-deploy-gotchas.mdc`; (b) `docs/Plan-Quick-Actions-Wiring.md`'s "## Deploy Commands" + "## What Is NOT Changing" blocks folded into **P4** (`phased-delivery-structure.md`); (c) `docs/HUD_FHA_Asset_Catalog.md` folded into **P5** as the "as-built asset inventory" flavor (Integration-status + Gaps tables). FHA added to each asset's provenance.
5. **DUPLICATE confirmed (not re-added):** `.cursorignore` (excludes `docs/research/` + `docs/FAQ/` for context scoping) is a **DUPLICATE of R7** (context-scoping ignore file). FHA's copy excludes *noise docs* rather than *dead code* — same mechanism, same ID; not re-homed.
6. **Inherited platform skills NOT re-homed:** the 23 `sf-*` skills under `.agents/skills/` are installed from the public `Jaganpro/sf-skills` GitHub repo (confirmed via `skills-lock.json`), not authored IP — referenced, not re-homed (consistent with the S-category rule).
7. **Promote-to-repo recommendation:** the `sf-docs-mcp` TypeScript server is genuinely forkable IP currently living inside a customer project. Recommend extracting it to a standalone GitHub repo so S3's skill doc can link a real install source.

**Findings from `NASA NSPIRES PostAwardBuild` (NASA NSPIRE post-award, PSS Grantmaking) — added 2026-06-04**
1. **No rules/skills/AGENTS.md exist in this project.** `.cursor/` is empty; no `.cursorrules`, no authored `SKILL.md`, no `AGENTS.md`; `.claude/settings.local.json` is only a docs-scrape permission allowlist (an **R5** instance, not re-added). → Install genericized R1–R4 templates here (same "rules never traveled forward" gap as `hrMelodyv3` / `PostAwardExtract`).
2. **Four NEW patterns contributed:** **P14** (Flow Approval Processes library), **P15** (OmniStudio DocGen recipe), **P16** (Grantmaking/PSS feature reference), **P17** (doc-derived implementation-guide workflow — the meta-pattern behind P14–P16). The three guides claim the previously-flagged orphan canonical files `nspires-postaward-{flow-approval-processes,omnistudio-docgen,grantmaking-psc-features}-guide.md` (see resolved orphan note above).
3. **Sibling-not-duplicate vs. P7:** P16 (PSS *feature/permissions* reference) and **P7** (PSS *object/field schema* reference, from `PostAwardExtract`) cover the same product from different angles — schema vs. behavior. Kept distinct; cross-referenced in both cards. Recommend a future merge into a single "PSS Grantmaking" library folder if a third PSS contribution lands.
4. **VARIANTs folded, no new ID:** `docs/award-demo-guide.md` (RTM-cited, step-by-step demo script tied to build phases) reinforces **P4** (phased-delivery/demo) and **P9** (demo playbook) — added to P4 provenance, not re-homed (NASA-specific content). `docs/oldorg-inventory.md` (Phase-0 subagent-generated metadata inventory) is a concrete instance of **P5** and overlaps **P8** (org metadata extraction) — added to P5 provenance.
5. **Drive (supporting material, not code):** `docs/Requirements Document: NSPIRES Moderniza.md` (customer requirements/RTM background) and `docs/Public Sector & Nonprofit – Grantmaking.json` (28 KB org metadata export) → Google Drive (Customer Background / Org Exports). Not versioned in the repo.
6. **Cross-project duplicate watch:** the PSS Grantmaking material now appears across `PostAwardExtract` (P7, schema) and `PostAwardBuild` (P16, features). Both pin Summer '26 / API v6x — keep each card's header version authoritative and re-stamp per release (P7/P17 convention).

**Findings from `FWA Federal Fraud` (synthetic-data → Data Cloud → GNN → Agentforce/MCP/Slack) — added 2026-06-03**
1. **This project HAS its own `.cursor/rules/` (unlike most prior contributors).** Four `.mdc` rules + a sub-project rule, plus `CLAUDE.md` orientation files and Data Cloud shell skills. So the contribution is real assets, not just "install templates here."
2. **One NEW rule contributed:** **R10** (Verification & Confidence Protocol, from `data-360.mdc`) — the first "never assert schema/API from memory; verify or say so" rule with a query-router and validate-before-write workflow. Canonical kept as `rules/canonical/fwa-data-360.mdc`.
3. **One NEW skill contributed:** **S4** (`data-cloud-sql-runner`, from `salesforce/.cursor/skills/dc-*.sh`) — packaged with `SKILL.md` + `README.md` + the three real scripts. First *executable-shell* skill in the catalog (S1–S3 are recipe/MCP skills); it's the deterministic enforcement arm of R10.
4. **Three NEW patterns contributed:** **P19** (live-demo run-of-show + fallback matrix, from the SE runbook/customer-demo playbook), **P20** (synthetic demo-data design/load/teardown, from `demo-data-guide.md`), **P21** (headless multi-surface MCP tool-plane, from `headless-360-build-plan.md`). Two full as-shipped docs preserved in `patterns/canonical/` (`fwa-headless360-se-runbook.md`, `fwa-demo-data-guide.md`). *(Note: P18 was already claimed by `FHA Call Center` for Service Assistant — these continue from P19.)*
5. **VARIANTs folded, no new IDs:**
   - `doc-sync.mdc` → DUPLICATE of **R2** (near-identical structure; the repo's R2 was *already sourced* from this exact pattern). Not re-added; FWA reinforces R2.
   - `workspace-map.mdc` (compact "where things live" map) → VARIANT of **R3**; reinforces the goals/doc-map idea. Not re-added.
   - `graph-schema.mdc` (always-on domain constants: node/edge types, typologies, naming) → VARIANT of **R1** (the "lock the constants so the agent doesn't reinvent them" flavor). Project-specific content; not re-homed.
   - `CLAUDE.md` / `demo/CLAUDE.md` "What NOT to do" + source-of-truth pointers → VARIANT of **R4**; folded a generic **"What NOT to Do (behavior guardrails)"** section into `rules/templates/AGENTS.md` with provenance.
   - `docs/templates/ReactAppContext.md` (a "here's what exists / backend contract / migration task" prompt) → VARIANT of **P3** (cross-surface handoff, migration-prompt flavor). Noted, not re-homed (FWA-specific).
6. **DUPLICATE-ish (not re-added):** `tools/sf-docs-mcp/` is the **same project** that is the catalog's **S3 / R9** source (`FHA Call Center` also shipped it). The FWA copy confirms S3/R9 generalize across projects; keep the existing canonical. `headless360-skills.md` overlaps the **P1/P5** "what is a skill / inventory" commentary — reference, not re-homed.
7. **Resolved a prior false alarm:** the FWA `dc-*.sh` scripts were briefly thought missing (a workspace-root scoping artifact); they exist at `salesforce/.cursor/skills/` exactly where the rules say. No path-drift bug.
8. **Demo-data drift to fix in the source project (not a repo issue):** FWA's four demo entities' unified IDs/scores disagree across `graph-schema.mdc`, the SE runbook, and `demo/CLAUDE.md` (IDs legitimately drift on Identity-Resolution re-run). The runbook already prescribes the fix (search by EIN, re-derive IDs) — captured here as a P20 "IDs that drift" caution.

**Findings from `ItineraryBuilder` (DOS PCS Portal — Experience Cloud / LWR; LWC + Apex, no Agent Script) — added 2026-06-04**
1. **No authored Cursor primitives in this project.** `.cursor/` is empty; no `.cursorrules`, no `AGENTS.md`/`CLAUDE.md`, no authored `SKILL.md`. All reusable IP lived as prose in markdown design docs (`AGENT_REFERENCE*.md`, QA Addendum, Phase 1.5 spec, UI-polish doc, `README.md`, `FIX-LOCATION-PICKLISTS.md`). Same "rules never traveled forward" gap as `hrMelodyv3`/`PostAwardExtract` → install the genericized R1–R4 + new R11–R13 templates here.
2. **First non-Agentforce / LWR-LWC contributor.** Prior catalog skewed Agentforce/Agent-Script; this project adds the LWC/Experience-Cloud surface (S5, S6, R13, P22, P24, P25) — broadening the catalog beyond agents.
3. **Three NEW rules:** **R11** (MCP-first org verification), **R12** (override-layer precedence), **R13** (design-language token rule, glob-scoped to LWC). None duplicated existing rules (R11 is distinct from R5's MCP *allowlist* and R10's generic *protocol*).
4. **Two NEW skills:** **S5** (`lwc-fullcalendar-integration`), **S6** (`sf-dependent-address-picklist`) — both packaged with `SKILL.md` + an as-shipped `README.md`.
5. **Five NEW patterns:** **P22** (LWC wire-vs-imperative), **P23** (Ask→Plan→Agent co-authoring), **P24** (LWC concurrent-save queue+debounce), **P25** (advisory validation), **P26** (scoped UI-polish prompt pack).
6. **VARIANTs folded, no new IDs:** *Locked Decisions* table → `agent-script-architecture.mdc` (R1) new optional section; LWR/LWC + compound-address deploy failures (7 lines) → `salesforce-deploy-gotchas.mdc`; phase-gating refinements → `phased-delivery-structure.md` (P4). `ItineraryBuilder` added to all three templates' provenance.
7. **DUPLICATE (not re-added):** the Phase 1 / 1.5 / 2 roadmap is a DUPLICATE of **P4** — only its two cheap refinements (explicit per-phase non-goals; `// TODO Phase N` code-seam markers) were folded into P4.
8. **Intra-project duplicate / drift to fix in the source repo (not a repo issue):** **two `AGENT_REFERENCE` files disagree** on Case-Id delivery — root `AGENT_REFERENCE_3.md` says URL param via `CurrentPageReference`; `DesignDocs/AGENT_REFERENCE.md` says `@api recordId` via LWR route. The `DesignDocs` copy is **newer** (post route-refactor, commit `785323e`) and should be **canonical**; archive/retire the root one. Logged in `patterns/canonical/itinerary-builder-lwr-reference.md`.
9. **Cross-project naming note:** this repo lives under a Hard Rock Drive path but now catalogs assets from unrelated engagements (DOS PCS, NASA NSPIRES, HUD FHA, FWA). Consider renaming the repo root / README scope line to a neutral "GPS Cursor Best Practices" to reflect that it's now multi-account. (Flag only — not changed here.)
