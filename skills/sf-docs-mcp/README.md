# sf-docs-mcp

> Make Salesforce documentation LLM-readable — an MCP server that converts `help.salesforce.com` and `developer.salesforce.com` pages to clean Markdown.

Salesforce docs are hostile to naive scraping: Help articles render via the Aura framework, and Developer docs use Shadow DOM / LWC. This MCP server handles both and caches the result, so an agent gets authoritative, current SF docs in context instead of hallucinating or burning tokens on broken HTML.

## TL;DR

- **help.salesforce.com** → Aura API via native `fetch()`. **No browser.** (Locker Service detects CDP, so a headless browser gets blocked — the Aura path sidesteps that entirely.)
- **developer.salesforce.com** → Playwright + stealth (Shadow DOM needs a real browser).
- **Cache** → SQLite (`better-sqlite3`), 24h TTL.
- **Self-healing** → the `fwuid` (framework unique id) rotates ~3×/year; the extractor re-scrapes the homepage to recover automatically.

## Tools

| Tool | Use when |
|---|---|
| `scrape_sf_docs` | You need to read any SF Help or Developer doc URL. |
| `analyze_page_structure` | A `developer.salesforce.com` page came back empty — inspect its DOM / shadow path. |

## Install

This is a Node MCP server, not a Markdown-only skill. To run it:

```bash
# 1. Build it (from the server's own repo)
npm ci && npm run build

# 2. Register it with your agent (.mcp.json / Cursor MCP settings)
{
  "mcpServers": {
    "sf-docs": { "command": "node", "args": ["<path>/sf-docs-mcp/dist/mcp-server.js"] }
  }
}
```

The `SKILL.md` in this folder documents *when/how the agent should call the tools*; the server source itself should be promoted to its own GitHub repo (see "Provenance").

## Verify

```bash
npm run test-url -- "https://help.salesforce.com/s/articleView?id=ind.psc_admin_concept_psc_welcom.htm&type=5"
# expect: pageType: "help-article", length > 1000

npm run test-url -- "https://developer.salesforce.com/docs/einstein/genai/guide/get-started.html"
# expect: pageType: "guide"
```

## Requirements

- **Node 18–24** (LTS 20 or 22 recommended). **Node 25+ is unsupported** — `better-sqlite3` has no prebuild and native compile fails.
- Playwright (developer.salesforce.com path only). `help-sf.ts` has zero Playwright dependency.

## Companion rule

Ship the architecture lock alongside the server: `rules/canonical/sf-docs-mcp.core.mdc` (catalog ID **R6**) records the non-obvious constraints (Aura-not-Playwright for Help, `fwuid` rotation, Node version ceiling) so an agent doesn't "fix" the design into breakage.

## Provenance

As-shipped from project `FHA Call Center` (`sf-docs-mcp/`). The full TypeScript server (extractors, Aura integration, SQLite cache, regression suite) lives in that project and should be promoted to a standalone GitHub repo; this folder documents the skill surface + install + verification.
