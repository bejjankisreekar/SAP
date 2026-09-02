# Workflow configuration, step by step

Building a real workflow end to end: the promotion approval from [Workflow fundamentals](06_Workflow_Fundamentals.md).

**Requirement:** a promotion routes to the manager's manager, then the HRBP; if the salary increase exceeds 10%, the Compensation team must also approve. Payroll is informed of the outcome. All other job changes need only the HRBP.

---

## Before you configure anything

Answer these, in writing, with the customer:

1. **What triggers it?** A change to Job Information with event reason Promotion.
2. **Who approves, and why does each step exist?** Manager's manager (budget), HRBP (policy), Compensation (only above the threshold).
3. **Who is informed?** Payroll, as a CC role.
4. **May approvers edit the data?** No — they approve or reject; edits go back to the initiator.
5. **What happens if nobody acts?** Reminder at 3 days, escalate at 7.
6. **Who can act when the approver is away?** Delegation; plus HRBP is a job relationship so it resolves to whoever currently holds it.
7. **What if the approver has left?** Administrative reassignment; process documented.

Skipping step 2 is how you end up with the six-step workflow nobody uses.

---

## Step 1 — prerequisites

| Prerequisite | Why |
|---|---|
| **HR Manager job relationship** maintained (ideally derived by rule) | So the HRBP step can resolve |
| **Dynamic group** for the Compensation team | So any member can approve |
| **Event reasons** configured, with derivation | Because the workflow rule keys off the reason |
| **Email notifications** enabled and templates reviewed | Approvers act on emails |
| **RBP**: approvers can see the data they are approving | Otherwise they approve blind or get an error |

That last one is a real and frequent defect: an approver who cannot see compensation fields cannot meaningfully approve a salary change.

---

## Step 2 — create the workflow configuration

Admin Center → **Manage Data** → *Create New* → **Workflow Configuration**.

| Field | Value |
|---|---|
| External code | `WF_PROMOTION_STD` |
| Name | Promotion approval — standard |
| Description | Manager's manager → HRBP. Used for promotions with an increase of 10% or less |
| Remind in days | 3 |
| Escalation | 7 days → escalate to the next step's approver |
| Alternate approver behaviour | Per customer policy |
| Edit transaction | No |

Then add the **steps**:

**Step 1**
| Setting | Value |
|---|---|
| Step sequence | 1 |
| Approver type | **Role** — Employee's Manager's Manager |
| Approval type | Approval |
| Context | Job Information |

**Step 2**
| Setting | Value |
|---|---|
| Step sequence | 2 |
| Approver type | **Job relationship** — HR Manager |
| Approval type | Approval |

**CC role**
| Setting | Value |
|---|---|
| Type | Dynamic group — Payroll team |
| When | On completion |

Save.

---

## Step 3 — create the second workflow

Create `WF_PROMOTION_HIGH` identically, plus a third step:

**Step 3**
| Setting | Value |
|---|---|
| Step sequence | 3 |
| Approver type | **Dynamic group** — Compensation team |
| Approval type | Approval |

And a third, simpler one for ordinary job changes:

`WF_JOBCHANGE_STD` — a single step, approver type job relationship = HR Manager.

**Design note:** three workflow configurations is better than one workflow with conditional steps, because each is readable on its own and the derivation rule makes the choice explicit. Where the product supports conditional steps within one workflow, either approach is defensible — but the readable one wins in a handover.

---

## Step 4 — the derivation rule

Build in **Configure Business Rules**, cross-entity scenario (Job Information + Compensation):

```
IF   Job Information.Event Reason = "JC_PROMOTION"
AND  ((New Annual Salary - Old Annual Salary) / Old Annual Salary) > 0.10
THEN Set Workflow = "WF_PROMOTION_HIGH"

ELSE IF Job Information.Event Reason = "JC_PROMOTION"
THEN Set Workflow = "WF_PROMOTION_STD"

ELSE IF Job Information.Event Reason is in ("JC_LATERAL", "TR_INTERNAL", "JC_FTE_CHANGE")
THEN Set Workflow = "WF_JOBCHANGE_STD"

ELSE Set Workflow = <none>
```

Points to note:

- The final `ELSE` returning **no workflow** is deliberate: everything else applies immediately. Without it, you may route changes you never intended to route.
- The percentage calculation needs the **old** salary, which is why this is a cross-entity rule.
- Guard against a null or zero old salary, or the division fails.

---

## Step 5 — attach the rule

**Manage Business Configuration** → `jobInfo` → attach the rule at **onSave**, sequenced **after** the event-reason derivation rule.

| Seq | Trigger | Rule |
|---|---|---|
| 10 | onSave | `JOBINFO_DERIVE_PAYGRADE` |
| 20 | onSave | `JOBINFO_DERIVE_EVENTREASON` |
| 30 | onSave | `COMP_VALIDATE_PAYRANGE` |
| 40 | onSave | `JOBINFO_DERIVE_WORKFLOW` ← this one, last |

Sequence 40 matters: the workflow rule reads the event reason, so it must run after the reason is derived.

---

## Step 6 — permissions

In **Manage Permission Roles**:

- Approvers need **view** on the data being approved — Job Information and Compensation fields.
- Approvers need the **workflow approval permission** in their role.
- The **workflow administrator** role needs Manage Workflow Requests, including reassignment.
- The initiator needs permission to raise the change in the first place.

---

## Step 7 — test, properly

The part that takes the longest, and the part that finds everything.

| Test | Expected |
|---|---|
| Promotion, 5% increase | Routes to manager's manager → HRBP. Payroll CC'd on completion |
| Promotion, 15% increase | Adds the Compensation step |
| Lateral move | Routes to HRBP only |
| Cost centre correction by HR | **No workflow** — applies immediately |
| Approver rejects at step 1 | Change discarded; live record unchanged; initiator notified |
| Approver sends back | Returns to the initiator for correction |
| Initiator withdraws | Request cancelled |
| Employee has no HR Manager relationship | **What happens?** This is the test people forget — the workflow cannot resolve the approver. Decide the behaviour and configure it |
| Approver is on holiday with delegation set | Routes to the delegate |
| Nobody acts for 3 days | Reminder sent |
| Nobody acts for 7 days | Escalation fires |
| Approver has left the company | Administrative reassignment works |
| Future-dated promotion | Approved now, applied on the effective date |
| While pending, someone edits the same record | Observe and document the behaviour — this is where conflicting requests arise |

**Test with Proxy** as each approver. The admin's view is not the approver's view.

---

## Step 8 — document

In the configuration workbook:

- Each workflow: code, purpose, steps, approver types, CC roles, reminders, escalation.
- The derivation rule and its sequence.
- The decision log: why the HR Director step became a CC role.
- The leaver-reassignment process.

---

## Common configuration mistakes

- Rule attached **before** event-reason derivation, so it routes on a stale reason.
- **No final ELSE**, so unintended changes route for approval.
- Percentage calculation with **no null/zero guard**.
- Approvers **cannot see** the fields they are approving.
- **No test** for the missing-relationship case, so the workflow fails silently in production.
- Named users as approvers.
- Reminder and escalation left at defaults nobody agreed.
- **Notifications not enabled**, so approvers never learn there is anything to do.

---

## Interview-grade Q&A

- **Walk me through configuring an approval workflow. [HIGH]** Agree the trigger, steps and purpose of each step with the customer; ensure prerequisites (job relationships, dynamic groups, event reasons, notifications, permissions); create the Workflow Configuration object with steps, approver types, CC roles, reminders and escalation; write a derivation rule choosing the workflow; attach it at onSave sequenced after event-reason derivation; permission the approvers; then test every path including rejection, send-back, escalation, missing approver and future dating.
- **Where is a workflow defined? [HIGH]** As a Workflow Configuration MDF object, maintained in Manage Data.
- **How does the system know which workflow to use? [HIGH]** A workflow derivation business rule at onSave returns the workflow configuration; no return means the change applies immediately.
- **Why must the workflow rule run after event-reason derivation? [HIGH]** Because routing keys off the event reason; running first would evaluate a stale or empty value.
- **How would you route conditionally on the size of a salary increase? [HIGH]** A cross-entity rule comparing the new and previous salary and returning a different workflow above the threshold — guarding against a null or zero previous value.
- **What do you test in a workflow? [HIGH]** Each routing branch, rejection, send-back, withdrawal, delegation, reminder, escalation, a missing approver relationship, an approver who has left, future-dated changes, and concurrent edits while pending — with Proxy as each approver.
- **What if the employee has no HR Manager relationship? [HIGH]** The step cannot resolve. Decide the behaviour deliberately — a fallback approver, a default group, or a validation preventing the change until the relationship exists — and test it.
- **Should you build one workflow with conditional steps or several workflows? [MED]** Several readable workflows chosen by an explicit derivation rule is usually easier to understand and hand over; conditional steps are defensible where the product supports them and the variation is small.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — transactions and workflows
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Workflow configuration](https://www.youtube.com/watch?v=qkmFdj4h4rA)
