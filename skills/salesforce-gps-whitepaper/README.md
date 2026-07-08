# salesforce-gps-whitepaper (a.k.a. `wpGenerator`)

> Convert a Markdown source into a **Salesforce-GPS-branded white-paper PDF** via a fixed, verified pipeline.
> This is the packaged support folder for catalog asset **S2**. The canonical skill body is documented at [`../salesforce-gps-whitepaper.md`](../salesforce-gps-whitepaper.md).

## What this is

A repeatable pipeline that takes a `.md` source and produces a branded PDF:

```
gather inputs → author HTML (build artifact) → headless Chrome render
            → PDFKit verification → PNG visual inspection → fix overflow
```

The **PDF is the deliverable**; the HTML is an internal build artifact. The whole pipeline exists to defeat one historical failure mode: **silent content truncation** when paged-media CSS clips overflowing pages with no warning.

## When to use / when NOT to

- ✅ Turning an internal `.md` into a branded PDF using the standard GPS template.
- ⛔ Non-GPS branding, decks (use Slides/PPTX tooling), or live/HTML deliverables.
- ⛔ Do **not** swap the renderer — the CSS is calibrated to Chrome's Skia/PDF print engine.

## Dependencies

- **macOS** (uses headless Google Chrome + `/usr/bin/swift` + PDFKit — all preinstalled on macOS).
- `whitepaper.css` (the FY26 Industries Ebook/White Paper template) + the Authoring guide. Both are **support materials → Google Drive** (see `discovery/google-drive-manifest.md`), referenced by the skill.

## What's inside this folder

```
salesforce-gps-whitepaper/
├── README.md                  ← this file
└── whitepaper-skeleton.html   ← the {{TOKEN}} HTML structural template (cover → exec → dividers → pages → back-cover)
```

The skill's full step-by-step body lives in the sibling flat file `../salesforce-gps-whitepaper.md` (kept there so existing catalog links remain valid).

## Install

```bash
# 1. Make the skill active
cp ../salesforce-gps-whitepaper.md ~/.cursor/skills/salesforce-gps-whitepaper/SKILL.md
# 2. Stage the support assets the skill expects (from Drive: Whitepaper Template/)
#    - whitepaper.css
#    - Salesforce GPS White Paper — Authoring.md
# 3. Use whitepaper-skeleton.html as the starting HTML structure.
```

> **Portability note (provenance):** the as-shipped skill body hardcodes
> `/Users/kvirtue/Desktop/WhitepaperGeneration/Salesforce GPS White Paper — Authoring.md`
> in its "Step 0 — read the guide" pointer. When installing into a new project, repoint that
> to wherever the guide lives locally (or to the Drive copy). This is the one non-portable line.

## Provenance

- Source project: `WhitepaperGeneration` (`~/Desktop/WhitepaperGeneration/`).
- A byte-identical copy of the skill exists at `~/.cursor/skills/salesforce-gps-whitepaper/SKILL.md` — keep the repo copy canonical; the user-dir copy is the *installed* instance.
