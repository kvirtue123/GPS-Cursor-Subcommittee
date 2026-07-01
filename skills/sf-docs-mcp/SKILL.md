---
name: sf-docs-mcp
description: Make Salesforce documentation LLM-readable. Use whenever you need authoritative, current content from help.salesforce.com or developer.salesforce.com in agent context. Converts JS-heavy/Shadow-DOM SF doc pages to clean Markdown via the Aura API (help) and Playwright+stealth (developer), with a 24h SQLite cache. TRIGGER when the agent needs to read SF docs, cite a Help article, or verify current platform behavior. DO NOT TRIGGER for offline work or when an in-repo build doc already has the spec.
---

# SF Docs MCP Skill

A custom MCP server that fetches Salesforce documentation and returns clean Markdown, solving the "SF docs are JS-heavy and unscrapeable" problem that otherwise wastes tokens and returns garbage.

- **help.salesforce.com** → Salesforce **Aura API** via native `fetch()` (no browser). Fast, cheap, no CDP fingerprint.
- **developer.salesforce.com** → **Playwright + stealth** (Shadow DOM / LWC require a real browser).
- Results cached in SQLite (`better-sqlite3`) with a 24-hour TTL.

## When to use `scrape_sf_docs`
Use this tool whenever you need to read Salesforce documentation content. Pass any `help.salesforce.com` or `developer.salesforce.com` URL.

### Success indicators
- **help.salesforce.com**: `pageType: "help-article"` means the Aura API returned real content.
- **developer.salesforce.com**: `pageType: "guide"` or `pageType: "reference"` means the Playwright extractor found substantive content.

### Failure recovery
If an article returns empty or error content:
1. The `fwuid` self-healing mechanism in `help-sf.ts` should handle stale tokens automatically.
2. If articles consistently return empty after a Salesforce release (~Feb, Jun, Oct), update the `KNOWN_FWUID` constant in `src/extractors/help-sf.ts` as a fallback.
3. For `developer.salesforce.com` pages that return empty content, use `analyze_page_structure` to inspect the DOM.

## When to use `analyze_page_structure`
Only for `developer.salesforce.com` pages where `scrape_sf_docs` returns empty/incomplete content. It reports custom elements, Shadow DOM elements, and a suggested CSS selector / shadow path to extract content — to diagnose why extraction failed.

## Operational notes (load-bearing)
- **Never introduce browser automation for help.salesforce.com** — Locker Service detects CDP. Keep `help-sf.ts` zero-Playwright.
- `aura.token` is always `null` for public articles; `release: ""` always fetches the latest version.
- The `fwuid` rotates ~3×/year (Feb/Jun/Oct); the extractor self-heals by scraping the homepage.
- Use **Node 18–24** (LTS 20/22). **Node 25+** breaks `better-sqlite3` prebuilds.
