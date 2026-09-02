# MDF security and picklists

Two loose ends that decide whether your object is usable and safe: **who can touch it**, and **what values it offers**.

---

## MDF security — the two switches

Access to an MDF object is controlled in two places, and both must be right.

### 1. On the object definition: `Secured`

In **Configure Object Definitions**, the object header has a **Secured** flag.

| Setting | Effect |
|---|---|
| **Secured = No** | The object is not permission-controlled — effectively anyone who can reach Manage Data can work with it |
| **Secured = Yes** | The object comes under **Role-Based Permissions**; access must be granted explicitly in a permission role |

**Default to Yes** for anything holding real data. `Secured = No` on an object containing personal data is a genuine audit finding.

Alongside it, the **Permission Category** decides *where in RBP* the object's permissions appear (for example under a Metadata Framework or Miscellaneous category). Choose a sensible one so administrators can find it.

### 2. In RBP: the permission role

Once secured, grant access in **Manage Permission Roles**:

| Permission | Meaning |
|---|---|
| **View** | Read records |
| **Create** | Add new records |
| **Edit / Update** | Change existing records |
| **Delete** | Remove records — grant sparingly, if at all |
| **Import/Export** | Bulk operations on the object |
| **Field-level** | Where supported, per-field view/edit control |

And, as with all RBP, the role is granted to a **permission group** and may be scoped by a **target population**. Full treatment: [08_Security_RBP](../08_Security_RBP/00_RBP_Learning_Path.md).

### The standard debugging order

"I can't see the object" is almost always one of these, in this order:

1. Is the object **Secured**, and is the permission actually granted in the user's role?
2. Is the object's **status Active**?
3. Is the record's **effective date** in the past (or is the user looking at a date before it starts)?
4. Is the record **Inactive**?
5. Is the field on the **Configuration UI**?
6. Is there a **field-level permission** hiding it?
7. For a People Profile block: is the **block configured** in Configure People Profile?

Work down that list rather than guessing; it resolves nearly every case.

---

## Role-based permissions and MDF objects — design guidance

- **One role per job to be done**, not one role per object. "Fleet Administration" may grant three objects.
- **Separate view from maintain.** Most users need view; very few need create/edit.
- **Delete should be nobody's permission** in production for anything with history. Use inactivation.
- **Import/export is a powerful permission** — it bypasses screen-level validation and can overwrite in bulk. Grant it to a small administrative group.
- **Test with Proxy.** Log in as the target user and confirm what they actually see; "it works for me" is not a test.

---

## MDF picklists

MDF fields draw their constrained values from **MDF picklists**, managed in **Picklist Center**. The mechanics are covered in [Picklists](../04_Data_Models/08_Picklists.md); what matters *here* is how they interact with MDF objects.

### Attaching a picklist to an MDF field

1. Create the picklist in **Picklist Center** with meaningful external codes.
2. **Configure Object Definitions** → the object → the field.
3. Set the field's data type to **Picklist** and select the picklist.
4. Save.
5. Add the field to the **Configuration UI** if it is not there.
6. Test in Manage Data.

### Field criteria on MDF fields

To filter a picklist (or an association) based on another field on the same record:

1. On the field, configure **field criteria**.
2. Specify the source field and the matching attribute.
3. Example: on `cust_ProfessionalRegistration`, filter the `cust_registrationBody` list by the employee's country.

This is the MDF equivalent of a cascading dropdown, and it is what stops users picking nonsense.

### Picklist gotchas specific to MDF

- **Rules reference external codes**, never labels. A rule comparing to `"Active"` will not match a value coded `ACTIVE`.
- **Imports match external codes.** A file containing labels fails.
- **Effective-dated picklist values** interact with effective-dated records: a value that becomes active next month cannot be used on a record dated today.
- **Inactivating a value** leaves existing records displaying it but stops new selections — the correct way to retire a value.
- **Changing which picklist a populated field uses** orphans every existing value.

---

## Putting it together — a secure, usable object

Checklist for any MDF object you build. If you can tick all of these, the object is production-ready:

- [ ] Object code follows the `cust_` convention and will never change
- [ ] Effective dating decided deliberately at creation
- [ ] `Secured = Yes` with a sensible permission category
- [ ] API visibility set to match the integration requirement
- [ ] External code convention is meaningful and documented
- [ ] Every field has the right type, length, and required/unique settings
- [ ] Picklists use stable external codes and are translated
- [ ] Field criteria configured wherever a list should be filtered
- [ ] Associations declared *and populated*
- [ ] Configuration UI built, grouped and ordered sensibly
- [ ] Business rules for conditional validation, attached and tested
- [ ] Permission roles created: view for the many, maintain for the few, no delete
- [ ] Tested with Proxy as each intended role
- [ ] Data loaded by import, with a post-load audit
- [ ] Definition and data exported, dated, filed with the configuration workbook

---

## Real world example

A works council agrees to a **disciplinary records** object on condition that access is provably restricted.

| Control | Implementation |
|---|---|
| Only HR case handlers may see records | `Secured = Yes`; a single permission role granting view/create/edit, assigned to a small static permission group |
| Managers must never see them | No permission granted; verified by Proxy testing as three different managers |
| Employees may see their own | A separate role with a target population of "self" only |
| Nobody may delete | Delete permission granted to no role; a status picklist handles closure |
| Case type must come from an agreed list | MDF picklist with fixed external codes, agreed with the works council |
| Country-specific case types | Field criteria filtering case type by the employee's country |
| Retention: purge after 3 years | Data retention policy configured on the object |
| Auditable | Effective dating on, audit fields retained, exports taken quarterly |

The configuration took an afternoon. The *evidence* that it works — the Proxy test results — is what got it approved.

---

## Interview-grade Q&A

- **How is access to an MDF object controlled? [HIGH]** Two layers: the `Secured` flag on the object definition brings it under RBP, then a permission role grants view/create/edit/delete/import, assigned to a permission group and optionally scoped by target population.
- **What does `Secured = No` mean? [HIGH]** The object is not permission-controlled, so anyone able to reach Manage Data can work with it — inappropriate for anything holding real or personal data.
- **What is the permission category on an object definition for? [MED]** It determines where the object's permissions appear within the RBP permission tree, so administrators can find them.
- **A user cannot see an MDF object. Diagnose. [HIGH]** Check the permission in their role, the object's active status, the record's effective date and status, whether the field is on the Configuration UI, any field-level permission, and — for a profile block — the People Profile configuration.
- **Would you grant Delete on an MDF object in production? [MED]** Normally no. Use an active/inactive status instead, so history and references survive.
- **Why is import/export a sensitive permission? [MED]** It operates in bulk and can bypass screen validation, so it can overwrite large volumes of data quickly.
- **How do you filter a picklist on an MDF object? [HIGH]** Field criteria — filtering the field's values based on another field's value on the same record.
- **Why must rules use picklist external codes? [HIGH]** Labels are display text and can be changed or localised; only the external code is stable, so a rule written against a label will silently fail.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF and permissions
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF security and configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
