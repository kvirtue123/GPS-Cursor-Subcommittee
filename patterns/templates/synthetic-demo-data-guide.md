# Synthetic Demo-Data Design / Generate / Load / Teardown — Template

A fill-in-the-blanks workflow for building **synthetic demo data** for a Salesforce (or Data Cloud) demo: discover the org schema first, design hero records around the demo beats, generate with a seeded Faker pipeline, bulk-load parent→child, then tear down repeatably.

> Source pattern: FWA SE demo-data guide (`FWA-Project/docs/templates/demo-data-guide.md`).
> The full as-shipped guide (with runnable Python and Data Cloud Ingestion-API steps) is preserved at
> `patterns/canonical/fwa-demo-data-guide.md`. This file is the condensed, token-conformed template.

---

## Section 1 — Prerequisites
- Tools: `sf` CLI, Python 3.10+, `pandas`, `faker`, `simple-salesforce` (+ `openpyxl`).
- Org access: alias authenticated; "Modify All Data"; API enabled; (Data Cloud) Ingestion API connected-app creds.

## Section 2 — Discover the org schema FIRST
> Never design data before you know the real field names. Use the Tooling API (CRM) or describe/list helpers (Data Cloud).

```bash
# Custom objects
sf data query -o {{ORG_ALIAS}} --use-tooling-api \
  -q "SELECT QualifiedApiName, Label FROM EntityDefinition WHERE QualifiedApiName LIKE '{{PREFIX}}%'"
# Fields on one object (required-only is the fast path)
sf data query -o {{ORG_ALIAS}} --use-tooling-api \
  -q "SELECT QualifiedApiName, DataType, IsRequired FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName='{{OBJECT}}'"
```
> For Data Cloud `__dlm`/`__dll`/`__cio`, use the **data-cloud-sql-runner** skill (`dc-describe.sh`) instead — do not guess field names.

Fill in an **object inventory** (API name · type · required fields · external-ID field).

## Section 3 — Design the demo slice
- **Scenario summary** — one paragraph: what's shown, which features, which audience.
- **Demo beats** — the exact clicks; each drives which records must exist.
- **Hero records** — the 2–4 records clicked live; every on-screen field must look plausible; give each a stable memorable ID.
- **Supporting records** — counts + relationship rules + **generation order (parent→child)**.
- **Field rules** — per field: approach (`Faker().company()` etc.) · range/pattern · example.
- **Data constraints** — required-no-default, picklist exact values, lookups that must resolve, unique fields, sandbox row caps.

## Section 4 — Generate (seeded for reproducibility)
```python
SEED = {{SEED}}            # record this; reproducibility depends on it
fake = Faker(); Faker.seed(SEED)
# one gen_<object>() per object; External_Id__c keeps upserts idempotent on re-run
# then OVERRIDE hero rows so their on-screen values match the demo beats
```
Load options: `simple-salesforce` bulk upsert (CRM) · Data Cloud **Ingestion API** + trigger Identity Resolution (DLO/DMO) · Data Loader (no-code).

## Section 5 — Load checklist
Pre-load (auth, inventory, hero overrides, seed recorded) → Load (parent first, zero FK errors) → Verify (open each hero record; run the beats end-to-end; dry-run recording).

## Section 6 — Reset & teardown
Delete children before parents (by External-ID prefix); (Data Cloud) delete DLO rows + re-run IR. Verify `COUNT()` is zero. Keep a decisions log so the next person can rebuild.

---

### When NOT to use
Production data migration (use a real ETL/DataLoader plan with validation), or demos that can run on existing org data. This is for **disposable, reproducible** synthetic demo slices.
