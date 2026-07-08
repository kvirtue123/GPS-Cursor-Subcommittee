---
name: lwc-fullcalendar-integration
description: Integrate FullCalendar v6 into a Lightning Web Component (Experience Cloud / LWR safe) as a single static-resource bundle with drag-and-drop, resize, external palette drop, multiple event sources, and read-only mode. TRIGGER when building or debugging a drag-and-drop calendar inside an LWC, a static-resource calendar bundle, or shadow-DOM event-styling issues. DO NOT TRIGGER for non-LWC calendars or server-side scheduling.
---

# LWC FullCalendar Integration

Render an interactive month/week calendar inside an LWC, deployed as a single static-resource JS bundle (no npm build step), that survives the LWR shadow-DOM boundary.

## When to use
- Any LWC that needs a drag-and-drop / resizable calendar with color-coded tiles.
- Experience Cloud (LWR) sites where you cannot rely on a build pipeline.

## When NOT to use
- Non-LWC web apps (use FullCalendar directly).
- Static read-only date displays (a table or `lightning-datatable` is lighter).
- Server-side scheduling logic — this is a UI library only.

## Dependencies & prerequisites
- FullCalendar v6 global bundle (`index.global.min.js`) uploaded as a static resource (single file or ZIP; confirm `dayGrid` + `interaction` plugins are present).
- `loadScript` from `lightning/platformResourceLoader`.
- An org/site on LWR if targeting Experience Cloud.

## Non-negotiable patterns (the hard-won parts)
1. **Init in `renderedCallback` with an `_fcInitialized` guard**, after `loadScript` resolves — NOT `connectedCallback` (DOM not ready).
2. **External drag palette must live inside this component's own template** — `FullCalendar.Draggable` needs the source elements in the same shadow DOM as the calendar.
3. **FullCalendar end dates are exclusive** — `_addOneDay` when writing to FC, `_subtractOneDay` in `eventDrop`/`eventResize` before saving back.
4. **Style events via `info.el.style` / `setProperty`** — FC renders event elements outside the LWC shadow DOM, so CSS classes/custom-properties don't pierce. Set inline (`backgroundColor`, `color`, and `--_tile-accent` for left-border accents).
5. **Read-only mode:** `editable: false` + `droppable: false`. Overlay events (e.g. related records) render as foreground tiles with `editable: false`, NOT `display: 'background'`.

## Canonical example
```javascript
_fcInitialized = false;
renderedCallback() {
  if (this._fcInitialized) return;
  this._initFullCalendar();
}
async _initFullCalendar() {
  await loadScript(this, fullcalendar + '/fullcalendar.min.js');
  this._fcInitialized = true;
  const paletteEl = this.template.querySelector('.palette-container');
  new FullCalendar.Draggable(paletteEl, { /* … */ });
  const cal = new FullCalendar.Calendar(this.template.querySelector('.cal'), {
    initialView: 'dayGridMonth',
    editable: !this.isReadOnly,
    droppable: !this.isReadOnly,
    eventDrop: (info) => {
      const end = new Date(info.event.end);
      end.setDate(end.getDate() - 1);          // exclusive-end fix
      this._save(info.event.id, info.event.start, end);
    }
  });
  cal.render();
}
```

See `README.md` in this folder for the full capabilities reference.

<!-- Provenance: project `ItineraryBuilder` (github.com/kvirtue/ItineraryBuilder). SKILL distilled from README.md (capabilities) + QA Addendum §2/§8 + PCSPortal_UIPolish §4 (shadow-DOM accent). Catalog ID S5. -->
