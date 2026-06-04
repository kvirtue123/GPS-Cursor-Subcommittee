# Scoped UI-Polish Prompt Pack — Template

A way to package several small, independent UI changes as a sequenced, self-contained prompt pack — each item names its component, exact file changes, model to use, expected result, a confirm-before-proceeding gate, and a per-item rollback.

> Source pattern: `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder) — `PCSPortal_UIPolish_DesignDoc`.

## When to use
- A batch of small, isolated visual/CSS refinements across components that you want to run one at a time, deploy, and eyeball before moving on.
- Delegating cheap changes to a smaller/faster model while reserving expensive models for hard work.

## When NOT to use
- Cross-cutting refactors where items aren't isolated (rollback-per-item won't hold).
- A single change — the packaging overhead isn't worth it.

## Structure (per item)
```markdown
## Item N — <short title>
**LWC:** <component>            **Model:** <e.g. Composer/Sonnet — no Opus needed>
**Files changed:** <exact files, e.g. foo.css only>
### Problem      <what's visually wrong>
### Approach     <the technique, incl. any shadow-DOM/runtime caveat>
### Changes      <before → after, file by file>
### Expected Result   <bullet checks a human can verify>
```

## Build order & gates
- A build-order table: run each prompt **separately**, deploy, **visually confirm**, then proceed to the next.
- Explicit **model assignment** per item (don't burn a frontier model on a CSS tweak).
- A **rollback** note per item: each change isolated to its component, so reverting one doesn't touch the others.

<!-- Provenance: contributed from project `ItineraryBuilder`. Catalog ID P26. -->
