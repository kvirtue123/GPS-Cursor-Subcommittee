# Fresh-Session Briefing — Template

A self-contained, hand-written document that brings a brand-new agent session fully up to
speed with **no prior conversation history needed**. Distinct from the cross-surface handoff
(P3): P3 delegates *one task* to *another surface*; this briefs *a fresh session on the same
surface* about overall project state so you can start clean without losing context (e.g. after
autocompaction, a crash, or just opening a new chat the next day).

> Source pattern: `sfdxMelodyv3/docs/debug/handoff-symphony-state.md`
> ("Use this doc to brief a fresh chat session. Self-contained context — no prior conversation needed.").

---

## Template

```markdown
# {{PROJECT}} Brief — Current State ({{DATE}})

> **Use this doc to brief a fresh chat session.** Self-contained context — no prior conversation needed.

## What we're building
{{1–2 paragraphs: the agent/app, the org, where the source lives, the demo/goal.}}

## What works
- {{verified-working capability}} — {{evidence}}

## What does NOT work
- {{current blocker, stated as the literal symptom}}

## Root cause (narrowed)
{{best current understanding of WHY it's broken, with the reasoning.}}

## Known paths to fix
### Path A — {{name}} ({{recommended? why}})
1. {{step}}
### Path B — {{name}} ({{status: unverified / exploring}})
1. {{step}}

## HARD CONSTRAINTS going forward
- **{{DO NOT do X}}** — {{why it breaks things; link the memory/gotcha that proves it.}}

## Context strategy for the new chat
Before doing anything else:
1. Read {{MEMORY/notes file}} — key entries: {{list}}
2. Read {{evidence log path}} end-to-end (see pattern P6).
3. Read {{design / scenario doc}} for {{what}}.
4. Read {{the primary source file}} for current state.

## Useful state
- Org / alias: {{…}}   - Agent/app API name: {{…}}   - Latest version: {{…}}
- Demo URL / key IDs / test data: {{…}}

## Open questions for the new chat
1. {{the unresolved decision the next session should drive}}

## Phases / work not yet done
- {{item}} — {{status}}
```

---

## Rules of the pattern
1. **Self-contained.** Assume zero memory of the prior chat. Inline or `@`-reference everything needed.
2. **State, not narrative.** "What works / doesn't / root cause / constraints" — not a transcript.
3. **Lead with constraints.** The HARD CONSTRAINTS section prevents the new session from repeating a destructive action (e.g. a publish that strips config).
4. **Point at the durable log.** The "context strategy" section should send the new session to the append-only evidence log (P6) and memory entries rather than re-deriving findings.
5. **Refresh, don't append.** Unlike the evidence log (P6), this is a *living snapshot* — overwrite it to reflect current state each time you hand off to a new session.

### When NOT to use
If the work is continuing in the *same* session with context intact, you don't need a brief. This is for crossing a session boundary, not a tool boundary (use P3 for that).
