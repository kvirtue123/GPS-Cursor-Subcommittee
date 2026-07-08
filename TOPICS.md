# Find by topic

`BEST-PRACTICES.md` is organized by *asset type* (Rules / Skills / Patterns / Knowledge). This file is organized by **feature area**, so you can answer "I'm starting a new project and need everything on X" in one lookup instead of searching four separate lists.

For each topic: grab the **Pattern/Skill** first (the reusable, install-ready asset), read the **Rule** if one applies to your project type, and use the **Knowledge** doc only if you need deeper reference material than the Pattern already gives you.

---

## Agent Script (the DSL)

- **Pattern:** [`patterns/canonical/agent-script-library.md`](patterns/canonical/agent-script-library.md) — **P1**, the curated playbook (patterns index, examples, reference)
- **Skill:** [`skills/agentforce-agentscript-clt/`](skills/agentforce-agentscript-clt/) — **S1**, building/debugging Custom Lightning Types *for* Agent Script agents
- **Rule:** [`rules/templates/agent-script-architecture.mdc`](rules/templates/agent-script-architecture.mdc) — **R1** variant, architecture lock for `.agent` file projects
- **Knowledge (raw source, already = P1):** `knowledge/agent-script/` (13-chapter compiled guide), `knowledge/agent-script.md`, `knowledge/ascript-*.md` (8 files), `knowledge/agent-api.md`

## Agentforce instruction & topic design (subagent content, not syntax)

- **Pattern:** [`patterns/canonical/agentforce-instruction-library.md`](patterns/canonical/agentforce-instruction-library.md) + [`agentforce-topic-design-patterns.md`](patterns/canonical/agentforce-topic-design-patterns.md) — **P28**
- **Rule:** [`rules/templates/agentforce-metadata-ops.mdc`](rules/templates/agentforce-metadata-ops.mdc) — **R8**, deploy/metadata discipline (does *not* cover instruction wording — that's P28)

## Custom Lightning Types (CLT) / Chip Menu

- **Pattern:** [`patterns/canonical/lightning-types-best-practices.md`](patterns/canonical/lightning-types-best-practices.md) — **P2** (13-section recipe incl. §14 chip-menu gotchas)
- **Skill:** [`skills/agentforce-agentscript-clt/`](skills/agentforce-agentscript-clt/) — **S1**
- **Rule:** deploy gotchas folded into [`rules/templates/salesforce-deploy-gotchas.mdc`](rules/templates/salesforce-deploy-gotchas.mdc) — **R1** companion
- **Knowledge (raw source, already = P2):** `knowledge/agentforce-custom-lightning-types.md`, `knowledge/agentforce-chip-menu.md`

## Agentforce Service Assistant

- **Pattern:** [`patterns/canonical/service-assistant-quick-actions.md`](patterns/canonical/service-assistant-quick-actions.md) — **P22** (quick-actions/subagents grounding model)
- **Knowledge (raw source, already = P22):** `knowledge/Service-Assistant-Quick-Actions-Subagents.md`, `knowledge/Plan-Quick-Actions-Wiring.md` (folded into **P4**)

## Agentforce Voice / telephony

- **Pattern:** [`patterns/canonical/agentforce-voice-best-practices.md`](patterns/canonical/agentforce-voice-best-practices.md) — **P29**

## Agentforce metadata / deploy discipline (`GenAiPlugin`, `GenAiPlanner`)

- **Rule:** [`rules/templates/agentforce-metadata-ops.mdc`](rules/templates/agentforce-metadata-ops.mdc) — **R8**; pairs with [`rules/templates/salesforce-deploy-gotchas.mdc`](rules/templates/salesforce-deploy-gotchas.mdc) — **R1** companion
- **Knowledge (raw source, already = R8):** `knowledge/Lessons-Learned-Agent-Sessions.md`

## Data Cloud (Data 360)

- **Pattern:** [`patterns/canonical/datacloud-theme-reference/`](patterns/canonical/datacloud-theme-reference/) — **P27** (data model, identity resolution, segmentation, analytics, ingestion, data graphs, AI/LLM models)
- **Rule:** [`rules/templates/verification-confidence-protocol.mdc`](rules/templates/verification-confidence-protocol.mdc) — **R10**, never guess a Data Cloud field/object name
- **Skill:** [`skills/data-cloud-sql-runner/`](skills/data-cloud-sql-runner/) — **S4**, query/describe Data Cloud objects from the shell

## Public Sector Solutions / Grantmaking

- **Pattern (schema):** [`patterns/canonical/grantmaking-object-reference.md`](patterns/canonical/grantmaking-object-reference.md) — **P7**
- **Pattern (features/permissions):** `patterns/canonical/` — **P16** *(canonical file — maintainers only)*
- **Pattern (org-extraction method):** [`patterns/canonical/org-metadata-extraction-audit.md`](patterns/canonical/org-metadata-extraction-audit.md) — **P8**
- **Knowledge (raw source, already = P7/P8/P16):** `knowledge/grantmaking-objects.md`, `knowledge/awardDetails.md`, `knowledge/grantmaking-psc-features-guide.md`

## Salesforce Flow Approval Processes

- **Pattern:** `patterns/canonical/` — **P14** *(canonical file — maintainers only)*
- **Knowledge (raw source, already = P14):** `knowledge/flow-approval-processes-implementation-guide.md`

## OmniStudio Document Generation

- **Pattern:** `patterns/canonical/` — **P15** *(canonical file — maintainers only)*
- **Knowledge (raw source, already = P15):** `knowledge/omnistudio-document-generation-guide.md`

## Headless multi-surface MCP (Headless 360)

- **Pattern (system topology):** [`patterns/templates/headless-mcp-tool-plane.md`](patterns/templates/headless-mcp-tool-plane.md) — **P21**
- **Pattern (Cursor prompt packs):** [`patterns/canonical/headless360-mcp-tools-prompt-pack.md`](patterns/canonical/headless360-mcp-tools-prompt-pack.md) + [`headless360-react-prompt-pack.md`](patterns/canonical/headless360-react-prompt-pack.md) — **P30**
- **Rule:** [`rules/templates/mcp-allowlist.settings.local.json`](rules/templates/mcp-allowlist.settings.local.json) — **R5**
- **Skill:** [`skills/sf-docs-mcp/`](skills/sf-docs-mcp/) — **S3**
- **Knowledge (raw source, already = P21):** `knowledge/headless-360-build-plan.md`, `knowledge/headless360-skills.md` (reference-only)

## Demoing (live, probabilistic AI, synthetic data)

- **Pattern (playbook):** [`patterns/canonical/probabilistic-ai-demo-playbook.md`](patterns/canonical/probabilistic-ai-demo-playbook.md) — **P9**
- **Pattern (run-of-show + fallback matrix):** [`patterns/canonical/fwa-headless360-se-runbook.md`](patterns/canonical/fwa-headless360-se-runbook.md) — **P19**
- **Pattern (synthetic data):** [`patterns/canonical/fwa-demo-data-guide.md`](patterns/canonical/fwa-demo-data-guide.md) — **P20**

## LWC / Experience Cloud

- **Pattern:** wire-vs-imperative (**P22**\*), concurrent-save queue (**P24**), advisory validation (**P25**), scoped UI-polish prompt pack (**P26**) — all in `patterns/templates/`
- **Skill:** [`skills/lwc-fullcalendar-integration/`](skills/lwc-fullcalendar-integration/) — **S5**; [`skills/sf-dependent-address-picklist/`](skills/sf-dependent-address-picklist/) — **S6**
- **Rule:** [`rules/templates/design-language-tokens.mdc`](rules/templates/design-language-tokens.mdc) — **R13**

\* Note: `CATALOG-INTERNAL.md`'s recommended-home table has a numbering collision — both "Service Assistant quick-actions/subagents" and "LWC wire-vs-imperative" are listed as P22. Not fixed here (out of scope for this pass); flagged as a gap.

## Project scaffolding (any new project)

- **Rules:** `AGENTS.md` (**R4**), architecture/reuse registry (**R1**), doc-sync (**R2**), goals/doc-map (**R3**)
- **Patterns:** fresh-session briefing (**P12**), phased-delivery structure (**P4**), Ask→Plan→Agent co-authoring (**P23**)

---

*Missing a topic? Add a section here when you catalog a new asset — see `CONTRIBUTING-PROMPT.md` and each folder's `README.md` for how assets get added.*
