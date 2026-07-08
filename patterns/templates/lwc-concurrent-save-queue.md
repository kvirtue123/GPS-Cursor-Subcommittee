# LWC Concurrent-Save Queue + Debounce — Template

Handles rapid, overlapping auto-saves in an LWC without race conditions or dropped edits, using a single in-flight flag + one queued save, plus input debounce.

> Source pattern: `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder) — QA Addendum §5 (Auto-Save Behavior).

## Problem
Auto-save on field change can fire a second save while the first is still in flight, causing races or lost updates. Saving on every keystroke also hammers the server.

## Pattern
1. **Debounce text inputs** (~500ms) so typing doesn't fire a save per keystroke; save on blur for textareas.
2. **Single-slot queue**: if a save is in flight, stash the latest payload in `_pendingSave` and return; when the current save finishes, run the pending one. Only the latest pending save matters.

## Canonical example
```javascript
_isLoading = false;
_pendingSave = null;
_locationDebounceTimer;

handleLocationChange(e) {
  clearTimeout(this._locationDebounceTimer);
  const data = this._buildSegmentData();
  this._locationDebounceTimer = setTimeout(() => this._handleSave(data), 500);
}

async _handleSave(data) {
  if (this._isLoading) { this._pendingSave = data; return; }
  this._isLoading = true;
  try {
    await saveSegment({ segmentJSON: JSON.stringify(data) });
    if (this._pendingSave) {
      const next = this._pendingSave;
      this._pendingSave = null;
      await this._handleSave(next);
    }
  } finally {
    this._isLoading = false;
  }
}
```

### When NOT to use
Explicit save-button forms (no auto-save) or components with a single, non-overlapping write. A single-slot queue is intentionally simple — if you need ordered multi-save batching, use a real queue.

<!-- Provenance: contributed from project `ItineraryBuilder`. Catalog ID P24. -->
