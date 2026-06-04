# Pattern P6 — Live-Site → Design-Token Extraction → Figma

> Genericized, project-agnostic template. Replace every `{{PLACEHOLDER}}`.
> Source/provenance: `WhitepaperGeneration/scripts/extract-design-tokens.js` + `DESIGN_TOKENS_SUMMARY.md` (Hard Rock Casino Hollywood token import).

## What this pattern does

Turns a **live website** into a structured **design system** without manual eyedropper work:

1. Drive a headless browser to the target URL.
2. Read **computed styles** off a representative set of elements.
3. De-duplicate + sort the raw values into **W3C Design Token Community Group** format.
4. Import the tokens into Figma (variable collections, text styles, effect styles) via the Figma MCP / Plugin API.

It is a *capture* pipeline: messy real-world CSS in, clean reusable tokens out.

## When to use

- You need to reproduce or reverse-engineer an existing site's look (colors, type scale, spacing, shadows, radii, transitions) as a Figma design system.
- Kicking off a rebrand/parity project where the source of truth is a shipped website, not a Figma file.
- You want a repeatable, auditable token snapshot instead of hand-picked values.

## When NOT to use

- The design system already exists in Figma — start there, don't re-derive from rendered CSS.
- You need *semantic* tokens (success/warning/brand-primary). This extracts **raw** values; semantic naming is a manual pass afterward.
- Sites behind auth, heavy bot-protection, or where computed styles are obfuscated — the capture will be unreliable.

## Dependencies / prerequisites

- Node + Playwright (`playwright` ^1.x). Chromium is the capture engine.
- Network access to `{{TARGET_URL}}` (and patience for SPA hydration — see the `waitForTimeout`).
- Figma MCP (`use_figma` / `create_new_file`) **only** for the import step; the extraction step is standalone.
- Permissions allowlist if running under Claude/Cursor settings: `WebFetch(domain:{{TARGET_DOMAIN}})`, `mcp__figma__*`.

## Canonical example (extraction script skeleton)

```js
// extract-design-tokens.js — run: node extract-design-tokens.js
const fs = require('fs');
const path = require('path');

async function extractDesignTokens() {
  const { chromium } = await import('playwright');
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();

  await page.goto('{{TARGET_URL}}', { waitUntil: 'load', timeout: 60000 });
  await page.waitForTimeout({{HYDRATION_WAIT_MS}}); // let SPA/dynamic CSS settle

  const tokens = await page.evaluate(() => {
    const result = { colors: {}, typography: {}, spacing: {}, shadows: {}, radii: {}, transitions: {} };
    const selectors = {{REPRESENTATIVE_SELECTORS}}; // e.g. ['h1','h2','p','a','button','[class*="card"]', ...]
    // for each selector: read getComputedStyle, collect color/bg/border, font family/size/weight/line-height,
    // padding/margin/gap, box-shadow, border-radius, transition into de-duped Sets,
    // then emit W3C tokens: { $value, $type } keyed as color-N / font-size-N / spacing-N / ...
    return result;
  });

  await browser.close();
  fs.writeFileSync(path.join(process.cwd(), '{{OUTPUT_JSON}}'), JSON.stringify(tokens, null, 2));
}
extractDesignTokens().catch(e => { console.error(e); process.exit(1); });
```

> The full as-shipped script (with the rgb→hex helper, per-selector sampling cap, and sorted emission) is preserved at `patterns/canonical/wpgen-extract-design-tokens.js`.

## Figma import step (summary)

After `{{OUTPUT_JSON}}` exists, import via the Figma MCP:
- **Variable collections** — one per token family (`Colors`, `Spacing`). Set scopes (`FRAME_FILL`, `TEXT_FILL`, `GAP`, `CORNER_RADIUS`).
- **Text styles** — one per `font-size`/`weight` pairing (Display / H1–H3 / Body / Caption / Label).
- **Effect styles** — one per unique shadow.
- Then a manual semantic pass: rename `color-7` → `brand-primary`, add success/warning/error.

## Recommended home

- **GitHub** — this template + the canonical script (installable/forkable).
- **Slack Canvas** — discovery card (what / when / when-not / link back).
- **Google Drive** — n/a (no binaries); the generated `extracted-tokens.json` is a per-project artifact, not a shared asset.

## Provenance

- `WhitepaperGeneration` (Hard Rock Casino Hollywood token import, 2026-05-22) — first contributor of this pattern.
