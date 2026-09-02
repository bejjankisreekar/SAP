# Addresses, contact information and National ID

Three person-level blocks that look trivial and cause a surprising amount of project work — because every country does them differently.

---

## Addresses (`homeAddress`)

**Person-level, effective-dated.** One person can hold several addresses of different **types**.

| Concept | Detail |
|---|---|
| Address type | Picklist: Home, Mailing, Emergency, Business, Temporary… Each type can hold its own effective-dated history |
| Effective dating | Yes — a house move on 1 May creates a new slice from 1 May |
| Country dependency | **The field layout changes per country** |
| Source of truth | Usually employee self-service, with an optional workflow |

### Country-specific address formats

This is the important part. An address in the UK, the US, Japan and Germany do not have the same fields:

| Country | Typical fields |
|---|---|
| UK | Address line 1/2, city, county, postcode |
| US | Street, apartment, city, **state (picklist)**, ZIP, county |
| Germany | Street + house number, postal code, city |
| Japan | Prefecture, city, district, block/lot, building |
| India | Address lines, city, state, PIN code |

EC ships these formats and you switch them on per country in the **Country/Region-Specific Succession Data Model**. You do *not* build one universal address with 20 optional fields — that is the classic beginner design, and it produces unusable data.

**Field-criteria / dependent picklists** matter here: choosing country US should restrict the state picklist to US states. See [Picklists](../04_Data_Models/08_Picklists.md).

### Practical gotchas

- **Which address does payroll use?** Agree the type (usually Home) and map only that one.
- **Address change workflows** — many customers route address changes to HR for audit, especially where the address drives tax jurisdiction (US state/local tax is the classic case).
- **Effective-dated address + retroactive tax** — a US address change dated in the past can trigger retro tax recalculation.
- **Multiple concurrent addresses of the same type** are not allowed for the same period; a new one supersedes.

---

## Email Information (`emailInfo`)

**Person-level, not effective-dated.** A list of typed emails.

- Types: Business, Personal, Other (picklist).
- One is flagged **primary** — that flag drives system notifications.
- The email used for **login** is a separate concept on the user account; do not confuse them.
- Business email is frequently written by an integration (from Active Directory), not typed by a human. Decide direction of flow and lock the field accordingly.

## Phone Information (`phoneInfo`)

**Person-level, not effective-dated.** Same shape: typed list (Business, Home, Cell, Fax…), one flagged primary, with country code fields.

- Mobile number quality matters if you use SMS or the mobile app.
- Format validation is usually done by rule or by a regex-style validation, since EC does not enforce international dialling formats by default.

## Social Accounts / Instant Message

Rarely a project focus. Person-level, not effective-dated, list-style. Mention them for completeness.

---

## National ID Card (`nationalIdCard`)

**Person-level, not effective-dated.** Holds government identifiers, **per country**.

| Field | Notes |
|---|---|
| Country/region | Determines which card types are valid |
| Card type | Picklist per country: NINO (UK), SSN (US), Aadhaar/PAN (India), Steuer-ID (Germany), My Number (Japan)… |
| National ID | The number itself |
| Is primary | Which ID is the main one for that person |

Points that come up constantly:

- **One person can hold several national IDs** — different countries (global assignments, dual nationality) and sometimes several types within one country (India: PAN *and* Aadhaar *and* UAN).
- **Validation** is country-specific: length, checksum, format. Some validations are shipped by SAP; others you implement with a business rule.
- **Uniqueness** — you usually want to prevent two people holding the same national ID. This is a duplicate-check design point (rule-based validation, or a check during data migration), not something to assume is automatic.
- **Sensitivity** — national IDs are among the most tightly permissioned fields in the system, and among the first purged after termination under retention policies.
- **Masking** — many customers mask all but the last digits in the UI for non-HR roles.

---

## Emergency Contact and Dependants

- **Emergency Contact** — person-level, not effective-dated; name, relationship, phone, address; one flagged primary. Usually maintained by the employee via self-service.
- **Dependants / Person Relationships** — person-level records for family members. In current releases these are used by **Benefits** (who is covered) and by Document Generation. Fields include relationship type, date of birth, and sometimes their own national ID and address.

Both are far more important than they look in a Benefits or Time implementation.

---

## Step by step — configure a country-specific address

1. Confirm the country is enabled in the instance.
2. Open the **Country/Region-Specific Succession Data Model** (or the country section in Manage Business Configuration, release-dependent).
3. Locate the `homeAddress` element for that country; review the shipped fields.
4. Adjust labels and required flags; attach the state/province picklist where relevant.
5. Ensure **field criteria** filter the state picklist by country.
6. Save/upload, then test by opening an employee assigned to that country and adding an address.
7. Check the payroll/downstream interface mapping still points at the right fields.
8. Permission the block in RBP for the roles that need it — including **self-service** if employees maintain their own.

---

## Real world example

A logistics company with staff in the US, Poland and the UAE:

- US addresses use the state picklist and drive **tax jurisdiction**, so address changes route through a workflow to HR and the payroll interface flags retro-dated changes.
- Poland uses a different postal-code format; validation is added by rule.
- UAE staff hold both a passport-based ID and an Emirates ID, so **two national ID card entries per person** — the data migration had to handle multiple rows per person, which the first attempt did not.
- Business email is written nightly from Active Directory, so the field is read-only in the UI for everyone except the integration user.

The Emirates ID point is a genuine migration failure mode: templates that assume one ID per person silently drop the second.

---

## Interview-grade Q&A

- **Is the address effective-dated? [HIGH]** Yes — Addresses are person-level and effective-dated, with a type (Home, Mailing…) per address.
- **How does EC handle different address formats per country? [HIGH]** Country-specific address configuration in the Country/Region-Specific Succession Data Model, with dependent picklists (e.g. state filtered by country).
- **Is National ID effective-dated? [HIGH]** No. It is a person-level list, and one person can hold several — different countries and types.
- **How do you validate a national ID? [MED]** Country-specific validation shipped where available, otherwise a business rule checking length/format; uniqueness is usually enforced by rule or during migration.
- **Which address type does payroll use? [MED]** Whichever is agreed in the interface design — usually Home; the important thing is that exactly one type is mapped.
- **Where are dependants stored and what uses them? [MED]** Person Relationships at person level; used by Benefits eligibility and document generation.
- **Are email and phone effective-dated? [HIGH]** No — they are typed lists with a primary flag.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [Employee data configuration](https://www.youtube.com/watch?v=qkmFdj4h4rA)
