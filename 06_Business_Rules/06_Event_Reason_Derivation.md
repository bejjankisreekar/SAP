# Event reason derivation

The most frequently requested rule in Employee Central, and one of the most frequently asked interview questions. It deserves its own note.

---

## The problem

Every Job Information row carries an **Event** and an **Event Reason** — the record of *what happened*. See [Events and Event Reasons](../07_Transactions_and_Workflows/01_Events_and_Event_Reasons.md) for the objects themselves.

Left to users, event reasons are chosen badly:

- Managers pick "Data Change" for a promotion, so the promotion never appears in turnover or compensation reporting.
- Different countries pick different reasons for the same action.
- The dropdown has 60 entries and people pick the first plausible one.
- Payroll receives the wrong reason and processes the change incorrectly.

**Bad event reasons are not a cosmetic problem.** They corrupt headcount and turnover reporting, retro payroll behaviour, workflow routing, and every downstream analysis.

The fix: **derive the event reason from what actually changed**, rather than asking the user.

---

## The design decision first

Three approaches, and you should be able to argue for each:

| Approach | How | When it fits |
|---|---|---|
| **Full derivation** | The user never chooses; a rule sets the reason from what changed | High-volume, standardised processes; low user maturity; strong reporting requirements |
| **Guided choice** | The user picks from a **filtered** list (cascading picklist by event), rule validates the choice | Where nuance matters — several valid reasons for the same field change |
| **Hybrid** | Rule derives a default; user may override within a filtered list; rule validates the override | Most real projects |

**The hybrid is the usual answer**, and saying so demonstrates judgement: derivation gives consistency, the filtered override handles the cases the rule cannot know (was this resignation to a competitor or for family reasons?).

---

## How derivation works

The rule compares the **new** values with the **previous** values on the record and infers what happened.

```
onSave on jobInfo:

IF   this is the first record (no previous)          → Event = HIRE,        Reason = NEW_HIRE
ELSE IF Company changed                              → Event = TRANSFER,    Reason = TRANSFER_INTERCOMPANY
ELSE IF Job Classification changed
     AND new Pay Grade > old Pay Grade               → Event = PROMOTION,   Reason = PROMOTION
ELSE IF Job Classification changed
     AND new Pay Grade = old Pay Grade               → Event = JOB_CHANGE,  Reason = LATERAL_MOVE
ELSE IF Job Classification changed
     AND new Pay Grade < old Pay Grade               → Event = DEMOTION,    Reason = DEMOTION
ELSE IF Department changed OR Manager changed        → Event = TRANSFER,    Reason = INTERNAL_TRANSFER
ELSE IF FTE changed                                  → Event = JOB_CHANGE,  Reason = WORKING_TIME_CHANGE
ELSE IF Location changed                             → Event = TRANSFER,    Reason = LOCATION_CHANGE
ELSE                                                 → Event = DATA_CHANGE, Reason = DATA_CORRECTION
```

Three things to notice:

1. **Order matters enormously.** The conditions are evaluated in sequence, so the most significant change must be tested first. A promotion that also changes department must be recorded as a promotion, not a transfer — so the job/grade branch comes before the department branch.
2. **It is a decision tree, not a set of independent rules.** Splitting this across several rules produces conflicting writes to the same field.
3. **The final `ELSE` matters.** Every path must produce a reason, or you will get blank event reasons — which break reporting just as badly as wrong ones.

---

## Accessing the previous value

The heart of the technique is comparing new and old. How you get the previous value depends on the scenario:

- Cross-entity and EC-specific scenarios typically expose the **previous record** or a "has changed" style comparison for fields.
- Where that is not available, an alternative is to read the record valid on the day before the effective date.

**In an interview**, describe the *concept* — "the rule compares the incoming values against the previous effective-dated record and derives the reason from the difference" — rather than claiming a specific function name, since these vary by release.

---

## Terminations and hires are special

Terminations and hires normally come through their own **transactions** (Take Action → Terminate, or the hire process), and the event is set by the transaction itself. What you derive there is the **reason**, usually from a user selection, filtered by a cascading picklist:

| Termination category (user picks) | Filtered reasons |
|---|---|
| Voluntary | Resignation, Resignation — competitor, Resignation — relocation, Retirement |
| Involuntary | Dismissal — performance, Dismissal — conduct, Redundancy |
| Other | End of fixed-term contract, Death in service, Transfer out |

And the reason drives the **employee status** (Terminated, Retired) plus downstream behaviour such as rehire eligibility. That is configuration on the event reason object, not a rule.

---

## Validating an override

If users may override the derived reason, validate the override:

```
IF   Event Reason = "PROMOTION"
AND  Job Classification has NOT changed
AND  Pay Grade has NOT changed
THEN Raise Error "Promotion requires a change of job or pay grade."
```

This is the rule that stops "promotion" being used for a pay rise with no job change — which is the most common event-reason data-quality complaint in live systems.

---

## Step by step — build it

1. **Design the event reason catalogue first** with HR: the list of reasons, their events, their employee statuses, and which are user-selectable. This is a workshop, not a configuration task.
2. Create the **Event Reason** records and the **cascading picklist** (Event → Event Reason).
3. Write the **derivation rule** as a single decision tree, ordered by significance.
4. Attach it at **onSave** on `jobInfo`, sequenced **before** the workflow-derivation rule (because workflow routing depends on the reason).
5. Write the **override validation rule**, sequenced after derivation.
6. **Test a matrix** of transactions — this is the part that takes real time:

   | Test | Expected reason |
   |---|---|
   | New hire | NEW_HIRE |
   | Job change, higher grade | PROMOTION |
   | Job change, same grade | LATERAL_MOVE |
   | Department change only | INTERNAL_TRANSFER |
   | Manager change only | INTERNAL_TRANSFER |
   | FTE change only | WORKING_TIME_CHANGE |
   | Legal entity change | TRANSFER_INTERCOMPANY |
   | Promotion **and** department change | PROMOTION (not transfer) |
   | Cost centre change only | DATA_CHANGE |
   | Salary change only (on compInfo) | The compensation event reason, not a job event |

7. Check the **downstream effects**: workflow routing, employee status, payroll replication, headcount and turnover reports.
8. Document the decision tree in the configuration workbook — in a table, so HR can read it.

Step 6's "promotion and department change" row is the test that finds ordering bugs. Always include it.

---

## Real world example

A retailer's first attempt let managers choose freely from 34 event reasons. After six months:

- 61% of all rows were "Data Change".
- Turnover reporting was unusable; promotions could not be counted.
- Payroll processed several changes with the wrong retro behaviour.

The fix:

1. Reduced the catalogue to 22 reasons, each mapped to an event and a status, agreed with HR and Payroll.
2. Built a **derivation rule** covering nine branches, ordered by significance.
3. Restricted the event reason field to **read-only for managers**; HR retains override rights within a filtered list.
4. Added the **override validation rule**.
5. Left historical data alone (it cannot be reconstructed) and flagged the cut-over date in reporting.

After the change, "Data Change" fell to 9% of rows and promotions became countable. That last point — *"we could not report on promotions until we fixed event reasons"* — is a very strong thing to be able to say in an interview.

---

## Common mistakes

- **Wrong condition order**, so a promotion with a department change is recorded as a transfer.
- **No final ELSE**, leaving blank event reasons.
- Splitting derivation across **several rules** that overwrite each other.
- Deriving at **onChange** instead of onSave, so it misses the final state and does not run for imports or API writes.
- Sequencing the derivation **after** the workflow rule, so routing uses a stale reason.
- Comparing to picklist **labels** instead of external codes.
- Building the rule **before** agreeing the catalogue with HR.
- Not testing the **combination** cases.

---

## Interview-grade Q&A

- **How do you derive an event reason? [HIGH]** A single onSave rule on Job Information structured as a decision tree, comparing the new values against the previous effective-dated record and setting the event and reason from the most significant change — with a final ELSE so no row is left blank.
- **Why does condition order matter? [HIGH]** Because changes overlap. A promotion that also changes department must be recorded as a promotion, so the job/grade test must precede the department test.
- **Where do you attach it, and why not onChange? [HIGH]** onSave, because the reason depends on the final state of the record and because onChange does not fire for imports or API writes.
- **How does it interact with workflow routing? [HIGH]** The workflow-derivation rule usually reads the event reason, so event-reason derivation must be sequenced first.
- **Should users be allowed to choose the event reason? [HIGH]** A hybrid is usual: derive a default for consistency, allow HR to override within a cascading, filtered list, and validate the override — because some distinctions (why someone resigned) cannot be inferred from the data.
- **How do you stop "Promotion" being used with no job change? [HIGH]** A validation rule that raises an error if the reason is Promotion but neither job classification nor pay grade changed.
- **What breaks when event reasons are wrong? [HIGH]** Headcount and turnover reporting, promotion analytics, workflow routing, employee status, retro payroll behaviour and downstream replication.
- **How do you test event reason derivation? [HIGH]** A matrix of transaction types including combination cases — a promotion that also changes department is the case that exposes ordering bugs.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC / configuring transactions
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business rules and event reason derivation](https://www.youtube.com/watch?v=90aPAtJbl9g)
