# data-cloud-sql-runner

> Run ANSI SQL, describe tables, and list objects against Salesforce **Data Cloud (Data 360)** from the shell — verified schema/data, clean JSON to stdout.

Three small, deterministic Bash helpers that let an agent (or a human) introspect and query Data Cloud `__dlm` / `__dll` / `__cio` objects without guessing field names. They wrap `ConnectApi.CdpQuery` and `pg_catalog` via `sf apex run`, so the only thing that enters the agent's context is the JSON result.

## Why this exists

Data Cloud is the canonical "do not assume the schema" system: it renames CSV columns, adds `ssot__` prefixes, and pluralizes object names during ingestion. Asserting a field name from memory is the #1 way agents waste a session writing SQL that returns 0 rows. This skill makes "verify it" a one-liner.

## Install

```bash
mkdir -p <sfdx-project-root>/.cursor/skills
cp scripts/*.sh <sfdx-project-root>/.cursor/skills/
chmod +x <sfdx-project-root>/.cursor/skills/*.sh
```

Run from the SFDX project root (scripts are intentionally **not** on PATH):

```bash
./.cursor/skills/dc-list-objects.sh
./.cursor/skills/dc-describe.sh ssot__Individual__dlm
./.cursor/skills/dc-query.sh "SELECT ssot__Id__c FROM ssot__Individual__dlm LIMIT 10"
```

## Pair with

- **R10 — Verification & Confidence Protocol** rule (`rules/templates/verification-confidence-protocol.mdc`): these scripts are the introspection commands that rule references.

## What's inside

```
data-cloud-sql-runner/
├── SKILL.md      ← loaded when triggered
├── README.md     ← this file
└── scripts/
    ├── dc-query.sh         ← run ANSI SQL (paginated), JSON out
    ├── dc-describe.sh      ← column names via pg_catalog
    └── dc-list-objects.sh  ← list all __dlm/__dll/__cio objects
```

## Requirements

`sf` CLI authed to a Data Cloud org · `jq` · `base64` · API 62+.

## Provenance

As-shipped from the FWA federal fraud project (`FWA-Project/salesforce/.cursor/skills/`). Verified against a live Data Cloud org. If Salesforce changes the `ConnectApi.CdpQuery` surface, re-verify `dc-query.sh`.
