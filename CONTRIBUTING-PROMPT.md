# Follow-up Audit Prompt (run in each other project)

Paste the block below into each of your other Cursor projects **after** running the original best-practices audit prompt there. It tells that project's agent to contribute its assets into the **already-established** `cursor-best-practices/` repo using the same structure, IDs, and conventions — instead of inventing a new layout.

> **Before running:** set the one variable so the agent knows where the shared repo is.
> `REPO_PATH = /path/to/cursor-best-practices`

---

## PROMPT — copy from here ↓

I previously ran a best-practices audit that produced a shared repository at:

`REPO_PATH = <paste the path above>`

That repo already defines the canonical structure, asset categories, documentation template, ID scheme, and home-routing rules. **Do not invent a new structure.** Your job is to audit THIS project and **merge** its reusable assets into that existing repo cleanly — additively, without duplicating or overwriting what's already there.

### Step 0 — Read the existing repo first (mandatory)
Read these before doing anything else, and conform to them exactly:
- `REPO_PATH/README.md` (repo map + install conventions)
- `REPO_PATH/BEST-PRACTICES.md` (the catalog, the per-asset documentation template, and the existing asset IDs R1–R4 / S1–S2 / P1–P5)
- `REPO_PATH/TOPICS.md` (feature-area index, cross-links rules/skills/patterns/knowledge)

### Step 1 — Audit THIS project
Scan this project tree for the three categories:
- **Rules** — `.cursor/rules/*.mdc`, any legacy `.cursorrules`, `AGENTS.md`/`CLAUDE.md`.
- **Skills** — authored `SKILL.md` packages (distinguish *authored* skills from inherited platform skills; only authored ones get re-homed).
- **Patterns** — knowledge libraries, best-practice guides, reusable workflow/handoff/architecture docs, prompt structures.

### Step 2 — Diff against the existing catalog (this is the important part)
For every asset you find, classify it as one of:
- **DUPLICATE** of an existing catalog asset (e.g. another copy of the Agent Script library or CLT guide) → do NOT re-add. Note it under "duplicates" and, if this project's copy is newer/better, flag which should become canonical.
- **VARIANT** of an existing asset (same idea, project-specific specifics) → do NOT create a new ID. Instead, fold its reusable specifics into the existing genericized template (e.g. add a recurring deploy gotcha to `rules/templates/salesforce-deploy-gotchas.mdc`), and add this project to that asset's provenance.
- **NEW** asset with no existing equivalent → assign the next sequential ID in its category (continue R5+, S3+, P6+; never reuse a number), and document it with the SAME template used in `BEST-PRACTICES.md`: *what / problem solved · when to use · when NOT to use · dependencies · canonical example · recommended home*.

### Step 3 — Write changes into the existing repo (additive only)
- Put genericized, drop-in versions in `rules/templates/` or `patterns/templates/` using the same `{{PLACEHOLDER}}` token convention already in the repo.
- Package authored skills as `skills/<skill-name>/` (copy `SKILL.md` + `README.md`).
- **Append** new asset cards to `BEST-PRACTICES.md` (don't rewrite existing entries); update the "Recommended-home summary" table and the "Gaps & duplicates" section.
- Add raw domain/reference content to `knowledge/` (tag with `feature_area`, `catalog_id`, `status`).

### Step 4 — Home-routing (use the repo's existing rules)
- **GitHub** → anything versioned/installable/forkable (rules, skill packages, guides, templates).
- **Slack Canvas** → a short discovery card per asset (what / when / when-not / link back to repo).
- **Google Drive** → supporting material only (decks, recordings, customer background, large binaries). Add to the manifest, don't commit binaries to the repo.

### Step 5 — Report back
Output a concise summary:
1. Assets found in this project, grouped by category.
2. Classification of each: DUPLICATE / VARIANT / NEW (+ the ID assigned to each NEW one).
3. Exact list of files you created or appended to in `REPO_PATH`.
4. Any cross-project duplicates or naming drift you noticed, and which copy you recommend as canonical.

### Guardrails
- Do not duplicate inherited Salesforce/Cursor platform skills — only authored IP.
- Do not renumber or rewrite existing R/S/P assets.
- Do not commit large binaries (`.mp4`, multi-MB images) — route them to the Drive manifest.
- Preserve placeholder tokens in templates; do not hardcode this project's specifics into a shared template.

## PROMPT — copy to here ↑
