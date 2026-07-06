# Layer-Elimination Debugging — Template

A disciplined method for debugging a failure that crosses many layers (e.g. Apex → planner
schema → CLT registry → renderer.json → LWC → channel). Instead of guessing, you enumerate
every layer the data passes through, then design **one decisive observation** that exonerates
or implicates a layer — so each step removes a chunk of the search space rather than poking
at random.

> Source pattern: `sfdxMelodyv3/docs/debug/symphony-carousel.md` — a single diagnostic LWC
> build (unconditional debug banner + plain list + the real component) let one screenshot
> disambiguate "never instantiated" vs. "instantiated but unbound" vs. "bound but not rendering,"
> collapsing days of guesswork.

---

## Step 1 — Enumerate the layers (the chain the data travels)

List every hop from source to symptom. Example (CLT render):

```
Apex returns data
  → planner/action output schema marks it displayable
    → CLT bundle (schema.json + renderer.json) registered & exposed
      → channel resolves the renderer override (lightningDesktopGenAi vs enhancedWebChat)
        → LWC instantiated in the DOM
          → LWC receives `value` (correct shape)
            → LWC renders the markup
```

## Step 2 — For each layer, define its decisive observation

| Layer | Decisive observation (what proves it works) | Tool |
|---|---|---|
| {{layer}} | {{the single signal that confirms/denies this layer}} | {{trace / SSE / DevTools DOM / SOQL / screenshot}} |

> The art is choosing observations that are **mutually exclusive**: a result that can only
> be explained by one diagnosis. A debug build that renders a banner *on mount regardless of input*
> is the classic move — its presence/absence alone separates "instantiated" from "not instantiated."

## Step 3 — Outcome → diagnosis table (pre-enumerate the branches)

| Observed outcome | Diagnosis | Next action |
|---|---|---|
| {{element absent from DOM}} | {{layer never reached — earlier hop is the gate}} | {{check the prior layer}} |
| {{element present, input undefined}} | {{binding/contract mismatch}} | {{…}} |
| {{element present, input correct, output empty}} | {{render logic / CSS}} | {{…}} |

## Step 4 — Record what each test eliminated

After each observation, write a one-liner into the evidence log (P6):
`**{{layer}} — EXONERATED/CONFIRMED.** {{evidence}}.`
Eliminated layers never get re-investigated — that's the whole point.

---

## Rules of the pattern
1. **One variable per test.** Change exactly one thing so the result is attributable.
2. **Prefer a test that splits the space in half.** Don't start at the ends; start where a single observation eliminates the most layers.
3. **Make the invisible visible.** Inject a deterministic, input-independent signal (debug banner, log line, sentinel value) to distinguish "code didn't run" from "code ran but produced nothing."
4. **Write eliminations down** (pairs with P6). The value compounds only if removed layers stay removed.
5. **Don't change production logic to test** beyond the minimal diagnostic scaffold — and remove the scaffold when done (the source log should note it was removed).

### When NOT to use
A one-layer bug with an obvious stack trace doesn't need this ceremony. Use it when the failure is silent, multi-layer, and "everything looks correct on paper."
