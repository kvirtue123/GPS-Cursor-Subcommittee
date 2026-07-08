# Advisory Validation Pattern — Template

Validation that informs without blocking writes: saves always succeed, rule violations surface as non-blocking alerts, and only a terminal action (Submit) is gated.

> Source pattern: `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder) — AGENT_REFERENCE "Validation is advisory" + QA Addendum §9.

## The rule
- **`save` always succeeds.** Validation is NEVER a DML gate.
- Rules run client-side after each save and produce a list of alerts (severity: error | warning).
- **Only the terminal action is gated** (e.g. Submit is disabled while any error-severity alert exists).
- No server-side enforcement that would reject the write — the user can save an in-progress, invalid state and fix it incrementally.

## Why
For draft-style builders (itineraries, configurators, multi-step forms), hard validation on every save is hostile — users need to save partial/invalid work and resolve issues as they go. Gating only at Submit gives freedom while still enforcing correctness at the boundary.

## Canonical example
- `saveSegment` upserts unconditionally and returns the saved record.
- `_validateRules()` runs after save, populates `_alerts`.
- `isSubmitDisabled => this._alerts.some(a => a.severity === 'error')`.
- Edge cases are *valid by design* (e.g. a one-day Departure+Arrival on the same date is NOT an overlap; a duplicate Departure saves fine but raises an alert that blocks Submit until resolved).

### When NOT to use
Transactional writes where an invalid record must never persist (payments, compliance records). There, validate *before* DML and reject.

<!-- Provenance: contributed from project `ItineraryBuilder`. Catalog ID P25. -->
