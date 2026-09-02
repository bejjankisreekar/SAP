# Debugging rules

"My rule isn't working." This note is the checklist that turns that sentence into a diagnosis in ten minutes instead of an afternoon.

---

## The diagnostic order

Work down this list. Do not skip ahead — the answer is usually in the first four.

### 1. Is the rule attached at all?

The most common cause, by a wide margin. Building a rule in Configure Business Rules does **nothing** until you attach it in Manage Business Configuration (HRIS elements) or Configure Object Definitions (MDF).

**Check:** open the element or object and look at its rules list.

### 2. Is it attached at the right trigger?

An onChange rule will not fire when the user never touches that field. An onInit rule will not fire mid-transaction. A validation attached at onPostSave cannot block anything.

**Check:** the trigger on the attachment, against what you actually need. See [Trigger points](03_Trigger_Points_onInit_onChange_onSave.md).

### 3. Is it attached to the right element?

A rule that writes a Compensation field but is attached to `jobInfo` may not have Compensation in scope, or may run before the compensation values are set.

**Check:** the element and the scenario's base object agree with where the fields live.

### 4. Is the condition actually true?

Usually a null, or a code/label mismatch.

- Fields the rule reads may be **empty** at that point in the flow.
- The rule may compare to a picklist **label** where the stored value is an **external code**.
- A date comparison may be evaluating against a different effective date than you assume.

**Check:** temporarily simplify the condition (e.g. make it always true) and see whether the action executes. If it does, the condition is the problem. If it does not, go back to steps 1–3.

### 5. Is another rule overwriting the result?

Two rules at the same trigger writing the same field: the later one wins.

**Check:** list every rule attached to that element at that trigger, in sequence order, and look for a second writer. Also check **propagation**, which can set the same field.

### 6. Is the sequence wrong?

A validation that runs before the derivation it depends on will validate against stale values.

**Check:** the sequence numbers on the attachments.

### 7. Is this a path where rules do not run?

| Path | Behaviour |
|---|---|
| UI | All triggers |
| **Import** | Depends on the import's rule setting — often deliberately off |
| **OData / Integration Center writes** | Save-time validation generally applies; onChange (a UI event) does not |
| Recruiting/Onboarding hire | Rules apply |

**Check:** how the data actually arrived. "It works when I do it but not overnight" is nearly always this.

### 8. Is the rule effective-dated out of range?

Rules have effective dates. A rule effective from 1 January will not apply to a transaction dated in December.

### 9. Is it a permission problem being mistaken for a rule problem?

If the rule writes a field the user cannot see or edit, the outcome can look like the rule failing.

**Check:** test with a role that has full access, and with Proxy as the affected user.

### 10. Is the rule inactive?

Simple, and occasionally the answer.

---

## The tools

| Tool | What it tells you |
|---|---|
| **Business Rule Execution Log / Rules Trace** | Which rules fired for a transaction, in what order, and their outcome. Naming varies by release — search Admin Center for "rule" and "log"/"trace" |
| **Check Tool** | Configuration inconsistencies, including some rule-related ones |
| **Proxy** | Reproduce the exact experience of the affected user |
| **Manage Business Configuration** | The authoritative list of what is attached where, with sequence |
| **A test employee** | Your most valuable tool — see below |

**The execution log is the single most useful tool** when it is available in your release. Enable it, run the transaction, and read which rules fired. If the rule is not listed, the problem is attachment or trigger (steps 1–3). If it is listed but did nothing, the problem is the condition (step 4).

---

## The bisection technique

When the logic is complex and the log does not make it obvious:

1. Copy the rule.
2. In the copy, **replace the condition with something always true** and attach the copy instead.
   - Action fires → the original condition is at fault.
   - Action still does not fire → attachment, trigger or scenario is at fault.
3. If the condition is at fault, add the conditions back **one at a time**, testing each.
4. The condition that stops it working is your bug — usually a null or a code/label mismatch.

Crude, and it works every time.

---

## Common root causes, ranked

From most to least common in real projects:

1. **Rule not attached**, or attached at the wrong trigger.
2. **Null values** at the moment the rule runs.
3. **Picklist label vs external code** mismatch.
4. **Two rules (or a rule and propagation) writing the same field.**
5. **Wrong sequence** — validation before derivation.
6. **Rules disabled on the import** that loaded the data.
7. **Wrong scenario**, so the rule cannot see the entity it needs.
8. **Effective dating** — of the rule, or of the record being evaluated.
9. **RBP** hiding the field the rule writes.
10. **Rule inactive.**

---

## Preventing rule problems

Cheaper than debugging:

- **One rule, one purpose.** Compound rules are hard to diagnose.
- **Systematic naming** — `JOBINFO_DERIVE_PAYGROUP` tells you the element, the action and the target.
- **Write the description**, including who asked for it and why.
- **Maintain the sequence table** in the configuration workbook.
- **Null-guard everything.**
- **Test negative cases** as standard, not as an afterthought.
- **Keep a set of test employees** covering the awkward shapes: part-time, dual-country, position-managed, concurrent employment, rehired, contingent worker. Run every rule change against all of them.
- **Test through every write path** that matters — UI, import, and API — not just the UI.

---

## Worked example

*"The pay-range validation lets managers save salaries above the maximum, but it works when I test it."*

Diagnosis, in order:

1. **Attached?** Yes, to `compInfo`, onSave. ✓
2. **Right trigger?** Yes. ✓
3. **Right element?** Yes. ✓
4. **Condition true?** Simplify: always-true version raises the error for managers too. So the condition is failing for them, not the attachment.
5. **What differs for a manager?** The rule reads Geozone from Job Information to find the pay range. Managers' permission role does **not** grant the Geozone field.
6. Geozone is therefore **null** in the rule's context for managers → the pay-range lookup returns nothing → the comparison is not true → no error.
7. **Fix:** grant view permission on Geozone to the manager role (it is not sensitive), and add a defensive branch: if no pay range is found, raise a warning rather than silently passing.

Note the two-part fix: the immediate cause (permission) and the resilience gap (a rule that silently does nothing when its lookup fails). Fixing only the first leaves the same trap for the next country you add.

---

## Interview-grade Q&A

- **A rule is not firing. Walk me through your diagnosis. [HIGH]** Is it attached; at the right trigger; on the right element; is the condition actually true (nulls, code vs label); is another rule or propagation overwriting the result; is the sequence right; did the data arrive through a path where rules do not run; is the rule effective-dated out of range; is a permission hiding the field; is the rule active.
- **What tool shows which rules fired? [HIGH]** The business rule execution log / rules trace, where available in the release — it lists the rules that ran, in order, with their outcomes.
- **How do you isolate a faulty condition? [MED]** Bisection: replace the condition with an always-true version to confirm attachment and trigger, then add the conditions back one at a time.
- **Why might a rule work in the UI but not overnight? [HIGH]** The data arrives by import or API. Imports may have rules disabled, and onChange rules never fire for non-UI paths.
- **Why might a rule work for you but not for a manager? [HIGH]** Field-level permissions — a field the rule reads is not granted to that role, so it evaluates as null.
- **What is the most common cause of a rule not working? [HIGH]** It is not attached, or it is attached at the wrong trigger.
- **How do you prevent rule defects? [MED]** One rule per purpose, systematic naming and descriptions, a documented sequence table, null guards, negative testing, a standing set of awkward test employees, and testing every write path.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business rules troubleshooting](https://www.youtube.com/watch?v=90aPAtJbl9g)
