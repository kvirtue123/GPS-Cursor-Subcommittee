# Platform Data-Model Reference Library — Template

A drop-in skeleton for capturing an out-of-the-box (or managed-package) Salesforce data model
as a single, `@`-referenceable knowledge file. Fill the `{{PLACEHOLDER}}` tokens.

> Source pattern: `PostAwardExtract/docs/grantmaking-objects.md` (PSS Grantmaking).

---

```markdown
# {{PRODUCT_NAME}} — Objects & Field Metadata Reference

> **Source**: {{SOURCE_GUIDE}} ({{RELEASE}} / API {{API_VERSION}})
> **License Required**: {{LICENSE_REQUIREMENT}}; {{PERMISSION_REQUIREMENT}} per object.

---

## Object Index

| Object | API Name | Available Since | Description |
| :--- | :--- | :--- | :--- |
| [{{OBJECT_LABEL}}](#{{object_anchor}}) | `{{ObjectApiName}}` | API v{{NN.0}} | {{one-line description}} |
| … | … | … | … |

---

## {{ObjectApiName}}

{{One-paragraph object description.}} Available in API version {{NN.0}} and later.

**Supported Calls**: {{create(), query(), update(), …}}

**Access**: {{LICENSE}} enabled + `{{PERMISSION}}`.

### Fields

| Field | API Name | Type | Properties | Description |
| :--- | :--- | :--- | :--- | :--- |
| {{Field Label}} | `{{FieldApiName}}` | {{Type / Reference → Target}} | {{Create, Filter, Nillable, …}} | {{description}} |
| … | … | … | … | … |
```

## Conventions
1. **Stamp provenance in the header** — source guide + API version + license/permission. A schema
   reference with no version is a trap.
2. **Index first, anchors everywhere** — the Object Index doubles as a table of contents so an
   `@`-reference jumps the agent to the right section.
3. **Uniform field tables** — keep the five columns identical across every object so the file is
   skimmable and diff-friendly.
4. **Mark reference targets** — write `Reference → {{TargetObject}}` so relationships are explicit.
5. **One domain per file** — don't merge unrelated platform clouds into one mega-file.

## Pairs with
- `goals-doc-map.mdc` (R3) — `@`-reference this file for "what does object X look like?" questions.
- `doc-sync.mdc` (R2) — refresh on each API-version bump.

### When NOT to use
Custom-object-light projects, or where the org schema is small enough to introspect live each time.
Don't hand-maintain a reference that drifts from the org — derive it from the Developer Guide and
re-stamp the version.
