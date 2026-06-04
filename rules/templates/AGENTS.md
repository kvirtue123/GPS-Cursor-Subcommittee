# {{PROJECT_NAME}} — Agent Orientation

**Project:** {{PROJECT_NAME}} (API name: `{{API_NAME}}`) | **Org alias:** `{{ORG_ALIAS}}` | **Target:** {{MILESTONE_DATE}}

**Architecture guardrails (always enforce):**
- Agent Script `.agent` DSL only — never `genAiPlugin`, `genAiFunction`, or `genAiPlanner`
- {{INTEGRATION_POLICY}} — e.g. "No live HTTP callouts; external systems are mocked"
- Flows over Apex for simple SOQL — `@InvocableMethod` Apex only when Flows can't do it

---

## Start Here
Read [`{{KNOWLEDGE_REF}}`]({{KNOWLEDGE_REF}}) first. It contains the subagent registry, action inventory, FSM state register, Lightning Type registry, data model summary, token rules, and the reuse registry.

---

## Build Phase Status
| Phase | Title | Status | Doc |
|---|---|---|---|
| 1 | {{PHASE_1_TITLE}} | {{STATUS}} | [{{PHASE_1_DOC}}]({{PHASE_1_DOC}}) |
| 2 | {{PHASE_2_TITLE}} | {{STATUS}} | [{{PHASE_2_DOC}}]({{PHASE_2_DOC}}) |
| N | … | … | … |

---

## Key Doc Index
| Need | Read |
|---|---|
| Topics, actions, global instructions | [{{DESIGN_A}}]({{DESIGN_A}}) |
| FSM states, percepts, business rules | [{{DESIGN_B}}]({{DESIGN_B}}) |
| Object/field specs, picklists, ERD | [{{DESIGN_C}}]({{DESIGN_C}}) |
| Master design doc (authoritative) | [{{MASTER_DESIGN}}]({{MASTER_DESIGN}}) |

---

## What NOT to Touch
- {{EXCLUSION_LIST}} — Named Credentials, live API Apex, etc. excluded from this build.

---

## What NOT to Do (behavior guardrails)
> Stop the agent re-deriving known facts. Each line points at the single source of truth instead.
- Do **not** re-derive {{SCHEMA_THING}} (table/field/API names) from memory — they live in {{SCHEMA_SOURCE}}; verify there.
- Do **not** rewrite {{DERIVED_DOCS}} — update them from the source-of-truth docs, don't regenerate from scratch.
- Do **not** guess {{MEASURED_VALUES}} (metrics, scores, accuracy) — read {{METRICS_SOURCE}}.
- Do **not** re-explore the repo each session — start from this file + the doc map.

> Provenance note: the "what NOT to do" + compact "source-of-truth pointers" pattern was reinforced by the FWA federal fraud project's root and `demo/` `CLAUDE.md` orientation files.
