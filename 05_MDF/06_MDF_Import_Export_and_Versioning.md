# MDF import, export and versioning

How MDF configuration and data move — in bulk, and between tenants.

---

## The tool: Import and Export Data

**Admin Center → Import and Export Data** handles both halves of MDF:

| What | Action |
|---|---|
| **Object definitions** (the blueprint) | Export/import definitions — used to move an object's design between tenants |
| **Object data** (the records) | Export/import records — used to load and update data in bulk |

The actions you will use:

| Action | Purpose |
|---|---|
| **Download Template** | A blank CSV with the correct columns for an object |
| **Export Data** | Current records as CSV — your backup and audit tool |
| **Import Data** | Load or update records |
| **Export/Import Object Definition** | Move the design |
| **Monitor Job** | Check status and download the result/error file |

---

## Importing records — step by step

1. **Download Template** for the object. Do this **from the target instance**, because templates reflect that instance's fields.
2. Open the CSV. Note the header rows — a technical id row and a label row. **Do not delete or reorder them.**
3. Fill in the data:

   | Column | Notes |
   |---|---|
   | `externalCode` | The key. Existing code → update; new code → insert |
   | `effectiveStartDate` | Required for effective-dated objects; use the instance's date format |
   | `status` | `A` / `I` |
   | Association columns | The **external code** of the target record, not its name |
   | Picklist columns | The picklist value's **external code**, not its label |
   | Translatable fields | One column per locale, e.g. `externalName.en_GB` |

4. Save as **UTF-8 CSV**.
5. **Import Data** → select the object → upload.
6. Choose the **purge type**:
   - **Incremental** — insert or update the rows in the file. The normal choice.
   - **Full Purge** — replace the object's entire data. Initial loads only, and never in production without a written decision.
7. Run, then **Monitor Job** and download the **result file**.
8. Read the errors. A job that reports "completed" with 300 failed rows is not a successful load.
9. Spot-check records in **Manage Data**.

### Load order

Same rule as foundation objects: **referenced records first**. If `cust_CompanyCar` references `cust_CarProvider`, load providers first. Composite children are loaded after (and often *with*) their parents.

---

## Effective-dated imports

For objects with effective dating, `effectiveStartDate` behaves exactly like an employee record's date:

| Scenario | What to send |
|---|---|
| New record | External code + effective start date |
| **Genuine change from a date** | Same external code + a **new** effective start date → creates a new time slice |
| **Correcting an existing slice** | Same external code + the **same** effective start date → overwrites that slice |

That is a powerful and dangerous property: re-sending a file with the same dates **overwrites history silently**. Before any repeat load, ask which of the two you intend.

---

## Exporting — and why to do it constantly

**Export Data** produces a CSV of an object's records. Use it for:

- **Backup before every bulk change** — the export is your undo.
- **Mass edits** — export, edit in Excel, re-import. Faster and more accurate than 200 manual edits.
- **Data quality auditing** — export and pivot: how many positions have no parent? How many records are inactive but still referenced?
- **Documentation** — dated exports filed with the configuration workbook.
- **Environment comparison** — export the same object from DEV and TEST and diff.

Ten minutes of export-and-pivot before UAT prevents a week of tickets. This is the single most under-used habit in EC projects.

---

## Moving MDF between tenants

Three routes, in order of preference:

### 1. Instance Synchronization

Copies supported configuration artefacts — including MDF object definitions and, depending on release, data — from source to target, with a preview of what will change. Check the supported-artefact list for your release.

### 2. Export/import of definitions and data

1. Source tenant → **Export Object Definition** for the object (and any objects it references).
2. Target tenant → **Import Object Definition**.
3. Then export and import the **data**, referenced objects first.
4. Re-create anything not carried across: Configuration UIs, business rules and their attachments, RBP permissions, picklists.

**What typically does *not* travel automatically** with the object definition — check each one:

- **Configuration UI** (may need separate migration or rebuilding)
- **Business rules** and their attachment to the object
- **RBP permission roles** granting the object
- **Picklists** used by its fields
- **Alerts/notifications** configured on it

That checklist is worth memorising; forgetting the rules and the RBP is why "we moved the object to TEST and it doesn't work".

### 3. Manual rebuild

Against the configuration workbook. Slow, but always available.

---

## Versioning and change control

MDF has **no version history of the definition itself** — you cannot roll back an object definition the way you roll back code. Your version control is what you build:

1. **Export the object definition** after every meaningful change, named `objectcode_definition_YYYYMMDD.csv`.
2. **Export the data** on the same schedule for objects that matter.
3. Keep both in a folder alongside the **configuration workbook**.
4. Record in the workbook: what changed, why, who approved it, and the date.
5. Diff exports to answer "what changed and when".

Effective-dated *records* do have history, and that is queryable in Manage Data. The *definition* does not. Knowing that distinction — and having a habit that compensates — is a senior-level answer.

---

## Common mistakes

- **Full purge** on an object that already holds data.
- Using association **names instead of external codes** in the file.
- Using picklist **labels instead of external codes**.
- Deleting the **technical header row** from the template.
- Using a template exported from a **different instance**.
- Re-running a load with the **same effective dates**, silently overwriting slices.
- Migrating the **definition without the rules, UI, picklists and permissions**.
- **No export taken before** a bulk change, leaving no way back.
- Non-UTF-8 encoding corrupting names.

---

## Real world example

Loading 1,800 positions before UAT.

1. Export the **object definition** for Position from DEV to confirm which custom fields exist.
2. Download the **Position data template** from the target tenant.
3. Map the customer's spreadsheet: position code → `externalCode`, job code → the job classification association (external code), parent position code → the self-referencing association.
4. **First attempt:** 1,800 failures — the parent positions did not yet exist. Split into two passes: pass 1 creates all positions without parents; pass 2 updates the parent association. This two-pass approach for self-referencing hierarchies is a standard technique.
5. **Second issue:** 40 positions referenced job codes that had not been loaded. Fixed at source.
6. Post-load audit by export + pivot: positions with no parent (expected: 12 top-level roles — found 19; three were data errors, four were genuine), positions with no job classification (0), inactive positions referenced as parents (2, corrected).
7. Export taken and filed.

The two-pass technique and the post-load audit are the parts worth remembering.

---

## Interview-grade Q&A

- **How do you bulk-load MDF data? [HIGH]** Import and Export Data: download the template from the target instance, fill it using external codes for associations and picklists, save as UTF-8 CSV, import incrementally, then read the result file and spot-check.
- **Incremental vs full purge? [HIGH]** Incremental inserts or updates only the rows in the file; full purge replaces the object's entire data set and is for initial loads only.
- **How do imports handle effective dating? [HIGH]** A new effective start date on an existing external code creates a new time slice; the same date overwrites the existing slice — so repeat loads can silently rewrite history.
- **How do you move an MDF object to another tenant? [HIGH]** Instance Sync where supported; otherwise export/import the object definition and data, then separately migrate or rebuild the Configuration UI, business rules and attachments, picklists, alerts and RBP permissions.
- **What does *not* come across with an object definition? [HIGH]** Typically the Configuration UI, business rules and their attachments, picklists, alerts and permission roles — check and migrate each.
- **Does MDF version object definitions? [MED]** No. Records can be effective-dated, but the definition has no history — so you export definitions after every change and keep them dated with the configuration workbook.
- **How would you audit MDF data quality? [MED]** Export to CSV and pivot: missing associations, orphaned parents, inactive records still referenced, duplicates.
- **How do you load a self-referencing hierarchy such as positions? [MED]** Two passes — create all records first, then update the parent references — because a parent must exist before it can be referenced.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF import and export](https://www.youtube.com/watch?v=yEsquQA-MxU)
