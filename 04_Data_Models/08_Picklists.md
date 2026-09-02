# Picklists

## What they are

A **picklist** is a configurable list of values behind a dropdown: marital status, address type, employee class, event reason, relationship type. They are everywhere in EC, and they are the single easiest thing to get wrong in a way that only shows up in reporting six months later.

Analogy: the drop-down list of countries on a web form. The *label* is what the user reads ("United Kingdom"); the *code* is what the system stores (`GB`). Change the label and nothing breaks. Change the code and every system downstream loses its mind.

---

## The anatomy of a picklist

| Concept | Meaning |
|---|---|
| **Picklist** | The list itself, identified by a picklist id (e.g. `maritalStatus`) |
| **Picklist value** | One entry in the list |
| **External code** | The stable key of the value — `MARRIED`, `SINGLE`. **This is what is stored, imported and integrated** |
| **Label** | What the user sees, per locale |
| **Status** | Active / Inactive |
| **Effective dating** | MDF picklists are effective-dated |
| **Parent value** | For cascading picklists |
| **Sort order / display order** | The order in the dropdown |
| **Non-unique / optional attributes** | Extra values some picklists carry (e.g. min/max for a rating scale) |

**The one rule that matters most:** *the external code is permanent; the label is cosmetic.* Design codes to be meaningful and stable — `RESIGNATION`, not `ER_007` and not `Resignation (voluntary) 2024`.

---

## Legacy picklists vs MDF picklists

Another historical split, like foundation objects.

| | **Legacy picklists** | **MDF picklists** |
|---|---|---|
| Managed in | Picklist Management (import/export CSV) | **Picklist Center** |
| Effective-dated | No | **Yes** |
| Cascading | Via parent picklist ids in the CSV | Configured in the UI |
| Extensible | Limited | Can carry additional attributes |
| Direction of travel | Being retired | The current model |

Modern instances use **Picklist Center**. If you meet an older instance still on legacy picklists, expect a migration to be on the roadmap — SAP has progressively moved picklists to MDF.

> As always: check your instance. If **Picklist Center** shows the list, it is MDF.

---

## Creating a picklist — step by step

1. Admin Center → **Picklist Center**.
2. **Create new picklist**: give it an id (e.g. `preferredPronouns`) and a name. Use a clear, consistent naming convention — you will have hundreds.
3. Add **values**, each with:
   - **External code** — uppercase, stable, meaningful (`THEY_THEM`)
   - **Label** in the default locale
   - **Labels in other locales** if the customer is multilingual
   - **Effective start date**
   - **Status** Active
   - **Sort order**
4. Save.
5. Attach the picklist to a field in **Manage Business Configuration** (for HRIS fields) or in **Configure Object Definitions** (for MDF object fields).
6. **Test** in the UI: is the list right, in the right order, in every language?
7. Export the picklist and file it with the configuration workbook.

For volume, use **import**: Picklist Center supports CSV import/export, and building 300 values in a spreadsheet is far faster and less error-prone than typing them.

---

## Cascading (dependent) picklists

A cascading picklist filters its values based on the value chosen in a **parent** field.

Classic examples:

| Parent | Child |
|---|---|
| Country | State / Province |
| Country | National ID card type |
| Event | **Event Reason** |
| Job Function | Job Family |
| Termination category | Termination reason |

**Event → Event Reason is the one you will build.** It is why a user choosing "Termination" sees only termination reasons, not "New Hire".

### How it works

Each child value declares its **parent value**. When the parent field is set, the child list is filtered to values whose parent matches.

### Cascading vs field criteria

They are easy to confuse:

| | **Cascading picklist** | **Field criteria** |
|---|---|---|
| Filters based on | Another **picklist value** | Another **field's value**, including foundation objects |
| Configured in | Picklist Center (parent-child) | Data model / object definition |
| Typical use | Country → State; Event → Event Reason | Job Classification filtered by Job Function; Pay Range filtered by grade and geozone |

In practice you use both, and when a dropdown shows the wrong values you check both.

---

## Translations

If the customer operates in more than one language, every picklist value needs a label per locale.

- Missing translations fall back to the default locale — so a French user sees English values, which looks broken and generates tickets.
- Translation is a **project workstream**, not an afterthought: hundreds of values × several languages, needing HR sign-off in each country.
- Export the picklists to CSV, send for translation, re-import. Doing it in the UI value by value is how projects lose a week.

---

## The rules nobody tells you until you break them

1. **Never delete a picklist value that is in use.** Historical records point at it. Set it **Inactive** instead — existing records keep their value and display it; new records cannot select it.
2. **Never change an external code after go-live.** Imports, integrations, business rules and reports all match on the code. Changing it silently orphans every record that used it.
3. **Changing a *label* is safe.** That is the correct way to respond to "we want it to say something different".
4. **Business rules reference the external code**, not the label. A rule written against `Married` will not fire on a value coded `MARRIED`.
5. **Imports match on the external code.** A migration file containing "Full Time" fails against a picklist coded `FULL_TIME`.
6. **Do not encode structure in the code.** `EVT_TERM_RESIGN_2024` is a code you will regret; `RESIGNATION` is not.
7. **Watch the picklist swap.** Changing which picklist a populated field uses orphans every existing value.

---

## Picklist, foundation object, or MDF object?

A design question that comes up constantly. Choose by asking *"does this thing need attributes of its own?"*

| Use a… | When |
|---|---|
| **Picklist** | It is a simple list of values with a code and a label — marital status, address type, employee class |
| **Foundation Object / MDF object** | It has attributes, associations, effective-dated history, or needs to drive derivation — department, job classification, pay grade, location |

Test: *"Will anyone ever ask 'what is the cost centre of that value?'"* If yes, it is an object, not a picklist. Getting this wrong — modelling departments as a picklist — is a classic beginner error that cannot be undone cheaply.

---

## Real world example

A customer needs event reasons for termination. First attempt:

| Code | Label |
|---|---|
| `1` | Resignation |
| `2` | Dismissal |
| `3` | Retirement |

Six months later they add "Resignation — competitor", "Resignation — relocation", "End of fixed-term contract" and two country-specific reasons. The codes are now `1`–`8` and nobody can read an integration file, a report, or a business rule without a lookup table.

Redone properly:

| Code | Label | Parent (Event) |
|---|---|---|
| `TERM_RESIGNATION` | Resignation | `TERMINATION` |
| `TERM_RESIGN_COMPETITOR` | Resignation — joining competitor | `TERMINATION` |
| `TERM_DISMISSAL` | Dismissal | `TERMINATION` |
| `TERM_RETIREMENT` | Retirement | `TERMINATION` |
| `TERM_END_CONTRACT` | End of fixed-term contract | `TERMINATION` |

Now: rules read clearly, integration files are self-documenting, reports group sensibly, and adding a reason costs nothing. The prefix convention also makes cascading obvious at a glance.

**The migration cost of fixing this after go-live is high** — you cannot simply rename codes, so you create new values, inactivate the old ones, and live with a mixed history. Which is exactly why you design codes on day one.

---

## Common mistakes

- **Numeric or opaque external codes.**
- **Deleting** values instead of inactivating them.
- **Changing codes** after go-live.
- **Missing translations** for a multilingual workforce.
- **Modelling an object as a picklist** — departments, job codes, cost centres.
- **Rules written against labels** instead of codes.
- **Migration files containing labels**, not codes.
- **No sort order**, so a 60-value list appears in creation order.
- **Duplicate values** created because two people built the same list independently — always search before creating.

---

## Interview-grade Q&A

- **What is a picklist? [HIGH]** A configurable list of values behind a dropdown, each with a stable external code and one or more localised labels.
- **External code vs label? [HIGH]** The external code is the stored, permanent key used by imports, integrations, rules and reports; the label is what the user sees and can be changed freely.
- **Can you delete a picklist value? [HIGH]** You should not, if it is in use — set it inactive. Existing records keep and display the value; new records cannot select it.
- **Legacy vs MDF picklists? [MED]** Legacy picklists were maintained by CSV import and are not effective-dated; MDF picklists are managed in Picklist Center, are effective-dated, and support richer configuration. MDF is the current model.
- **What is a cascading picklist? [HIGH]** A picklist whose values are filtered by the value of a parent field — country → state, or **event → event reason**.
- **Cascading picklist vs field criteria? [MED]** Cascading filters on a parent *picklist value*; field criteria filter on another *field's value*, including foundation objects such as job function or pay grade.
- **When do you use a picklist rather than a foundation object? [HIGH]** When the value is just a code and a label. If it needs attributes, associations, history or derivation, it must be an object.
- **A migration file keeps failing on employee class. Why? [HIGH]** The file contains labels rather than external codes, or codes that do not exist, or values that are inactive as of the record's effective date.
- **What happens if you change the picklist attached to a populated field? [MED]** Existing values are orphaned — they no longer resolve against the new list.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring foundation objects
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Picklists and configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
