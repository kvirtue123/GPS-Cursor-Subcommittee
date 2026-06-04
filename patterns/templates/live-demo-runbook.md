# Live Demo Run-of-Show + Fallback Matrix — Template

A reusable structure for a **high-stakes live demo** (customer-facing, on-stage, recorded). It front-loads everything that prevents a live failure and pre-scripts the recovery for everything that can still go wrong, so the presenter never debugs live.

> Source pattern: FWA "Headless 360" SE runbook + customer-demo playbook
> (`FWA-Project/docs/headless360_se_runbook.docx.md`, `docs/phase_3/phase3-customer-demo-prompt.md`).

---

## 1. The Story (½ page)
- **What the customer asked for** — in their words.
- **What we're showing** — one or two sentences.
- **Why it lands** — the single "aha" moment.

## 2. Demo Arc
> Order matters: start where the audience already lives, expose the platform underneath progressively.

| # | Segment / surface | Who it speaks to | Time | What they see |
|---|---|---|---|---|
| 1 | {{SURFACE_1}} | {{AUDIENCE}} | {{MIN}}–{{MAX}} | {{BEAT}} |
| 2 | {{SURFACE_2}} | {{AUDIENCE}} | … | … |

## 3. Pre-flight checklist (do this {{N}} min before, NOT on stage)
> Every minute of pre-flight saves five minutes of live recovery. Group by system.

- **{{SYSTEM_A}} (e.g. org):** {{checks}}
- **{{SYSTEM_B}} (e.g. Slack/IDE):** {{checks}}
- **Auth:** confirm every independent auth/token is green (list each one — they cache separately).
- **Clean state:** clear stale demo records / stale threads so the first action is at the top.

## 4. Golden records / copy-paste inputs
> The 2–4 hero entities the demo runs against. Keep this open in a side window.

| Name | Search by | Expected result | Notes / gotcha |
|---|---|---|---|
| {{HERO_1}} | {{QUERY}} | {{EXPECTED}} | {{e.g. "search by ID, not name — duplicates"}} |

> Note any IDs that **drift** (e.g. on re-run of a job) and the fallback lookup (search by a stable key instead).

## 5. On-stage prompts (paste verbatim)
> Bold = what you paste. Plain = expected response, paraphrased.

```
{{EXACT_PROMPT_TO_PASTE}}
```

## 6. Talk track (speaker notes)
> Conversational, not scripted. One short block per beat. Include the "keep your hands off the keyboard — the silence is the demo" moments.

## 7. Fallback matrix (the reusable gold)
> Pre-enumerate every likely failure with its most-likely cause and a <60s recovery. **Rule: never debug live unless the fix is under 30 seconds — otherwise switch tracks.**

| Fallback | Symptom | Most-likely cause | Recovery (≤60s) |
|---|---|---|---|
| A | {{symptom}} | {{cause}} | {{steps; or "fall forward to segment N+1"}} |
| B | {{symptom}} | {{cause}} | {{steps}} |

### Global emergency fallbacks
- **If {{SYSTEM_A}} is down:** {{reorder — open with the surface that still works}}.
- **If {{SYSTEM_B}} is down:** {{switch to the pre-staged artifact}}.
- **If the whole environment is down:** pivot to the architecture conversation; promise a recorded follow-up. Don't pretend it works.

### Composure rules
- Never apologize twice for the same failure. Acknowledge once, recover, move on.
- Never debug live if the fix is >30s. Switch tracks.
- Never blame the platform — "we're in beta on this one" / "demo env, prod has different SLAs."

## 8. Wrap & QA
- 90-second recap tying every surface back to the one "aha."
- **Anticipated questions** table (question → honest answer, including limitations).

## 9. Appendix
- Tool/endpoint inventory, architecture-in-one-paragraph, useful links, timing cheat sheet (min/max/where-to-cut-if-tight).

---

### When NOT to use
Async or recorded-only demos, or internal smoke tests — the fallback matrix and on-stage talk track are overhead you don't need. Use a plain checklist instead.
