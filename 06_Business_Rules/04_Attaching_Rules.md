# Attaching rules

A rule that is not attached does nothing. Attachment is where the rule meets the object, the trigger and the sequence — and it is the step beginners forget, then report as "my rule doesn't work".

---

## Where rules get attached

| Attach to | Tool | Typical rules |
|---|---|---|
| **HRIS element** (`jobInfo`, `compInfo`, `personalInfo`, `employmentInfo`) | **Manage Business Configuration** | Validation, event reason derivation, cross-field logic |
| **HRIS field** | **Manage Business Configuration** | onChange derivation for one field |
| **MDF object** (save rules, validation rules) | **Configure Object Definitions** | Validation and defaulting on custom and standard MDF objects |
| **MDF field** | **Configure Object Definitions** | Conditional visibility, field-level derivation |
| **Position Management Settings** | Manage Data → Position Management Settings | Position-to-job synchronisation behaviour |
| **Workflow derivation** | Workflow configuration / element attachment | Choosing which workflow to trigger |
| **Eligibility (Benefits, Time)** | The module's configuration objects | Who is eligible for what |
| **Alerts and notifications** | Alert object configuration | When to raise an alert |

---

## Attaching to an HRIS element — step by step

1. Build and save the rule in **Configure Business Rules** first. It must exist before it can be attached.
2. Admin Center → **Manage Business Configuration**.
3. Open the **HRIS element** — for example `jobInfo`.
4. Find the rules/details section for the element.
5. Add a rule entry:
   - **Rule** — select yours
   - **Trigger / event type** — `onInit`, `onChange`, `onSave`, `onPostSave`
   - **Sequence / order** — where it runs relative to other rules at the same trigger
6. Save.
7. **Test in the UI** — the happy path and the negative case.
8. Document in the configuration workbook: rule id, element, trigger, sequence, purpose.

### Element level vs field level

| Attach at | Fires | Use for |
|---|---|---|
| **Element** | For the whole block, at the chosen trigger | Validation, cross-field logic, event reason derivation |
| **Field** | When that specific field changes | Dependent derivation driven by one field |

A rule attached to the *element* at onChange behaves differently from one attached to a *field* at onChange — the field-level one is scoped to that field's change. Choose deliberately.

---

## Attaching to an MDF object — step by step

1. Build the rule with an MDF-appropriate scenario, base object = your object.
2. **Configure Object Definitions** → open the object.
3. In the object's rules section, add the rule and choose the event: typically **Save Rules** (validation before save), **Validate Rules**, or **Initialize Rules**.
4. Save the definition.
5. Test in **Manage Data**: create a record that should pass, then one that should fail.
6. Document.

For **field-level** rules on an MDF object, attach on the field itself — commonly used for conditional visibility and defaulting.

---

## Sequencing — the part that bites

When several rules are attached to the same element at the same trigger, they execute in the configured **sequence**.

Two consequences:

1. **Dependencies must be ordered.** If rule A derives the pay grade and rule B validates the salary against that grade's range, B must run after A. Get it backwards and B validates against the *old* grade — a defect that only appears on promotions, which is exactly when it matters.
2. **The last writer wins.** If two rules write the same field, the later one overwrites the earlier. Never do this deliberately; when you find it, consolidate the two rules into one.

**Documentation habit:** keep a table in the configuration workbook — element, trigger, sequence, rule, purpose. It is the only readable view of execution order.

| Element | Trigger | Seq | Rule | Purpose |
|---|---|---|---|---|
| `jobInfo` | onSave | 10 | `JOBINFO_DERIVE_PAYGRADE` | Pay grade from job classification |
| `jobInfo` | onSave | 20 | `JOBINFO_DERIVE_EVENTREASON` | Event reason from what changed |
| `jobInfo` | onSave | 30 | `COMP_VALIDATE_PAYRANGE` | Salary within range for grade |

---

## Rules that trigger workflows

A special and important case. A rule can decide **whether** a change needs approval and **which** workflow applies:

```
IF   [Job Information . Company] has changed
AND  [Job Information . Event Reason] is equal to "TRANSFER_INTERCOMPANY"
THEN Set [Workflow] to "WF_INTERCOMPANY_TRANSFER"
```

Points to know:

- The workflow rule is evaluated on save; if it returns a workflow, the change becomes **pending data** rather than being applied.
- Multiple conditions usually collapse into one workflow-derivation rule per element, with branches — rather than several competing rules.
- If no workflow is returned, the change applies immediately.

Full treatment: [Workflow fundamentals](../07_Transactions_and_Workflows/06_Workflow_Fundamentals.md).

---

## Detaching and deactivating

| Action | Effect |
|---|---|
| Remove the attachment | The rule still exists but never runs |
| Inactivate the rule | It stops running everywhere it is attached |
| Delete the rule | Blocked while attached; detach first |
| Change the sequence | Immediate effect on execution order |

**Before deleting any rule, search for its attachments.** A rule attached in three places and deleted from one still runs in the other two — and a rule you *think* you removed but only detached from one element is a genuinely confusing defect.

---

## Real world example

A customer's promotion process needs four rules on `jobInfo`:

| Seq | Trigger | Rule | Purpose |
|---|---|---|---|
| — | onChange (field: Job Classification) | `JOBINFO_DEFAULT_TITLE` | Default job title from the job code, so the user sees it immediately |
| 10 | onSave | `JOBINFO_DERIVE_PAYGRADE` | Authoritative pay grade from the job classification |
| 20 | onSave | `JOBINFO_DERIVE_EVENTREASON` | Event reason based on what actually changed |
| 30 | onSave | `JOBINFO_DERIVE_WORKFLOW` | Route promotions to HRBP + Comp; route simple data changes to nobody |

And on `compInfo`:

| Seq | Trigger | Rule | Purpose |
|---|---|---|---|
| 10 | onSave | `COMP_VALIDATE_PAYRANGE` | Error if outside range, warning if in the top decile |

Notice the pattern: **onChange for user convenience, onSave for correctness**, sequenced so derivation precedes validation, with the workflow decision last. That structure works for almost every EC transaction.

---

## Common mistakes

- **Rule built but never attached** — the single most common "it doesn't work".
- Attached at the **wrong trigger**.
- Attached to the **wrong element** (e.g. to `compInfo` when the field lives on `jobInfo`).
- **Sequence wrong**, so validation runs before derivation.
- **Two rules writing one field** at the same trigger.
- **Deleted from one attachment only**, so it still runs elsewhere.
- **No documentation of the sequence**, so nobody can reason about execution order.
- Attaching a heavy rule at onChange of a frequently-edited field, making the screen sluggish.

---

## Interview-grade Q&A

- **Where do you attach a rule to Job Information? [HIGH]** In Manage Business Configuration, on the `jobInfo` HRIS element, choosing the trigger and the sequence.
- **Where do you attach a rule to an MDF object? [HIGH]** In Configure Object Definitions, on the object (save/validate/initialize rules) or on a specific field.
- **Element-level vs field-level attachment? [HIGH]** Element-level fires for the whole block at the chosen trigger and is used for validation and cross-field logic; field-level fires when that specific field changes and is used for dependent derivation.
- **What decides the order rules run in? [HIGH]** The configured sequence at each trigger — derivation must be sequenced before any validation that depends on it.
- **What happens if two rules write the same field? [HIGH]** The later one in the sequence wins; the fix is to consolidate them, not to reorder repeatedly.
- **How does a rule trigger a workflow? [HIGH]** A workflow-derivation rule evaluated on save returns a workflow configuration; the change then becomes pending data until approved. If no workflow is returned, the change applies immediately.
- **My rule isn't firing. First thing you check? [HIGH]** Whether it is attached at all, and at which element and trigger.
- **Before deleting a rule, what do you do? [MED]** Find every place it is attached — a rule detached from one element may still run elsewhere.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Attaching and triggering rules](https://www.youtube.com/watch?v=90aPAtJbl9g)
