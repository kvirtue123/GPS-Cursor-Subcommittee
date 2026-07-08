# Doc-Derived Implementation Guide — Template

A repeatable workflow + document structure for turning **official vendor documentation** (Salesforce Help, developer docs, etc.) into a single project-local **implementation guide** the agent can `@`-reference. Replaces ad-hoc "go search the web every time" with one curated, source-cited knowledge file per feature area.

> Source pattern: `{{PROJECT_NAME}}/docs/*-implementation-guide.md` (NASA NSPIRES PostAward: flow-approval-processes, omnistudio-document-generation, grantmaking-psc-features).

---

## When to use
- You're building on a platform feature with **dense, authoritative, but scattered** official docs (approval flows, document generation, a managed-package framework).
- The same feature will be touched repeatedly across phases, and you want the agent to read **one curated page** instead of re-scraping.
- You need traceability — every claim should point back to a canonical source URL.

## When NOT to use
- One-off features you'll never revisit (just read the doc inline).
- Topics where official docs change weekly — a snapshot rots fast; prefer a live `@`-ref or re-validate often.

## Authoring workflow (the prompt pattern)
1. **Enumerate sources.** List the exact `help.salesforce.com` / `developer.salesforce.com` article URLs for the feature.
2. **Scrape, don't paraphrase from memory.** Pull the real content (use the docs MCP / scraper). Record any page that **could not be scraped** (JS-rendered Metadata API pages are a known trap) and link it directly so a human can open it.
3. **Consolidate into one guide** with a stable section structure (see skeleton below).
4. **Cite inline.** Every section starts with its `**Source:** <url>`.
5. **Add a Metadata/Deployment reference** section — what the deployable artifact actually is (`.flow-meta.xml`, etc.) and where it lives.
6. **Drop it in `docs/<feature>-implementation-guide.md`** and point the project's `goals-doc-map.mdc` (R3) at it.

## Guide skeleton (genericize per feature)
```markdown
# {{FEATURE_NAME}} — Implementation Guide
> Audience: Cursor AI agent building {{FEATURE_NAME}} in {{PLATFORM}}.
> Sources: scraped from {{DOC_HOST}}. All URLs listed per section.
> Metadata API Note: {{ANY_JS_RENDERED_PAGES_THAT_FAILED_TO_SCRAPE + direct links}}

## Table of Contents
1. Feature Overview          (**Source:** {{url}})
2. Permissions & Editions    (**Source:** {{url}})
3. Key Concepts
...
N-1. Considerations & Pitfalls
N.   Metadata Deployment Reference  ← the deployable artifact + path
```

## Why this works
- **Token-efficient:** the agent reads one scoped guide instead of N web pages per session.
- **Auditable:** source URLs make every recommendation verifiable.
- **Resilient to scraper gaps:** the explicit "could not be scraped" note prevents silent omissions.

## Pairs with
- **R3** (goals/doc-map) — register the guide so the agent loads it on demand.
- **P4** (phased delivery) — guides are the "knowledge" layer feeding sequenced build phases.
- Produces guides like **P14** (Flow Approval Processes), **P15** (OmniStudio DocGen), **P16** (Grantmaking/PSS features).
