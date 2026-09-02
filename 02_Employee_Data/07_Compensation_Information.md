# Compensation Information

## What it is

**Compensation Information (`compInfo`)** holds what an employee is paid. It is employment-level and effective-dated, and it has a **header plus children** structure that surprises people coming from a single "basic pay" field.

```
Compensation Information (header, effective-dated)
├── Pay Group, Pay Grade, Pay Range, currency, frequency
├── Recurring Pay Components          ← salary, car allowance, housing
│     └── amount · currency · frequency · percentage
└── Non-Recurring Pay Components      ← bonus, sign-on, one-off award
      └── amount · currency · payment date
```

Analogy: the header is the **payslip's cover** — which payroll, which grade, which currency. The pay components are the **individual lines** on it.

---

## The header fields

| Field | Meaning |
|---|---|
| **Pay Group** | Which payroll run the employee belongs to (often derived from legal entity + country) |
| **Pay Grade** | The grade in the pay structure |
| **Pay Range** | The min/mid/max for that grade in that geozone — used for compa-ratio |
| **Compa-ratio / Range penetration** | Calculated: salary vs the range midpoint |
| **Annualised salary / Annual Salary** | Calculated from components, frequency and FTE |
| **Target pay / Total target compensation** | Base plus target variable |
| **Currency** | Employment currency |
| **Pay Type / Salary Type** | Salaried vs hourly, where used |
| **Event / Event Reason** | Yes — compensation rows also carry events |

---

## Pay components — the core concept

A **Pay Component** is one element of pay. It is a **foundation object** defining, for example:

| Attribute | Example |
|---|---|
| Code / name | `BASE_SALARY`, `CAR_ALLOW`, `BONUS` |
| Type | Recurring or non-recurring |
| Value type | Amount, percentage, or number |
| Currency handling | Fixed currency or employee currency |
| Frequency | Monthly, annual, weekly, hourly |
| Payroll mapping | The wage type it maps to in the payroll system |
| Whether it counts toward totals | Included in annualised salary? In target pay? |

**Pay Component Groups** aggregate components for calculation and display — for example "Total Base Pay" = base salary + shift allowance; "Total Target Cash" = base + target bonus.

This is the direct analogue of **wage types** in SAP HCM, and the mapping between EC pay components and payroll wage types is a core deliverable in any payroll integration.

---

## Recurring vs non-recurring

| | **Recurring** | **Non-recurring** |
|---|---|---|
| Paid | Every pay period until changed or ended | Once, on a specific date |
| Effective-dated | Yes — a history of amounts | Has a payment date, not a period |
| Examples | Base salary, car allowance, housing allowance, shift premium | Annual bonus, sign-on bonus, referral award, one-off recognition |
| Typical source | Entered at hire, changed at promotion, updated by Compensation cycles | Written by Variable Pay, or entered by HR |

Interviewers like the follow-up: *"an employee gets a monthly car allowance and a one-off relocation payment — how do you store them?"* Car allowance → recurring pay component. Relocation → non-recurring pay component.

---

## Frequency, FTE and annualisation

Three numbers interact and beginners mix them up:

- **Amount** — what is entered, e.g. 5,000
- **Frequency** — per what, e.g. Monthly (12 per year)
- **FTE** — how much of a full-time job, e.g. 0.6

**Annualised amount** = amount × frequency-per-year (and, depending on configuration and whether the amount is entered as full-time or actual, adjusted by FTE).

Worked example: 5,000 monthly, frequency Monthly (12), FTE 0.5.

- If the amount stored is the **actual** amount paid: annual = 5,000 × 12 = 60,000.
- If the amount stored is the **full-time equivalent** amount: actual annual = 5,000 × 12 × 0.5 = 30,000.

**Which convention the customer uses must be decided and documented**, because compa-ratio, compensation planning and the payroll interface all depend on it. Getting this wrong is a genuine production incident, not a theoretical one.

---

## Pay Grade, Pay Range and compa-ratio

- **Pay Grade** — a level in the structure (G1…G12, or "Senior Professional").
- **Pay Range** — min / mid / max for a grade, usually **per geozone and per currency**, because London and Warsaw pay differently for the same grade.
- **Compa-ratio** = salary ÷ range midpoint. 1.0 means paid at midpoint.
- **Range penetration** = (salary − min) ÷ (max − min).

A frequent requirement: **validate on save** that the salary sits within the pay range for the grade, warning or blocking outside it. That is a business rule, and it is a favourite practical exercise. See [Common EC rule recipes](../06_Business_Rules/05_Common_EC_Rule_Recipes.md).

---

## Who writes compensation data

| Source | What it writes |
|---|---|
| HR / manager via Change Job and Compensation | Ad-hoc changes, promotions |
| **Compensation module** (merit cycle) | Publishes new base salary as of the cycle's effective date |
| **Variable Pay** | Publishes bonus as a non-recurring component |
| **Data migration** | Initial load |
| **Integrations** | Rare, but exists (e.g. union-agreed uplifts) |

When Compensation "publishes" results into EC, it creates effective-dated Compensation Information rows with a specific event reason. Knowing that pathway exists is a good senior-level answer.

---

## Step by step — configure and test

1. **Create pay components** in Manage Organization, Pay and Job Structures (legacy FO) — code, type, frequency, currency, whether included in totals.
2. **Create pay component groups** for the totals the customer needs.
3. **Create pay grades and pay ranges**, per geozone and currency.
4. **Manage Business Configuration → `compInfo`** — set field visibility, labels, required flags.
5. Attach rules: default pay group from legal entity; derive pay grade from job classification; validate salary against pay range.
6. **Permission** the block tightly — compensation is the most sensitive data after national ID.
7. Test: hire with a salary, run a promotion with an increase, add a one-off bonus, and check history and annualisation.
8. Check the **payroll interface mapping** for each new pay component. A pay component with no wage type mapping is invisible to payroll.

---

## Real world example

A company implements EC with monthly-paid staff in Germany and hourly-paid staff in the US.

- Germany: `BASE_SALARY` recurring, frequency Monthly, entered as FTE amount; a 13th-month payment as a separate recurring component with its own payroll mapping.
- US: `HOURLY_RATE` recurring with frequency Hourly; annualisation uses standard hours from Job Information.
- Shift premium is a recurring percentage component, calculated on base.
- Annual bonus arrives from Variable Pay as a non-recurring component each March.
- A rule blocks saving a salary above the pay-range maximum unless the user holds a specific permission; otherwise it warns.

The two frequency conventions coexisting in one instance is what makes the annualisation question non-trivial — and it is exactly the sort of detail an experienced interviewer probes.

---

## Interview-grade Q&A

- **What is Compensation Information? [HIGH]** The employment-level, effective-dated element holding pay group, pay grade, currency and frequency, with recurring and non-recurring pay components underneath.
- **Recurring vs non-recurring pay component? [HIGH]** Recurring is paid every period and is effective-dated; non-recurring is a one-off with a payment date.
- **What is the SAP HCM equivalent of a pay component? [HIGH]** A wage type. Pay component groups aggregate them, and the interface maps components to wage types.
- **How is annual salary calculated? [HIGH]** Amount × frequency per year, adjusted for FTE depending on whether amounts are stored as actual or full-time equivalent — a convention that must be defined and documented.
- **What is compa-ratio? [MED]** Salary divided by the midpoint of the pay range for the employee's grade and geozone.
- **How do you stop someone entering a salary outside the range? [HIGH]** An onSave business rule comparing the amount to the pay range, raising an error or a warning; optionally bypassed for privileged roles.
- **How does the Compensation module write back to EC? [MED]** Publishing a compensation cycle creates effective-dated Compensation Information rows with a defined event reason; variable pay writes non-recurring components.
- **Where does pay group come from? [MED]** Usually derived by rule from legal entity and country; it determines the payroll run.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [Compensation information walkthrough](https://www.youtube.com/watch?v=6xrD8Vkm0QI)
