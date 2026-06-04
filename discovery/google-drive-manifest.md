# Google Drive Manifest — Supporting Materials

Google Drive is for **supporting materials**: recordings, decks, customer background, onboarding — anything that isn't versioned/installable code. Code and docs stay in GitHub; discovery cards stay in the Slack Canvas.

## Suggested Drive folder structure

```
Cursor Best Practices/
├── Recordings/
│   ├── Melody V2 - Salesforce Demo.mp4
│   ├── Live and Simulated Agents.mp4
│   └── live_vs_demo-melody.mp4
├── Decks/                         ← exec / enablement decks (add as produced)
│   └── Agentforce Custom Lightning Types Implementation Guide (Aki Hirano, Mar 2026)  ← source deck behind S1/P2

├── Customer Background/
│   ├── Windsurfer Reservations Requirements (BRD 03.09.26).md/pdf
│   ├── Melody Metrics (provided by Customer).md
│   ├── Hard Rock KT.md
│   └── NSPIRES Modernization — Award Phase Requirements (RTM).md
├── Org Exports/                  ← raw metadata/config exports (reference, not installable)
│   └── Public Sector & Nonprofit – Grantmaking.json  (PSS Grantmaking config export)
├── Whitepaper Template (S2 support)/
│   ├── whitepaper.css
│   └── Salesforce GPS White Paper — Authoring.md
└── Onboarding/
    └── "How to use the Cursor Best Practices repo" (1-pager)
```

## Items found in the workspace to upload

| File | Current location | → Drive folder |
|---|---|---|
| `Melody V2 - Salesforce Demo.mp4` | `hrMelodyv3/` | Recordings/ |
| `Live and Simulated Agents.mp4` | `hrMelodyv3/` | Recordings/ |
| `live_vs_demo-melody.mp4` | `hrMelodyv3/` | Recordings/ |
| Windsurfer BRD | `hrMelody/docs/Background/`, `sfdxMelodyv3/docs/` | Customer Background/ (de-dupe first) |
| Melody Metrics | `hrMelody/docs/Background/` | Customer Background/ |
| Hard Rock KT | `hrMelody/docs/Background/` | Customer Background/ |
| `whitepaper.css` + Authoring guide | referenced by S2 skill (`~/Desktop/WhitepaperGeneration/`) | Whitepaper Template/ |
| GPS brand assets (cover/section PNGs, `spark_*.png` ~3–5 MB each, logo SVGs, fonts) | `WhitepaperGeneration/assets/` | Whitepaper Template/Brand Assets/ (binaries — do NOT commit to git) |
| `extracted-tokens.json` (sample W3C token output, P6) | `WhitepaperGeneration/` | optional → Whitepaper Template/ as a P6 example artifact (per-project, not shared) |
| Aki Hirano CLT Implementation Guide (original deck) | source of `sfdxMelodyv3/docs/agentforce-custom-lightning-types.md` | Decks/ — the `.md` extract stays canonical in P2; the deck is human-facing supporting material |
| `Requirements Document: NSPIRES Moderniza.md` (Award-phase RTM/requirements) | `NASA NSPIRES PostAwardBuild/docs/` | Customer Background/ — customer requirements, not ours to version publicly |
| `Public Sector & Nonprofit – Grantmaking.json` (~28 KB PSS config export) | `NASA NSPIRES PostAwardBuild/docs/` | Org Exports/ — raw export, reference only; the curated P7/P16 guides are the installable assets |
| `Demo-Script-v3-FINAL.md` (HUD FHA demo narration) | `FHA Call Center/docs/` | Decks/ or a new `Demo Scripts/` — human-facing presenter material, not an installable asset |
| `BuildandConfigure_409022.webp` (~240 KB build/config screenshot) | `FHA Call Center/` | Org Exports/ or `Screenshots/` — binary, do NOT commit to git |
| `ServiceAgentData.csv` (~12 MB demo seed data) | `FHA Call Center/` | Org Exports/ — large binary seed file; reference only, never commit to the repo (the `scripts/demo/` seeders are the reusable part) |
| `headless360_se_runbook.docx` (polished SE demo runbook, source of P19) | `FWA Federal Fraud/docs/` (the `.docx.md` extract) | `Demo Scripts/` — human-facing presenter runbook; the genericized P19 template is the installable asset, this is the polished deliverable |
| `Salesforce Headless 360` enablement deck (if produced from the runbook) | — (to be produced) | Decks/ — exec/enablement deck for the Headless-360 four-channel story (P21) |

## Contributing-project notes

| Project | Supporting materials for Drive | Notes |
|---|---|---|
| `PostAwardExtract` (NASA NSPIRE post-award) | _None_ | Contributed patterns **P7** (data-model reference) and **P8** (org extraction & audit) are text-only GitHub assets — no decks/recordings/binaries to upload. If a NASA NSPIRE demo recording or customer BRD surfaces later, route it to `Customer Background/` or `Recordings/` here (don't commit to the repo). |
| `sfdxMelodyv3` (Symphony/Sonata booking agent) | Aki Hirano CLT deck → `Decks/`; Windsurfer BRD (already listed, de-dupe with `hrMelody`) → `Customer Background/` | Contributed rule **R7** and patterns **P11/P12/P13** are text-only GitHub assets. The only supporting material is the Aki Hirano deck (source of the P2 CLT guide). |
| `NASA NSPIRES PostAwardBuild` (NASA NSPIRE post-award) | NSPIRES Award-phase RTM/requirements → `Customer Background/`; PSS Grantmaking config export → `Org Exports/` | Contributed patterns **P14–P17** are text-only GitHub assets (knowledge libraries + the doc-derived workflow). Two supporting items only; no recordings/decks/binaries. |
| `FHA Call Center` (HUD Agentforce Service Assistant) | Demo script → `Demo Scripts/`; build/config screenshot + `ServiceAgentData.csv` seed file → `Org Exports/` | Contributed rules **R8/R9**, skill **S3** (`sf-docs-mcp`), and pattern **P18** are GitHub assets. Note: the `sf-docs-mcp` *server source* is forkable IP — promote it to its own GitHub repo (not Drive). The HUD/FHA `docs/research/` + `docs/FAQ/` content is customer/domain background — route to `Customer Background/` if shared, don't commit. |
| `FWA Federal Fraud` (synthetic-data → Data Cloud → GNN → Agentforce/MCP/Slack) | SE demo runbook (`headless360_se_runbook.docx`) → `Demo Scripts/`; any Headless-360 enablement deck → `Decks/` | Contributed rule **R10**, skill **S4** (`data-cloud-sql-runner`, scripts committed), and patterns **P19/P20/P21** are GitHub assets. The synthetic **dataset CSVs** and **GNN model weights** (`gnn/model_best.pt`, multi-MB) are large binaries — keep in the project / model registry, never commit to this repo; reference only. No customer-confidential background (data is synthetic). |

## Items from `ItineraryBuilder` (DOS PCS Portal) to upload — added 2026-06-04

These are **supporting/reference material only** (links + customer-facing context). The reusable code/docs were re-homed as templates/skills; nothing below is committed to the repo.

| Item | Current location | → Drive folder |
|---|---|---|
| Demo wireframe (customer) — `pcsdemo.netlify.app` ("Orders") | linked in `Demo Requirements.md` | Customer Background/ (link doc) |
| Live demo site — `dos-agentforce-sdo-20250131.my.site.com/pcs` | `AGENT_REFERENCE.md` | Customer Background/ (link doc) |
| Reference inspiration — Safari Portal / Itinerary Builder blog | `Demo Requirements.md` | Customer Background/ (link doc) |
| Any recorded PCS Portal demo (`.mp4`, if produced) | not in repo | Recordings/ |

> No large binaries currently exist in the `ItineraryBuilder` project — these are URLs/context to capture as a short "PCS Portal — background & links" doc in Drive, cross-linked to the repo + Slack Canvas.

## Why these go to Drive (not GitHub)
- Large binaries (`.mp4`, multi-MB PNGs) bloat git history.
- Customer-provided background isn't ours to version publicly.
- Decks/recordings are consumed by humans, not installed by agents.

## Cross-links
- Each Drive folder should link back to the **GitHub repo** and the **Slack Canvas**.
- The Slack Canvas links to both the repo and this Drive folder (fill in URLs at the top of `slack-canvas.md`).
