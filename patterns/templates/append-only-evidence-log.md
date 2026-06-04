# Append-Only Evidence Log — Template

A single, durable Markdown file that accumulates evidence across a long or multi-session
debug. It is **append-only**: new findings are added under dated headings, never rewritten.
This survives context autocompaction and session boundaries — a fresh agent (or teammate)
can reconstruct the entire investigation from one file.

> Source pattern: `sfdxMelodyv3/docs/debug/symphony-carousel.md` (a multi-day, multi-tool
> carousel-render investigation that crossed Claude Code ↔ Cursor and several sessions).

---

## Header block (write once, at the top)

```markdown
# {{INVESTIGATION_NAME}} — Evidence Log

> **Purpose:** Append-only evidence log so context survives autocompact / session boundaries.
> **Started:** {{DATE}}
> **Symptom:** {{exact, reproducible symptom — quote literal strings the user/agent sees}}
> **Repeats on:** {{versions / environments where the symptom occurs}}
```

## Standing status sections (update in place — these are the exception to append-only)

```markdown
## What's verified ✅
- {{fact}} — {{how it was confirmed (command, trace, direct observation)}}

## What's NOT verified
- {{open question still to be tested}}

## What's the smoking gun
- {{the single most decisive piece of evidence so far}}
```

## Per-session evidence entries (append a new one each session — never edit prior ones)

```markdown
## {{DATE TIME}} {{SURFACE/AGENT}} findings — {{short label}}

### Method
{{what you did to produce this evidence — the experiment, the capture, the query}}

### Findings
- {{raw observation 1 (paste trace excerpts, SOQL output, SSE frames, console errors)}}
- {{raw observation 2}}

### Layers eliminated / confirmed this session
- **{{layer}} — {{EXONERATED | CONFIRMED FIRING | STILL SUSPECT}}.** {{why}}

### Narrowed conclusion
{{what the evidence now points to; what's left to test}}
```

---

## Rules of the pattern
1. **Append, don't rewrite.** Prior entries are the audit trail; correcting them in place destroys the record. If a past conclusion was wrong, add a new entry that says so.
2. **Raw evidence over summary.** Paste the literal trace/SSE/SOQL/console output. Summaries hide the detail that cracks the case later.
3. **Date + surface every entry.** Multi-tool work (Claude Code, Cursor, Builder) needs to know *who* observed *what*, *when*.
4. **One file per investigation.** Reference it from chat by path so every session writes to the same log.
5. **Pairs with the cross-surface handoff (P3) and session-brief (P7).** The handoff says "go look"; this log is where findings land; the brief points new sessions at this log.

### When NOT to use
A short bug fixed in one sitting doesn't need a durable log — just fix it. Reach for this only when the investigation spans sessions/tools or the context is at risk of being compacted away.
