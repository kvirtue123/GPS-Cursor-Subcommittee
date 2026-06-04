# LWC Wire-vs-Imperative Decision Pattern — Template

A one-table rule for deciding whether an LWC↔Apex call uses `@wire` or an imperative call. Removes per-method guesswork and keeps caching/error-handling behavior consistent.

> Source pattern: `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder) — AGENT_REFERENCE "Wire vs. Imperative" table.

## The rule
| Operation kind | Method | Rationale |
|---|---|---|
| Initial data load on mount | `@wire` | Cacheable; benefits from LDS caching + `refreshApex` |
| Any read-only / cacheable lookup | `@wire` | Read-only; cacheable |
| Create / update a record | **Imperative** | Write; needs full error control |
| Junction CRUD (delete-and-replace) | **Imperative** | Write |
| Server-side recalculation after a change | **Imperative** | Write; deterministic ordering |
| Status transition / submit | **Imperative** | Write; status update |

## How to apply
1. Reads → `@wire` adapters; refresh them with `refreshApex` after any imperative write so the UI reflects the new state.
2. Writes → imperative `import`ed methods wrapped in `try/finally` with explicit error toasts.
3. Never write through a `@wire`; never rely on a cacheable method for fresh post-write data.

### When NOT to use
Trivial single-call components where there's only one read and no writes — the table is overkill.

<!-- Provenance: contributed from project `ItineraryBuilder`. Catalog ID P22. -->
