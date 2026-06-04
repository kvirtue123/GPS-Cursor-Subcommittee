---
name: data-cloud-sql-runner
description: Run ANSI SQL, describe tables, and list objects against Salesforce Data Cloud (Data 360) from the shell, returning clean JSON to stdout. Use whenever you need VERIFIED Data Cloud field names, table names, or row data for __dlm / __dll / __cio objects — instead of guessing. Enforces the "verify, don't assume" confidence protocol. Do NOT use for CRM/platform sObjects (use SOQL).
---

# Data Cloud SQL Runner

Three deterministic shell helpers that wrap `ConnectApi.CdpQuery` (the Data Cloud Query SQL API) and `pg_catalog` introspection via `sf apex run`. Only their JSON stdout enters the agent's context — the Apex plumbing stays out.

This skill is the enforcement arm of the **R10 Verification & Confidence Protocol** rule: when that rule says "verify field names against the live org," these scripts are how.

## The three commands

| Command | Purpose | Example |
|---|---|---|
| `dc-query.sh "<ANSI SQL>"` | Run a query, paginated, JSON out | `dc-query.sh "SELECT ssot__Id__c FROM ssot__Individual__dlm LIMIT 10"` |
| `dc-describe.sh <Table>` | List a table's real column names (pg_catalog) | `dc-describe.sh Individual__dlm` |
| `dc-list-objects.sh` | List all `__dlm` / `__dll` / `__cio` objects | `dc-list-objects.sh` |

## How to use (workflow)

1. **List** objects to confirm exact, case-sensitive names: `dc-list-objects.sh`
2. **Describe** the target to get real field API names: `dc-describe.sh FWA_Owns__dlm`
3. **Query** using only verified names, always with a `LIMIT`: `dc-query.sh "SELECT source_id__c FROM FWA_Owns__dlm LIMIT 5"`

Never skip steps 1–2 and assume field names from CSV headers or prior docs — Data Cloud transforms names (casing, `ssot__` prefix, pluralization).

## When to use
- You need verified Data Cloud schema or data for CIOs, identity-resolution rules, data graphs, or analysis.
- A rule/doc tells you to confirm a field name before asserting it.

## When NOT to use
- CRM/platform sObjects (`Account`, `User`, `PermissionSet`, …) → use `sf data query` (SOQL).
- Data graph *definitions* / runtime graph JSON → use the Data Cloud REST API, not these scripts.

## Dependencies / prerequisites
- `sf` CLI authenticated to a Data Cloud-enabled org (`sf org display`).
- `jq` and `base64` on PATH.
- Scripts live in the project at `<sfdx-root>/.cursor/skills/` and are invoked by relative or absolute path — they are **not** on `PATH`. Run from the SFDX project root: `./.cursor/skills/dc-query.sh "..."`.
- API 62+ (`ConnectApi.CdpQuery.querySql`).

## Notes baked into the scripts
- Queries are base64-encoded before injection into Apex to dodge quote-escaping.
- `dc-query.sh` polls `querySqlStatus` to completion, then pages `querySqlRows` in 10k chunks.
- `dc-list-objects.sh` queries `pg_catalog` first (DLOs aren't in `getGlobalDescribe()`), then falls back to `Schema.getGlobalDescribe()`.
- Output is the last `@@@…@@@` USER_DEBUG payload, parsed to JSON.

## Provenance
As-shipped from the FWA federal fraud project (`FWA-Project/salesforce/.cursor/skills/`), verified against a Data Cloud org, API v62.
