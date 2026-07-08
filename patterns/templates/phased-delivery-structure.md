# Phased-Delivery + Design-Doc-Map — Template

A repeatable project structure that separates **design** from sequenced **build phases**, giving humans and agents a deterministic, resumable, auditable execution path.

> Source pattern: `hrMelody/docs/{design,build}/` + `Melody_R2_Design_Document.md`.

---

## Folder layout

```
docs/
├── design/
│   ├── A_agent_goals.md        ← topics, actions, global instructions, assumptions
│   ├── B_percepts_states.md    ← percepts, FSM transitions, business rules, metrics
│   └── C_data_model.md         ← object/field specs, picklists, validation, ERD
├── build/
│   ├── phase-1-<name>.md       ← self-contained: deploy order, COPY-vs-BUILD, deps, commands
│   ├── phase-2-<name>.md
│   └── phase-N-<name>.md
├── knowledge/                  ← reference material (e.g. Agent Script library)
└── <Master_Design_Document>.md ← authoritative consolidation of design/A,B,C
```

## Rules of the pattern
1. **Design vs. build are separate.** Design docs hold *decisions*; build docs hold *steps*. Never mix.
2. **Each build phase is self-contained.** It states its CLI commands, exact relative file paths, dependencies on earlier phases, and labels every artifact **COPY** (verbatim from a source) or **BUILD** (written from scratch).
3. **Phases execute in order.** Later phases depend on earlier deploys (e.g. roll-up summary after child object).
4. **A status table lives in `AGENTS.md`** so anyone can see progress at a glance (Complete / In progress / Not started).
5. **The Master Design Doc stays in sync** with design/A,B,C (enforced by the `doc-sync.mdc` rule).

## Canonical phase-doc skeleton
```markdown
# Phase N — <Title>
**Status:** <Not started | In progress | ✅ Complete>
**Workspace root / force-app path:** …

## N.0 Overview
| Deploy Order | Artifact | Action (COPY/BUILD/RUN) | Dependency |
|---|---|---|---|
| 1 | … | … | … |

## N.1 <Artifact>
<commands, paths, notes>
```

## Pairs with
- `goals-doc-map.mdc` (R3) — `@`-references this structure.
- `doc-sync.mdc` (R2) — keeps it fresh cheaply.

### When NOT to use
Tiny one-shot changes. Don't scaffold four phase docs for a two-file edit.

## Phase-gating refinement (reinforced by `ItineraryBuilder`)
Two cheap, high-value additions seen in a second project that uses this pattern:
- **Explicit "do NOT build X yet" lines** in each phase scope (e.g. "Phase 1: no rate lookup, no AI flagging, no multiple drafts"). Prevents scope creep into future phases.
- **`// TODO Phase N:` code markers** at the exact seams where future work attaches (e.g. at a null-check where a "Create" path lands later), so the empty state is trivially upgradable.

<!-- Provenance: phase-gating refinement contributed from project `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder); its Phase 1 / 1.5 / 2 roadmap is a DUPLICATE of this pattern, folded here rather than re-IDed. -->

## Two-block refinements (reinforced by `FHA Call Center`)
A plan doc (`Plan-Quick-Actions-Wiring.md`) using this pattern added two cheap blocks worth copying into every phase plan:
- **A per-phase `## Deploy Commands` block** with the exact named-component `sf project deploy start --metadata "..."` invocation, so executing a phase is copy-paste, not recall. (Pairs with `salesforce-deploy-gotchas.mdc` / `agentforce-metadata-ops.mdc`.)
- **A closing `## What Is NOT Changing` block** that enumerates assets deliberately left alone. Prevents the agent from "helpfully" editing untouched files and makes the blast radius auditable.

<!-- Provenance: deploy-command + "What Is NOT Changing" blocks contributed from project `FHA Call Center` (docs/Plan-Quick-Actions-Wiring.md), a VARIANT of this pattern, folded rather than re-IDed. -->

