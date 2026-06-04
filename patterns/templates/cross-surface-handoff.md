# Cross-Surface Agent Handoff — Template

A structured "handoff packet" for delegating a task from one agent/surface to another (e.g. Claude Code → Cursor, or model A → model B) when the current surface lacks a needed capability (browser DevTools, a specific MCP, a different model).

> Source pattern: `sfdxMelodyv3/docs/debug/cursor-handoff-lwc-inspect.md`.

---

## Why {{TARGET_SURFACE}}
> One or two lines: what capability the target surface has that the current one lacks, and the single goal of the handoff.

## Context for the receiving agent
> Everything the receiver needs and can't infer. Be concrete.
- Project / org / environment: {{…}}
- The exact symptom or task: {{…}}
- What's already confirmed working: {{…}}
- Relevant IDs, URLs, file paths: {{…}}
- Relevant prior findings: {{…}}

## Diagnostic / execution steps
> Numbered, runnable steps. Include exact commands and what to look for at each.
1. {{step}} — expected: {{…}}
2. {{step}} — expected: {{…}}

## Expected-outcomes table
> Pre-enumerate the branches so the receiver can self-diagnose.

| Outcome | Diagnosis / next action |
|---|---|
| {{outcome A}} | {{…}} |
| {{outcome B}} | {{…}} |

## Reporting back
> The contract for what to return and where.
- Append findings to: `{{REPORT_FILE}}` under heading `## {{DATE}} {{SURFACE}} findings`
- Include: {{checklist of required findings}}
- Then return to {{ORIGIN_SURFACE}} with the report.

---

### When NOT to use
If the current agent can finish the task itself, skip the packet — it's pure overhead. Use this only for genuine capability/surface gaps.
