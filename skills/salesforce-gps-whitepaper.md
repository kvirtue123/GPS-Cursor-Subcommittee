---
name: salesforce-gps-whitepaper
description: Converts a Markdown document into a Salesforce GPS-branded white paper PDF using the standard whitepaper.css template. Use when the user provides or points to a .md file and asks to create, generate, or update a GPS white paper or branded PDF. The HTML is an internal build artifact — the deliverable is the PDF (and optionally a clean .md mirror). Covers the full pipeline: inputs gathering, HTML authoring, headless Chrome rendering, PDFKit Swift verification, and PNG preview inspection.
---

# Salesforce GPS White Paper Pipeline

**Input:** a `.md` source file  
**Deliverable:** a branded `.pdf` (HTML is a build artifact, not a deliverable)

---

## Step 0 — Read the authoring guide first

```
/Users/kvirtue/Desktop/WhitepaperGeneration/Salesforce GPS White Paper — Authoring.md
```

Read it in full before authoring any HTML. It is the authoritative reference for layout rules, CSS constraints, render command, and the end-of-session checklist. This skill summarises the critical points; the guide is the source of truth.

---

## Step 1 — Gather inputs (ask only these)

| # | Input | Notes |
|---|---|---|
| 1 | **Source Markdown** | Path or pasted content |
| 2 | **Output filename** | e.g. `My_Paper` → produces `My_Paper.pdf` |
| 3 | **Cover title** | Exact string; note preferred line-break point |
| 4 | **Cover subtitle** | e.g. `Internal GPS Briefing · May 2026` |
| 5 | **Section divider titles** | One per section; indicate which words get accent color |
| 6 | **Stat-card callouts** | Eyebrow / big number / body — up to one per content page (optional) |
| 7 | **Back cover CTA** | Email address + button label (omit if internal-only) |
| 8 | **Shorten-to-fit allowed?** | Default: supporting prose may trim ~10%; hero content kept verbatim |

Do **not** ask about renderer, fonts, colors, page size, or padding — those are fixed by `whitepaper.css`.

---

## Step 2 — Author the HTML (build artifact)

Save as `White Paper/<Name>.html`. Follow the skeleton from guide §4 exactly:

```
cover → exec-summary .page → (section-divider + .page …) × N → [appendix .page] → back-cover
```

### CSS rules (non-negotiable)

- **Never** `position: absolute` or `overflow: hidden` on `.page` / `.page__body` — this is the silent-truncation trap (guide §2.1).
- **Never** modify `whitepaper.css`. Scoped additions go in a `<style>` block in `<head>`.
- Use `page-break-inside: avoid; break-inside: avoid` on any callout card.

### Key classes (all in `whitepaper.css`)

| Component | Class |
|---|---|
| Cover | `section.cover` |
| Content page | `section.page` |
| Section divider | `section.section-divider` |
| Back cover | `section.back-cover` |
| Running head | `div.running-head` |
| Page number | `div.folio` |
| Stat callout | `aside.stat-card` → `__eyebrow`, `__number`, `__body` |
| Standard table | `table.matrix` |
| Compact table | `table.matrix--compact` (9 px font) |
| Vendor profile | `div.vendor` → `__head`, `__cols`, `__col`, `__col--limit` |

### Markdown → HTML mapping (guide §2.7)

| Markdown pattern | Required HTML |
|---|---|
| Bullet lists | `<ul><li>` — never `<p>` elements |
| Callout stat row | `<aside class="stat-card">` |
| Comparison table | `<table class="matrix">` |
| Section H1 | `section-divider` + `section.page` pair |
| Final section / closing | `section.back-cover` with real `mailto:` CTA |

### Pagination heuristic (guide §2.5)

If a section has > ~6 H3s + a matrix + a stat-card, pre-split into two sibling `.page` blocks. Label the second: `(continued)` in the title. Update all subsequent `<div class="folio">` numbers after any split.

---

## Step 3 — Render PDF

```bash
cd "White Paper"
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf-no-header \
  --virtual-time-budget=10000 \
  --print-to-pdf="<Name>.pdf" \
  "file://$(pwd)/<Name>.html"
```

Do **not** use weasyprint, wkhtmltopdf, or any Chromium variant — the CSS is calibrated specifically against Chrome's Skia/PDF print engine (guide §2.8).

---

## Step 4 — Verify with PDFKit (mandatory before declaring done)

```bash
/usr/bin/swift - << 'EOF'
import Foundation
import PDFKit
guard let doc = PDFDocument(url: URL(fileURLWithPath: "<Name>.pdf")) else { exit(1) }
print("Total pages: \(doc.pageCount)")
var all = ""
for i in 0..<doc.pageCount {
    let p = doc.page(at: i)!
    let raw = p.string ?? ""
    all += raw.lowercased() + "\n"
    let first = raw.split(separator: "\n").first.map(String.init)?
                   .prefix(80).trimmingCharacters(in: .whitespaces) ?? "(empty)"
    let flag = raw.count < 80 ? " *** REVIEW ***" : ""
    print("  p\(i+1) len=\(raw.count): \(first)\(flag)")
}
// Populate with every section heading, stat-card number, and key term
let mustAppear: [String] = []
// Populate with any content that was intentionally removed
let mustNOT: [String] = []
print("\n--- Must appear ---")
for m in mustAppear { print("  [\(all.contains(m.lowercased()) ? "OK" : "MISSING")] \(m)") }
print("\n--- Must NOT appear ---")
for m in mustNOT { print("  [\(all.contains(m.lowercased()) ? "STILL PRESENT" : "REMOVED")] \(m)") }
EOF
```

> **Letter-spacing gotcha:** CSS `letter-spacing` causes Chrome to extract styled eyebrow text as `E X E C U T I V E  S U M M A R Y`. If a term reports MISSING, dump the surrounding page content before assuming it is absent.

---

## Step 5 — PNG preview (visual inspection)

```bash
mkdir -p /tmp/wp_preview
/usr/bin/swift - << 'EOF'
import AppKit, Quartz
guard let doc = CGPDFDocument(URL(fileURLWithPath: "<Name>.pdf") as CFURL) else { exit(1) }
for i in 1...doc.numberOfPages {
    guard let page = doc.page(at: i) else { continue }
    let box = page.getBoxRect(.mediaBox)
    let scale: CGFloat = 1.5
    let w = Int(box.width * scale), h = Int(box.height * scale)
    let rep = NSBitmapImageRep(bitmapDataPlanes: nil, pixelsWide: w, pixelsHigh: h,
        bitsPerSample: 8, samplesPerPixel: 4, hasAlpha: true,
        isPlanar: false, colorSpaceName: .deviceRGB, bytesPerRow: 0, bitsPerPixel: 0)!
    let ctx = NSGraphicsContext(bitmapImageRep: rep)!
    NSGraphicsContext.current = ctx
    let cg = ctx.cgContext
    cg.setFillColor(CGColor.white); cg.fill(CGRect(x:0,y:0,width:w,height:h))
    cg.scaleBy(x: scale, y: scale); cg.drawPDFPage(page)
    try! rep.representation(using: .png, properties: [:])!
        .write(to: URL(fileURLWithPath: "/tmp/wp_preview/p\(String(format:"%02d",i)).png"))
}
print("Done — \(doc.numberOfPages) pages → /tmp/wp_preview/")
EOF
```

Use the Read tool to inspect each PNG. At minimum verify: cover, exec summary, every section divider, first content page of each section, back cover.

---

## Step 6 — Fix orphan / overflow pages

1. Any page with `len < 80` in PDFKit output needs investigation.
2. Dump it: replace `IDX` with the 0-based index.

```bash
/usr/bin/swift - << 'EOF'
import Foundation; import PDFKit
let doc = PDFDocument(url: URL(fileURLWithPath: "<Name>.pdf"))!
print(doc.page(at: IDX)!.string?.prefix(600) ?? "(empty)")
EOF
```

3. Fix by splitting the overflowing `<section class="page">` earlier, re-render, re-verify.

---

## End-of-session checklist (guide §5)

- [ ] Rendered with the Chrome command above
- [ ] PDFKit ran — all `mustAppear` terms show OK, no `*** REVIEW ***` pages
- [ ] PNGs generated and visually inspected (cover, dividers, back cover at minimum)
- [ ] Page count reported; orphan/unexpected pages listed
- [ ] Back cover CTA uses real `mailto:` href
- [ ] No `<p>` elements where `<ul><li>` is required
- [ ] No `position: absolute` on `.page` or `.page__body`
- [ ] `whitepaper.css` is unchanged

---

## Deliverables

| File | Status |
|---|---|
| `White Paper/<Name>.pdf` | **Primary deliverable** |
| `White Paper/<Name>.html` | Build artifact (keep, but not the user's output) |
| `White Paper/<Name>.md` | Optional clean Markdown mirror — update if content was edited during PDF production |
