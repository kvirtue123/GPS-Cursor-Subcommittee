# GPS Cursor Subcommittee — Best Practices

A shared, collaborative catalog of reusable Cursor assets — **Rules**, **Skills**, and **Patterns** — contributed by the subcommittee, for the subcommittee. The goal is simple: stop re-solving the same problems on every engagement. Someone figures it out once, it lands here, everyone else pulls from it.

Every entry comes from real project work, not speculation. It grows as members add to it.

## Asset types

- **Rules** — behavioral guardrails you install as small config files in `.cursor/rules/`. They shape how the agent works on your project (architecture to follow, when to update docs, what to verify before writing code).
- **Skills** — reusable task recipes you invoke by name. The agent follows a complete, tested workflow instead of you re-explaining it each time.
- **Patterns** — repeatable workflows and approaches in plain Markdown. Not installed code — structured guidance you keep in your project and reference when you need it.

See **[`BEST-PRACTICES.md`](BEST-PRACTICES.md)** for the catalog inventory — what exists and what each asset covers. Installable files live in `rules/`, `skills/`, and `patterns/`.

## How to use it

1. **Browse [`BEST-PRACTICES.md`](BEST-PRACTICES.md)** to see what exists and find what fits your problem.
2. **Grab the file** for the asset you want from the repo.
3. **Drop it into your project** — Rules go in `.cursor/rules/`, Skills are invoked by name, Patterns live in your `docs/` for reference.

New to it? **R4 (AGENTS.md)**, **R1 (architecture lock)**, **R2 (doc-sync)**, and **P12 (session briefing)** pay off immediately on any project.

## Contribute

This is the whole point — the catalog is only as good as what members put in. If something worked on a project, or a failure mode is worth encoding so no one else hits it, add it.

**The bar:** it came from real project work, it's reusable in more than one context, and it has a clear "use this when / not when."

**How:** open a PR adding your file to the right folder — `rules/templates/`, `skills/`, or `patterns/templates/` — or post it in the channel Feed tagged `[Rule]`, `[Skill]`, or `[Pattern]` and a co-lead will help land it and assign an ID. Each folder's `README` says exactly what goes where.

## Links

- 💬 **Slack:** #gps-ae-club-cursor-subcommittee
- 📖 **Catalog:** [`BEST-PRACTICES.md`](BEST-PRACTICES.md)

---

_GPS Cursor Subcommittee · Leads: Keegan Virtue, Steve Shin_
