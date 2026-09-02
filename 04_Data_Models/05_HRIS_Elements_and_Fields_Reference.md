# HRIS elements and fields — reference

A lookup table for the standard Employee Central HRIS elements: what each one is, its level, whether it is effective-dated, and its main fields.

> Field ids and availability vary by release and configuration. Use this to orient yourself, then confirm against **Manage Business Configuration** in the instance you are working in.

---

## Quick index

| Element id | Name | Level | Effective-dated |
|---|---|---|---|
| `personInfo` | Biographical Information | Person | No |
| `personalInfo` | Personal Information | Person | **Yes** |
| `globalInfo` | Global Information (country-specific) | Person | **Yes** |
| `homeAddress` | Addresses | Person | **Yes** |
| `emailInfo` | Email Information | Person | No |
| `phoneInfo` | Phone Information | Person | No |
| `imInfo` | Instant Messaging | Person | No |
| `socialAccountInformation` | Social Accounts | Person | No |
| `nationalIdCard` | National ID | Person | No |
| `emergencyContactPrimary` | Emergency Contact | Person | No |
| `personRelationshipInfo` | Dependants / Person Relationships | Person | Varies |
| `employmentInfo` | Employment Information | Employment | Container |
| `jobInfo` | Job Information | Employment | **Yes** |
| `compInfo` | Compensation Information | Employment | **Yes** |
| `payComponentRecurring` | Recurring Pay Components | Employment | **Yes** |
| `payComponentNonRecurring` | Non-Recurring Pay Components | Employment | Payment date |
| `jobRelationsInfo` | Job Relationships | Employment | **Yes** |
| `workPermitInfo` | Work Permit | Employment/Person | Validity dates |
| `globalAssignmentInfo` | Global Assignment Details | Employment | **Yes** |

---

## Person-level elements

### `personInfo` — Biographical Information

Not effective-dated. Identity facts.

| Field | Notes |
|---|---|
| `person-id-external` | The person key |
| `first-name`, `last-name`, `middle-name` | Where the customer keeps names on the biographical element |
| `date-of-birth` | Drives age-based eligibility |
| `place-of-birth`, `country-of-birth` | Required by several payrolls |
| `gender` | Picklist |
| `date-of-death` | Triggers specific processes |
| `perPersonUuid` | System identifier used by APIs |

### `personalInfo` — Personal Information

**Effective-dated.** Facts that change.

| Field | Notes |
|---|---|
| `first-name`, `last-name`, `middle-name` | Most customers keep names here so changes create history |
| `preferred-name`, `display-name` | What appears in the UI and org chart |
| `salutation`, `suffix`, `second-last-name` | Country-dependent |
| `marital-status`, `marriage-date` | Often drives benefits |
| `nationality`, `native-preferred-lang` | |
| `challenge-status` | Disability indicator — highly sensitive |

### `homeAddress` — Addresses

**Effective-dated**, one history per address type.

| Field | Notes |
|---|---|
| `address-type` | Home, Mailing, Business, Emergency… |
| `country` | Determines the layout of everything below |
| `address1`, `address2`, `address3` | Country-specific labels |
| `city`, `state`, `province`, `county` | Availability by country |
| `zip-code`/`postal-code` | Format varies |
| `start-date` | The move date |

### `emailInfo` / `phoneInfo`

Not effective-dated. Typed lists with a primary flag.

| Field | Notes |
|---|---|
| `email-type` / `phone-type` | Business, Personal, Cell, Fax… |
| `email-address` / `phone-number` | |
| `is-primary` | Drives notifications |
| `country-code`, `area-code`, `extension` | Phone only |

### `nationalIdCard`

Not effective-dated. One row per country + card type.

| Field | Notes |
|---|---|
| `country` | Determines valid card types |
| `card-type` | SSN, NINO, PAN, Aadhaar, Steuer-ID… |
| `national-id` | The number |
| `is-primary` | The main ID for this person |

### `emergencyContactPrimary` / `personRelationshipInfo`

| Field | Notes |
|---|---|
| `name`, `relationship` | |
| `phone`, `email`, address fields | Emergency contact |
| `is-primary` | |
| `date-of-birth`, `gender`, `is-dependant`, `is-beneficiary` | Dependants — used by Benefits |

### `globalInfo`

**Effective-dated, country-specific.** Contents differ entirely per country — see [Country-specific data models](04_Country_Specific_Data_Models.md).

---

## Employment-level elements

### `employmentInfo` — Employment Information

The container for one working relationship.

| Field | Notes |
|---|---|
| `start-date` | Hire date |
| `original-start-date` | First ever start with the company |
| `seniority-date`, `service-date` | Entitlement calculations |
| `end-date` | Termination date |
| `last-date-worked`, `payroll-end-date` | Often differ from termination date |
| `is-primary-employment` | Concurrent employment |
| `assignment-id-external` | Employment key |
| `okay-to-rehire`, `regret-termination` | Set at termination |
| `probation-period-end-date`, `notice-period` | Frequently rule-derived |

### `jobInfo` — Job Information

**The biggest element.** Effective-dated.

| Group | Fields |
|---|---|
| Organisation | `company` (legal entity), `business-unit`, `division`, `department`, `cost-center`, `location`, `geozone` |
| Job | `job-code` (job classification), `job-title`, `position`, `job-function`, `employee-class`, `employment-type`, `regular-temp` |
| Management | `manager-id` (supervisor), `hr-manager`, `second-manager` |
| Time | `fte`, `standard-hours`, `working-days-per-week`, `time-type`, `work-schedule` |
| Status | `employee-status`, `pay-group` |
| Transaction | `event`, `event-reason`, `start-date`, `position-entry-date` |
| Country-specific | Added by CSF-SDM |

### `compInfo` — Compensation Information

Effective-dated header.

| Field | Notes |
|---|---|
| `pay-group` | Sometimes here, sometimes on jobInfo |
| `pay-grade`, `pay-range` | Usually propagated from the job |
| `currency`, `frequency` | |
| `annual-salary`, `target-pay`, `compa-ratio`, `range-penetration` | Calculated |
| `event`, `event-reason` | Compensation rows carry events too |

### `payComponentRecurring` / `payComponentNonRecurring`

| Field | Notes |
|---|---|
| `pay-component` | Which component (foundation object) |
| `pay-value`, `currency-code`, `frequency` | The amount |
| `start-date` / `pay-date` | Recurring uses effective dates; non-recurring a payment date |
| `sequence-number` | Where multiple rows of the same component are allowed |

### `jobRelationsInfo`

| Field | Notes |
|---|---|
| `relationship-type` | HR Manager, Matrix Manager, Custom Manager |
| `rel-user-id` | The related person |
| `start-date` | Effective-dated |

### `workPermitInfo`

| Field | Notes |
|---|---|
| `document-type`, `document-number` | Visa, work permit |
| `issuing-authority`, `country` | |
| `valid-from`, `valid-to` | Drives expiry alerts |

### `globalAssignmentInfo`

| Field | Notes |
|---|---|
| `assignment-type` | Long-term, short-term, commuter |
| `planned-start-date`, `planned-end-date` | |
| `host` / `home` company | |

---

## Custom field placeholders

Every element carries a set of unused placeholders you claim as custom fields:

| Placeholder family | Use for |
|---|---|
| `custom-string1` … `custom-stringN` | Text or picklist-backed fields |
| `custom-date1` … | Dates |
| `custom-double1` … | Numbers |
| `custom-long1` … | Integers |

**Two non-negotiable habits:**

1. **Record every placeholder you use** in the configuration workbook — element, placeholder, business meaning, owner, date added.
2. **Never reuse a placeholder** that has ever held data. The old values are silently re-labelled as the new field.

---

## How to check any element in an instance

1. Admin Center → **Manage Business Configuration**.
2. HRIS Elements → pick the element.
3. You now see every field, its id, label, visibility, required flag and picklist — the authoritative answer for *that* instance and release.
4. For MDF objects (Department, Position, Location…) use **Configure Object Definitions** instead.
5. For what the **API** exposes, use the **OData API Data Dictionary** in Admin Center — it maps HRIS elements to OData entities such as `PerPersonal`, `EmpJob`, `EmpCompensation`.

That last tool is the fastest way to answer "which entity holds this field?" during an integration discussion.

---

## Interview-grade Q&A

- **List the person-level HRIS elements. [HIGH]** `personInfo`, `personalInfo`, `globalInfo`, `homeAddress`, `emailInfo`, `phoneInfo`, `nationalIdCard`, `emergencyContactPrimary`, `personRelationshipInfo`, `imInfo`, `socialAccountInformation`.
- **List the employment-level HRIS elements. [HIGH]** `employmentInfo`, `jobInfo`, `compInfo`, `payComponentRecurring`, `payComponentNonRecurring`, `jobRelationsInfo`, `workPermitInfo`, `globalAssignmentInfo`.
- **Which are effective-dated? [HIGH]** `personalInfo`, `globalInfo`, `homeAddress`, `jobInfo`, `compInfo`, `payComponentRecurring`, `jobRelationsInfo`. Not: `personInfo`, `emailInfo`, `phoneInfo`, `nationalIdCard`, `emergencyContactPrimary`.
- **Where does FTE live? And date of birth? [HIGH]** FTE → `jobInfo` (employment, effective-dated). Date of birth → `personInfo` (person, not effective-dated).
- **How do you add a custom field, and what must you document? [HIGH]** Claim an unused placeholder of the right type, set label/visibility/required/picklist, permission it, test — and record element, placeholder, meaning, owner and date in the workbook. Never reuse a placeholder that held data.
- **How would you find which OData entity holds a field? [MED]** The OData API Data Dictionary in Admin Center — for example `EmpJob` for Job Information, `PerPersonal` for Personal Information.
- **Which element holds event and event reason? [HIGH]** `jobInfo` — and also `compInfo` for compensation changes.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Data models and HRIS elements](https://www.youtube.com/watch?v=yEsquQA-MxU)
