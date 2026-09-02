# MDF object definitions

## Where you work

**Admin Center → Configure Object Definitions.** This is where the blueprint of every MDF object lives — standard ones you can inspect, and custom ones you create.

The screen has two halves:

1. **Object header** — the object's own settings (code, label, effective dating, status, security, API visibility).
2. **Fields and associations** — the attributes and relationships.

---

## The header settings, one at a time

| Setting | What it means | How to decide |
|---|---|---|
| **Code** | The technical name, e.g. `cust_CompanyCar` | Prefix custom objects with `cust_`. Never change it afterwards |
| **Effective Dating** | `None`, `From Basic`, `From Parent`, `Basic` | **The most consequential choice.** See below |
| **API Visibility** | `Not Visible`, `Read Only`, `Editable` | Set to Editable if integrations must write it; Read Only if only extracting |
| **Label** | Display name | Translatable |
| **Status** | Active / Inactive | |
| **Secured** | Yes / No | Yes means RBP controls access. Almost always **Yes** |
| **Permission Category** | Which group of permissions it appears under in RBP | Choose a sensible category so admins can find it |
| **Menu Category** | Whether it appears in Manage Data / admin menus | |
| **Todo Category / Workflow** | Whether records can trigger workflows | |
| **Owner / Namespace** | System-managed for standard objects | Do not modify standard objects' namespace |

### Effective dating — decide this first

| Option | Behaviour | Use for |
|---|---|---|
| **None** | The record has one current version; changes overwrite | Reference data with no history value |
| **Basic** | The record is effective-dated: changes create time slices with a validity period | Anything whose history matters — positions, departments, rates |
| **From Parent** | Inherits the parent object's effective dating | Child objects in a composite relationship |

**Why it matters so much:** turning effective dating on later, for an object that already holds data, is disruptive and sometimes not supported. Ask at design time: *"will anyone ever need to know what this record looked like last year?"* If yes — or if you are unsure — choose effective dating.

### API visibility

If the object is not API-visible, it does not exist for OData, and therefore not for Integration Center or any integration. Set this deliberately at creation; discovering it during integration testing costs a day and a change request.

---

## Creating an object — step by step

Requirement: track **company cars** issued to employees.

1. **Design on paper first**: fields, types, which are mandatory, relationships, who maintains it, who sees it, whether history matters, whether integrations need it.
2. Admin Center → **Configure Object Definitions** → *Create New* → **Object Definition**.
3. Set the header:
   - Code: `cust_CompanyCar`
   - Label: *Company Car*
   - Effective Dating: **Basic** (a car's details and assignment change over time)
   - API Visibility: **Editable** (the fleet system will write to it)
   - Secured: **Yes**, Permission Category: an appropriate admin category
4. Add **fields** (see [Fields and field types](03_Fields_and_Field_Types.md)):

   | Field | Type | Notes |
   |---|---|---|
   | `externalCode` | String | The record key — every object has one |
   | `externalName` | Translatable string | The display name |
   | `registration` | String, required, max 12 | |
   | `model` | String | |
   | `issueDate` | Date, required | |
   | `returnDate` | Date | |
   | `insuranceExpiry` | Date, required | Drives an alert |
   | `carStatus` | Picklist (`cust_CarStatus`) | Active / Returned / Written off |
   | `employee` | User | The holder |

5. Add **associations** if the object relates to other objects — e.g. a valid-values association to a `cust_CarProvider` object.
6. **Save.** The object now exists.
7. **Manage Data** → create a couple of test records to check the fields behave.
8. **Manage Configuration UI** → design the screen if the default layout is not good enough.
9. **Configure Business Rules** → add validation (e.g. return date must be after issue date) and attach it to the object's `onSave`.
10. **RBP** → create or extend a permission role granting create/edit/view on the object; assign it to the Fleet Admin group.
11. **Import and Export Data** → export the definition and the data; file with the workbook.
12. Test end to end, then promote to TEST and PROD.

---

## The standard fields every object has

Every MDF object gets a small set of system fields:

| Field | Purpose |
|---|---|
| `externalCode` | The record's key. Required, unique. **Design the convention** |
| `externalName` | Display label for the record, usually translatable |
| `effectiveStartDate` | Present when effective dating is on |
| `createdBy`, `createdDate`, `lastModifiedBy`, `lastModifiedDate` | Audit |
| `mdfSystemRecordStatus` / status | Active / Inactive |

The `externalCode` deserves the same care as a foundation object's external code: stable, meaningful, and never changed after go-live.

---

## Modifying standard objects

You *can* add custom fields to standard MDF objects — a custom field on Position, or on Department. This is normal and expected.

Rules:

- **Add**, do not remove or repurpose standard fields.
- Prefix custom fields with `cust_` so the next consultant can tell them apart.
- Check whether the field needs to be added to **Configuration UI** to appear on screen.
- Check whether it needs to flow to **payroll or downstream systems**.
- Re-test the object's standard behaviour afterwards — especially for Position, where synchronisation logic touches many fields.

---

## Object statuses and lifecycle

| State | Meaning |
|---|---|
| Active object definition | Usable; records can be created |
| Inactive object definition | Hidden from Manage Data; existing data remains |
| Deleted definition | Destructive; usually blocked when data exists |

For **records**, MDF supports an active/inactive status too, and — like foundation objects — you inactivate rather than delete anything historical data points at.

---

## Common mistakes

- **Choosing "no effective dating"** on something whose history matters, then discovering the gap in UAT.
- **Forgetting API visibility**, so the integration team cannot see the object.
- **Not securing the object** (`Secured = No`), so everyone with Manage Data access can edit it.
- **Vague `externalCode` conventions** — auto-generated GUIDs make integration files unreadable.
- **Building an object for a single attribute** that belonged on `jobInfo`.
- **Creating experiments and leaving them** — instances accumulate `cust_test1`, `cust_test_new`, `cust_test_final`.
- **Changing the object code** after records exist.

---

## Interview-grade Q&A

- **Where do you create an MDF object? [HIGH]** Admin Center → Configure Object Definitions; the records then live in Manage Data.
- **What are the key header settings? [HIGH]** Code, label, effective dating, API visibility, secured flag and permission category, plus status and menu/todo categories.
- **What are the effective-dating options? [HIGH]** None, Basic (the object is effective-dated in its own right) and From Parent (inherits the parent's dating in a composite relationship).
- **Why does effective dating have to be decided up front? [HIGH]** Changing it later on an object that already holds data is disruptive and may not be supported.
- **What is API visibility for? [HIGH]** It determines whether the object is exposed as an OData entity — without it, no integration or Integration Center report can reach it.
- **What does "Secured = Yes" do? [MED]** It brings the object under Role-Based Permissions, so access is granted through permission roles rather than being open to anyone with Manage Data.
- **What standard fields does every object have? [MED]** `externalCode`, `externalName`, audit fields, a status, and `effectiveStartDate` when effective dating is on.
- **Can you add fields to standard MDF objects like Position? [HIGH]** Yes — add custom fields prefixed `cust_`, add them to the Configuration UI if they must show, permission them, and retest the object's standard behaviour.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF object definitions](https://www.youtube.com/watch?v=yEsquQA-MxU)
