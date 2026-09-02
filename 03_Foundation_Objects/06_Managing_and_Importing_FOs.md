# Managing and importing Foundation Objects

How foundation data actually gets created, changed, loaded and kept clean.

---

## Which tool for which object

| Object type | Create/edit one record | Bulk load |
|---|---|---|
| **MDF Foundation Objects** (Legal Entity, Business Unit, Division, Department, Cost Centre, Location, Location Group, Geozone, Job Function, Job Classification, Pay Grade, Pay Range, Pay Group, Position) | **Manage Data** | **Import and Export Data** |
| **Legacy Foundation Objects** (Pay Component, Pay Component Group, Frequency, Event Reason, Dynamic Role) | **Manage Organization, Pay and Job Structures** | **Import Foundation Data** |
| Object *definitions* (adding a field to an FO) | **Configure Object Definitions** (MDF) or the Corporate Data Model XML (legacy) | Import and Export Data (definitions) |

The quickest way to tell which world an object lives in: search for it in **Manage Data**. If it is there, it is MDF.

---

## Creating a record in Manage Data

1. Admin Center → **Manage Data**.
2. Top-right dropdown → **Create New** → choose the object (e.g. *Department*).
3. Fill in:
   - **External Code** — the stable key, following your documented convention
   - **Effective Start Date** — when this version becomes valid
   - **Name** (and translations if the customer is multilingual)
   - **Status** — Active
   - **Associations** — its parent objects
   - Any propagating attributes (e.g. Cost Centre on a Department)
4. Save.
5. To change it later, open the record and choose the effective date — you are creating a **new time slice**, exactly like employee data.

**Searching:** the search box in Manage Data matches on external code and name. For objects with thousands of records, use the *Advanced* search or export to CSV instead of scrolling.

---

## Effective dating of Foundation Objects

FOs are effective-dated, and this is under-appreciated:

- **Renaming** Department "Treasury" → "Group Treasury" from 1 January creates a new slice. Reports as of December still say "Treasury".
- **Restructuring** — moving a department to a different division — is an effective-dated change on the association.
- **Retiring** — set Status = Inactive from a date. **Never delete.** Historical employee records point at it.
- **Future-dating** works: create next year's pay ranges in December, effective 1 January.

A useful consequence: **most re-organisations are foundation-object work plus a mass employee update**, not one or the other. Renaming and re-parenting units happens on the FOs; moving people between them happens on Job Information.

---

## Bulk loading with Import and Export Data (MDF)

This is how real foundation data gets loaded — nobody hand-creates 240 locations.

**Step by step**

1. Admin Center → **Import and Export Data**.
2. Select action **Download Template**, choose the object, and include dependencies if offered.
3. Open the CSV. Note the header rows — MDF templates carry a technical header row plus a label row; **do not delete or reorder them**.
4. Fill in the data:
   - `externalCode` — the key
   - `effectiveStartDate` — required for effective-dated objects, in the instance's date format
   - `status` — `A` (active) / `I` (inactive)
   - Association columns — the **external code** of the parent record, not its name
   - Translatable fields — usually one column per locale (`name.en_GB`, `name.de_DE`)
5. Save as **UTF-8 CSV**.
6. Back in Import and Export Data, action **Import Data**, choose the object, upload the file.
7. Choose **purge type**: *Incremental* (add/update the rows in the file — the normal choice) or *Full Purge* (replace the whole object's data — dangerous, initial loads only).
8. Run. Read the **result/error file** — it lists failed rows with reasons.
9. Spot-check in Manage Data.

**Load order matters** — parents before children, always:

```
1. Legal Entity
2. Business Unit
3. Division
4. Department
5. Cost Centre  (or replicate from Finance)
6. Location Group → Geozone → Location
7. Job Function → Job Family → Job Classification
8. Pay Grade → Pay Range → Pay Group
9. Legacy: Frequency → Pay Component → Pay Component Group
10. Event Reasons
11. Positions (after everything they reference)
```

A child row whose parent does not yet exist fails the association validation — that is by far the most common bulk-load error.

---

## Exporting — and why you should do it constantly

**Export and Export Data → Export Data** produces a CSV of an object's records (and, separately, of object *definitions*).

Use it for:

- **Backup before a change** — export, then edit. The export is your undo.
- **Documentation** — attach exports to the configuration workbook, dated.
- **Migrating between tenants** — export from DEV, import to TEST (or use Instance Sync).
- **Mass edits** — export, edit in Excel, re-import. Far faster and safer than 300 manual edits.
- **Data quality checks** — export and pivot: how many departments have no cost centre? How many locations have no geozone?

That last use is genuinely valuable and rarely done. A ten-minute pivot table before UAT prevents a week of tickets.

---

## Keeping foundation data clean

Build these checks *before* go-live, and run them monthly afterwards:

| Check | Why |
|---|---|
| FOs with missing associations (department with no division, location with no geozone) | Causes empty picklists and failed propagation |
| Active FOs with no employees | Candidates for retirement; also spots typos creating duplicates |
| Inactive FOs still referenced by active employees | Data quality defect; will fail validations on the next change |
| Duplicate names with different external codes | "Treasury" and "Treasury " — created by two people on the same day |
| Pay ranges missing for a live grade × geozone combination | Validation rules silently pass or fail unpredictably |
| Job classifications with no pay grade | Propagation delivers nothing |

Most of these are a Story report or an exported CSV plus a pivot.

---

## Who owns foundation data after go-live?

A governance question that consultants must raise and customers routinely forget:

- **Who may create a department?** Uncontrolled creation produces 40 variants of "Marketing".
- **What is the approval process?** Many customers put an MDF workflow or a ticket process in front of FO creation.
- **Where is the master?** Cost centres usually come from Finance; job codes from Compensation; locations from Facilities. EC is not always the master.
- **How are changes communicated?** A new legal entity affects payroll, integrations, permissions and reporting — it is not a five-minute admin task.

Raising this in an interview ("who owns foundation data after go-live?") signals experience, because it is the thing that quietly degrades over the first two years of live running.

---

## Real world example

A customer needs 240 store locations loaded before UAT.

1. Facilities provides a spreadsheet: store number, name, address, country, opening date, region.
2. Mapping: store number → `externalCode`, region → **Geozone** (mapped via a lookup table agreed with Compensation), country → **Legal Entity**.
3. First import into DEV fails 240/240 — the geozones did not exist yet. Geozones loaded first; second attempt: 240 succeed.
4. Third issue found in testing: 11 stores had opened *after* the effective start date used, so employees hired earlier could not select them. Fixed by setting `effectiveStartDate` to the company's go-live date rather than each store's opening date — a deliberate decision, documented.
5. Export taken after the load and filed with the workbook.

Points 3 and 4 are the realistic part. The mechanics of importing are easy; the effective-date decisions are where the thinking is.

---

## Common mistakes

- Loading **children before parents**.
- Using association **names instead of external codes** in the import file.
- Deleting the **technical header row** from a template.
- **Full purge** on an object that already has data.
- **Deleting** an FO rather than inactivating it.
- Non-UTF-8 encoding mangling accented location names.
- Effective start dates set to today, so historical hires cannot reference the object.
- No **export taken before** a bulk change, so there is no way back.

---

## Interview-grade Q&A

- **Where do you maintain foundation objects? [HIGH]** MDF FOs in **Manage Data**, legacy FOs in **Manage Organization, Pay and Job Structures**; bulk via **Import and Export Data** or **Import Foundation Data** respectively.
- **What order do you load foundation data in? [HIGH]** Parents before children: legal entity → business unit → division → department → cost centre; geozone → location; job function → family → classification; pay grade → pay range → pay group; then event reasons and positions.
- **Are foundation objects effective-dated? [HIGH]** Yes — renames, re-parenting and retirement are all effective-dated changes.
- **How do you retire a department? [HIGH]** Set it inactive from an effective date. Never delete it, because historical employee records reference it.
- **What is the difference between incremental and full purge import? [HIGH]** Incremental adds or updates the rows in the file; full purge replaces the object's entire data set and is for initial loads only.
- **If I change a department's cost centre, do existing employees change? [HIGH]** No — propagation applied when the value was selected; existing records need a mass update.
- **How do you move foundation configuration from DEV to TEST? [MED]** Instance Sync where supported, or export from DEV and import into TEST, keeping the exports with the configuration workbook.
- **Who owns foundation data after go-live? [MED]** A governance decision: usually HR operations for org units, Finance for cost centres, Compensation for jobs and grades — with a controlled creation process, because uncontrolled creation degrades reporting quickly.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring foundation objects
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Foundation objects and imports](https://www.youtube.com/watch?v=yEsquQA-MxU)
