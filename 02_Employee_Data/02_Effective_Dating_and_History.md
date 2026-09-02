# Effective dating and history

## The idea

An **effective-dated** record is valid **from a date onwards**, until another record supersedes it. EC does not overwrite; it **slices time**.

Analogy: a bank statement. You don't erase last month's balance when this month's arrives — you add a new line with a date. To answer "what was my balance in March?", you read the line that was valid in March.

```
Job Information history for Priya
┌────────────┬──────────────────┬──────────────┬─────────────────┐
│ Effective  │ Job title        │ Department   │ Event / Reason  │
├────────────┼──────────────────┼──────────────┼─────────────────┤
│ 01-Jul-2021│ Analyst          │ Finance      │ Hire / New Hire │
│ 01-Jan-2023│ Analyst          │ Treasury     │ Transfer / Move │
│ 01-Apr-2025│ Senior Analyst   │ Treasury     │ Job Change / Promotion │
└────────────┴──────────────────┴──────────────┴─────────────────┘
```

"What was Priya's department on 15-Jun-2023?" → Treasury. The system reads the row whose validity covers that date. That is the whole mechanism.

---

## Start date, end date, and why you rarely type an end date

Each row has an **effective start date**. Its **end date is implicit**: the day before the next row starts. The final row runs to a high date (commonly 31-Dec-9999).

Consequences:

- Inserting a row in the middle automatically **re-slices** the surrounding rows.
- You cannot have two rows with the same effective date on the same element — the second replaces the first.
- Deleting the middle row extends the previous one forward.

---

## Which EC elements are effective-dated

| Effective-dated | Not effective-dated |
|---|---|
| Personal Information | Biographical Information |
| Addresses (Home Address) | Email Information |
| Global Information (country-specific person data) | Phone Information |
| Job Information | National ID Card |
| Compensation Information | Emergency Contact |
| Recurring Pay Components | Social Accounts |
| Job Relationships | (Most "list-style" person data) |
| Position (MDF, effective-dated) | |

Non-effective-dated elements simply hold current values — change them and the old value is gone (an audit trail may still record it, but there is no time slice to query).

**Interview tip:** the classic pair to quote is *"Biographical Information is not effective-dated; Personal Information is."* Date of birth doesn't change on a Tuesday; a surname can.

---

## Insert vs Correct — the single most important operational skill

Every effective-dated block offers a **History** view with two very different actions:

| | **Insert New Record** | **Correct / Make Correction** |
|---|---|---|
| What it does | Adds a **new time slice** starting on a new date | **Overwrites** the existing time slice |
| History | Old values remain valid for the old period | Old values are gone (replaced) |
| Use when | A real change happened on a real date | The original entry was **wrong** |
| Example | Promotion effective 1 April | Someone typed the salary as 5,000 instead of 50,000 for the row starting 1 January |

**Getting this wrong is the most common data defect in EC projects.** Two failure modes:

1. **Correcting when you should insert** — the promotion is applied retroactively to the start of the previous slice. Now the person appears to have been a Senior Analyst since 2021, and any retroactive payroll calculation is wrong.
2. **Inserting when you should correct** — the history now shows a "change" that never happened, complete with a misleading event reason, and headcount/turnover reporting counts an event that did not occur.

**Rule:** *"Did reality change on a date, or did I mistype?"* Reality changed → Insert. Mistyped → Correct.

---

## Future dating

EC handles **future-dated records** as normal business, not as a workaround:

- A transfer effective 1 October can be entered in August.
- The employee's current data stays current until the date arrives.
- The row is visible in History with a future date and can be edited or deleted before it takes effect.
- Downstream systems must be told **when** to pick it up — most replications run "as of today", so a future change flows on its effective date, not on entry date.

Things to watch:

- **Workflows** on future-dated changes: approval usually happens now, the change still takes effect later.
- **Reports** default to "as of today" — a future-dated transfer will not appear until you change the as-of date.
- **Position Management** must also be future-dated consistently, or the position and the job data disagree for a period.

---

## Retroactive changes

A change entered *after* its effective date. EC accepts them; the surrounding systems are where the pain lives.

- **Payroll** may need retro-calculation; the interface must flag that the change is retroactive.
- **Time and absence** balances may need recalculation.
- **Reports already published** for the earlier period are now out of date.
- Some customers restrict how far back non-HR users may date a change (enforced through a business rule).

Common design: allow managers a 30-day retro window; anything older goes to HR.

---

## Reading data "as of" a date

Every consumer of EC data has an as-of concept:

- **UI** — the History view; also "view as of date" on some screens.
- **Reports** — an as-of-date parameter, or a date range for period reporting.
- **OData** — `asOfDate` for a snapshot, or `fromDate`/`toDate` for all records in a period; without either, you may get only the current slice.
- **Compound Employee API** — delta and full-transmission modes with effective-dating parameters.

If a report and the UI disagree, the first question is always **"as of which date?"**

---

## Step by step — practise the mechanics

In a practice system, on one test employee:

1. Open **Job Information → History**. Note the existing rows and dates.
2. **Insert** a record effective the first of next month; change the department. Save.
3. Re-open History: you now have a new slice; the earlier slice ends the day before.
4. **Correct** the row you just created — change the department again. Save. Note that no new slice appeared.
5. Insert a row effective **last month** (retroactive) and observe what the system warns you about.
6. Look at **Compensation Information** and repeat — notice the pay component rows also slice.
7. Run a report as of today, then as of a future date, and confirm the values differ.

Doing this once beats reading about it five times.

---

## Common mistakes

- Using **Correct** to record a real change (destroys history and misleads payroll).
- Entering the effective date as *today* out of habit when the business date is the 1st of the month.
- Forgetting that **deleting a middle record** extends the previous one forward — it does not leave a gap.
- Assuming a future-dated change is "live" — it is not, and integrations will not send it until its date.
- Reporting "as of today" and then arguing with an HR user who is looking at a different date.

---

## Interview-grade Q&A

- **What is effective dating? [HIGH]** Records are valid from a start date until superseded, so the system holds full history and can answer questions as of any date.
- **Insert vs Correct? [HIGH]** Insert creates a new time slice for a genuine change; Correct overwrites the existing slice to fix a data entry error.
- **Which EC elements are not effective-dated? [HIGH]** Biographical Information, Email, Phone, National ID, Emergency Contact, Social Accounts.
- **How is the end date of a record determined? [MED]** Implicitly — the day before the next record's start date; the last record runs to the high date.
- **Can you enter a future-dated transfer? [MED]** Yes; it is standard. The change applies on its effective date, and downstream systems pick it up then.
- **What are the risks of retroactive changes? [HIGH]** Retro payroll calculation, recalculated time balances, already-published reports becoming wrong; often controlled by policy plus a rule limiting how far back non-HR users can date changes.
- **A manager says the promotion "didn't work". What do you check? [HIGH]** The effective date (is it in the future?), whether it is pending workflow approval, whether Correct was used instead of Insert, and the as-of date of whatever screen or report they are looking at.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — configuring employee data
- Video — [Employee data and effective dating walkthrough](https://www.youtube.com/watch?v=qkmFdj4h4rA)
