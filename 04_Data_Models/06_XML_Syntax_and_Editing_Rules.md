# XML syntax and safe editing rules

You will not write data-model XML from scratch. You *will* read it, diff it, spot what a previous consultant did, and occasionally change it. This note gives you the grammar and — more importantly — the habits that stop you breaking an instance.

---

## The grammar, in five minutes

XML is nested tags with attributes:

```xml
<hris-field id="marital-status" visibility="both" required="false" max-length="32">
    <label>Marital Status</label>
    <label xml:lang="fr-FR">Situation familiale</label>
</hris-field>
```

| Part | Name | Rule |
|---|---|---|
| `<hris-field …>` | Opening tag | Must be closed |
| `id="marital-status"` | Attribute | Always in quotes |
| `<label>…</label>` | Child element | Nesting must not overlap |
| `</hris-field>` | Closing tag | Order matters: last opened, first closed |
| `<picklist id="x"/>` | Self-closing tag | Ends with `/>` |

Three rules cover 95% of breakages:

1. **Every tag closes.** `<hris-field>` needs `</hris-field>`, or must self-close.
2. **Nesting cannot overlap.** `<a><b></a></b>` is invalid.
3. **Special characters must be escaped** in text: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`. A label of "R&D" written literally will break the file.

---

## The elements you will meet

| Tag | Meaning |
|---|---|
| `<hris-element id="…">` | A block of data — an object or a section of the employee record |
| `<hris-field id="…">` | One field inside it |
| `<label>` | The display label; repeat with `xml:lang` for each locale |
| `<picklist id="…">` | The picklist supplying the field's values |
| `<hris-associations>` / `<association>` | Relationships to other objects |
| `<standard-element>` | A pre-EC user-record field (Succession Data Model only) |
| `<background-element>` | A talent profile list (Succession Data Model only) |
| `<field-criteria>` | Filtering one field's values based on another |

---

## Attribute reference

| Attribute | Applies to | Values | Notes |
|---|---|---|---|
| `id` | element, field | technical name | **Never change after go-live** |
| `visibility` | field | `both`, `view`, `edit`, `none` | Global, not per user |
| `required` | field, association | `true`, `false` | Affects every write path, not just the UI |
| `max-length` | field | number | Must fit anything inbound interfaces send |
| `pii` | field | `true`, `false` | Flags personal data for privacy handling |
| `multiplicity` | association | `ONE_TO_ONE`, `ONE_TO_MANY` | |
| `destination-entity` | association | object id | The other end of the link |
| `xml:lang` | label | locale code | `de-DE`, `fr-FR`, `ja-JP` |

---

## Reading a data model like a consultant

When you open an unfamiliar instance's XML, look for these five things in order:

1. **Which elements exist** — does this instance use `globalAssignmentInfo`? `workPermitInfo`? That tells you which features are live.
2. **Which custom placeholders are in use, and what are they labelled** — this is the fastest survey of what the previous team built.
3. **Which fields are `required="true"`** — these are the constraints every interface must satisfy.
4. **Which fields are `visibility="none"`** — usually deliberately hidden, occasionally forgotten.
5. **Which rules are attached, and at which trigger** — the behaviour that will surprise you later.

Twenty minutes of that beats a week of asking questions.

---

## The safe-editing checklist

Print this. It is the difference between a routine change and an outage.

**Before**

1. Write down the requirement and the exact change.
2. **Download the current XML.** Name it `<MODEL>_before_YYYYMMDD.xml`. This is your backup *and* your diff baseline.
3. Check whether the change can be made in **Manage Business Configuration** instead. If it can, do it there — BCUI validates, XML editing does not.
4. Confirm you are working on **DEV**.

**During**

5. Use an editor that validates XML (VS Code, Notepad++ with the XML plugin, any IDE). Not Notepad.
6. Change one thing at a time.
7. Escape special characters in labels.
8. Take an **unused** custom placeholder — check the whole file, not just the element.
9. Do not change any `id`.
10. Do not delete an element or field that holds data — set `visibility="none"` instead.

**After**

11. Validate well-formedness before upload.
12. Upload to **DEV only**.
13. Open the affected screen and confirm the field renders and saves.
14. **Permission the field in RBP** — otherwise it is invisible and you will think the change failed.
15. Check the **import template** and the **OData entity** now include it.
16. Download the new XML as `<MODEL>_after_YYYYMMDD.xml` and **diff** the two.
17. Update the configuration workbook — element, placeholder, meaning, owner, date, and the reason.
18. Promote to TEST, retest, then PROD.

---

## Diffing — the underrated skill

Two XML exports plus a diff tool answers questions nobody else in the project can answer:

- *"What changed between last Tuesday and today?"*
- *"Is DEV actually the same as TEST?"*
- *"What did the release change in our data model?"*
- *"Who added this field, and when?"*

Any diff tool works. On Windows, VS Code (**Compare Selected**) or WinMerge. Keep dated exports in a folder next to the workbook — six exports a project is enough to reconstruct any history.

```bash
code --diff SDM_before_20250902.xml SDM_after_20250902.xml
```

---

## What actually goes wrong

| Symptom | Likely cause |
|---|---|
| Upload rejected | Malformed XML — unclosed tag, overlapping nesting, unescaped `&` |
| Field does not appear after upload | Not permissioned in RBP; or `visibility="none"`; or not placed on People Profile |
| Old data now shows under a new label | A custom placeholder was reused |
| Nightly hire interface starts failing | A field was made `required="true"` |
| A rule stops firing | Its attachment was lost when the element was replaced wholesale |
| Values disappear from a dropdown | The picklist id was changed on a populated field |
| Everything looks wrong | The XML from a *different customer or instance* was uploaded |

That last one happens more than anyone admits. Check the file you are about to upload came from this instance.

---

## When to use XML and when not to

| Task | Do it in |
|---|---|
| Add a field, change a label, change visibility/required | **BCUI** |
| Attach a rule to an element | **BCUI** |
| Add a picklist value | **Picklist Center** |
| Change an MDF object | **Configure Object Definitions** |
| Bulk relabel 40 fields | XML — genuinely faster |
| Something BCUI does not expose | XML |
| Document the instance | XML export, always |
| Compare two environments | XML export + diff |

**The modern default is BCUI for changes and XML for documentation.** Saying that in an interview shows you are current, not stuck in 2016.

---

## Interview-grade Q&A

- **Do you still edit data models as XML? [HIGH]** Most day-to-day changes are made in Manage Business Configuration; XML is used for what BCUI does not expose, for bulk edits, and — always — for documentation and comparison.
- **What do you do before uploading a data model? [HIGH]** Download and archive the current version, work in DEV, and validate the XML is well-formed.
- **What are the key attributes of an `hris-field`? [HIGH]** `id`, `visibility` (`both`/`view`/`edit`/`none`), `required`, `max-length`, `pii`, plus labels per locale and an optional picklist.
- **Why must you never change a field `id`? [HIGH]** Rules, imports, integrations and reports all reference it; changing it breaks every one of them silently.
- **How do you retire a field? [MED]** Set `visibility="none"`. Do not delete it — the data stays and reappearing later with a different meaning is worse than a hidden field.
- **How would you find out what changed in the data model last month? [MED]** Diff dated XML exports; that is why you export after every change.
- **What breaks when a field is made mandatory? [HIGH]** Every write path that does not supply it — imports, OData integrations, Recruiting/Onboarding hires — starts failing.
- **You are handed an unfamiliar instance. How do you learn its configuration fast? [MED]** Export the data models, list the elements in use, the custom placeholders and their labels, the required fields, the hidden fields, and the attached rules.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Data model configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
