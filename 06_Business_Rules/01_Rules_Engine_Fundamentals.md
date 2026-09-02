# The rules engine — fundamentals

## What a business rule is

A **business rule** is configured logic in the form:

```
IF   <condition>
THEN <action>
ELSE <action>
```

It is built in **Admin Center → Configure Business Rules**, given a name and an id, and then **attached** to the place where it should run — an HRIS element, a field, an MDF object, or a specific process.

Analogy: a rule is a **standing instruction to a very literal assistant**. "If the department is Treasury, set the cost centre to CC-2400." It will do exactly that, every time, including in the cases you did not think about. Most rule defects are not the engine misbehaving; they are the instruction being more literal than you intended.

**What rules replaced from SAP HCM:** dynamic actions, features (PINCH, ABKRS), and small user exits. If an interviewer with an HCM background asks "how do you do a dynamic action in EC?", the answer is a business rule at the right trigger point.

---

## What rules can do

| Capability | Example |
|---|---|
| **Default a value** | On hire, set probation end date = hire date + 6 months |
| **Derive a value** | Set pay group from legal entity and employee class |
| **Validate** | Salary must be within the pay range for the grade |
| **Raise a message** | Error (blocks save) or warning (informs, allows save) |
| **Control visibility** | Show a field only when another has a certain value |
| **Make a field required** | Conditionally mandatory |
| **Trigger a workflow** | Route this change for approval when the conditions match |
| **Set eligibility** | Who may enrol in a benefit; who may take an absence type |
| **Calculate** | Annualised amounts, date arithmetic, FTE-based values |

What rules **cannot** do: call an external system, loop over unlimited collections, or perform arbitrary computation. When you hit those limits, the answer is an integration or a BTP extension, not a cleverer rule.

---

## Anatomy of a rule

| Part | Meaning |
|---|---|
| **Rule ID** | The technical key. Immutable in practice — name it well |
| **Rule Name** | Human-readable |
| **Description** | *Write this.* Six months later it is the only explanation of why the rule exists |
| **Rule Scenario** | Determines the rule's purpose and the objects it can see — see [Rule scenarios](02_Rule_Scenarios_and_Contexts.md) |
| **Base Object** | The main object the rule operates on (e.g. Job Information) |
| **Parameters** | Additional objects the rule needs access to |
| **Effective start date** | Rules are effective-dated too |
| **If / Then / Else** | The logic |

---

## The rule editor

The editor builds expressions from dropdowns rather than free text. A condition is assembled as:

```
IF   [Job Information . Department]  is equal to  [Value: D-450]
AND  [Job Information . Company]     is equal to  [Value: UK01]
THEN Set [Job Information . Cost Center]  to  [Value: CC-2400]
```

Each operand is either:

- a **field** on the base object or a parameter object,
- a **literal value**,
- a **function** result,
- or the output of a **lookup** into another object.

### Operators

| Type | Operators |
|---|---|
| Comparison | equal to, not equal to, greater than, less than, greater or equal, less or equal |
| Logical | AND, OR, NOT |
| Null checks | is null / is not null (essential — see below) |
| Set | is in / is not in (against a list) |

### Common functions

Exact names vary slightly by release; the categories are stable.

| Category | Typical functions | Use |
|---|---|---|
| Date | `Date()`, `Today()`, `DateAdd`/`AddDays`/`AddMonths`, `DateDiff`, `Year()`, `Month()` | Probation end dates, seniority calculations, expiry checks |
| String | Concatenate, Substring, ToUpper, ToLower, Trim, Length | Building titles, formatting codes |
| Number | Round, Abs, arithmetic | Annualisation, FTE calculations |
| Lookup | Value lookup into another object/table | Deriving a value from a foundation object |
| Conditional | If-then-else nested inside an expression | Compact multi-branch logic |
| Message | Raise error / raise warning | Validation |
| Null handling | IsNull / IsNotNull, coalesce-style defaults | **The most important category** |

### Null handling — the source of half of all rule bugs

In EC, fields are frequently empty at the moment a rule runs — especially in `onInit` and `onChange` before the user has filled the screen in.

```
IF [Job Information . Department] is equal to "D-450"
```

If Department is null, that comparison is **not true** — and, depending on the rule and release, may behave unexpectedly in combination with other conditions.

**Habits that prevent this:**

1. Guard with a null check: `IF Department is not null AND Department is equal to …`
2. Think about the **order** in which the user fills the screen — an onChange rule on Cost Centre cannot rely on a field the user has not reached yet.
3. Test the **empty case** explicitly, not just the happy path.

---

## A first rule, end to end

**Requirement:** on hire, set the probation end date to six months after the hire date.

1. Admin Center → **Configure Business Rules** → *Create New Rule*.
2. Choose the **scenario** — a Basic rule with Job Information (or Employment Information) as the base object, depending on where the field lives.
3. Name it clearly: `EMP_DERIVE_PROBATION_END`, "Derive probation end date = hire date + 6 months".
4. Write the description — including *who asked for it and why*.
5. Logic:

   ```
   IF   [Employment Information . Probation End Date] is null
   AND  [Employment Information . Start Date]         is not null
   THEN Set [Employment Information . Probation End Date]
         to  AddMonths([Employment Information . Start Date], 6)
   ```

   Note the null guard on the target field — it stops the rule overwriting a value someone deliberately set.
6. Save.
7. **Manage Business Configuration** → `employmentInfo` → attach the rule at **onSave** (or onChange of the start date, depending on the desired behaviour).
8. **Test**: hire someone and confirm the date; then hire someone with a manually entered probation date and confirm the rule leaves it alone.
9. Document in the configuration workbook.

Steps 4, 5's null guard, and 8's negative test are what make this a professional rule rather than a liability.

---

## Rule design principles

1. **One rule, one purpose.** A rule that defaults five unrelated fields is impossible to debug or change safely.
2. **Name rules systematically.** `JOBINFO_DERIVE_COSTCENTER`, `COMP_VALIDATE_PAYRANGE`. When there are 90 rules, the naming convention is the documentation.
3. **Always write the description.**
4. **Guard against nulls.**
5. **Prefer propagation and field settings** over rules where they do the job — see [Associations and propagation](../03_Foundation_Objects/05_Associations_and_Propagation.md). Fewer rules is better.
6. **Never let two rules write the same field** at the same trigger. This is the classic "why is this value wrong?" defect.
7. **Test the negative case.** A rule that always fires is as broken as one that never does.
8. **Watch performance.** Rules run synchronously; dozens of rules with lookups on one screen make it slow.
9. **Rules are effective-dated** — you can introduce a rule from a future date, which matters for policy changes.

---

## Common mistakes

- No **null guard**, so the rule misbehaves on a half-filled screen.
- Comparing to a picklist **label** instead of its **external code**.
- Rule attached at the **wrong trigger** — onChange when it needed onSave.
- Two rules writing one field.
- A rule doing what **propagation** would do for free.
- No description, no naming convention, so nobody can maintain it.
- Only testing the happy path.
- Assuming a rule runs during **imports** — it depends on the import setting, and for migrations rules are usually off.

---

## Interview-grade Q&A

- **What is a business rule? [HIGH]** Configured If/Then/Else logic created in Configure Business Rules and attached to an HRIS element, field, MDF object or process, used to default, derive, validate, control visibility and trigger workflows.
- **What do business rules replace from SAP HCM? [HIGH]** Dynamic actions, features and small user exits — because there is no ABAP layer in the cloud.
- **What can a rule not do? [MED]** Call external systems, perform arbitrary computation or unbounded loops. Those need an integration or a BTP extension.
- **What are the main parts of a rule? [HIGH]** Rule id and name, description, scenario, base object and parameters, effective date, and the If/Then/Else logic.
- **Why is null handling important? [HIGH]** Fields are often empty when a rule runs, particularly at onInit and onChange, so comparisons behave unexpectedly; you guard with null checks and test the empty case.
- **Should you compare to a picklist label or code? [HIGH]** The external code — labels are display text, are localised and can change.
- **When would you not write a rule? [HIGH]** When propagation, a required flag, a unique flag, max length, or field criteria achieve the same result declaratively — they are cheaper, faster and need no maintenance.
- **What happens if two rules write the same field? [HIGH]** The last one in the execution sequence wins, producing results that look random; you should never configure it that way.
- **Do rules run during data imports? [MED]** It depends on the import setting; for migration loads rules are usually switched off so they do not overwrite migrated values.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business rules explained](https://www.youtube.com/watch?v=90aPAtJbl9g)
