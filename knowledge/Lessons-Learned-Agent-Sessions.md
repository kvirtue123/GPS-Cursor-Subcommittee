---
feature_area: agentforce-metadata-ops
status: duplicate-of-catalog
catalog_id: R8
catalog_ref: rules/canonical/FHA-Lessons-Learned-Agent-Sessions.md
note: "Source for R8 (Agentforce metadata operational rule). This copy has slightly more detail (e.g. T4.3 HECM Annual Compliance subagents) than the canonical copy — worth a follow-up diff/merge, not done here. Do not re-catalog."
---

# Lessons learned: Agent sessions (FHA Call Center / Agentforce work)

This note captures friction from real agent work so future runs spend less time retrying, retrieving noise, or fighting Metadata API shape mismatches. Last updated: 2026-04-26 (T4.3 — 6 HECM Annual Compliance subagents).

---

## Salesforce CLI and org operations

**Deploy the smallest surface area.** A broad deploy of `force-app/main/default/objects/Case/fields` failed because an unrelated custom field (`Case.SDO_Sub_Type__c`) had picklist control values that did not match the target org. Always deploy by explicit named metadata component, not by folder:

```bash
sf project deploy start \
  --metadata "CustomField:Case.Identity_Verified_Borrower__c" \
  --target-org <alias>
```

**Never use unscoped metadata pulls for "explore."** `sf project retrieve start --metadata GenAiPlugin` (no member) bloated the repo with hundreds of unrelated `genAiPlannerBundle` files that had to be deleted. Always retrieve **named** components, e.g. `GenAiPlugin:HECM_Verify_Caller_Identity`, after you know the API name.

**Do not echo org credentials.** `sf org list --json` includes access tokens; never paste that output into chat or docs.

---

## Service Assistant / GenAiPlugin (subagent) as metadata

**`GenAiPlanner` may not be retrievable** (`Entity type 'GenAiPlanner' is not available in this api version`). Topics are deployed as **`GenAiPlugin`** with `pluginType: Topic`. Linking to the agent is done in legacy Agentforce Builder — not via a `plannerField` element, which fails parse in API v66.

**`sortOrder` is normalised to 0 by the org — do not retrieve-to-verify.** The org flattens all instruction `sortOrder` values to 0 regardless of what was deployed (confirmed T4.1 and T4.3). The deploy's `Deployed Source` table (`State: Created` / `State: Changed`) is sufficient proof of success. A post-deploy retrieve only rewrites `sortOrder` and marks the file `Changed` — skip it. Use **lexicographically ordered `developerName` values** (e.g. `HECM_Tax_Instruction_01` … `_06`) so the legacy Builder UI shows steps in the correct order regardless. The **repo XML** is the source of truth for human-readable order.

**`GenAiPlugin` is not queryable via Tooling API.** `SELECT … FROM GenAiPlugin` returns `sObject type 'GenAiPlugin' is not supported`. Verify plugin existence via the deploy output table or the legacy Agentforce Builder Topics tab — not via SOQL.

---

## When to read the build doc first

**`docs/HUD-FHA-Build.md` is the spec — read it first, every time.** It contains verbatim label, description, scope, and instruction strings for every task. Re-deriving those from Salesforce Help articles or MCP is unnecessary and adds tokens. Only reach for external sources when the build doc explicitly has a gap.

**Do not read `docs/knowledge/instruction-library.md` or `docs/knowledge/topic-design-patterns.md` for `GenAiPlugin` tasks.** These files contain generic Agentforce topic design patterns. They have no bearing on `GenAiPlugin` metadata shape and add significant token cost with zero payoff. Omit them from context for all subagent authoring tasks.

---

## Repo hygiene

- **Do not** retrieve `GenAiPlannerBundle` with wildcards. Cleanup cost is 200+ files.
- **Do** add new agent artifacts under `force-app/main/default/genAiPlugins/<DeveloperName>.genAiPlugin-meta.xml` and document deploy commands in `docs/HUD-FHA-Build.md` next to the task.

---

## Apex invocable actions (T4.2 lessons)

**Match `sourceApiVersion` in the class meta-xml.** This project pins `sourceApiVersion: 66.0` in `sfdx-project.json`. Use `<apiVersion>66.0</apiVersion>` in every `*.cls-meta.xml`.

**Ship the test class in the same named deploy.**

```bash
sf project deploy start \
  --metadata "ApexClass:HECMLookupComplianceStatus" \
  --metadata "ApexClass:HECMLookupComplianceStatusTest" \
  --target-org FHA-Demo --wait 15
```

**`--test-level RunSpecifiedTests` silently skips on sandboxes.** `◯ Running Tests - Skipped` in the deploy log does not mean tests passed. Run tests separately:

```bash
sf apex run test --tests HECMLookupComplianceStatusTest \
  --target-org FHA-Demo --result-format human \
  --code-coverage --synchronous --wait 10
```

**`Org Wide Coverage 0%` on single-class `--synchronous` runs is meaningless.** Trust the per-class row (`HECMLookupComplianceStatus 100%`).

**Treat the build doc's Apex snippet as a starting spec.** Match its `@InvocableMethod` label/description, field names, and demo values verbatim. Safe hardening on top: add `with sharing`, make bulk-safe (1:1 input/output), handle null input, add `label=` to `@InvocableVariable`. Document deviations in the chat response.

---

## Checklist for "Service Assistant subagent" tasks

1. Read `docs/HUD-FHA-Build.md` for the task — verbatim strings are there.
2. Author `GenAiPlugin` with `pluginType: Topic`, `canEscalate: false`, no `genAiFunctions` (unless the task explicitly says to wire an action). Only include `genAiPluginInstructions` that the build doc literally specifies; do not invent instructions from narrative hints.
3. Deploy by named component — one `--metadata "GenAiPlugin:<Name>"` flag per plugin.
4. Verify via the deploy output table (`State: Created`). Do not retrieve-to-verify.
5. Confirm topics appear in legacy Agentforce Builder → Topics tab; add from library if the org only surfaces library topics.

---

## One-line summary

**Narrow deploys, named retrieves, trust the in-repo runbook, expect `sortOrder=0` normalisation (use ordered `developerName` values), skip retrieve-to-verify for GenAiPlugin, never query GenAiPlugin via Tooling API, and never read generic pattern docs for subagent tasks — the build doc has the spec.**
