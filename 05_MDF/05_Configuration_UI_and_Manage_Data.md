# Configuration UI and Manage Data

Two tools that sit either side of the object definition: **Configuration UI** designs the screen, **Manage Data** uses it.

---

## Manage Data — where records live

**Admin Center → Manage Data** is the generic maintenance screen for every MDF object in the instance.

### What you can do

| Action | How |
|---|---|
| **Find a record** | Search dropdown → pick the object → search by external code or name |
| **Create** | *Create New* → pick the object |
| **Edit** | Open the record → *Take Action* → *Make Correction* or *Insert New Record* (effective-dated objects) |
| **View history** | Effective-dated objects show their time slices |
| **Inactivate** | Set status to Inactive; do not delete records with history |
| **Delete** | Available, but almost never the right answer |
| **See related records** | Associations are navigable |

### Effective dating in Manage Data

Exactly like employee data — and with the same trap:

| Action | Effect |
|---|---|
| **Insert New Record** | New time slice from a new effective date; history preserved |
| **Make Correction** | Overwrites the current slice; history rewritten |

Renaming a department because the business renamed it → **Insert**. Fixing a typo made this morning → **Correct**. The reasoning is identical to [Effective dating and history](../02_Employee_Data/02_Effective_Dating_and_History.md).

### Practical habits

- For more than a handful of records, **use Import and Export Data**, not this screen.
- Use **export + pivot** to audit data quality rather than clicking through records.
- The search is on external code and name; if you cannot find a record, check its **effective date** and **status** — inactive or future-dated records may not appear with default filters.

---

## Configuration UI — designing the screen

**Admin Center → Manage Configuration UI** builds the layout used when a record is created or edited.

Every MDF object gets a default screen. You build a Configuration UI when the default is not good enough — which, for anything users touch, is most of the time.

### What you control

| Element | Capability |
|---|---|
| **Which fields appear** | Include or omit fields from the definition |
| **Order** | Sequence of fields |
| **Grouping** | Sections/panels with headings |
| **Labels** | Screen-specific labels |
| **Read-only / editable** | Per field, on this screen |
| **Required on this screen** | In addition to the definition's required flag |
| **Child object tables** | Composite children rendered as sub-tables |
| **Visibility rules** | Show or hide fields conditionally, driven by business rules |

### Step by step — build one

1. **Manage Configuration UI** → *Create New*.
2. Select the **base object** (e.g. `cust_CompanyCar`).
3. The tool generates a default layout containing all fields.
4. **Remove** fields users should not see, **reorder** the rest into a logical sequence.
5. **Group** related fields into sections — *Vehicle*, *Assignment*, *Insurance*.
6. Set per-field **read-only** where the value is derived or comes from an integration.
7. Add composite **child tables** if the object has them.
8. Save with a clear id (e.g. `cust_CompanyCar_UI`).
9. **Attach the UI to the object** — in Configure Object Definitions, set the object's default Configuration UI (or reference it where the object is consumed, e.g. from a workflow or a custom block).
10. Test in Manage Data: create a record and confirm the layout, ordering and read-only behaviour.
11. Test as a **non-admin user** with the intended permission role — the admin's view is not the user's view.

### Where Configuration UIs are used

- The create/edit screen in **Manage Data**
- **Custom MDF blocks on People Profile** — an MDF object rendered on the employee's profile (this is how "company car" or "professional registration" appears on an employee record)
- **Position** screens and other standard object screens
- Screens surfaced through **Take Action** on an employee

That second one is the important one for EC work: adding an MDF object as a **People Profile block** is how you extend the employee record with structured, multi-row data that a custom field could never hold.

---

## Putting an MDF object on People Profile

Step by step, because this is a genuinely common requirement:

1. Build the object and its Configuration UI (above).
2. Ensure the object has a **User field** linking it to the employee (or the appropriate standard association).
3. Admin Center → **Configure People Profile**.
4. Add a block of the MDF/custom type, pointing at your object and its Configuration UI.
5. Place it in the right section of the profile.
6. **RBP** — grant view/edit on the object to the relevant roles; without this the block appears empty or not at all.
7. Test as an employee (ESS), as a manager (MSS), and as HR. Three different results are expected — confirm each is the intended one.

---

## Common mistakes

- **No Configuration UI**, so users get the raw default layout with 30 fields in definition order.
- **Fields on the definition but not on the UI**, so they can never be filled in — a genuinely common and confusing defect.
- Setting a field required **only on the UI**, so imports and API writes bypass it (sometimes intended, often not).
- Testing only as an admin, so RBP-driven differences are missed.
- Using **Manage Data** for bulk work instead of import.
- Using **Make Correction** for a genuine change, destroying the history of a department rename.
- Forgetting that a **new field** added to the object definition later must also be added to the Configuration UI.

---

## Real world example

A customer wants **company cars** visible on the employee's profile, editable by Fleet Admin, viewable by the employee, invisible to line managers.

| Step | What was done |
|---|---|
| Object | `cust_CompanyCar`, effective-dated, secured, API visible |
| UI | Three sections: Vehicle (registration, model), Assignment (employee, issue date, return date), Insurance (provider, expiry). Provider and expiry read-only — they come from the fleet system nightly |
| Profile | Custom MDF block added to People Profile under a "Benefits & Assets" section |
| RBP | Fleet Admin: create/edit. Employee (ESS): view own. Manager: no permission at all |
| Integration | Nightly Integration Center job upserts records from the fleet provider's file |
| Alert | Rule-driven notification 30 days before insurance expiry |

Note that the *screen design* is what makes it usable, and the *permission design* is what makes it acceptable to the works council. The object definition was the easy part.

---

## Interview-grade Q&A

- **What is Manage Data? [HIGH]** The generic Admin Center screen for creating, editing, searching and inactivating records of any MDF object, including foundation objects.
- **What is Configuration UI? [HIGH]** The configurable screen layout for an MDF object — which fields appear, in what order, grouped how, read-only or editable — built in Manage Configuration UI.
- **How do you show an MDF object on the employee's profile? [HIGH]** Give the object a User reference to the employee, build a Configuration UI, add a custom MDF block in Configure People Profile pointing at the object and UI, then permission the object in RBP.
- **Insert vs Correct in Manage Data? [HIGH]** Identical semantics to employee data: Insert creates a new effective-dated slice for a genuine change; Correct overwrites the current slice to fix an error.
- **A new field does not appear on the screen. Why? [HIGH]** It was added to the object definition but not to the Configuration UI — or it is not permissioned, or its visibility is set to not visible.
- **When would you not use Manage Data? [MED]** For bulk creation or updates — use Import and Export Data instead.
- **Can a field be required on the screen but not in the definition? [MED]** Yes — Configuration UI can add a screen-level required flag, which does not apply to imports or API writes. Decide deliberately which behaviour you want.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF configuration UI](https://www.youtube.com/watch?v=yEsquQA-MxU)
