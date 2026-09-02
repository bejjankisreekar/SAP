# MDF fields and field types

Choosing the right field type is a five-second decision that either works forever or causes a data-quality problem you cannot fix without a migration.

---

## The field types

| Type | Holds | Use for | Watch out for |
|---|---|---|---|
| **String** | Text | Names, codes, references | Set `maxLength` sensibly; too short breaks inbound interfaces |
| **Translatable String** | Text with a value per locale | Anything users read: object names, descriptions | Only for display text, never for keys |
| **Number / Integer** | Whole numbers | Counts, sequence | |
| **Decimal / Double** | Decimals | FTE, percentages, rates | Set precision; decide rounding behaviour |
| **Date** | A date | Start, end, expiry | Time zones do not apply — this is a plain date |
| **DateTime** | Date and time | Timestamps | Time zone handling matters |
| **Boolean** | true/false | Flags | A picklist with Yes/No is often better — it translates and reports more cleanly |
| **Picklist** | A value from an MDF picklist | Any constrained list | Use external codes, never labels, in rules and imports |
| **Generic Object** | A reference to another MDF object record | Links to Department, Location, custom objects | This is how you point at another object |
| **User** | A reference to a user | Owner, holder, approver, responsible person | Stores the user id |
| **Attachment** | An uploaded file | Certificates, scanned documents | Storage limits and privacy implications |
| **Auto Number** | A system-generated sequence | Record numbering where no natural key exists | Makes integration files harder to read — prefer a meaningful code |
| **Data Source / Foundation Object reference** | A reference to a legacy FO | Pay component and similar | Legacy pattern |

---

## Field-level settings

For each field you can set:

| Setting | Meaning | Guidance |
|---|---|---|
| **Identifier / name** | Technical id | Prefix custom fields with `cust_`; never change it |
| **Label** | Display name, translatable | Change this when the wording is wrong, not the id |
| **Data type** | From the table above | Cannot be changed freely once data exists |
| **Required** | Mandatory | Applies to imports and API writes too |
| **Max length / precision** | Size | Must accommodate every source system |
| **Default value** | Static default | Use a rule when the default is conditional |
| **Valid values source** | Picklist or association | |
| **Visibility** | Visible / Read only / Not visible | Global; per-user control is RBP |
| **Status** | Active / Inactive | Inactivate rather than delete |
| **Unique** | Enforces uniqueness across records | Useful for registration numbers and codes |
| **Field criteria** | Filter this field's values by another field | The MDF equivalent of dependent dropdowns |
| **Rules** | Validation and defaulting rules on the field | |

---

## Choosing between the confusing pairs

**String vs Translatable String**
Use Translatable String for anything a user reads in their own language (object name, description). Use plain String for codes and references. A translatable key is a maintenance nightmare.

**Boolean vs Yes/No picklist**
Boolean is simpler; a picklist translates, reports more cleanly, and can be extended later ("Yes / No / Not applicable"). Most experienced consultants default to a picklist for anything a user selects, and Boolean for internal flags.

**Picklist vs Generic Object reference**
If the value is just a code and a label → picklist. If it has attributes of its own → an object, referenced with a Generic Object field. Same test as [picklist vs foundation object](../04_Data_Models/08_Picklists.md).

**User field vs String holding a user id**
Always the **User** type. It validates, renders a person picker, and integrates with permissions. A string holding a user id has none of that and will contain typos.

**Auto Number vs meaningful external code**
Auto numbers are convenient and unreadable. If a human will ever read an integration file or a report row, design a meaningful code.

---

## Validation options

Three layers, in increasing order of power:

1. **Field settings** — required, max length, unique, valid-values source. Free, declarative, and always evaluated.
2. **Field criteria** — restrict values based on another field on the same record.
3. **Business rules** — anything conditional: cross-field checks, date comparisons, lookups, calculated defaults, error and warning messages.

**Prefer the highest layer that does the job.** A required flag is better than a rule that checks for blank, because it needs no maintenance and cannot be bypassed by a rule-execution order problem.

Typical rule-level validations on an MDF object:

- End date must be after start date
- Expiry date must be in the future when the record is created
- If status is "Returned", return date is mandatory
- Registration number must match a country-specific format

---

## Worked example — designing the fields

Object: `cust_ProfessionalRegistration` (from the hospital example).

| Field | Type | Required | Notes |
|---|---|---|---|
| `externalCode` | String | Yes | `{employeeId}-{bodyCode}-{number}` — meaningful and unique |
| `externalName` | Translatable String | Yes | Display name |
| `cust_employee` | User | Yes | Whose registration it is |
| `cust_registrationBody` | Generic Object → `cust_RegBody` | Yes | Bodies have their own attributes (country, website, renewal period) → object, not picklist |
| `cust_registrationNumber` | String, max 30, **unique** | Yes | Uniqueness enforced by the field setting |
| `cust_issueDate` | Date | Yes | |
| `cust_expiryDate` | Date | Yes | Drives the alert |
| `cust_status` | Picklist `cust_RegStatus` | Yes | Valid / Expired / Suspended / Withdrawn |
| `cust_evidence` | Attachment | No | Scanned certificate |
| `cust_notes` | String, max 500 | No | |

Design choices worth noticing:

- Registration body is an **object**, not a picklist, because it has attributes (renewal period feeds the alert timing).
- Registration number is **unique** at field level — no rule needed.
- Status is a **picklist**, not a Boolean, because there are four states, not two.
- Employee is a **User** field, so it validates and renders a person picker.
- `externalCode` is **meaningful**, so an integration file is readable.

---

## Changing fields after go-live

| Change | Safe? |
|---|---|
| Change a label | **Yes** |
| Add a new field | **Yes** |
| Make an optional field required | Risky — every write path must supply it |
| Increase max length | Usually safe |
| **Decrease** max length | Dangerous — existing values may exceed it |
| Change data type | Generally not supported once data exists |
| Change the field identifier | **Never** — rules, imports, integrations and reports break |
| Inactivate a field | Safe; data is retained but hidden |
| Delete a field with data | Destructive; avoid |

The pattern is the same as for data models: **add and hide freely; rename and retype never.**

---

## Common mistakes

- **Boolean where a picklist was needed**, discovered when the customer asks for a third state.
- **String user references** instead of the User type.
- **Max length too short** for what the source system sends — silent truncation or failed imports.
- **Auto-number codes** that make every integration file unreadable.
- **No uniqueness** on a field that must be unique, then duplicates in production.
- **Everything required**, so imports of legacy data with gaps fail entirely.
- **Translatable strings used as keys.**
- **Validation written as a rule** when a field setting would have done it.

---

## Interview-grade Q&A

- **What field types does MDF support? [HIGH]** String, Translatable String, Number/Integer, Decimal, Date, DateTime, Boolean, Picklist, Generic Object reference, User, Attachment, Auto Number, and foundation object references.
- **When would you use a Generic Object field rather than a picklist? [HIGH]** When the referenced thing has attributes of its own — a picklist stores only a code and a label.
- **How do you reference an employee on an MDF object? [HIGH]** With a **User** field, which validates the reference and renders a person picker.
- **How do you enforce that a field is unique? [MED]** With the field-level unique setting, not a business rule.
- **Boolean or Yes/No picklist? [MED]** Picklist for anything a user selects — it translates, reports cleanly and can be extended; Boolean for internal flags.
- **Which changes to a field are unsafe after go-live? [HIGH]** Changing the identifier, changing the data type, decreasing max length, and making an existing field required without checking every write path.
- **Where should validation live? [HIGH]** At the highest declarative level that works — required/unique/max length as field settings, then field criteria, then business rules for conditional or cross-field logic.
- **What is field criteria on an MDF object? [MED]** Filtering one field's valid values based on another field's value on the same record.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF object configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
