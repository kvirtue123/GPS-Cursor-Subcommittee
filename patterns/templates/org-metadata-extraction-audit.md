# Org Metadata Extraction & Audit Workflow — Template

A drop-in checklist + report skeleton for reverse-engineering an existing Salesforce feature into
a DX project and recording ground truth. Fill the `{{PLACEHOLDER}}` tokens.

> Source pattern: `PostAwardExtract/docs/awardDetails.md` (Funding Award extraction).

---

## Step 1 — Scope & extract
- [ ] Feature anchor object: `{{ANCHOR_OBJECT}}`
- [ ] Retrieve anchor + siblings/related objects (incl. `{{NAMESPACE}}_` custom fields/objects)
- [ ] Retrieve automation: Flows, Approval Processes, record-triggered logic
- [ ] Retrieve code & UI: Apex classes, LWC/Aura, Layouts, FlexiPages
- [ ] Retrieve access: Permission Sets / Permission Set Groups referencing the object
- [ ] Build manifest: `manifest/package.xml` (or `sf project retrieve start -m ...`)

## Step 2 — Trace runtime paths
- [ ] For each key action, follow it to its implementation (flow / Apex / managed framework).
- [ ] Identify managed-package frameworks by their static resources / components
      (e.g. `{{omnistudio__...}}`) rather than assuming custom build.

## Step 3 — Map the status lifecycle
```markdown
## Status Lifecycle
### N. {{Status}}
- **Action**: {{action / flow API name / approval process unique name}}
```

## Step 4 — Write the audit (both halves)
```markdown
## Extraction Summary

**What was found:**
- **Objects**: {{…}}
- **Flows**: {{… by API name …}}
- **Approval Process**: {{Object.ProcessUniqueName}}
- **Apex & UI**: {{…}}
- **{{Framework}} Pathway**: {{evidence → conclusion}}
- **Permissions & Layouts**: {{…}}

**What was missing / unexpected:**   ← the high-value, reusable half
- {{Thing you looked for and did NOT find}} → {{what that implies}}
- {{Metadata type not available/found}} → {{confirms X rather than Y}}
```

## Conventions
1. **Record negative findings explicitly.** "We looked for custom EmailTemplates and found none"
   prevents the next session from re-investigating.
2. **Cite API names, not just labels** — flows/approval processes are unambiguous by unique name.
3. **Name the framework from evidence** — point at the static resource / component that proves it.
4. **Pair with `AGENTS.md` (R4)** so this audit is the project's ground-truth front door.

### When NOT to use
Greenfield builds with nothing to reverse-engineer. Don't run a full extraction audit for a
single known field change.
