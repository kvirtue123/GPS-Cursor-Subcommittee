# Cursor Best Practices — Quick Reference

> **Reference cards** — one card per asset, enough to know what exists and when to reach for it.
> For the full explainer (what this is, how to get started, how to contribute) → read the **Google Doc** first.

📄 **Explainer (start here):** _(add Google Doc URL)_
🔗 **Repo (installable templates):** [GPS-Cursor-Subcommittee](https://github.com/kvirtue123/GPS-Cursor-Subcommittee) → start at `BEST-PRACTICES.md`
🎥 **Drive (recordings, decks):** _(add Google Drive folder URL)_

---

## 🛡️ Rules — behavioral guardrails

**R1 · Reuse Registry & Architecture** (`alwaysApply`)
Locks architecture + COPY/ADAPT/SKIP asset registry + recurring deploy gotchas.
✅ Use when constraints are firm. ⛔ Not for greenfield/exploration. → `rules/templates/agent-script-architecture.mdc` (+ `salesforce-deploy-gotchas.mdc`)

**R2 · Doc-Sync** (glob-scoped, token-efficient)
Tells the agent *when* to update docs vs. skip — stops doc rot AND token waste.
✅ Maintained doc sets. ⛔ Throwaway prototypes; never `alwaysApply`. → `rules/templates/doc-sync.mdc`

**R3 · Goals & Doc-Map** (`@`-reference index)
States the goal once + maps where each design decision lives → scopes context.
✅ Multi-doc projects. ⛔ Doc-light projects. → `rules/templates/goals-doc-map.mdc`

**R4 · AGENTS.md** (orientation)
The "start here" front door: guardrails + phase status + key-doc index.
✅ Every non-trivial project. → `rules/templates/AGENTS.md`

**R5 · MCP/WebFetch allowlist** (`settings.local.json`)
Pre-approve a *narrow* set of MCP tools + fetch domains so the agent works from ground truth without prompts.
✅ Stable MCP/doc set (sf-docs, Figma, SLDS). ⛔ Don't over-broaden. → `rules/templates/mcp-allowlist.settings.local.json`

**R6 · Agent runtime guardrails** (anti-jailbreak)
Scope-lock + injection resistance for the **deployed** Agent Script agent (not the coding agent).
✅ Every customer-facing agent. ⛔ Internal open-Q&A agents. → `rules/templates/agent-runtime-guardrails.agent.md`

**R7 · Context-scoping ignore file** (`.cursorignore`/`.claudeignore`)
Removes dead/superseded/known-broken code from the index so the agent never cites or copies it. Complements R1's COPY/SKIP.
✅ A project carrying a superseded twin (legacy agent, rewritten module). ⛔ Hiding *active* code (blinds the agent); not for secrets. → `rules/templates/context-scoping-ignore.cursorignore`

**R8 · Agentforce metadata ops** (glob-scoped)
Deploy by named component; *don't retrieve-to-verify* (`sortOrder`→0); no wildcard retrieves; `GenAiPlugin` isn't Tooling-queryable; read the build doc first.
✅ Authoring/deploying `GenAiPlugin`/`aiAuthoringBundle`. ⛔ Non-Agentforce work; never `alwaysApply`. → `rules/templates/agentforce-metadata-ops.mdc`

**R9 · Single-component architecture lock** (`alwaysApply`)
Locks the *non-obvious* design of one component (MCP server, worker) so the agent doesn't "fix" a deliberate counter-intuitive choice into breakage.
✅ A self-contained component with a surprising-but-correct decision. ⛔ Multi-component repos (use R1); keep it tiny. → `rules/templates/architecture-lock.mdc`

**R10 · Verification & confidence protocol** (glob-scoped)
"Never assert a field/API name from memory — verify or say *'I'm not certain'*." Plus a query-router (one access path per data surface) + validate-before-write workflow.
✅ Data Cloud / external DB / undocumented API the agent can't safely guess. ⛔ Well-known sObjects; never `alwaysApply`. → `rules/templates/verification-confidence-protocol.mdc` (enforced by S4)

**R11 · MCP-first org verification** (token-efficient)
Prefer the live org via DX MCP over static metadata; verify every field/object API name before writing SOQL/Apex. (Distinct from R5's tool *allowlist* and R10's generic *protocol*.)
✅ SFDX projects with DX MCP + drifty local metadata. ⛔ No MCP/org; deploys stay on CLI. → `rules/templates/mcp-first-verification.mdc`

**R12 · Override-layer precedence**
One authoritative override doc supersedes contradictions in all other design docs; fixed reading order; don't back-edit superseded docs.
✅ Multi-doc projects with contradicting, time-layered docs (Q&A addenda). ⛔ Single/fresh doc sets. → `rules/templates/override-layer-precedence.mdc`

**R13 · Design-language token rule** (glob-scoped to LWC)
Every LWC matches one visual language: `:host` tokens, card structure, semantic pills + left-border accents, AI/sparkle pattern, shadow-DOM `setProperty` caveat.
✅ Multi-component portal with a design system. ⛔ Single-component/non-UI; never `alwaysApply`. → `rules/templates/design-language-tokens.mdc`

---

## 🧰 Skills — task-specific agents

**S1 · agentforce-agentscript-clt**
Build/deploy/render Custom Lightning Types from Agent Script + envelope-unwrap + failure catalogue.
✅ Custom card/carousel rendering or debugging a blank CLT. ⛔ Plain Apex/Flow/Prompt work. → `skills/agentforce-agentscript-clt/`

**S2 · salesforce-gps-whitepaper** (`wpGenerator`)
Markdown → branded GPS white-paper PDF (Chrome render → PDFKit verify → PNG check).
✅ Branded PDF from `.md`. ⛔ Non-GPS branding, decks, live HTML. (macOS-only) → `skills/salesforce-gps-whitepaper.md`

**S3 · sf-docs-mcp** (MCP server)
Salesforce docs → clean Markdown: Aura API for Help (no browser — Locker detects CDP), Playwright for Developer, 24h cache, self-heals `fwuid`.
✅ Need ground-truth SF docs in context. ⛔ Offline, or the in-repo build doc already has the spec (R8). (Node 18–24) → `skills/sf-docs-mcp/`

**S4 · data-cloud-sql-runner** (shell scripts)
`dc-query`/`dc-describe`/`dc-list-objects` against Data Cloud (Data 360) via `ConnectApi.CdpQuery` — verified schema/data, JSON to stdout. The deterministic arm of R10.
✅ Verified Data Cloud field/object names + queries on `__dlm`/`__dll`/`__cio`. ⛔ CRM sObjects (use SOQL); data-graph defs (use REST). → `skills/data-cloud-sql-runner/`

**S5 · lwc-fullcalendar-integration**
FullCalendar v6 in an LWC as a single static resource (no npm): drag/resize/external-drop + the 5 LWR shadow-DOM survival patterns (renderedCallback+guard, palette in-template, exclusive-end ±1 day, `setProperty` styling, overlay tiles `editable:false`).
✅ Drag-drop calendar in an LWC, esp. Experience Cloud/LWR. ⛔ Non-LWC apps; static date displays. → `skills/lwc-fullcalendar-integration/`

**S6 · sf-dependent-address-picklist**
Read compound-Address State/Country dependent picklists in Apex (`validFor` bitVector) → dependent `lightning-combobox` (country-first, clear-state-on-change) so users pick ISO codes, not free text that throws DML.
✅ Address field with State/Country Picklists; DML errors on typed state/country. ⛔ Plain text address fields. → `skills/sf-dependent-address-picklist/`

---

## 🧩 Patterns — reusable workflows & architecture

**P1 · Agent Script knowledge library** — curated DSL playbook (patterns/examples/reference). Reference, don't fork. → `patterns/canonical/agent-script-library.md`

**P2 · CLT best-practices guide** — the 13-section CLT recipe + failure-mode table + PR checklist. → `patterns/canonical/lightning-types-best-practices.md`

**P3 · Cross-surface handoff** — structured packet to delegate work to another agent/surface (DevTools, MCP, other model). ⛔ Skip if current agent can finish it. → `patterns/templates/cross-surface-handoff.md`

**P4 · Phased-delivery + design-doc-map** — separates design (A/B/C) from sequenced, self-contained build phases. ⛔ Don't over-scaffold tiny changes. → `patterns/templates/phased-delivery-structure.md`

**P5 · AI tooling inventory** — periodic snapshot of available rules/skills/MCP before deciding what to build.

**P6 · Design-token extraction** — Playwright reads a live site's computed styles → W3C tokens → Figma variables/styles. ✅ Reverse-engineer a shipped site into a design system. ⛔ DS already in Figma; auth-walled sites. → `patterns/templates/design-token-extraction.md` (+ `patterns/canonical/wpgen-extract-design-tokens.js`)

**P7 · Platform data-model reference** — curated `@`-referenceable object/field reference for a standard platform domain (e.g. PSS Grantmaking). ✅ Standard-platform projects. ⛔ Custom-light/small schemas. → `patterns/templates/platform-data-model-reference.md`

**P8 · Org metadata extraction & audit** — reverse-engineer an org feature, then record "what was found / what was **missing**" (negative findings). ✅ Brownfield/org-takeover. ⛔ Greenfield or single-field changes. → `patterns/templates/org-metadata-extraction-audit.md`

**P9 · Probabilistic AI demo playbook** — LIVE/DEMO toggle to demo an LLM product without faking it or gambling live; decouple language/routing from UI/data/critical flows. ✅ Demoing a probabilistic product to a customer. ⛔ Deterministic products. → `patterns/templates/probabilistic-ai-demo.md` (+ `patterns/canonical/probabilistic-ai-demo-playbook.md`)

**P10 · Text-only API hybrid UI** — proxy + live/scripted engines + **client-side intent detection** (text phrase → injected card) for AI APIs that return text only. ✅ Text-only API but you need rich cards. ⛔ API already returns structured payloads. → `patterns/templates/text-only-api-hybrid-ui.md`

**P11 · Append-only evidence log** — one durable file that accumulates dated findings; survives autocompaction/session boundaries. ✅ Long, cross-session/cross-tool investigations. ⛔ One-sitting bugs. → `patterns/templates/append-only-evidence-log.md`

**P12 · Fresh-session briefing** — self-contained "brief a new chat" doc (works/doesn't/root-cause/HARD CONSTRAINTS/context strategy). ✅ Crossing a session boundary (next-day, post-compaction). ⛔ Same session with context intact (and use P3 for a *tool* gap). → `patterns/templates/session-brief.md`

**P13 · Layer-elimination debugging** — enumerate the layers, design one decisive observation each, eliminate methodically; inject an input-independent signal to split "didn't run" from "ran but empty." ✅ Silent multi-layer "looks correct on paper" failures. ⛔ Obvious single-layer stack traces. → `patterns/templates/layer-elimination-debugging.md`

**P14 · Flow Approval Processes library** — consolidated, source-cited guide to Salesforce Flow Approval Processes (stages/steps, orchestration runs, record locking, legacy migration, deploy artifact). ✅ Multi-step/multi-user approvals. ⛔ Simple single-approver records. → `patterns/canonical/nspires-postaward-flow-approval-processes-guide.md`

**P15 · OmniStudio DocGen recipe** — `fndSingleDocxLwc` button + step props + URL-param template preselect + `fndmultiPDFConvertLwc` + browser preview. ✅ `.docx`/`.pdf` from records via OmniStudio. ⛔ Non-OmniStudio docgen (GPS white-paper = S2). → `patterns/canonical/nspires-postaward-omnistudio-docgen-guide.md`

**P16 · Grantmaking/PSS feature reference** — Budget Mgmt, recurring Funding Award Requirements, Compliant Data Sharing, Form Framework (+ editions/permissions). ✅ PSS/Grantmaking feature *behavior*. ⛔ Raw schema → use P7. → `patterns/canonical/nspires-postaward-grantmaking-psc-features-guide.md`

**P17 · Doc-derived guide workflow** — turn scattered official docs into one curated, source-cited, `@`-referenceable guide; link JS-rendered pages that can't be scraped; always include a metadata/deploy-artifact section. ✅ Dense vendor-doc features you'll revisit. ⛔ One-offs; weekly-changing docs. → `patterns/templates/doc-derived-implementation-guide.md`

**P22 · Service Assistant quick-actions/subagents** — the grounding model: Service Assistant actions are NOT autonomous (treated as grounding text); only **Case Quick Actions** (single `CaseId`) become plan-step buttons. Subagent scoping + 4-part instruction structure + migration checklist. ✅ Any Service Assistant/Service Planner build or "why won't my button appear?" debug. ⛔ Autonomous Agentforce agents (use P1). (legacy Builder only) → `patterns/canonical/service-assistant-quick-actions.md`
> ⚠️ _ID drift to reconcile: this card was labeled **P21** on the canvas but the authoritative catalog (`BEST-PRACTICES.md`) cards Service Assistant as **P18**. Aligned to the catalog here. A human should confirm and fix any lingering `P21` references in other surfaces._

**P19 · Live-demo runbook + fallback matrix** — story → demo arc → off-stage pre-flight → golden records → verbatim prompts → talk track → **exhaustive fallback matrix** (symptom→cause→≤60s recovery) + composure rules ("never debug live >30s"). ✅ High-stakes customer/on-stage demos. ⛔ Async/recorded or internal smoke tests. → `patterns/templates/live-demo-runbook.md`

**P20 · Synthetic demo-data guide** — discover schema FIRST → design hero records → seeded Faker generate → load parent→child (or Data Cloud Ingestion API + IR) → repeatable teardown. ✅ Disposable, reproducible demo data. ⛔ Production migration; demos on existing data. → `patterns/templates/synthetic-demo-data-guide.md` (+ `patterns/canonical/fwa-demo-data-guide.md`)

**P21 · Headless multi-surface MCP tool-plane** — publish capability **once** as MCP tools, consume from N surfaces (agent/Slack/web/script) on one contract; boundary rule (decision→agent, fetch→MCP direct) + least-privilege read/write server split. ✅ Multi-surface AI systems. ⛔ Single surface, no reuse. Distinct from P10 (rendering). → `patterns/templates/headless-mcp-tool-plane.md`

> ⚠️ _Earlier canvas drafts numbered Service Assistant as both P18 and P21. The authoritative catalog (`BEST-PRACTICES.md`) is the source of truth: Service Assistant = **P21**; the demo/data/tool-plane cards above it are P18–P20. A human should reconcile any stray P18-labeled Service Assistant references on other surfaces._

**P22 · LWC wire-vs-imperative** — one table: reads→`@wire` (cacheable, `refreshApex`), writes→imperative (error control). ✅ Non-trivial LWC mixing reads+writes. ⛔ Trivial single-read components. → `patterns/templates/lwc-wire-vs-imperative.md`

**P23 · Ask→Plan→Agent co-authoring** — context-gather → section-by-section refine → reader-tested implement, mapped to Cursor Ask/Plan/Agent with a plan-approval gate before code. ✅ Co-authoring docs an agent will implement. ⛔ Tiny unambiguous changes. Distinct from P4 (structures the docs; P23 structures producing+executing them). → `patterns/templates/ask-plan-agent-coauthoring.md`

**P24 · LWC concurrent-save queue + debounce** — ~500ms input debounce + single-slot in-flight queue (`_isLoading` + one `_pendingSave`). ✅ Auto-saving LWC forms with overlapping writes. ⛔ Save-button forms; ordered multi-save batching. → `patterns/templates/lwc-concurrent-save-queue.md`

**P25 · Advisory validation** — save always succeeds (never a DML gate); violations are non-blocking alerts; only Submit is gated. ✅ Draft-style builders where users save partial/invalid work. ⛔ Transactional writes that must never persist invalid (payments/compliance). → `patterns/templates/advisory-validation.md`

**P26 · Scoped UI-polish prompt pack** — batch small UI changes as sequenced items: component + exact files + **model to use** + expected result + confirm-gate + per-item rollback. ✅ Isolated CSS/visual tweaks run one-at-a-time; delegate cheap work to a smaller model. ⛔ Cross-cutting refactors; a single change. → `patterns/templates/scoped-ui-prompt-pack.md`

> _**LWC/Experience-Cloud bundle (S5 + S6 + R13 + P22 + P24 + P25):** the first non-Agentforce surface in the catalog — reach for these together on Experience Cloud / LWR builds (calendar UI, address pickers, design tokens, wire/imperative, auto-save, advisory validation)._
> _P6 (live-site) and P8 (org metadata) are the two "capture existing reality → structured artifact" reverse-engineering patterns — different domains, same shape._
> _**Demo toolkit (P9 + P19 + P20 + P21):** P19 is the run-of-show & recovery plan, P9 the LIVE/DEMO toggle for probabilistic beats, P20 the data the demo runs on, P21 the multi-surface "headless" story being shown. Reach for them together when prepping a customer demo._
> _**Verify-don't-guess pair (R10 + S4):** R10 is the rule ("never assert schema from memory"); S4 (`data-cloud-sql-runner`) is the one-command introspection that satisfies it for Data Cloud._
> _**Salesforce knowledge libraries (P7 + P14 + P15 + P16), authored via P17:** P7 = object/field *schema*; P14/P15/P16 = feature *behavior/how-to*. P17 is the reusable workflow that produces all of them. Reach for P17 first when starting on a dense-doc platform feature._
> _P9 (playbook) and P10 (architecture) are a pair: P9 is the **how to present**, P10 is the **how to build** the reliable-yet-authentic AI demo._
> _**Debugging toolkit (P3 + P11 + P12 + P13):** P3 delegates an observation to another surface → P13 is the method that produces it → P11 is where evidence lands → P12 re-briefs the next session. Reach for them together on hard, cross-session/cross-tool investigations._

---

## ⚠️ Known gaps / cleanup
- Rules live only in `hrMelody`; **install templates into `hrMelodyv3`** (R1–R4 + new R5/R6).
- De-dupe `docs/design/{A,B,C}`, the BRD, and the CLT guide across `hrMelody` ↔ `sfdxMelodyv3`.
- Naming drift Melody→Harmony→Symphony — templates use `PROJECT_NAME`/`ORG_ALIAS` placeholders.
- Archive old versioned `.agent` files; keep only the latest.
- `hrMelodyv3/melody-ui/` and `melody-ui-Demo/` have **byte-identical** `HANDOFF.md` + `DEMO-CLICKPATH.md` — keep `melody-ui/` canonical, archive the twin.
- `sfdxMelodyv3/docs/demo-scenario-map.md` has a stale `Harmony/Melody` header but a Symphony/Sonata body — fix the header in-project.
- Install R7 (`.cursorignore`) where a superseded twin still gets indexed (e.g. legacy `Harmony` agent dirs).
- `NASA NSPIRES PostAwardBuild` has no rules/skills/AGENTS.md — **install R1–R4 templates** there.
- PSS Grantmaking now spans P7 (schema, `PostAwardExtract`) + P16 (features, `PostAwardBuild`) — consider merging into one "PSS Grantmaking" library if a third contribution lands.
- `FHA Call Center`'s `sf-docs-mcp` server (S3) is forkable IP living inside a customer project — **extract it to a standalone GitHub repo** so the skill can link a real install source.
- `FHA Call Center` has no root `.cursor/rules/` — its hard-won lessons (R8) live in a doc the agent won't auto-load. **Promote `Lessons-Learned` into `.cursor/rules/` at root** in that project, and pair R9 + `AGENTS.md` with the MCP server.
- `ItineraryBuilder` has **empty `.cursor/`** and no `AGENTS.md` — all IP is prose in design docs. **Install R1–R4 + R11–R13 templates** there; convert its design docs' reading-order into an `AGENTS.md` (R4).
- `ItineraryBuilder` has **two contradictory `AGENT_REFERENCE` files** (root vs `DesignDocs/`) disagreeing on Case-Id delivery — keep the newer `DesignDocs/AGENT_REFERENCE.md` canonical, retire the root one.
- This repo now catalogs **multiple unrelated accounts** (Hard Rock, DOS PCS, NASA, HUD, FWA) but lives under a Hard Rock Drive path — consider renaming the repo scope to a neutral "GPS Cursor Best Practices."
- **ID drift (catalog ↔ canvas):** Service Assistant is **P18** in `BEST-PRACTICES.md` but was **P21** on this canvas — aligned to P18 above; sweep other surfaces for stale `P21=Service Assistant` references.
- `FWA Federal Fraud`'s four demo entities' unified IDs/scores **disagree across its own docs** (`graph-schema.mdc` vs SE runbook vs `demo/CLAUDE.md`) because IDs drift on Identity-Resolution re-run. Fix in-project: search by **EIN** and re-derive live IDs (captured as a P20 caution).
- `FWA Federal Fraud` is the first contributor that **already had** `.cursor/rules/` + shell skills — no "install templates" gap; instead it *donated* R10 + S4. Promote its `sf-docs-mcp` (same as S3) consolidation note.
