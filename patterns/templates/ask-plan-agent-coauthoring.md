# Ask → Plan → Agent Co-Authoring Lifecycle — Template

A three-stage workflow for producing design docs and then implementing against them, structured around the Cursor Ask → Plan → Agent lifecycle. Forces alignment *before* code is written.

> Source pattern: `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder) — AGENT_REFERENCE "Structured Co-Authoring Workflow".

## Stage 1 — Context Gathering
1. Review the current state (data model, existing components) — confirm what exists vs. what's proposed.
2. Close gaps — ask clarifying questions (auth, approvals, lookups, drafts, AI features…).
3. Document assumptions explicitly as you go.

## Stage 2 — Refinement & Structure
1. Iterate **one section at a time**.
2. Brainstorm options; refine on feedback.
3. Write so a developer can implement without guessing — developer-ready, not aspirational.

## Stage 3 — Reader Testing (the Cursor lifecycle)
1. **Ask** — the agent reads all docs and asks clarifying questions before doing anything.
2. **Plan** — the agent produces an implementation plan; human reviews/approves before any code.
3. **Agent** — the agent executes the approved plan, **one file at a time**.

## Why it works
- Separates "deciding what to build" from "building it" — fewer wasted edits.
- The mandated reading order + approval gate prevents the agent from charging ahead on a wrong assumption.

### When NOT to use
Tiny, unambiguous changes where a plan-review gate is pure overhead.

<!-- Provenance: contributed from project `ItineraryBuilder`. Catalog ID P23. -->
