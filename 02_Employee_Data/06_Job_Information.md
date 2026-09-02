# Job Information — the most important element in EC

## Why this element matters most

**Job Information (`jobInfo`)** answers *"what is this person doing, where, for whom, and under what terms — as of any date?"* It is:

- the element with the **most fields**
- the one carrying **Event and Event Reason**, so it also *is* the transaction history
- the one that drives **payroll, position management, permissions, org charts and most reports**
- the one that almost every business rule you will ever write touches

If Job Information is designed badly, everything downstream is wrong. Interviewers know this, which is why they ask about it more than any other element.

---

## The field groups

### 1. Organisational assignment

| Field | Meaning | Notes |
|---|---|---|
| **Company / Legal Entity** | Which legal employer | Drives country, currency, and country-specific fields |
| **Business Unit** | Top-level business grouping | |
| **Division** | Next level down | |
| **Department** | Where they sit organisationally | Often the level users care about |
| **Cost Centre** | Who pays | May come from Finance; can be split (alternative cost distribution) |
| **Location** | Physical work location | Drives holiday calendar, sometimes tax |
| **Geozone** | Grouping of locations for compensation | Used by pay ranges |

The relationships between these are enforced by **associations** — see [Associations and propagation](../03_Foundation_Objects/05_Associations_and_Propagation.md).

### 2. Job assignment

| Field | Meaning |
|---|---|
| **Job Classification** (job code) | The catalogue job — "Senior Analyst" |
| **Job Title** | The displayed title; may differ from the job code's name |
| **Position** | The position held (if Position Management is on) |
| **Job Function / Job Family** | Grouping for career paths and reporting |
| **Employee Class** | Permanent, fixed-term, apprentice… |
| **Employment Type** | Full-time, part-time |
| **Regular/Temporary** | Contract nature |

### 3. Reporting and management

| Field | Meaning |
|---|---|
| **Manager** (supervisor) | The line manager — drives the org chart, MSS and RBP target populations |
| **Second Manager / Matrix Manager** | Where used; matrix relationships usually live in Job Relationships |
| **HR Manager** | HRBP responsible — used for workflow routing and permissions |

### 4. Working time and status

| Field | Meaning |
|---|---|
| **FTE** | Full-time equivalent, e.g. 0.6 |
| **Standard Hours** | Contracted hours per week |
| **Working Days per Week** | |
| **Employee Status** | Active, Terminated, Suspended, Unpaid Leave, Retired… — driven by the event reason |
| **Time Type / Work Schedule** | If Time Off is implemented |
| **Notice period, probation** | Sometimes here, sometimes on Employment Information |

### 5. The transaction fields

| Field | Meaning |
|---|---|
| **Event** | The category of change: Hire, Job Change, Transfer, Promotion, Termination, Data Change, Leave of Absence, Return to Work… |
| **Event Reason** | The specific reason within the event: "Promotion — Merit", "Termination — Resignation" |
| **Effective date** | When this row starts |
| **Position entry date, time in position** | Derived dates used for reporting |

**Every Job Information row carries an event and event reason.** That is how EC records "what happened" without a separate action record. Full treatment: [Events and Event Reasons](../07_Transactions_and_Workflows/01_Events_and_Event_Reasons.md).

### 6. Payroll-relevant fields

| Field | Meaning |
|---|---|
| **Pay Group** | Which payroll run the employee belongs to |
| **Payroll system / country grouping** | For replication routing |
| **Custom country-specific job fields** | From the country-specific data model |

---

## How Job Information behaves

- **Fully effective-dated.** Every change creates a new row (or corrects an existing one).
- **One row per effective date per employment.** You cannot have two rows for the same day.
- **Derived by rules more than typed.** In a mature design, the user picks a position or a job code, and rules derive department, job title, pay grade, standard hours, FTE and event reason.
- **Position-driven where Position Management is on.** Selecting the position populates a set of fields, either as a default the user may override or as a locked synchronisation. See [Sync between Position and Job Info](../09_Position_Management/04_Sync_Between_Position_and_Job_Info.md).
- **Employee status is not free-form.** It is driven by the event reason's configured status, which is how a termination reason makes someone Terminated.

---

## The manager field, and why it is special

The **manager** on Job Information:

- draws the **org chart**
- determines the **MSS target population** in RBP ("manager hierarchy")
- routes most **workflows**
- is copied to the suite-level user record so talent modules see the same hierarchy

Common problems:

- **Circular hierarchies** — A reports to B reports to A. EC will usually block obvious cases; complex loops appear after a bulk load.
- **The "no manager" case** — the CEO. Every implementation needs a rule for the top of the hierarchy.
- **Manager change without effective date discipline** — a re-org dated inconsistently across a hundred employees produces a hierarchy that is broken for a few days.
- **Position-driven manager** — if positions drive the hierarchy, the manager field should come from the parent position, not be typed.

---

## Worked example — one transfer

Karan moves from Sales to Marketing on 1 September, keeping his job code, changing department, manager and cost centre.

**Before:**

| Eff. date | Dept | Manager | Cost centre | Event / Reason |
|---|---|---|---|---|
| 01-Mar-2023 | Sales | R. Iyer | CC-1100 | Hire / New Hire |

**Action:** Take Action → Change Job and Compensation → effective 01-Sep-2025 → Department = Marketing.

**Rules fire:** onChange of Department derives the new cost centre from the department; an onSave rule derives Event = Transfer and Event Reason = Internal Transfer; manager is derived from the target position.

**After:**

| Eff. date | Dept | Manager | Cost centre | Event / Reason |
|---|---|---|---|---|
| 01-Mar-2023 | Sales | R. Iyer | CC-1100 | Hire / New Hire |
| 01-Sep-2025 | Marketing | S. Devi | CC-2400 | Transfer / Internal Transfer |

The March row is untouched. Reporting as of August still shows Sales.

---

## Step by step — configuring Job Information

1. **Manage Business Configuration** → HRIS Elements → `jobInfo`.
2. Review every field: visibility (`both`/`view`/`none`), required, label, picklist, default.
3. Hide what the customer does not use — a screen with 60 fields is a training problem and a data-quality problem.
4. Attach rules:
   - **onInit** to default values when the screen opens
   - **onChange** to derive dependent fields (department → cost centre)
   - **onSave** to derive event reason and to validate
5. Configure **field criteria** so picklists filter correctly (job classifications filtered by job function, for example).
6. Add country-specific fields via the country-specific model.
7. Permission the block and its fields in **RBP** — separately for ESS, MSS and HR.
8. Test each transaction type, and test **history** (insert and correct).

---

## Design decisions you will actually be asked to make

| Question | Considerations |
|---|---|
| Does the customer use Position Management? | If yes, most of Job Information becomes derived, and the design centres on the position |
| Is department the org unit, or is the position hierarchy the org? | Both are used; mixing them inconsistently is the usual mess |
| Which fields may a manager change, and which are HR-only? | Field-level RBP plus workflow design |
| How many event reasons? | Enough to report meaningfully, few enough for users to choose correctly — usually 25–60 |
| Cost centre from department, or entered per employee? | Derive by default, allow override where finance requires it |
| Job title free text or from the job code? | Free text pleases the business and ruins reporting; a common compromise is derive-then-allow-override with a rule |

---

## Interview-grade Q&A

- **What is Job Information? [HIGH]** The effective-dated employment-level element holding organisational assignment, job assignment, manager, working time, employee status and the event/event reason for each change.
- **Where are Event and Event Reason stored? [HIGH]** As fields on each Job Information row — EC has no separate action record.
- **How does the org chart get built? [HIGH]** From the manager field on Job Information, propagated to the user record.
- **What drives employee status? [HIGH]** The event reason's configured status (Active, Terminated, Unpaid Leave, and so on).
- **How would you default the cost centre? [HIGH]** An onChange business rule deriving cost centre from department, or propagation from the Department foundation object.
- **What is FTE used for? [MED]** Part-time calculation, headcount reporting, salary annualisation, and position FTE control.
- **How do you stop users typing an arbitrary job title? [MED]** Derive it from the Job Classification by rule; restrict edit rights, or accept override with a documented reporting consequence.
- **What breaks if manager data is wrong? [HIGH]** Org chart, MSS access via RBP target populations, workflow routing, and every talent module that reads the hierarchy.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — configuring employee data
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Job Information walkthrough](https://www.youtube.com/watch?v=qkmFdj4h4rA)
