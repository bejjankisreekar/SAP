# Common EC rule recipes

The dozen rules almost every Employee Central project builds. Each is written as: the requirement, the design decisions, and the logic in readable pseudo-code.

> The editor builds these from dropdowns; the pseudo-code below shows the *shape*, not literal syntax. Field and function names vary by release.

---

## 1. Derive the pay grade from the job classification

**Requirement:** the user picks a job; the grade must follow.

- **Scenario:** cross-entity (Job Information + Compensation), or propagation if the grade is a plain attribute
- **Trigger:** onSave on `jobInfo` (authoritative), optionally onChange for immediate feedback
- **First ask:** can [propagation](../03_Foundation_Objects/05_Associations_and_Propagation.md) do this? If the pay grade is simply an attribute of the job classification, propagate it and write no rule.

```
IF   Job Information.Job Classification is not null
THEN Set Compensation.Pay Grade
      to  Job Classification.Pay Grade
```

---

## 2. Validate the salary against the pay range

**Requirement:** block a salary outside the range for the employee's grade and geozone; warn near the top.

- **Scenario:** cross-entity (Compensation + Job Information + Pay Range)
- **Trigger:** onSave on `compInfo`
- **Design:** error for most users, warning for HR — implemented either as two rules with different conditions, or by checking the user's role where the scenario supports it

```
Lookup PayRange WHERE Pay Grade = Compensation.Pay Grade
                 AND  Geozone   = Job Information.Geozone
                 AND  Currency  = Compensation.Currency

IF   Annualised Salary > PayRange.Max
THEN Raise Error "Salary {x} exceeds the maximum {max} for grade {grade} in {geozone}."

ELSE IF Annualised Salary < PayRange.Min
THEN Raise Error "Salary {x} is below the minimum {min} for grade {grade} in {geozone}."

ELSE IF Annualised Salary > (PayRange.Mid + ((PayRange.Max - PayRange.Mid) * 0.8))
THEN Raise Warning "Salary is in the top of the range. HR approval may be required."
```

**Note the message text.** A message naming the actual numbers turns a support ticket into a self-service correction.

---

## 3. Derive the cost centre from the department

- **Scenario:** Basic / EC rule on Job Information
- **Trigger:** onChange of Department
- **First ask:** propagation again — this is the textbook propagation case. Write the rule only if the derivation is conditional (e.g. different logic per country).

```
IF   Job Information.Department is not null
AND  Job Information.Cost Center is null
THEN Set Job Information.Cost Center to Department.Cost Center
```

The `Cost Center is null` guard preserves a deliberate override. Drop it if the cost centre must always follow the department.

---

## 4. Derive the pay group

**Requirement:** route the employee to the right payroll.

- **Trigger:** onSave on `jobInfo`
- **Design:** derive from legal entity + employee class; never let users type it

```
IF   Company = "UK01"
THEN Set Pay Group = "UK-MTH"

ELSE IF Company = "US01" AND FLSA Status = "EXEMPT"
THEN Set Pay Group = "US-BIWK-EX"

ELSE IF Company = "US01" AND FLSA Status = "NONEXEMPT"
THEN Set Pay Group = "US-BIWK-NONEX"

ELSE IF Company = "DE01"
THEN Set Pay Group = "DE-MTH"
```

For more than a handful of branches, use a **lookup into a configuration object** rather than a long If/Else chain — a `cust_PayGroupMapping` object with company, employee class and pay group is far easier to maintain than a rule with 40 branches, and HR can maintain it.

---

## 5. Default the probation end date

- **Trigger:** onInit or onChange of the hire date; onSave as backstop
- **Design:** period differs by country

```
IF   Employment.Probation End Date is null
AND  Employment.Start Date is not null
THEN
    IF   Company.Country = "DE" THEN Set Probation End = AddMonths(Start Date, 6)
    ELSE IF Company.Country = "GB" THEN Set Probation End = AddMonths(Start Date, 3)
    ELSE Set Probation End = AddMonths(Start Date, 6)
```

---

## 6. Default the seniority date — and protect it

**The migration trap** from [Employment Information](../02_Employee_Data/05_Employment_Information.md).

```
IF   Employment.Seniority Date is null
AND  Employment.Start Date is not null
THEN Set Employment.Seniority Date = Employment.Start Date
```

The null guard is the whole point: without it, a migration carrying real seniority dates has them overwritten with the hire date, and 300 acquired employees lose years of service.

---

## 7. Default and validate FTE / standard hours

```
IF   Job Information.FTE is null AND Job Information.Employment Type = "FULL_TIME"
THEN Set FTE = 1.0

IF   Job Information.FTE > 1.0 OR Job Information.FTE <= 0
THEN Raise Error "FTE must be greater than 0 and no more than 1.0."

IF   Job Information.Standard Hours is null
THEN Set Standard Hours = Company.Standard Hours * FTE
```

---

## 8. Default the job title from the job classification

```
IF   Job Information.Job Title is null
AND  Job Information.Job Classification is not null
THEN Set Job Title = Job Classification.Name
```

Attached at **onChange** of Job Classification, with a null guard so a deliberate custom title survives. See the design trade-off in [Job structure FOs](../03_Foundation_Objects/03_Job_Structure_FOs.md).

---

## 9. Prevent a termination date before the hire date

```
IF   Employment.End Date is not null
AND  Employment.Start Date is not null
AND  Employment.End Date < Employment.Start Date
THEN Raise Error "The termination date cannot be before the hire date."
```

Trivial, and it catches real typos every month.

---

## 10. Limit retroactive dating for non-HR users

**Requirement:** managers may not back-date a change more than 30 days.

```
IF   Job Information.Effective Date < AddDays(Today(), -30)
AND  <user is not in the HR role>
THEN Raise Error "Changes cannot be dated more than 30 days in the past. Please contact HR."
```

The role check depends on what the scenario exposes; where it is not available, the common alternative is to permit the field only for HR roles and to raise a warning for everyone else.

---

## 11. Conditional visibility

**Requirement:** show "Reason for leaving — other" only when the termination reason is "Other".

- **Trigger:** onChange of the reason field
- **Effect:** set the target field's visibility (and required flag) from the rule

```
IF   Job Information.Event Reason = "TERM_OTHER"
THEN Set Field Visibility of Reason Detail = Editable
     Set Field Required   of Reason Detail = true
ELSE Set Field Visibility of Reason Detail = Hidden
```

---

## 12. Derive the workflow

```
IF   Event Reason is in ("PROMOTION", "TRANSFER_INTERCOMPANY")
THEN Set Workflow = "WF_HRBP_COMP_APPROVAL"

ELSE IF Event Reason = "DATA_CHANGE"
THEN Set Workflow = <none>          -- applies immediately

ELSE Set Workflow = "WF_MANAGER_APPROVAL"
```

Attached at onSave, after event-reason derivation in the sequence. See [Attaching rules](04_Attaching_Rules.md).

---

## 13. Alert before an expiry date

**Requirement:** notify HR 90 days before a work permit expires.

Implemented as an **alert/notification** with a rule condition rather than a screen rule:

```
IF   Work Permit.Valid To is not null
AND  DateDiff(Work Permit.Valid To, Today()) <= 90
THEN Raise alert to <HR role / job relationship>
```

Alerts run as a scheduled job, not on save — which is precisely why an onSave rule would not solve this requirement.

---

## Design patterns worth stealing

**Use a configuration object instead of a long If/Else.**
Any mapping with more than about five branches — pay group, event reason, probation period, workflow — should be a small MDF object the customer can maintain, with the rule doing a lookup. Fewer rule changes, no consultant needed for a new country.

**Null-guard every default.**
`IF <target> is null THEN set it` is the difference between a helpful default and a rule that destroys migrated data.

**Put correctness at onSave, convenience at onChange.**
onChange makes the screen feel intelligent; onSave makes the data right.

**Write messages that name the values.**
"Salary 92,000 exceeds maximum 81,000 for G7 in UK-LONDON" instead of "Validation error".

**One rule, one purpose, named systematically.**
`JOBINFO_DERIVE_PAYGROUP`, `COMP_VALIDATE_PAYRANGE`. With 90 rules, the naming convention is the documentation.

---

## Interview-grade Q&A

- **Give three rules you have built in EC. [HIGH]** Event reason derivation on save; pay-range validation raising an error with the actual numbers; pay group derivation from legal entity and employee class.
- **How would you validate a salary against the pay range? [HIGH]** A cross-entity onSave rule on Compensation that looks up the pay range by grade, geozone and currency and raises an error outside min/max — with a warning band near the top and a role-based exception where required.
- **How do you default the probation end date? [HIGH]** An onInit or onChange rule adding the country-specific period to the hire date, guarded so it only fires when the field is null.
- **Why is the null guard important on a defaulting rule? [HIGH]** Without it, the rule overwrites values that were deliberately set — most damagingly during data migration, where real seniority dates get replaced by hire dates.
- **How would you handle a pay-group mapping with 40 combinations? [HIGH]** A configuration object holding the mapping, with a lookup rule — not a 40-branch If/Else, so the customer can maintain it without a consultant.
- **How do you make a field conditionally mandatory? [MED]** An onChange rule setting the field's required flag and visibility based on another field's value.
- **How would you alert HR 90 days before a work permit expires? [HIGH]** An alert/notification with a rule condition, executed by a scheduled job — not an onSave rule, because nothing is being saved at that moment.
- **How do you limit back-dating for managers? [MED]** An onSave rule comparing the effective date to today minus the allowed window and raising an error, combined with RBP so HR retains the ability.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business rules in practice](https://www.youtube.com/watch?v=90aPAtJbl9g)
