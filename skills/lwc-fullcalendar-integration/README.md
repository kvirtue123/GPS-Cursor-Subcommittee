# FullCalendar

A full-sized, interactive JavaScript event calendar for scheduling and visualization.

> Provenance: copied as-shipped from `ItineraryBuilder/README.md` (github.com/kvirtue/ItineraryBuilder). This is the capabilities reference that backs the `lwc-fullcalendar-integration` skill (catalog S5).

## What It Does

FullCalendar renders a professional calendar UI with month and week views. Users can view events as colour-coded blocks, drag them to reschedule, resize to adjust duration, and drop new items from an external palette.

## Key Capabilities

- **Month and week views** with configurable toolbar buttons
- **Drag and drop** to move existing events between dates
- **Resize** event edges to change duration
- **External drop** — drag items from outside the calendar to create new events (via `FullCalendar.Draggable`)
- **Multiple event sources** — load from arrays, JSON feeds, or functions; each source is independently refreshable
- **Event styling** — apply colours, CSS classes, and custom rendering per event
- **Read-only mode** — disable editing by setting `editable: false` and `droppable: false`

## Quick Start

```javascript
const calendar = new FullCalendar.Calendar(containerElement, {
    initialView: 'dayGridMonth',
    headerToolbar: {
        left:   'prev,next today',
        center: 'title',
        right:  'dayGridMonth,timeGridWeek'
    },
    editable: true,
    droppable: true,
    events: [ /* event objects */ ],
    eventClick:  (info) => { /* handle click */ },
    eventDrop:   (info) => { /* handle move */ },
    eventResize: (info) => { /* handle resize */ }
});
calendar.render();
```

## Resources

- [Project website and demos](https://fullcalendar.io/)
- [Documentation](https://fullcalendar.io/docs)
- [Support](https://fullcalendar.io/support)
