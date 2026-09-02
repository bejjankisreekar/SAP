# The Corporate Data Model (CDM)

## What it is

The **Corporate Data Model** is the XML file that defines the **organisation side** of Employee Central: the legacy foundation objects, their fields, and the associations between them.

Think of it as the **blueprint of the company's structures**, as opposed to the Succession Data Model, which is the blueprint of the people.

> **Scope reality check.** Because most foundation objects migrated to MDF, a modern instance's CDM is much smaller than it was in 2015. What remains is typically **Pay Component, Pay Component Group, Frequency, Event Reason** and **Dynamic Role**, plus associations for those. The org/job/pay objects you configure in *Manage Data* are defined in *Configure Object Definitions*, not here. Always check your own instance rather than assuming.

---

## Structure of the file

At the top level the CDM is a list of `<hris-element>` blocks, each containing `<hris-field>` entries, plus association declarations.

```xml
<hris-element id="payComponent">
    <label>Pay Component</label>

    <hris-field max-length="32" id="pay-component-code" required="true" visibility="both">
        <label>Pay Component Code</label>
    </hris-field>

    <hris-field max-length="128" id="name" required="true" visibility="both">
        <label>Name</label>
    </hris-field>

    <hris-field id="pay-component-type" required="true" visibility="both">
        <label>Type</label>
    </hris-field>

    <hris-field id="frequency" required="false" visibility="both">
        <label>Frequency</label>
    </hris-field>

    <hris-field id="currency" required="false" visibility="both">
        <label>Currency</label>
    </hris-field>

    <hris-field id="custom-string1" required="false" visibility="both">
        <label>Wage Type Mapping</label>
    </hris-field>
</hris-element>
```

*(Illustrative — exact field ids and attributes vary by release. Always work from the XML your instance actually returns.)*

Read that block as: **"there is an object called Pay Component; it has these fields; each field has an id, a label, a visibility and a required flag."** That is the whole grammar. Every element in every data model follows it.

---

## The anatomy of an `hris-field`

| Attribute | Meaning | Values |
|---|---|---|
| `id` | The technical field name — used by rules, imports and APIs | e.g. `pay-component-code`, `custom-string1` |
| `visibility` | Whether it appears at all, and whether it is editable | `both` (view + edit), `view`, `edit`, `none` |
| `required` | Whether it must be filled | `true` / `false` |
| `max-length` | Character limit | numeric |
| `pii` | Marks personally identifiable information | `true` / `false` |
| `<label>` | The display label, one per locale | `<label xml:lang="de-DE">…</label>` |
| `<picklist>` / picklist id | The picklist supplying values | picklist id |

**Custom fields** are pre-declared placeholders — `custom-string1`…`custom-stringN`, `custom-date1`…, `custom-double1`… You do not invent new field names in a data model; you take an unused placeholder, give it a label, set its visibility, and it becomes your field. That is why documenting *which placeholder is used for what* is essential — `custom-string7` means nothing to the person who inherits the instance.

---

## Associations in the CDM

Legacy foundation objects declare relationships with association tags, for example:

```xml
<hris-element id="eventReason">
    <label>Event Reason</label>
    ...
    <hris-associations>
        <association multiplicity="ONE_TO_ONE"
                     destination-entity="event"
                     required="true">
            <label>Event</label>
        </association>
    </hris-associations>
</hris-element>
```

Key attributes:

| Attribute | Meaning |
|---|---|
| `multiplicity` | `ONE_TO_ONE` or `ONE_TO_MANY` |
| `destination-entity` | The object at the other end |
| `required` | Whether the link must be populated |

The effect is the same as MDF associations: filtered picklists, validation, and the ability to propagate. See [Associations and propagation](../03_Foundation_Objects/05_Associations_and_Propagation.md).

---

## What you actually do with the CDM

Realistically, on a modern project:

1. **Read it** to understand what the previous consultant did, especially which custom fields are in use.
2. **Add a custom field** to a legacy FO — for example a wage-type mapping field on Pay Component.
3. **Change a label** so the UI speaks the customer's language.
4. **Change `visibility`** to hide fields the customer does not use.
5. **Add or adjust an association** where a legacy object needs one.
6. **Export it as documentation**, dated, into the configuration workbook.

Most day-to-day organisational configuration now happens in **Manage Data** and **Configure Object Definitions** instead.

---

## Step by step — a CDM change, done safely

1. **Write the requirement down.** Which object, which field, why, who needs to see it.
2. **Download the current CDM** from Provisioning. Save as `CDM_before_YYYYMMDD.xml`. *This is your backup.*
3. Open it in an XML-aware editor (one that validates well-formedness — a plain text editor will let you save a broken file).
4. Find the `hris-element`. Take the **next unused custom field placeholder**.
5. Set `id` (leave the placeholder id), `<label>`, `visibility`, `required`, `max-length`, and a picklist if needed.
6. **Validate the XML** — well-formed, correct nesting, no stray characters.
7. **Upload to DEV only.** Never to production first.
8. Open the object in **Manage Organization, Pay and Job Structures** and confirm the field appears and behaves.
9. **Permission it in RBP** — a new field is invisible until you do.
10. Test the downstream effects: imports (a new column appears in the template), integrations, reports.
11. **Download the new XML** as `CDM_after_YYYYMMDD.xml` and **diff** it against the before file. The diff is your change record.
12. Update the configuration workbook, including *which placeholder you used for what*.
13. Promote to TEST, retest, then PROD.

Steps 2, 11 and 12 are the ones that separate professionals from people who will one day be unable to explain what a field is.

---

## What breaks, and how badly

| Mistake | Consequence |
|---|---|
| Malformed XML uploaded | Upload rejected — or, worse, partially applied, leaving the instance inconsistent |
| Removing a field that holds data | Data becomes unreachable through the UI; reports and integrations break |
| Reusing a custom placeholder already in use | The old data is now labelled as something else — silent, and very hard to spot |
| Changing a field id | Rules, imports, integrations and reports referencing it all break |
| Uploading a CDM from another instance | You have just overwritten this customer's configuration with someone else's |
| Editing production directly | No rollback except your backup — if you took one |

The single habit that prevents all of these: **download before you upload, and diff afterwards.**

---

## Real world example

Requirement: *"Payroll needs to know which SAP wage type each pay component maps to, and it must show on the pay component record."*

Approach:

1. Pay Component is a legacy FO in this instance → the change belongs in the **CDM**.
2. Download `CDM_before_20250902.xml`.
3. In `hris-element id="payComponent"`, take `custom-string1`, label it "Wage Type", `visibility="both"`, `required="false"`, `max-length="10"`.
4. Upload to DEV, populate a few components in *Manage Organization, Pay and Job Structures*.
5. Permission it to the Payroll Admin role only.
6. Update the **payroll interface** to include the field, and the pay component **import template** now carries the extra column.
7. Diff before/after, file both with the workbook, noting: *"payComponent.custom-string1 = SAP wage type, added 02-Sep-2025, owner: Payroll."*

Six months later, when somebody asks what `custom-string1` is, the answer takes ten seconds instead of a day.

---

## Interview-grade Q&A

- **What is the Corporate Data Model? [HIGH]** The XML defining the organisation side of EC — legacy foundation object elements, their fields, and associations between them.
- **What is in it today versus historically? [HIGH]** Historically all foundation objects; today typically only the objects that did not migrate to MDF — pay component, pay component group, frequency, event reason, dynamic role — because MDF FOs are defined in Configure Object Definitions.
- **What is an `hris-element` and an `hris-field`? [HIGH]** An element is a block of data (an object or a section of the employee record); a field is one attribute inside it, with an id, label, visibility and required flag.
- **What values can `visibility` take? [HIGH]** `both`, `view`, `edit`, `none`.
- **How do you add a custom field? [HIGH]** Take an unused custom placeholder (`custom-string1`, `custom-date2`…), give it a label, set visibility and required, add a picklist if needed — then permission it in RBP and document which placeholder holds what.
- **What is the first thing you do before uploading a data model? [HIGH]** Download and archive the current version — it is both the backup and the diff baseline.
- **What happens if you upload malformed XML? [MED]** It is rejected, or partially applied, potentially leaving the instance inconsistent — which is why you change DEV first and never edit production directly.
- **Why must you never reuse a custom field placeholder? [MED]** Existing data stored in it is silently re-labelled as something else, and the error is extremely hard to detect afterwards.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Data models and XML](https://www.youtube.com/watch?v=yEsquQA-MxU)
