# Termination

The transaction with the most downstream consequences, the most dates, and the most legal exposure. Get it wrong and someone keeps getting paid, or loses access on the wrong day, or the company breaches a retention obligation.

---

## What termination does

**Take Action → Manage Worker → Terminate**, with an effective date.

| Record | Effect |
|---|---|
| **Job Information** | A new effective-dated row with a Termination event and reason; **employee status** becomes Terminated (or Retired, per the reason) |
| **Employment Information** | End date, last date worked, payroll end date, rehire eligibility |
| **User record** | Set **inactive** on (or after) the termination date — the user is not deleted |
| **Position** | The incumbent is removed; the position becomes vacant or is closed — a design decision |
| **Compensation** | Recurring pay components end; final payments may be entered as non-recurring |
| **Workflows** | In-flight approvals owned by this person need reassignment |
| **Person data** | Retained until a retention policy purges it |

**Nothing is deleted.** The employee remains in the system as an inactive user with full history, which is what makes historical reporting and legal defence possible.

---

## The dates — and why there are so many

This is the part interviewers probe, because in practice they differ.

| Date | Meaning | Who cares |
|---|---|---|
| **Termination date / end date** | The last day of employment | HR, reporting |
| **Last date worked** | The last day physically worked — may be earlier (garden leave, notice not worked) | Managers, time recording |
| **Payroll end date** | The last date payroll should pay | Payroll |
| **Benefits end date** | When cover ceases — often the end of the month | Benefits providers |
| **System access revocation date** | When accounts are disabled — often *before* the termination date for dismissals | IT and security |
| **Notice period start/end** | Contractual | HR, legal |

A dismissal for misconduct might have: last date worked = today, access revoked = today, termination date = end of notice, payroll end = end of notice, benefits end = end of month. Five different dates for one event.

**Design consequence:** the payroll and identity-management interfaces must each be told *which* date they use. Sending "termination date" to everyone is how people keep their laptop access for a month, or lose their health cover a week early.

---

## Termination reasons and status

The event reason drives the outcome:

| Reason | Employee status | Notes |
|---|---|---|
| Resignation | Terminated | Voluntary |
| Resignation — competitor | Terminated | Often flagged for the exit-risk report |
| Dismissal — performance / conduct | Terminated | Involuntary; often `okay-to-rehire` = No |
| Redundancy | Terminated | Involuntary, no fault; usually rehire-eligible |
| End of fixed-term contract | Terminated | Not a "leaver" in some turnover metrics |
| Retirement | **Retired** | Different status; often different benefits treatment |
| Death in service | Terminated (or a specific status) | Triggers a sensitive, distinct process |
| Transfer out to associated company | Terminated | Excluded from turnover reporting |

**Voluntary vs involuntary** is usually a custom attribute on the event reason, because turnover reporting depends on it and the event alone does not distinguish them.

---

## Termination step by step

1. **Take Action → Manage Worker → Terminate.**
2. Enter the **termination date** and select the **termination reason** (filtered by category through a cascading picklist).
3. Enter **last date worked**, **payroll end date** and any other required dates.
4. Set **eligible for rehire** — a genuine HR decision, not a default.
5. Enter any additional fields: notice period served, exit interview flag, return of company property.
6. **Save** → normally a workflow routes to HRBP (and Payroll for some reasons).
7. On approval, the records above are updated.

### What must happen around it

| Area | Action |
|---|---|
| **Position** | Release it (vacant, possibly To Be Hired) or close it |
| **Direct reports** | Reassign them to a new manager — otherwise you have orphaned employees and a broken hierarchy |
| **Workflows** | Reassign or cancel approvals awaiting this person |
| **Job relationships** | Everyone who had this person as HR Manager or matrix manager needs updating |
| **Delegations** | Remove any proxy or delegation rights |
| **Integrations** | Payroll, identity management, benefits, learning, badge access |
| **Assets** | Company car, phone, laptop — often tracked in MDF objects |

**The direct-reports point is the one people forget**, and it is a strong answer to "what else must happen when a manager leaves?"

---

## Data retention and purge

Terminated employees' data is retained, but not forever.

- **Data Retention Management / Data Purge** configures rules to delete or anonymise personal data after a defined period following termination.
- Retention periods differ by country and by data type — payroll data is often kept far longer than recruitment data.
- Purge is **irreversible**, so it runs on a defined schedule with sign-off, not ad hoc.
- Legal holds must be respected: a person involved in litigation must be excluded from purge.

Under GDPR and similar regimes this is a legal obligation in both directions: keep it long enough, and no longer. Being able to say "termination starts a retention clock, and purge is configured in Data Retention Management with country-specific periods" is a strong, current answer.

---

## Reversing a termination

Someone is terminated in error, or withdraws their resignation.

| Situation | Handling |
|---|---|
| Entered in error, not yet approved | Withdraw the pending workflow request |
| Entered in error, already applied | Delete or correct the termination row in Job Information history, restore employment dates and status, reactivate the user |
| Genuine change of mind after leaving | Depending on the gap and policy: reverse the termination, or **rehire** as a new employment |
| Already replicated to payroll | Reversing in EC is not enough — payroll must be corrected too, and identity management may need to restore accounts |

**The important point:** reversing a termination in EC does not reverse what downstream systems already did. Always ask what has already run.

---

## Real world example

A manager of 14 people resigns with three months' notice.

The sequence that actually has to happen:

1. Termination entered with the future termination date; a workflow routes to HRBP.
2. Because the date is in the future, the person **stays active** until then — they still manage their team, still approve, still appear in the org chart.
3. Two weeks before the date, HR reassigns the **14 direct reports** with an effective date matching the departure — otherwise on day one they are orphaned.
4. **Job relationships** where this person is the HRBP or matrix manager for others are updated.
5. **In-flight workflows** awaiting their approval are reassigned.
6. Their **position** is marked To Be Hired and a requisition is raised.
7. On the termination date: user set inactive, identity management disables accounts at 23:59, payroll processes the final pay using the payroll end date.
8. Benefits cease at month end, per the benefits end date.
9. The retention clock starts; purge is scheduled per the country's policy.

Steps 3–5 are the ones that generate support tickets when skipped. This sequence is a very good answer to "walk me through a termination".

---

## Common mistakes

- Using a **single date** for everything, so access, pay and benefits all end on the wrong day.
- **Not reassigning direct reports**, leaving orphaned employees and a broken hierarchy.
- Not reassigning **in-flight workflow approvals**.
- Setting **rehire eligibility** by default instead of as a decision.
- A termination reason whose **employee status is not a terminated status**, so the person stays Active and keeps being paid.
- **Deleting** the employee instead of terminating them.
- Assuming reversing in EC reverses downstream systems.
- No **retention/purge** policy, so personal data is kept indefinitely.
- Forgetting the **position** — vacancies never appear and headcount planning breaks.

---

## Interview-grade Q&A

- **What happens when you terminate an employee? [HIGH]** A new Job Information row with a termination event reason sets the employee status; Employment Information gets end date, last date worked, payroll end date and rehire eligibility; the user is set inactive; the position is released or closed; recurring pay components end. Nothing is deleted.
- **Why are there several termination dates? [HIGH]** Termination date, last date worked, payroll end date, benefits end date and access revocation date serve different consumers and frequently differ — for example garden leave or a dismissal where access ends immediately but employment continues through notice.
- **What determines whether someone becomes Terminated or Retired? [HIGH]** The employee status configured on the event reason used.
- **What else must happen when a manager leaves? [HIGH]** Reassign direct reports with a matching effective date, reassign in-flight workflow approvals, update job relationships where they were the HRBP or matrix manager, remove delegations, and handle the position and requisition.
- **Is the employee deleted? [HIGH]** No — the user is set inactive and the data is retained for history, reporting and legal purposes, until a retention policy purges it.
- **How is data retention handled? [HIGH]** Data Retention Management defines country- and data-type-specific periods after termination, after which data is deleted or anonymised; purge is irreversible, scheduled, and must respect legal holds.
- **How do you reverse a termination? [MED]** Withdraw the workflow if pending; otherwise delete or correct the termination row, restore the employment dates and status, and reactivate the user — then correct payroll and identity management, because reversing in EC does not undo what downstream systems already processed.
- **Where is "eligible for rehire" stored and who uses it? [MED]** On Employment Information, set at termination and checked at rehire and by Recruiting when a former employee applies.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring transactions
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Lifecycle transactions](https://www.youtube.com/watch?v=qkmFdj4h4rA)
