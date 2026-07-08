# Demoing a Probabilistic AI Product — Template

A reusable playbook for giving a confident, repeatable demo of a probabilistic product (an LLM-driven agent) without either faking it or gambling on a live model in front of a customer.

> Source pattern: `{{PROJECT_NAME}}` (`hrMelodyv3/melody-ui-Demo/DEMO-EXPLAINER.md`). Keep this template product-agnostic; put build-specifics in your project's `HANDOFF.md` / clickpath doc.

---

## The core idea

**Decouple the unpredictable part (language + routing) from the part that has to look perfect (UI, data, critical flows), and give the presenter a toggle to pick the risk level per moment.**

The model's language and routing are probabilistic. The visuals, data, and critical flows can and should be deterministic. That mirrors how you'd ship to production: keep the live intelligence, engineer guardrails around it.

## The three modes

| Mode | What it is | Use it for | Trade-off |
|---|---|---|---|
| **LIVE** | Every message hits the real {{AI_PRODUCT}} backend | Prove authenticity; free-form "ask it anything" | Probabilistic — can take an unrehearsed path |
| **DEMO** | A scripted client-side state machine, no network | The beats that must not fail | Only follows the rehearsed path |
| **Hybrid** | Shared UI layer both modes render into | Making LIVE and DEMO look identical | Requires intent-detection (below) |

## The hybrid insight (the reusable lesson)

Many AI APIs return **only text** — no structured payloads for rich UI. So the UI does **intent detection on the model's own words**: when the agent emits a recognizable phrase (e.g. `{{TRIGGER_PHRASE}}`), the UI injects the matching rich component client-side. The model stays smart; the visuals stay reliable. Because the rich-UI layer is **shared** between engines, LIVE and DEMO look visually identical.

## Risk-per-moment dial

| Beat | Mode | Why |
|---|---|---|
| Free-form "ask it anything" | **LIVE** | Proves authenticity; variation acceptable |
| Standard happy path | **LIVE** | Usually reliable; keep prompts tight to avoid known loops |
| The single biggest "wow" moment | **DEMO** | Must land identically every time |
| Any flow the model handles poorly | **DEMO** | Never demo a known-weak path live |
| Offline / resilience story | **DEMO** | Kill the backend, keep going — proves the safety net |

## Guardrails (the deliverable)

- **Never demo a known-weak flow live.** Identify failure modes in prep; route those beats through DEMO.
- **Stay authentic — don't fake the product.** Keep a genuine LIVE mode; be transparent that critical beats run in a controlled mode.
- **Decouple language from visuals.** Highest-leverage design choice.
- **Set production expectations explicitly.** A demo shows the *target experience*; production adds guardrails, structured actions, testing.
- **Keep credentials server-side.** Proxy the AI API so the browser never holds secrets.
- **Make the asset reusable.** Centralize prompts, intent rules, and scripts.

## When NOT to use

Deterministic products with no probabilistic surface don't need the toggle. For a pure happy-path screenshot walkthrough, this is overkill.
