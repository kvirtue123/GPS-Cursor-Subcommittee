# Knowledge

Raw domain/product reference material — scraped official docs, feature guides, language references — that hasn't been (or doesn't need to be) formally packaged as an installable **Rule**, **Skill**, or **Pattern**. If Rules/Skills/Patterns are "things you install," `knowledge/` is "things you read."

See the [root README](../README.md) for how this fits alongside Rules/Skills/Patterns, and **[`TOPICS.md`](../TOPICS.md)** for a feature-area index that cross-links everything (Rules + Skills + Patterns + Knowledge) by subject — that's the fastest way to answer "what do I need for X on a new project?"

## How this folder works

- **Flat, not nested by feature area.** Every file lives directly in `knowledge/` (with the one exception of `agent-script/`, a pre-existing 13-chapter guide kept intact as a unit). Feature area is a **tag**, not a folder — see the frontmatter block at the top of each file:

  ```yaml
  ---
  feature_area: agentforce-agent-script   # which topic bucket this belongs to
  status: new | duplicate-of-catalog | folded-into-catalog | reference-only
  catalog_id: P1                          # set if this file already has a Rule/Skill/Pattern ID elsewhere
  catalog_ref: patterns/canonical/...      # path to that asset, if any
  note: "..."                             # human-readable context
  ---
  ```

- **`status: duplicate-of-catalog` / `folded-into-catalog` files are intentionally kept here** even though the same material is already a cataloged Rule/Skill/Pattern. This folder is a browsable knowledge base, not a one-time staging inbox — but the frontmatter means nobody should re-catalog them as a *new* asset. Always check the `catalog_id` before adding something to `BEST-PRACTICES.md`.
- **`status: new` files are things that haven't been evaluated for cataloging yet.** If you drop a new file into `knowledge/`, add frontmatter with `status: new` and a best-guess `feature_area`; a maintainer will triage it into a Pattern/Rule/Skill if it's reusable, or leave it here as pure reference if not.
- **`status: reference-only`** means it was reviewed and deliberately *not* cataloged as its own asset (e.g. it overlaps commentary already covered elsewhere) — see its `note` for why.

## Feature areas in use

| `feature_area` tag | What's in it |
|---|---|
| `agentforce-agent-script` | Agent Script (DSL) language docs, examples, API reference — **already cataloged as P1**, see `patterns/canonical/agent-script-library.md` |
| `agentforce-lightning-types` | Custom Lightning Type implementation + chip-menu renderer notes — **already cataloged as P2** |
| `agentforce-metadata-ops` | `GenAiPlugin`/deploy lessons-learned — **already cataloged as R8** |
| `agentforce-service-assistant` | Service Assistant quick-actions/subagents grounding model — **already cataloged as P22** (with a variant folded into P4) |
| `salesforce-flow` | Flow Approval Processes implementation guide — **already cataloged as P14** |
| `omnistudio` | OmniStudio Document Generation guide — **already cataloged as P15** |
| `grantmaking-pss` | PSS/Grantmaking objects, features, and org-extraction audit — **already cataloged as P7, P8, P16** |
| `headless360` | Headless 360 build plan + skills commentary — **already cataloged as P21** (prompt packs moved out to P30) |

New/uncataloged material that came out of this same audit was promoted straight into `patterns/canonical/` rather than left here, since it's genuinely new IP:

- **P27** — Data Cloud (Data 360) theme reference library → `patterns/canonical/datacloud-theme-reference/`
- **P28** — Agentforce instruction & topic design pattern library → `patterns/canonical/agentforce-instruction-library.md`, `agentforce-topic-design-patterns.md`
- **P29** — Agentforce Voice refactoring best practices → `patterns/canonical/agentforce-voice-best-practices.md`
- **P30** — Headless 360 Cursor build-mode prompt packs → `patterns/canonical/headless360-mcp-tools-prompt-pack.md`, `headless360-react-prompt-pack.md`

One file was routed elsewhere: `AI Capabilities in Government Cloud Plus.md` was internal-only sales/SE enablement content, not reusable dev IP — moved to `discovery/ai-capabilities-government-cloud-plus.md` (internal-only).

## Adding a new knowledge file

1. Drop the `.md` file directly into `knowledge/` (flat — no new subfolders).
2. Add the frontmatter block above. Pick an existing `feature_area` tag if it fits, or introduce a new one and add a row to the table above.
3. Search `BEST-PRACTICES.md` / `CATALOG-INTERNAL.md` for the same topic first — if it's already a Rule/Skill/Pattern, set `status: duplicate-of-catalog` and point `catalog_ref` at it instead of leaving it as `status: new`.
4. If it's genuinely new and reusable across projects, flag it for promotion to `patterns/canonical/` + a `BEST-PRACTICES.md`/`CATALOG-INTERNAL.md` card (next sequential `P#`) rather than leaving it here indefinitely — `knowledge/` is for reference material, not a place installable assets go to hide.
