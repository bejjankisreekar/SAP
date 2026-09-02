# Job change, transfer and promotion

The everyday transactions. Mechanically they are all the same thing — **a new effective-dated Job Information row** — and what distinguishes them is the **event reason**, the **fields that changed**, and therefore the **approval and downstream behaviour**.

---

## They are all one transaction

```
Take Action → Change Job and Compensation
   → choose the effective date
   → change whichever fields changed
   → rules derive the event and event reason
   → workflow derivation decides whether approval is needed
   → save
```

A promotion is not a different screen from a transfer. It is the same screen with different fields changed, producing a different derived reason. Understanding that is the point of this note.

| Transaction | What typically changes | Typical event / reason |
|---|---|---|
| **Data change** | An administrative attribute — cost centre correction, a custom field | Data Change / correction |
| **Lateral move** | Job classification, same pay grade | Job Change / lateral move |
| **Promotion** | Job classification and/or pay grade upward, usually with a salary increase | Promotion / promotion |
| **Demotion** | Job or grade downward | Demotion |
| **Internal transfer** | Department, manager, location — same legal entity | Transfer / internal transfer |
| **Intercompany transfer** | Legal entity | Transfer / intercompany |
| **Working time change** | FTE, standard hours, employment type | Job Change / working time change |
| **Manager change** | Manager only | Data Change or Transfer, per customer design |
| **Position change** | Position (with values flowing from it) | Position change / transfer |
| **Leave of absence** | Employee status changes to a leave status | Leave of Absence |
| **Return to work** | Status back to Active | Return to Work |

---

## What each transaction touches

| Transaction | Job Info | Comp Info | Position | Workflow typically |
|---|---|---|---|---|
| Data change | ✓ | | | None or manager |
| Lateral move | ✓ | Maybe | ✓ | Manager → HRBP |
| Promotion | ✓ | ✓ | ✓ | Manager → HRBP → Compensation |
| Transfer (internal) | ✓ | Maybe | ✓ | Both managers → HRBP |
| Transfer (intercompany) | ✓ | ✓ (pay group, currency) | ✓ | Managers → HRBP → Payroll/Legal |
| Working time change | ✓ | ✓ (annualisation, FTE-based amounts) | ✓ (position FTE) | Manager → HRBP |
| Leave of absence | ✓ (status) | Sometimes (stop pay) | Position held or released? | HRBP |
| Return to work | ✓ (status) | Sometimes | | Manager |

That "Position: held or released?" question for leave is a real design decision — see [Position Management in transactions](../09_Position_Management/05_Position_Management_in_Transactions.md).

---

## The transfer, in detail

Transfers are where the most goes wrong, because more changes at once.

**Internal transfer** (same legal entity, new department and manager):

1. New Job Information row from the effective date.
2. Department, manager and cost centre change; cost centre usually derived from the new department.
3. The **old manager loses** the person from their team on the effective date; the **new manager gains** them. RBP target populations follow automatically.
4. Any in-flight workflows addressed to the old manager may need attention.
5. Time and absence: entitlements usually continue; location-based holiday calendars may change.
6. Position: the person leaves one position and occupies another; the old one may become vacant or be closed.

**Intercompany transfer** (different legal entity) adds:

- **Country and currency** may change → country-specific fields and pay components change.
- **Pay group changes** → the employee moves to a different payroll.
- Payroll needs to treat it as a **leaver in one entity and a starter in another**, which the interface must handle explicitly.
- Statutory reporting in both countries.
- Benefits eligibility resets.
- The design decision from [Hire and rehire](02_Hire_and_Rehire.md): transfer, or terminate-and-rehire?

**The interview-worthy point:** an intercompany transfer is a *payroll event*, not just an HR data change, and the interface design is the hard part.

---

## The promotion, in detail

A promotion normally spans two elements, which is why it is the standard example of a **cross-entity** requirement:

1. **Job Information** — new job classification, possibly new position and title, event reason Promotion.
2. **Compensation Information** — new pay grade (derived from the job), new salary.
3. **Validation** — salary within the new grade's pay range.
4. **Workflow** — usually the longest chain, because money is involved.
5. **Downstream** — Compensation and Succession see the new grade; payroll gets the new salary from the effective date; retro applies if it is back-dated.

Design questions that come up in every project:

| Question | Typical answer |
|---|---|
| Can a promotion happen without a salary change? | Yes, and the rule validating "promotion requires job or grade change" must allow it |
| Can a salary change happen without a promotion? | Yes — that is a pay rate change, a different event |
| Who initiates? | The manager, usually, with HR and Compensation approving |
| Effective date rules? | Often restricted to the first of a month for payroll simplicity |
| Off-cycle vs the annual comp cycle? | Off-cycle promotions must not be overwritten when the Compensation module publishes; agree the sequencing |

---

## Mass changes

A reorganisation moves 400 people to a new department on 1 April. Options:

| Approach | When |
|---|---|
| **Foundation object change** — re-parent the department | If the *structure* changed, not the people's assignment. Often the correct and much cheaper answer |
| **Mass import** of Job Information rows | If people genuinely move between existing units |
| **Mass change tool** (where available) | For moderate volumes with a common change |
| Manual | Only for small numbers |

**Always ask first: did the people move, or did the structure change?** A rename or re-parenting is FO work — see [Managing and importing FOs](../03_Foundation_Objects/06_Managing_and_Importing_FOs.md). Doing 400 employee transactions when a single FO change was needed is a classic waste, and it pollutes the history with events that never happened.

For a genuine mass move:

1. Extract the affected population with their current values.
2. Build the Job Information import file with the new values, the effective date and the correct event reason.
3. Decide **rules on or off** — usually on for ongoing changes, so derivation and validation apply.
4. Decide **workflow on or off** — normally off for an approved mass change, since the approval happened as a project decision.
5. Load into TEST first, check counts and a sample, then PROD.
6. Verify the org chart, RBP target populations and any in-flight workflows.

---

## Leave of absence and return to work

Often overlooked, and a favourite scenario question.

- **Leave of absence** is a Job Information row with a leave event reason whose **employee status** is Paid or Unpaid Leave.
- The employee is **not terminated** — they remain in headcount (usually), but pay and benefit behaviour changes.
- **Return to work** is another row restoring status to Active.
- If Time Off is implemented, absences of certain types can create these rows automatically.
- **Position:** decide whether the position stays occupied (so it cannot be filled) or is released (so a temporary hire is possible). Both are valid; the customer must choose.

---

## Real world example

A promotion that went wrong, and why:

A manager promotes someone effective 1 March, entered on 15 March. Payroll had already run for March.

What happened:
- The change was **retroactive**, so payroll needed a retro calculation for the two weeks.
- The interface sent the change without flagging retroactivity, so payroll processed it as a current-period change and underpaid.
- The event reason had been chosen manually as "Data Change", so the compensation report did not show a promotion at all.

The fixes:
1. Event reason **derivation** so "Data Change" cannot be chosen for a job change.
2. A rule limiting non-HR users to a 30-day retro window, with a warning on any back-dated change.
3. The payroll interface updated to flag retroactive changes explicitly.
4. A policy that promotions take effect on the first of the month following approval, agreed with payroll.

Every one of those fixes is a topic in this repo. That is what "the transaction is where everything comes together" means.

---

## Common mistakes

- Using **Correct** instead of **Insert**, so the promotion applies retroactively to the whole previous period.
- **Wrong effective date** — today's date out of habit instead of the business date.
- Doing 400 **employee transactions** when a foundation object change was needed.
- Not checking **in-flight workflows** after a transfer — approvals addressed to the old manager.
- **Intercompany transfers** treated as simple data changes, so payroll mishandles them.
- Promotions entered during the **annual compensation cycle** and then overwritten when the cycle publishes.
- Leave of absence with an event reason whose **status is Active**, so people on unpaid leave keep being paid.

---

## Interview-grade Q&A

- **How do you promote someone in EC? [HIGH]** Take Action → Change Job and Compensation with the effective date; change the job classification and salary; rules derive pay grade, event and event reason and validate the salary against the range; a workflow routes it; on approval, new effective-dated rows appear in Job Information and Compensation.
- **What is the difference between a promotion, a transfer and a data change? [HIGH]** Mechanically nothing — all create a new Job Information row. They differ by which fields changed, which event reason is derived, and therefore which approvals and downstream behaviour apply.
- **What extra work does an intercompany transfer involve? [HIGH]** Country and currency change, country-specific fields, a new pay group, payroll treating it as a leaver and a starter, statutory reporting in both countries, benefits eligibility, and often legal approval.
- **How do you handle a reorganisation of 400 people? [HIGH]** First determine whether the *structure* changed (a foundation-object change, effective-dated) or the *people* moved (a mass Job Information import with the right event reason). Load to TEST first, decide rules and workflows on or off, verify the org chart, RBP populations and in-flight workflows.
- **How is a leave of absence recorded? [HIGH]** A Job Information row with a leave event reason whose employee status is Paid or Unpaid Leave; return to work is another row restoring Active. The employee is not terminated.
- **A promotion was entered after payroll ran. What now? [HIGH]** It is a retroactive change: payroll needs a retro calculation, the interface must flag it as retroactive, and policy usually limits how far back non-HR users may date changes.
- **Can a promotion happen without a pay rise? [MED]** Yes — and the validation rule requiring a job or grade change must allow it, just as a pay rise without a job change is a pay rate change rather than a promotion.
- **What happens to permissions when someone transfers? [MED]** RBP target populations based on the manager hierarchy update automatically on the effective date; in-flight workflows addressed to the previous manager may need reassignment.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring transactions
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Lifecycle transactions](https://www.youtube.com/watch?v=qkmFdj4h4rA)
