# Text-Only AI API → Rich-UI Hybrid Architecture — Template

A reference architecture for building a standalone chat UI on top of an AI/agent API that returns **text only** (no structured action payloads), while still rendering rich components (cards, carousels, summaries) reliably.

> Source pattern: `{{PROJECT_NAME}}` (`hrMelodyv3/melody-ui/` — React+Vite+TS chat UI for an Agentforce concierge agent). This is the *implementation* companion to the conceptual P6 demo playbook.

---

## Problem it solves

The {{AI_PRODUCT}} API streams text but returns no structured data to drive rich UI, and the platform's native rich-rendering path (e.g. Custom Lightning Types in Embedded Chat) is broken or unavailable. You still need branded cards/carousels in the chat.

## Architecture

```
Browser UI ({{FRONTEND}})  ⇄  {{PROXY}} (holds OAuth)  ⇄  {{AI_PRODUCT}} API  ⇄  LLM agent
```

- **Frontend** renders messages and detects intent in the agent's text.
- **Proxy** holds the OAuth `{{AUTH_FLOW}}` flow so the browser never sees credentials; caches tokens.
- **Two engines, one UI:** a live engine (streams SSE from the API) and a scripted engine (client-side state machine) both feed the **same** component layer.

## Key components (rename to your stack)

| Concern | Example file | Responsibility |
|---|---|---|
| Live engine | `engine/liveEngine.ts` | Session mgmt, SSE parsing, card detection |
| Scripted engine | `engine/scriptedEngine.ts` | Deterministic demo state machine |
| Scenarios | `engine/scenarios.ts` | Scripted flows |
| Card triggers | `services/cardTriggers.ts` | **Text pattern → card injection rules** (the hybrid core) |
| Stream parser | `services/streamParser.ts` | SSE event parsing (text chunk / inform / end-of-turn) |
| Proxy: API | `server/agentApi.ts` | Session create / stream / end |
| Proxy: auth | `server/auth.ts` | OAuth, token cache |

## The hybrid core: intent detection

Map recognizable agent phrases to components. When the live agent emits the phrase, inject the matching card client-side:

```
"{{TRIGGER_PHRASE}}"  →  inject <{{CARD_COMPONENT}}>
```

This is the same rendering layer the scripted engine uses, so LIVE and DEMO look identical. A strong "watch me change it" demo beat: add one phrase→component rule, save, and the live agent renders a new component next time it says the trigger.

## Dependencies

{{FRONTEND}} (e.g. React + Vite + TS), a small {{PROXY}} (e.g. Express), an AI/agent API that streams text, an OAuth client-credentials (or equivalent) app. Deliberately minimal deps so agent edits stay surgical.

## When NOT to use

If the AI API already returns structured action/component payloads, render those directly — you don't need client-side intent detection. If the platform's native rich rendering works, prefer it.
