# Business Configuration UI (BCUI)

## What it is

**Manage Business Configuration** — universally called **BCUI** — is the Admin Center tool that lets you edit the **Succession Data Model** through a user interface instead of XML.

It is the single most-used configuration tool in an EC build, and it removed the biggest historical bottleneck: needing an implementation partner with Provisioning access for every label change.

Analogy: XML editing is writing HTML by hand; BCUI is the page editor. Same result, far fewer broken pages.

---

## What you can do in BCUI

| Area | Capability |
|---|---|
| **HRIS elements** | Browse every element (`personalInfo`, `jobInfo`, `compInfo`…) and its fields |
| **Fields** | Change label, visibility, required, max length, default value, picklist |
| **Custom fields** | Claim an unused placeholder and turn it into a real field |
| **Rules** | Attach business rules to an element or a field, choosing the trigger |
| **Field criteria** | Configure filtering of one field's values by another |
| **Country-specific** | Access the country sections of the data model, where exposed |
| **Standard elements** | View and configure the pre-EC user-record fields |
| **Propagation** | Configure a field to take its value from an associated object |
| **Export** | Download the resulting XML for documentation |

## What you cannot do in BCUI

- Upload a wholesale replacement data model (Provisioning)
- Enable features and modules (Provisioning)
- Edit **MDF object** definitions — that is **Configure Object Definitions**
- Edit the **Corporate Data Model** for legacy foundation objects (Provisioning XML)
- Create picklist values — that is **Picklist Center**
- Decide who sees a field — that is **RBP**

Knowing where each of those lives is the practical skill. See the decision table in [The data model landscape](01_Data_Model_Landscape.md).

---

## The screen, field by field

Opening an HRIS element in BCUI shows a table of its fields. Per field you will typically set:

| Setting | Meaning | Guidance |
|---|---|---|
| **Identifier** | The technical id | Never change it |
| **Label** | Display name, per locale | Change this, not the id, when the customer wants different wording |
| **Enabled / visibility** | `both`, `view`, `edit`, `none` | Global visibility; not per user |
| **Mandatory / required** | Must be filled | Check every inbound interface first |
| **Maximum length** | Character limit | Must fit what source systems send |
| **Picklist** | Which list supplies values | Changing it on a populated field orphans values |
| **Default value** | Static default | Prefer a rule if the default is conditional |
| **Details / rules** | Attached rules and their triggers | The main reason to open BCUI after the initial build |

---

## Attaching a rule in BCUI

This is where most of your rules end up, so learn the flow:

1. Build the rule first in **Configure Business Rules** (it must exist before you can attach it).
2. Admin Center → **Manage Business Configuration**.
3. Open the **HRIS element** (e.g. `jobInfo`).
4. Choose whether the rule attaches to the **element** (fires for the whole block) or a **field** (fires on that field's change).
5. Pick the **trigger**: `onInit`, `onChange`, `onSave`, `onPostSave`.
6. Select the rule and save.
7. Test on a real record — and test the *negative* case too (does it correctly not fire?).

Multiple rules can be attached to the same element and trigger; they execute in the configured sequence, and **sequence matters** when two rules touch the same field. Full detail: [Attaching rules](../06_Business_Rules/04_Attaching_Rules.md).

---

## Step by step — the canonical BCUI change

Adding "Preferred Pronouns" to Personal Information:

1. **Requirement written down**: field name, purpose, values, who maintains, who sees, whether payroll needs it. *(Answer for this one: HR and the employee maintain it; everyone can view; payroll does not need it.)*
2. **Export the current XML** from BCUI and save it as `SDM_before_YYYYMMDD.xml`.
3. **Picklist first** — Picklist Center → create `preferredPronouns` with external codes (`SHE_HER`, `HE_HIM`, `THEY_THEM`, `PREFER_NOT`) and labels.
4. **BCUI** → HRIS Elements → `personalInfo`.
5. Find the next unused `custom-string` placeholder. Set:
   - Label: *Preferred Pronouns* (plus translations)
   - Visibility: `both`
   - Required: `false`
   - Picklist: `preferredPronouns`
6. **Save.**
7. **RBP** → grant view to all employees, edit to the employee (ESS) and HR roles.
8. **Configure People Profile** → confirm the Personal Information block shows it where the customer wants it.
9. **Test**: open your own profile, set a value, save, check History shows a new effective-dated row.
10. **Check the import template** — Personal Information now has a new column.
11. **Export the XML again**, diff, and record in the workbook: `personalInfo.custom-string4 = Preferred Pronouns, picklist preferredPronouns, added 02-Sep-2025, owner HR Ops`.
12. Promote to TEST via Instance Sync or XML, retest, then PROD.

Every BCUI change follows that shape. The tool takes two minutes; steps 1, 7, 10 and 11 are the job.

---

## BCUI vs XML — when to use which

| Situation | Use |
|---|---|
| Add or change a handful of fields | **BCUI** |
| Change labels for one element | **BCUI** |
| Attach or re-sequence rules | **BCUI** |
| Relabel 60 fields across five elements | XML (genuinely faster) |
| Something BCUI does not expose | XML via Provisioning |
| Document the current state | XML export |
| Compare DEV against TEST | XML export + diff |
| First-time model load on a new tenant | XML upload |

**BCUI validates; hand-edited XML does not.** That alone is a strong reason to default to BCUI.

---

## Gotchas

- **BCUI does not permission anything.** A new field is invisible until RBP grants it. Roughly half of "my BCUI change didn't work" reports are this.
- **Changing visibility to `none` hides it from everyone**, including admins and, in some paths, from imports — it is not a way to hide it from *some* users.
- **Required is enforced everywhere**, including OData writes and the hire interface.
- **BCUI shows the model, not the screen.** Where the block appears is Configure People Profile; for MDF objects it is Configuration UI.
- **Country sections** may be partially exposed depending on release — if you cannot find a country field, it may still need the CSF XML.
- **No undo.** Your undo is the export you took in step 2.

---

## Real world example

A customer raises: *"the Job Information screen says 'Cost Center' but Finance calls it 'Cost Object', and half our managers put the wrong value in the FTE field because it's not clear it's a decimal."*

Two BCUI changes and one rule, in under an hour:

1. **Label change** on `jobInfo.cost-center` → "Cost Object" (and the French and German labels too).
2. **Label change** on `fte` → "FTE (e.g. 1.0 = full time)".
3. A **validation rule** attached onSave to `jobInfo`: if FTE > 1 or FTE ≤ 0, raise an error message.

Then: test, export the XML, diff, document, promote. The customer's actual complaint — data quality on FTE — is fixed by the third item; the first two stop the tickets.

---

## Interview-grade Q&A

- **What is BCUI? [HIGH]** Manage Business Configuration — the Admin Center tool for editing the Succession Data Model (elements, fields, labels, visibility, required, picklists, rule attachments) without editing XML.
- **What can you *not* do in BCUI? [HIGH]** Enable features or upload a full data model (Provisioning), edit MDF object definitions (Configure Object Definitions), create picklist values (Picklist Center), or set who sees a field (RBP).
- **How do you attach a business rule to Job Information? [HIGH]** Create the rule in Configure Business Rules, then in BCUI open `jobInfo`, choose element or field level, pick the trigger (onInit/onChange/onSave/onPostSave) and select the rule.
- **You added a field in BCUI and cannot see it. Why? [HIGH]** It is not permissioned in RBP; or its visibility is `none`; or the block is not on People Profile.
- **Do you still need Provisioning for data models? [HIGH]** For most day-to-day changes, no — BCUI covers them. Provisioning is still needed for feature enablement, wholesale model uploads and anything BCUI does not expose.
- **BCUI or XML? [MED]** BCUI for changes because it validates; XML for bulk edits, unexposed settings, documentation and environment comparison.
- **What is the risk of setting a field to required in BCUI? [HIGH]** Every write path must supply it, so imports, OData integrations and Recruiting/Onboarding hires can start failing.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business configuration walkthrough](https://www.youtube.com/watch?v=yEsquQA-MxU)
