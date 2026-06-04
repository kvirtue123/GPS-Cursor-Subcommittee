# Rules

Behavioral guardrails for Cursor projects — small `.mdc` config files that change how the agent behaves (architecture to follow, when to update docs, what to verify before writing code). See the [catalog](../BEST-PRACTICES.md) and [root README](../README.md) for context.

## Where to add

- **`templates/`** — install-ready rule files others can drop into their project's `.cursor/rules/`. **Add your contributions here.**

## Contributing a rule

Open a PR adding your `.mdc` file to `templates/`, or post it in the channel Feed tagged `[Rule]`. The bar: it came from real project work, it's reusable in more than one context, and it has a clear "use this when / not when."
