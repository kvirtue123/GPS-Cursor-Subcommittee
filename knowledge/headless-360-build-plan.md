---
feature_area: headless360
status: duplicate-of-catalog
catalog_id: P21
catalog_ref: patterns/templates/headless-mcp-tool-plane.md
note: "Already cataloged as P21 (Headless multi-surface MCP tool-plane). Kept here for local browsing — do not re-catalog."
---

# Headless 360 — Internal Build Plan

### MCP Tool Plane

Salesforce Hosted MCP Servers expose org data, automation, and product capabilities to any MCP-compatible AI client over per-user OAuth. They are the **tool plane** — not the reasoning brain.

| Server ID | Capability | Notes |
|---|---|---|
| `platform/sobject-all` | Full CRUD + query + search | FLS/sharing/object perms enforced on every call |
| `platform/sobject-reads` | Read + query only | Use for any agent that must never mutate |
| `platform/sobject-mutations` | Create + update, no delete | Audit-by-construction: write-back without delete risk |
| `platform/sobject-deletes` | Delete only | Scope-isolated; rarely needed in these UCs |
| `data-cloud-sql` | ANSI SQL against DLOs/DMOs/CIOs | Unified customer data; per-user OAuth, not service account |
| `analytics/tableau-next` | Semantic-model discovery, KPI queries, NL analytics | Requires Tableau Next enablement |
| Flows | Lightning Flows as MCP tools | Reuse existing automation without new endpoints |
| Invocable Actions | `@InvocableMethod` classes as MCP tools | Preferred over net-new REST endpoints |
| Apex REST | `@RestResource` classes mapped to MCP tools | For complex joins/queries not suited to vanilla SOQL |
| Prompt Builder | Admin-curated prompt templates as MCP prompts | LLM-assisted summaries with org-aware context |
| API Catalog | Registered REST/Connect API endpoints as tools | Extends tool surface without custom server code |

**Governance:** Admin-configurable per External Client App; tool surface is auditable. Never expand beyond the MCP tools each use case actually needs — every unused tool schema bloats context and invites speculative calls.

**Cross-platform additions:**
- **Salesforce DX MCP Server (Beta)** — natural-language CLI replacement: org queries, metadata retrieval, deploys, test runs. IDE-side via Agentforce Vibes MCP Client.
- **Heroku MCP Server** — control Heroku apps/teams/deploys from an LLM context; relevant when the scheduled job (UC4) or BFF (UC1/UC2) is hosted on Heroku.

### Agent API Contract

The Agent API is the headless contract between any experience-layer surface (React, Slack, mobile) and an Agentforce agent. Build every AI-assisted UI surface against this, not against raw LLM APIs.

| Endpoint | Method | Purpose |
|---|---|---|
| `/einstein/ai-agent/v1/agents/{AGENT_ID}/sessions` | POST | Start session; random UUID session key + `bypassUser` flag |
| `/sessions/{SESSION_ID}/messages` | POST | Send synchronous message (batch automations) |
| `/sessions/{SESSION_ID}/messages` (streaming) | POST + `Accept: text/event-stream` | SSE stream for real-time chat UIs |
| `/sessions/{SESSION_ID}` | DELETE | Explicit teardown (120-second per-call timeout) |
| `/sessions/{SESSION_ID}/feedback` | POST | Thumbs/ratings → Data 360; feeds audit trail + tuning |

**SSE event sequence:** `ProgressIndicator` → `TextChunk` (incremental) → `Inform` (final) → `EndOfTurn` → `ValidationFailureChunk` (on failure, clears prior chunks).

**React-side implementation pattern:**
- **Token broker** — Apex or Heroku service issues short-lived access tokens to the browser; never embed client secret in frontend code.
- **`useAgentSession()` hook** — holds `sessionId`, increments `sequenceId` per turn.
- **EventSource / fetch + ReadableStream** — consumes `text/event-stream`; appends `TextChunk` events incrementally; finalizes on `Inform + EndOfTurn`.
- **`citedReferences` + `inlineMetadata`** — render inline footnotes from `startIndex`/`endIndex` so analysts can trace every AI assertion back to its source record.
- **`submitFeedback`** — feedback widget posts with `sessionId`; stored in Data 360 for audit.

**Important:** Agent API is not supported for agents of type "Agentforce (Default)". Confirm agent type eligibility with the customer's Agentforce license before architecture commitment.

### Agentforce Vibes Skills

Skills are modular, progressively-loaded instruction packs for the agentic IDE (Agentforce Vibes / Cursor). They extend IDE behavior without bloating the always-on context that Rules impose.

**Loading model:** Metadata always loaded (~100 tokens per skill). Instructions loaded only on trigger. Resources loaded on demand — effectively unlimited. Opposite of Rules (always active).

**Storage:** `.a4drules/skills/` (workspace, version-controlled — recommended). Commit this directory so every contributor's Cursor/Vibes session inherits the same playbook.

**Notable OOTB skill:** `afv-library/skills/developing-agentforce` — authoritative for building, debugging, deploying Agent Script agents.

**Abilities** — domain-scoped activation limiting which MCP tools light up per context (e.g., DevOps tools only, never SObject mutations). Use to stay under the ~20-tool limit. Configured via `a4d_mcp_settings.json`.

### Connected App & Auth Patterns

| Pattern | When to use | Notes |
|---|---|---|
| OAuth 2.0 PKCE + refresh token | Human-facing internal apps, MCP clients | **Default for all Hosted MCP Server access**; short-lived tokens; no client secret on frontend |
| OAuth 2.0 JWT Bearer (server-to-server) | Backend services, scheduled jobs, integration agents | No user interaction; requires private key + certificate in Connected App |
| Named Credentials | Callouts from Apex/Flow back out to external systems | Use with External Credentials for per-user principal mapping |
| Connected App with IP allowlist | Compliance tools requiring strict network boundary | Combine with PKCE or JWT |

**MCP-specific:** Per-user OAuth 2.0 with PKCE is the default and non-negotiable for Data Cloud (`data-cloud-sql`) — never username/password. FLS, sharing rules, and object permissions are enforced on every MCP tool call; the MCP layer provides audit-by-construction.

### API-Only Integration User

For service-to-service integrations (drift detector, audit agent, code review agent): provision a dedicated Salesforce **integration user** with a Salesforce (full) or API-only license, assigned a permission set granting only the object/field access the integration needs. Do not use a named human user's credentials.

### Common APIs in Play

| API | Primary use in this engagement |
|---|---|
| Salesforce REST API | CRUD on standard + custom objects, SOQL via `/query` |
| Salesforce Connect API | Composite requests, UI API for record layout metadata |
| Salesforce Metadata API (SOAP) | Retrieve/deploy org metadata (Flows, Apex, PermSets, etc.) |
| Salesforce Tooling API (REST) | Fine-grained component queries, SetupAuditTrail, symbol tables |
| Salesforce Bulk API 2.0 | Large data exports (wagering history, case volumes) |
| Platform Events / CometD | Real-time push from SF to headless clients |
| Change Data Capture (CDC) | Subscribe to record-level change streams |
| Agent API | Headless Agentforce agent sessions (UC1, UC2) |
| Salesforce DX MCP Server | Natural-language metadata/query/deploy (UC3, UC4, UC5) |

---

## Use Case 1: VIP Host Dashboard (Internal Tool)

### Overview

A React/Next.js view of a high-value player: contact details, open and recent cases, wagering history, responsible gambling flags, and deposit/withdrawal summary. 

### High-Level Demonstration Plan
1. **React/Next.js frontend — MVP** — Player search, 360 summary card (contact + RG flag + tier), case list with open/recent filter, wagering summary (read-only). SSE streaming for any AI-assisted panel content. 
2. **Write-back: case actions** — Add case update capability (status change, note logging) through `platform/sobject-mutations` MCP tool — FLS/sharing enforced at the MCP layer. All writes generate standard SF audit trail.
3. **Real-time case updates** — Subscribe to CDC or Platform Events via BFF/token broker to push new case activity to the dashboard without manual refresh.
4. **UI parity pattern** — Surface the same agent session (same `sessionId`) in both the React dashboard and Slack (see UC2 for Slack pattern). Single conversation memory, single audit trail in Data 360.

**Agentforce Vibes Skill:** Use `afv-library/skills/developing-agentforce` when building the headless agent; `data360-sql-runner` for Data Cloud queries backing the player 360.

### Salesforce APIs & Tooling Required

| Component | API / Tool |
|---|---|
| Player 360 data (read) | `platform/sobject-reads` MCP server; Salesforce REST API `/query` (SOQL composites) for non-MCP paths |
| Multi-object composite fetch | Salesforce Connect API — Composite request (`/composite`) |
| Case create/update | `platform/sobject-mutations` MCP server (enforces FLS/sharing); REST API as fallback |
| Real-time case push | Platform Events + CometD **or** CDC subscription in token broker |
| Custom wagering data | Apex REST (`@RestResource`) surfaced via API Catalog MCP tool or direct REST call |
| LLM-assisted summaries | Prompt Builder MCP prompts; headless Agentforce agent via Agent API |
| Agent API (if headless agent) | `POST /einstein/ai-agent/v1/agents/{ID}/sessions`; SSE streaming; `submitFeedback` |
| Auth | OAuth 2.0 PKCE via Connected App; token broker (Apex/Heroku) issues short-lived tokens |
| Frontend framework | Next.js (App Router) + React Query; `useAgentSession()` hook for agent sessions |
| Agentforce Vibes Skills | `developing-agentforce` (OOTB), `data360-sql-runner`, `gnn-score-fetch` |

### Open Questions

- What custom objects and fields represent wagering history, RG flags, and VIP tier? Do they already exist in the org?
- Is Neccton surfacing responsible gambling alerts into Salesforce already (custom object? Platform Event?)? What is the trigger and data shape?
---

## Use Case 2: AML / EDD Case Worker Tool (Internal Tool)

### Overview

A purpose-built compliance review interface for the Risk/Compliance team replacing the standard Salesforce case UI for AML and EDD workflows. Analysts need to see case details, linked EDD flags, Neccton responsible gambling alerts, and account risk indicators in a single view — and every action they take (decision, note, escalation) must be written back to Salesforce for audit trail. 

### High-Level Demonstration Plan

1. **Case queue + case detail** — Analyst-facing queue of open AML/EDD cases with EDD flags and Neccton alerts in a single view. All reads via `platform/sobject-reads` MCP server (FLS/sharing enforced at the tool layer).
2. **Write-back: decisions and notes** — Action panel (case status, decision field, investigation notes) writing back to Salesforce through `platform/sobject-mutations` — audit-by-construction, no additional middleware logging required.
3. **Real-time Neccton alert push** — Platform Events or CDC subscription surfacing new Neccton alerts to the analyst without page refresh.
4. **Compliance assistant agent (optional)** — Headless Agentforce agent backed by `data-cloud-sql` (unified player risk profile) surfacing recommended next actions via SSE streaming; `submitFeedback` stores analyst ratings in Data 360 as part of the regulatory audit trail.

**Agentforce Vibes Skill:** Commit a custom `aml-case-workflow` skill to `.a4drules/skills/` scoped to the analyst workflow — encapsulates case object patterns, AML/EDD field names, and write-back action sequences so every developer on the team gets consistent IDE guidance.

### Salesforce APIs & Tooling Required

| Component | API / Tool |
|---|---|
| Case + AML/EDD object reads | `platform/sobject-reads` MCP server; REST API `/query` (SOQL) |
| Case action writes (decisions, notes) | `platform/sobject-mutations` MCP server (FLS/sharing enforced; audit-by-construction) |
| Neccton alert read | Platform Events (CometD subscribe) **or** CDC **or** REST query on custom object — TBD |
| Real-time alert push to UI | CometD / Streaming API in token broker; WebSocket relay to React client |
| Compliance assistant agent | Agent API — `startSession`, SSE streaming, `submitFeedback` → Data 360 audit trail |
| Auth | OAuth 2.0 PKCE + IP-restricted Connected App; token broker issues short-lived tokens |
| Audit trail | `platform/sobject-mutations` MCP (primary); SF standard audit fields; token-broker action log (secondary) |
| Frontend | React (no SSR needed); role-gated component rendering; `useAgentSession()` hook if agent surfaced |
| Agentforce Vibes Skills | `developing-agentforce` (OOTB), `aml-case-workflow` (custom, committed to `.a4drules/skills/`) |

### Open Questions for Customer

- How does Neccton currently push data into Salesforce — Platform Event, batch scheduled Apex, direct REST upsert? What is the object/field structure of the alert record?
- Do compliance analysts have Salesforce licenses today? If not, how do they authenticate — external IdP with a service account mapping?
- What are the specific regulatory requirements for audit trail retention? (UK GC? MGA? Specific field set that must be captured per decision?)
- Is there a requirement for MFA enforcement regardless of network location?
- What data must **not** leave the SF org boundary — can PII fields appear in token-broker logs, or must they be masked outside of SF?
- Are case decisions legally binding in a way requiring a digital signature or secondary approval workflow?
- Is a headless compliance assistant agent confirmed, or is the case worker tool purely REST-driven? (Affects agent type selection — see Cross-Cutting.)

---

## Use Case 3: Metadata-Aware Code Review Agent

### Overview

An AI agent with live read access to the Salesforce org's metadata (Flows, Apex classes, permission sets, sharing rules, custom objects) that automatically runs at PR/push time to catch conflicts, deprecated API usage, and sharing rule violations before deployment. Static analysis tools like PMD catch code-level issues; this agent catches **org-context** issues — e.g., a new permission set that grants field access already restricted by a sharing rule, or an API version deprecated in the current org's release. In a regulated sports betting org, pre-deploy conflict detection reduces the risk of accidental access escalation reaching production.

### High-Level Demonstration Plan

1. **Live org metadata read at PR time** — Salesforce DX MCP Server (Beta) retrieves the relevant component from the production or long-lived sandbox org when a PR is opened; no custom SOAP wrapper needed.
2. **Org-context conflict detection** — Agent cross-references the PR diff against live org metadata: deprecated API versions, sharing rule conflicts, permission escalation, missing FLS on new fields. Context scoped via DevOps/Metadata Ability in `a4d_mcp_settings.json` (~20-tool limit enforced).
3. **Structured findings posted to MR** — Agent runs in Plan mode (list findings, categorize severity) then Act mode (post GitLab MR comment or Jira sub-task). Hooks enforce the mode boundary — no accidental comment spam.
4. **Team-wide rule library** — `metadata-review-rules` skill committed to `.a4drules/skills/`; every developer on the team gets the same org-aware review context in their IDE.

### Salesforce APIs & Tooling Required

| Component | API / Tool |
|---|---|
| Metadata retrieval | Salesforce DX MCP Server (Beta) — replaces direct Metadata API SOAP calls |
| Fine-grained component queries | Salesforce Tooling API via DX MCP Server — `ApexClass`, `FlowDefinition`, `PermissionSet`, symbol tables |
| API version + SetupAuditTrail | Tooling API via DX MCP Server — `ApexClass.ApiVersion`, `SetupAuditTrail` |
| MCP client (IDE) | Agentforce Vibes MCP Client; `a4d_mcp_settings.json` for tool/server enablement |
| Abilities | DevOps/Metadata Ability only; gated via `a4d_mcp_settings.json`; ~20-tool limit |
| Hooks | Pre/post-tool injections for CI guardrails (validate input, format output) |
| CI integration | GitLab CI job (`.gitlab-ci.yml` stage); GitLab REST API for MR comments |
| Static analysis (complement) | PMD (`sf scanner run`) for Apex; runs before the AI agent for baseline |
| Auth | JWT Bearer Connected App (integration user); DX MCP Server holds credentials |
| Agentforce Vibes Skills | `metadata-review-rules` (custom, committed to `.a4drules/skills/`); `developing-agentforce` (OOTB) |

### Open Questions for Customer

- Which GitLab tier — SaaS or self-managed? (Affects network egress from CI runner to DX MCP Server and LLM API.)
- Is there an approved LLM vendor for internal tooling (e.g., must use Azure OpenAI within tenant boundary, or is Anthropic Claude API acceptable)?
- Who maintains the `metadata-review-rules` skill — platform/DevOps team, or does compliance sign off on rule additions/removals?
- What is the current branching strategy? Does every component change go through a PR, or are some changes pushed directly to integration branches?
- Are there Sandbox refresh cycles that would cause the org metadata used for review to drift from production? (Point agent at prod or a long-lived sandbox.)
- Is Agentforce Vibes (VS Code) adopted by the engineering team today? The DX MCP Server and Abilities model requires it. (See Cross-Cutting: Agentforce Vibes adoption.)
- What is the appetite for blocking PRs on agent findings vs. advisory-only comments? (Recommended: advisory first, blocked after 30-day observation period.)

---

## Use Case 4: Automated Org Drift Detector

### Overview

A scheduled job that retrieves the current state of the production org's metadata, diffs it against the source-controlled baseline in GitLab, and raises a Jira ticket for any drift — including which component changed and which Salesforce user made the change (via SetupAuditTrail). In a regulated sports betting org, undocumented production changes are a licensing and audit risk: regulators expect that the org's configuration matches the approved, version-controlled baseline.

### High-Level Demonstration Plan

1. **Scheduled metadata retrieval** — Scheduled job (Lambda or GitLab pipeline) retrieves production org metadata via Salesforce DX MCP Server; diffs against the `main`/`release` branch in GitLab.
2. **Drift identification + attribution** — Flags components where production diverges from source-controlled baseline; queries `SetupAuditTrail` via Tooling API to surface the responsible Salesforce user and timestamp.
3. **Jira ticket creation (Plan → Act)** — Agent lists drifted components in Plan mode first, then raises a Jira ticket per component in Act mode. Hooks enforce the boundary — no ticket spam on re-runs. Severity-1 drift (permission sets, sharing rules) also triggers Slack/Teams notification.
4. **Deduplication** — Does not re-raise an open ticket for the same component; closes tickets when drift is remediated.

### Salesforce APIs & Tooling Required

| Component | API / Tool |
|---|---|
| Metadata retrieval from prod | Salesforce DX MCP Server (preferred) **or** `sf project retrieve` CLI / Metadata API SOAP (`retrieve()`) |
| Change attribution | Tooling API `SetupAuditTrail` via DX MCP Server |
| Baseline comparison | GitLab REST API — file content retrieval, tree listing |
| Ticket creation | Jira REST API (v3) — issue create, issue search (deduplication) |
| Alerting | Slack Incoming Webhooks **or** Teams connector |
| Scheduler | AWS Lambda + EventBridge (cron), GitLab Scheduled Pipelines, or Heroku Scheduler |
| Heroku hosting | Heroku MCP Server — control deploys/scheduling from LLM context if hosted on Heroku |
| Plan/Act guardrails | Hooks — pre-Act Hook requires non-empty Plan output; post-Act Hook logs created ticket IDs |
| Auth (Salesforce) | JWT Bearer Connected App (integration user) |
| Auth (GitLab) | GitLab Project Access Token (read-only) |
| Auth (Jira) | Jira API token (service account) |
| Runtime | Python or Node.js Lambda; `jsforce` or `simple-salesforce` for fallback Metadata API calls |

### Open Questions for Customer

- Is the GitLab repo currently source-tracked (`sf project generate` with source format)? Or is metadata stored as unpackaged XML without a clear 1:1 mapping? (Affects diff fidelity.)
- What is the authoritative GitLab branch for production baseline — `main`, a release tag, or a deployment branch?
- Which Jira project and issue type should drift tickets be raised under? Is there a compliance/change control board that reviews them?
- What is the acceptable drift detection latency — daily, hourly, or near-real-time? (Near-real-time requires SetupAuditTrail polling rather than full metadata retrieval; different architecture.)
- Are there metadata types intentionally modified directly in production (e.g., email templates, quick text) that should be excluded?
- Who is the Salesforce user running the scheduled job — use a dedicated integration user with a recognizable username to filter from SetupAuditTrail results.
- Is Jira the preferred ticketing destination, or does the customer use a different ITSM (ServiceNow, etc.)?
- Is the job hosted on Heroku? (Determines whether Heroku MCP Server is relevant.)

---

## Use Case 5: Permission Set & Security Audit Agent

### Overview

An agent that traverses the org's permission sets, profiles, sharing rules, field-level security configurations, and connected app access, then produces a structured audit report suitable for internal engineering review and submission to a gaming license access review. UK Gambling Commission, MGA, and similar bodies require operators to demonstrate that access to sensitive player data and financial records is limited to authorized personnel — this agent automates the evidence gathering.

### High-Level Demonstration Plan

1. **Metadata extraction** — Salesforce DX MCP Server retrieves permission sets, profiles, and sharing rules; `platform/sobject-reads` handles `PermissionSetAssignment` queries (FLS enforced at the MCP layer — no accidental over-read).
2. **Structured audit report** — Produces: (a) role → object access matrix, (b) role → field access matrix for sensitive fields, (c) sharing rule summary, (d) connected app + OAuth scope inventory. Output in Markdown + JSON.
3. **Risk auto-flagging** — Flags over-broad profiles (System Administrator with PII access), delete permissions on financial records, public sharing rules on sensitive objects, connected apps with `full` OAuth scope.
4. **Live matrix via Tableau Next (if available)** — If `analytics/tableau-next` is enabled, surfaces the permission matrix as a filterable live semantic-model query rather than a static report — no re-run needed for ad-hoc role/field lookups.

### Salesforce APIs & Tooling Required

| Component | API / Tool |
|---|---|
| Permission set + profile retrieval | Salesforce DX MCP Server — Metadata API (`PermissionSet`, `Profile`, `SharingRules`) |
| Granular FLS / object permissions | Tooling API via DX MCP Server — `FieldPermissions`, `ObjectPermissions` |
| `PermissionSetAssignment` queries | `platform/sobject-reads` MCP server (**preferred** path; FLS enforced at MCP layer) |
| `SetupEntityAccess` (connected apps) | Tooling API via DX MCP Server |
| Sharing rule analysis | Metadata API via DX MCP Server — `SharingRules` per object |
| User → permission set mapping | `platform/sobject-reads` MCP server — `/query` on `PermissionSetAssignment` |
| Permission matrix analytics | `analytics/tableau-next` MCP server — semantic-model query (if Tableau Next enabled) |
| Report output | Local Markdown/JSON; optional Confluence REST API push or SharePoint via Graph API |
| Runner | Python or Node.js CLI; wrappable as scheduled Lambda for recurring runs |
| Auth | JWT Bearer Connected App (read-only integration user with Metadata API + targeted object access) |
| Agentforce Vibes Skills | `perm-audit-runner` (custom, committed to `.a4drules/skills/`) |

### Open Questions for Customer

- Which gaming license(s) is the operator holding (UK GC, MGA, state-by-state US)? Each has different access review format requirements — affects report structure.
- Is this a one-time audit ahead of a license renewal, or a quarterly/annual recurring process? (Determines whether delta reporting is needed.)
- Who receives and reviews the report — internal compliance team only, or external auditor? (External distribution affects format and redaction requirements.)
- Are there Salesforce profiles that should be excluded from the audit scope (e.g., System Administrator, integration users)?
- Is there an existing GRC platform (e.g., OneTrust, Vanta, Drata) that the report should integrate with?
- What is the expected audit run turnaround — is a 30–60 minute Metadata API retrieve acceptable, or must it complete in minutes? (Affects full retrieve vs. Tooling API query-per-component approach.)
- Does the customer's org use Permission Set Groups? (Adds a traversal layer — groups must be resolved to constituent sets for accurate FLS reporting.)
- Is Tableau Next enabled? (Determines whether `analytics/tableau-next` is viable for live permission matrix queries.)

---

## Cross-Cutting Open Questions

These span multiple use cases and should be resolved before sprint planning across the engagement.

| Question | Affects |
|---|---|
| **Hosted MCP Server enablement** — has the customer org enabled Salesforce Hosted MCP Servers? Which External Client App will own the `platform/sobject-*` and `data-cloud-sql` tool surface? Admin-configurable; must be set up before any MCP tool calls can be made. | UC1, UC2, UC3, UC4, UC5 |
| **Agentforce Vibes adoption** — is the engineering team using Agentforce Vibes (VS Code) today? The Salesforce DX MCP Server and Abilities model requires it. `.a4drules/skills/` commitment is only effective if the team's IDE activates skills. | UC3, UC4 |
| **Agent type selection** — for UC1 and UC2, if a headless Agentforce agent is used, it cannot be type "Agentforce (Default)". Confirm agent type eligibility with the customer's Agentforce license before committing the Agent API architecture. | UC1, UC2 |
| **Tableau Next availability** — does the customer have Tableau Next enabled? Relevant for UC5 permission matrix live queries and UC1 VIP analytics. Confirm before committing `analytics/tableau-next` MCP server. | UC1, UC5 |
| **Skills governance** — who owns and reviews `.a4drules/skills/` additions (`metadata-review-rules`, `perm-audit-runner`, `aml-case-workflow`, etc.)? Recommend the same review process as Cursor rules — PR required, platform team approves. | UC2, UC3, UC5 |
| **Integration user strategy** — one shared integration user for all headless tooling, or one per use case? Shared is simpler to manage; per-use-case is least-privilege and easier to audit via SetupAuditTrail. | UC1, UC2, UC3, UC4, UC5 |
| **LLM/AI vendor approval** — is there an approved vendor list for AI components? (Affects UC3 code review agent and UC5 if AI-assisted report generation is used.) | UC3, UC5 |
| **Network topology** — are all headless services deployed within a customer VPC, or can they be hosted externally (Vercel, AWS, Heroku)? UK GC data residency may require UK region for token broker and BFF. | UC1, UC2, UC4 |
| **Salesforce org structure** — production + sandbox(es)? How many environments? Does the drift detector (UC4) run against prod only, or also sandboxes? Which org does UC3 code review agent point at? | UC3, UC4 |
| **GitLab structure** — mono-repo or per-project? Is source tracking already enabled (`sf project` format)? Affects both the code review agent and drift detector. | UC3, UC4 |
| **Jira workspace** — cloud vs. server/data center? Affects API version and auth method for UC4 ticket creation. | UC4 |
| **Neccton integration status** — is Neccton already sending data into Salesforce? What is the current mechanism? Blocks UC1 (RG flags) and UC2 (compliance alerts). | UC1, UC2 |
| **Salesforce license inventory** — how many Salesforce seats does the customer have? Do compliance analysts and VIP hosts have existing seats? Determines whether the headless apps authenticate as named users or via service accounts. | UC1, UC2 |
| **Responsible gambling data sensitivity** — is RG flag data subject to additional access controls beyond standard FLS (e.g., must not appear in shared views, access must be logged)? | UC1, UC2, UC5 |
| **Timeline and prioritization** — which use cases are P1? Recommended sequence: UC1 (fastest to value, clearest scope) → UC2 (high compliance value, Neccton dep) → UC4 (compliance control) → UC5 (audit prep) → UC3 (DevOps maturity). | All |
