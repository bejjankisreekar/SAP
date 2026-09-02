# Biographical and Personal Information

Two person-level blocks that beginners merge into one. They are different elements, with different behaviour, and interviewers use them to test whether you actually worked in the system.

---

## Biographical Information (`personInfo`)

**Person-level. Not effective-dated.** Facts that identify the human being.

| Typical field | Notes |
|---|---|
| `person-id-external` | The person key |
| `first-name`, `last-name`, `middle-name` | Also formal name, preferred name, initials depending on config |
| `date-of-birth` | Drives age-based eligibility rules |
| `place-of-birth`, `country-of-birth` | Required in several countries for payroll/legal |
| `gender` | Picklist |
| `date-of-death` | Yes, it exists, and yes, processes depend on it |
| `perPersonUuid` | System-generated unique identifier used by APIs |

Because it is **not effective-dated**, changing a value here replaces it. There is no "as of" version of someone's date of birth.

---

## Personal Information (`personalInfo`)

**Person-level. Effective-dated.** Facts about the person that legitimately change over time.

| Typical field | Notes |
|---|---|
| `first-name`, `last-name` (again) | Yes — names appear in both elements; see below |
| `preferred-name` / display name | What appears in the UI and org chart |
| `marital-status` | Picklist; often drives benefits eligibility |
| `marriage-date` | |
| `nationality` / `citizenship` | Sometimes here, sometimes in Global Information |
| `native-preferred-lang` | |
| `salutation`, `suffix`, `second-last-name` | Very country-dependent |
| `challenge-status` / disability indicator | Sensitive; usually tightly permissioned |

**Why names live in both:** Biographical Information holds the legal identity captured once; Personal Information holds the *effective-dated* name, so a name change on marriage on 12 June is a new time slice from 12 June. Which one the UI shows depends on configuration, and most customers configure name fields on Personal Information and keep Biographical minimal. Whatever you choose, **choose one as the source of truth for downstream systems** — the classic defect is payroll showing the old surname because it was mapped to the wrong element.

---

## Global Information (country-specific person data)

**Person-level, effective-dated, and country-specific.** Holds fields that only exist for a particular country: for example a national insurance category, a religious denomination for church tax, a residence status, an ethnicity field required by local reporting.

- Defined in the **Country/Region-Specific Succession Data Model** — see [Country-specific data models](../04_Data_Models/04_Country_Specific_Data_Models.md).
- Which country block appears is driven by the employee's country (usually derived from the legal entity or work location).
- One person can hold Global Information for **more than one country** — useful for global assignments and cross-border workers.

---

## Name formats and the global problem

Names are the most under-estimated design topic in a global rollout:

- Some countries use **two surnames** (Spain, Mexico → `second-last-name`).
- Some need **local-script names** alongside Latin script (Japan, China, Russia → alternative name fields).
- Some have **no concept of a middle name**; others require initials.
- Display order differs (family name first in Japan, Hungary, and elsewhere).

EC handles this with country-specific name format configuration plus the alternative-name fields. The consulting decision is *which name appears where* — in the org chart, in emails, in the payroll interface, on generated documents.

---

## Data privacy — this block, more than any other

Biographical and Personal Information contain the most sensitive data in the system: date of birth, marital status, disability indicators, ethnicity in some countries.

- Permission these fields **narrowly** in RBP; "HR can see everything" is not a defensible design.
- Field-level permissions matter: a manager may need the preferred name and not the date of birth.
- Local works councils and privacy regulations may prohibit storing certain fields at all in certain countries — that is a design constraint you must ask about, not assume.
- Purge policies (Data Retention Management) apply after termination.

---

## Configuring these elements

Everything is driven by the **Succession Data Model** (employee-level HRIS elements), edited today in **Manage Business Configuration**:

- Add or hide a field → set `visibility` to `both` / `view` / `none`
- Make it mandatory → `required="true"`
- Change the label → the `<label>` entry (per locale)
- Attach a picklist → the picklist ID on the field
- Add validation or defaulting → attach a business rule (onSave/onChange)

Country-specific variations go into the **CSF Succession Data Model**, not the base one.

Step by step to add a custom field to Personal Information:

1. Admin Center → **Manage Business Configuration**
2. HRIS Elements → **personalInfo**
3. Find an available `custom-string` / `custom-date` field, set its ID, label, visibility and required flag
4. Save
5. Grant the field in the relevant **permission roles** (it is invisible until you do)
6. Test on a real employee record, then re-export the XML for the workbook

---

## Real world example

A retailer operating in the UK, Spain and Japan:

- **Base Personal Information** keeps first name, last name, preferred name, marital status.
- **Spain** adds `second-last-name` through the country-specific model; without it, Spanish employees' names are wrong on every document.
- **Japan** adds Kana name fields for local-script display and payroll.
- **Global Information** for Spain carries the social security affiliation data; for Japan, the local residence data.
- A **business rule** defaults preferred name from first name on hire, so the org chart is never blank.
- **RBP** exposes date of birth only to HR and payroll roles; managers see name and preferred name only.

That is a realistic slice of a global design, and it uses every concept in this note.

---

## Interview-grade Q&A

- **Biographical vs Personal Information? [HIGH]** Both are person-level. Biographical is not effective-dated (date of birth, place of birth, gender); Personal is effective-dated (name, marital status) so changes create history.
- **Where does a name change on marriage go? [HIGH]** A new effective-dated record in Personal Information starting on the change date.
- **What is Global Information? [HIGH]** Country-specific person data, effective-dated, defined in the country-specific Succession Data Model; a person can hold it for multiple countries.
- **How do you add a custom field to Personal Information? [HIGH]** In Manage Business Configuration, configure an available custom field on `personalInfo` (ID, label, visibility, required, picklist), then permission it in RBP and test.
- **How would you handle Spanish second surnames or Japanese Kana names? [MED]** Through country-specific fields in the CSF Succession Data Model plus the alternative-name configuration.
- **Which fields would you restrict most tightly? [MED]** Date of birth, marital status, disability/ethnicity indicators, national ID — permissioned to HR and payroll roles only, with field-level RBP.
- **Is Biographical Information effective-dated? [HIGH]** No.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — configuring employee data
- Video — [Employee data configuration](https://www.youtube.com/watch?v=qkmFdj4h4rA)
