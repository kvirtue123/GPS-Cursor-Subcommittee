# Best Practices Catalog

The inventory of what's in the catalog. For what these asset types are, how to use them, and how to contribute, see the **[README](README.md)**.

Each asset has a stable ID (`R#`, `S#`, `P#`). Installable files live in their folders as they're added.

---

## Rules (R1–R*)

Behavioral guardrails installed as `.mdc` files in `.cursor/rules/`. Two kinds: **always-on** (loaded on every request — use sparingly) and **glob-scoped** (loaded only when relevant files change — preferred for most).

Examples:

- Architecture locks and reuse registries
- Doc-sync and context-scoping
- MCP tool allowlists and verification protocols
- Agentforce metadata operations
- Design language enforcement for LWC

→ `rules/templates/`

---

## Skills (S1–S*)

Examples:

- Building and debugging Custom Lightning Types (CLTs) for Agentforce
- Generating branded GPS whitepaper PDFs from Markdown
- Making Salesforce documentation LLM-readable via a custom MCP server
- Running verified queries against Salesforce Data Cloud
- Embedding FullCalendar in a Lightning Web Component
- Driving dependent State/Country address picklists in LWC

→ `skills/`

---

## Patterns (P1–P*)

Examples:

- Delivery structure and phased project management
- Multi-session debugging workflows
- Demo preparation and fallback planning
- Synthetic demo data generation
- Multi-surface agent architecture
- LWC data access, validation, and auto-save patterns
- Co-authoring workflows with Cursor's Ask/Plan/Agent modes

→ `patterns/templates/`

---

*GPS Cursor Subcommittee · Leads: Keegan Virtue, Steve Shin*