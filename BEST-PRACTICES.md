# Best Practices Catalog

This is the GPS Cursor Subcommittee's shared catalog of reusable Cursor assets. Every entry is extracted from a real GPS delivery project — not invented speculatively.

Assets are organized into three categories: **Rules**, **Skills**, and **Patterns**. Each asset has a stable ID (e.g. `R1`, `S1`, `P1`) and will be documented here as it's released. Installable files drop alongside weekly posts in the Slack channel.

💬 **Channel:** #gps-ae-club-cursor-subcommittee
🔗 **Repo:** https://github.com/kvirtue123/GPS-Cursor-Subcommittee

---

## Rules

Rules are behavioral guardrails installed into a Cursor project as small config files (`.mdc` format, placed in `.cursor/rules/`). They shape how the AI agent behaves during a session — what architecture to follow, when to update documentation, what to verify before writing code, what to ignore.

Two types:
- **Always-on** — loaded on every agent request. Use sparingly; they cost tokens on every interaction.
- **Glob-scoped** — loaded only when relevant files are touched. The preferred approach for most rules.

The catalog currently contains **13 rules** (R1–R13), covering:
- Architecture locks and reuse registries
- Doc-sync and context-scoping
- MCP tool allowlists and verification protocols
- Agentforce metadata operations
- Design language enforcement for LWC

_Installable templates will be added to `rules/templates/` as they are introduced in the channel._

---

## Skills

Skills are reusable task recipes — Markdown files that give the agent a complete, tested workflow for a specific job. You install them once and invoke them by name.

The catalog currently contains **6 skills** (S1–S6), covering:
- Building and debugging Custom Lightning Types (CLTs) for Agentforce
- Generating branded GPS whitepaper PDFs from Markdown
- Making Salesforce documentation LLM-readable via a custom MCP server
- Running verified queries against Salesforce Data Cloud
- Embedding FullCalendar in a Lightning Web Component
- Driving dependent State/Country address picklists in LWC

_Packaged skill files will be added to `skills/` as they are introduced in the channel._

---

## Patterns

Patterns are reusable workflow templates — Markdown files you keep in your project's `docs/` folder and reference in context. Not installed code; structured guidance.

The catalog currently contains **26 patterns** (P1–P26), covering:
- Delivery structure and phased project management
- Multi-session debugging workflows
- Demo preparation and fallback planning
- Synthetic demo data generation
- Multi-surface agent architecture
- LWC data access, validation, and auto-save patterns
- Co-authoring workflows with Cursor's Ask/Plan/Agent modes

_All pattern templates are available now in `patterns/templates/`._

---

## How to contribute

If something worked on a project — or a failure mode is worth encoding so no one else hits it — share it. The bar: it came from a real project, it's reusable in more than one context, and it has a clear "use this when / not when."

Post in the channel Feed tagged `[Rule]`, `[Skill]`, or `[Pattern]`. A co-lead will triage and assign an ID.

---

_Updated as assets are released · GPS Cursor Subcommittee · Leads: Keegan Virtue, Steve Shin_
