# Job Relationships and Dependants

Two small elements that do disproportionate work — one drives approvals and permissions, the other drives benefits.

---

## Job Relationships (`jobRelationsInfo`)

**Employment-level, effective-dated.** Records people *other than the line manager* who have a defined relationship to this employee.

| Relationship type (configurable picklist) | Typical use |
|---|---|
| **HR Manager / HR Business Partner** | Workflow approval step; RBP target population for HR |
| **Matrix Manager** | Dotted-line reporting in matrix organisations |
| **Second Manager** | Backup approver, or the manager's manager for escalation |
| **Custom Manager** | A named relationship the customer invents — e.g. "Project Lead", "Mentor", "Works Council Rep" |

Each row holds the relationship type, the related person, and effective dates. One employee can have several relationships at once, and each relationship type can have its own history.

### Why job relationships matter more than they look

1. **Workflow routing.** A workflow step can be addressed to a *relationship* rather than a named person: "route to the employee's HR Manager". That is how one workflow definition serves 20,000 employees.
2. **RBP target populations.** A permission group can be granted access to "everyone where I am the HR Manager" — a relationship-based target population. This is how HRBPs get exactly their own population without maintaining a list.
3. **Matrix organisations.** The line manager on Job Information is one hierarchy; matrix managers in Job Relationships are another. Reporting and approvals can use either.

### Design points

- **Keep the list short.** Every relationship type is a maintenance burden — someone has to keep it accurate for every employee, forever.
- **Decide how they are maintained.** Typed by HR? Derived by rule from department or position? Loaded from an integration? "Typed by HR and never updated" is the default failure mode.
- **Derive where possible.** A rule that sets HR Manager from the employee's Legal Entity + Business Unit removes thousands of manual entries.
- **Effective dating still applies.** When an HRBP changes, insert a new record; don't correct the old one, or the historical approvals no longer make sense.
- **Termination handling.** When the person *holding* a relationship leaves, everyone pointing at them has a dangling relationship. Plan a report or a mass update; this is a real hypercare ticket.

---

## Dependants / Person Relationships

**Person-level.** Family members and other related persons: spouse, child, parent, domestic partner, emergency contact in some designs.

| Field | Notes |
|---|---|
| Relationship type | Picklist: spouse, child, parent, partner, other |
| Name, date of birth, gender | May be a full sub-record with its own address and national ID |
| Is beneficiary / is dependant flags | Used by Benefits |
| Effective dates | Depending on configuration |

### What consumes dependant data

- **Benefits** — eligibility and enrolment ("family cover" needs to know who the family is).
- **Document Generation** — visa letters, insurance nominations.
- **Country-specific payroll** — tax allowances based on number of children in several countries.
- **Time Off** — some absence types are tied to dependants (parental leave).

### Design points

- **Privacy.** Dependant data is personal data about people who are not your employees. Permission it narrowly, and include it in retention/purge policy.
- **Self-service.** Most customers let employees maintain their own dependants, sometimes with an approval workflow for benefit-relevant changes.
- **Data quality.** Dependant date of birth drives child-benefit and insurance eligibility; a missing DOB silently removes a person from cover.

---

## Emergency Contact vs Dependant

They are different, and the distinction is frequently muddled in requirements workshops:

| | **Emergency Contact** | **Dependant** |
|---|---|---|
| Purpose | Who to call in an emergency | Who is covered by benefits / drives entitlements |
| Overlaps? | Often the same person, but not always | |
| Stored as | Emergency Contact element | Person Relationship / Dependants |
| Consumed by | HR, safety processes | Benefits, payroll, documents |

Do not merge them just because the names overlap. A colleague can be an emergency contact; they are not a dependant.

---

## Step by step — configure a job relationship type and use it

1. **Picklist**: add the relationship type (e.g. `HR_MANAGER`) to the job relationship type picklist in Picklist Center.
2. **Manage Business Configuration → `jobRelationsInfo`** — confirm the field configuration and visibility.
3. **Business rule (optional but recommended):** onSave of Job Information, derive the HR Manager from Legal Entity + Business Unit and write the relationship.
4. **Workflow:** create a workflow step with approver type = *Job Relationship* → HR Manager.
5. **RBP:** create a permission group and grant a role with target population = "HR Manager relationship".
6. **Test** with a real employee: change their business unit and confirm the derived HR Manager changes, then trigger a workflow and confirm it routes to the right person.
7. **Report** on employees with a missing or inactive relationship holder — build this before go-live, not after.

---

## Real world example

A bank with 12,000 staff:

- Every employee has an **HR Manager** relationship, derived by rule from Legal Entity + Division. No one types it.
- Promotions route: line manager → HR Manager (from the relationship) → Compensation Partner.
- HRBPs get their population through an RBP target population based on that relationship — when the derivation rule changes, access follows automatically.
- **Matrix Manager** is maintained manually for around 900 project-based staff, and a monthly report lists matrix managers who have left.
- Dependants are employee-maintained through self-service, feeding the medical insurance interface weekly.

The lesson: derive relationships where you can, report on the ones you cannot.

---

## Interview-grade Q&A

- **What are job relationships used for? [HIGH]** Recording non-line-manager relationships (HR Manager, matrix manager, custom) that drive workflow routing and RBP target populations.
- **How would you route a workflow to an employee's HRBP? [HIGH]** Define an HR Manager job relationship, then use approver type *job relationship* in the workflow step.
- **How do you avoid maintaining job relationships by hand? [MED]** Derive them with a business rule from organisational data (legal entity, business unit, department, position).
- **Are job relationships effective-dated? [MED]** Yes — insert a new record when the relationship changes, so historical approvals remain explainable.
- **What happens when someone holding a relationship leaves? [MED]** Everyone pointing at them has a stale relationship; you need a report and a mass-update process.
- **Where are dependants stored, and what uses them? [HIGH]** Person Relationships at person level; used by Benefits eligibility, some country payrolls, Time Off entitlements and document generation.
- **Is an emergency contact the same as a dependant? [MED]** No — different purpose, different element, and often a different person.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [Employee data configuration](https://www.youtube.com/watch?v=qkmFdj4h4rA)
