# Rule scenarios and contexts

## Why the scenario matters more than the logic

When you create a rule, the **first** thing you choose is the **rule scenario**. That choice decides:

- which **base object** the rule operates on,
- which **other objects** the rule can see (its parameters),
- **where** the rule can be attached,
- and which **functions and actions** are available.

Pick the wrong scenario and the rule either cannot be attached where you need it, or cannot see the data it needs. You cannot change a rule's scenario afterwards — you rebuild it. **This is the most common wasted hour for beginners.**

Analogy: choosing a scenario is like choosing the right form at a government office. Fill in the wrong form perfectly and you still have to start again.

---

## The scenario families

Exact names and availability vary by release and by which modules are enabled. The families are stable, and this is how to think about them:

| Family | Purpose | Typical base objects |
|---|---|---|
| **Basic** | General-purpose logic on a single object | Any MDF object, or an HRIS element |
| **Rules for Employee Central** (EC-specific scenarios) | Logic across employee data — job, compensation, personal | Job Information, Compensation, Employment, Personal Info |
| **Cross-entity / multi-object EC rules** | Logic needing several employee entities at once | Job Info **plus** Compensation, plus Employment |
| **Position Management** | Position-related derivation and validation | Position, Job Information |
| **Workflow-related** | Deriving workflow, approvers, or triggering routing | Job Information, MDF objects |
| **Time / Benefits / other module scenarios** | Module-specific eligibility and validation | Time Type, Benefit, Absence |
| **Object-based / MDF scenarios** | Validation and defaulting on a custom object | The MDF object |

**The practical shortcut:** in the scenario picker, look for the one whose description names the entities you need. If the requirement mentions Job Information *and* Compensation, you need a scenario that exposes both.

---

## Base object and parameters

| Concept | Meaning |
|---|---|
| **Base object** | The primary record the rule reads and writes. Usually the element the rule is attached to |
| **Parameters** | Additional objects made available to the rule so it can read (and sometimes write) them |

Worked example: *"When the job classification changes, set the pay grade from the job, and validate the salary against the pay range."*

- Base object: **Job Information** (that is where the change happens).
- Parameters needed: the **Job Classification** foundation object (to read its pay grade) and **Compensation Information** (to read the salary).
- Therefore: a scenario that exposes Job Information as base with Compensation available — not a Basic rule on Job Information alone.

If you find yourself thinking "I need one more object", that is the signal you chose the wrong scenario.

---

## Cross-entity rules

A frequent EC requirement: a change in one entity must affect another.

| Requirement | Entities involved |
|---|---|
| On promotion, set the pay grade from the new job | Job Info → Compensation |
| On termination, clear the position and set employee status | Job Info → Employment |
| On a legal entity change, derive the new pay group | Job Info → Compensation |
| When FTE changes, recalculate the annualised salary | Job Info → Compensation |

These need a **cross-entity scenario** — one that can read the changed record and write another. Attempting them with two separate single-entity rules is a common wrong turn: the second rule cannot see the first rule's in-flight values reliably, and the sequencing is fragile.

**Interview answer:** *"For logic spanning Job Information and Compensation I use a cross-entity EC rule scenario, so both entities are in scope in one rule and the values are consistent within a single save."*

---

## Rule contexts — where a rule may be used

Alongside the scenario, a rule has a **context** determining where it can be attached:

| Context | Attached at | Example |
|---|---|---|
| HRIS element | `jobInfo`, `compInfo`, `personalInfo` | Defaulting and validation on employee data |
| HRIS field | A single field | onChange logic for one field |
| MDF object | Object definition, save rules | Validation on a custom object |
| MDF field | A field on an MDF object | Conditional visibility |
| Position Management | The Position Management Settings object | Position-to-job sync behaviour |
| Workflow derivation | Workflow configuration | Choosing which workflow applies |
| Eligibility | Benefits, Time | Who is eligible for what |
| Alerts | Alert/notification objects | When to raise an alert |

---

## Choosing a scenario — a decision procedure

1. **Write the requirement as a sentence** with the entities named: *"When X changes on Job Information, set Y on Compensation, unless Z on Employment."*
2. **List every entity** the sentence mentions. That is your minimum scope.
3. **Decide where it must be attached** — which screen and which trigger.
4. **Open the scenario picker** and choose the scenario that covers all listed entities *and* can be attached where you need it.
5. **Before writing any logic**, save a stub rule and check you can attach it in the intended place. If you cannot, you chose wrong — find out now, not after an hour of building.
6. Only then build the logic.

Step 5 is the one that saves the wasted hour.

---

## Effective dating of rules

Rules are effective-dated. Two consequences worth knowing:

- You can introduce a rule **from a future date** — useful when a policy changes on 1 January and you want the configuration in place beforehand.
- When a rule runs against an effective-dated record, **the record's effective date matters**: a rule may evaluate against the values valid on that date, not today's values. This is exactly why retroactive and future-dated transactions sometimes produce results people find surprising, and why you test rules with past, present and future effective dates.

---

## Real world example

Requirement: *"When an employee transfers to a different legal entity, derive the new pay group, set the event reason to Transfer — Intercompany, and warn the user if the employee has an open work permit for the old country."*

Scenario analysis:

| Element of the requirement | Entity needed |
|---|---|
| "transfers to a different legal entity" | Job Information (old and new value of company) |
| "derive the new pay group" | Compensation Information (or Job Information, depending on where pay group sits) |
| "set the event reason" | Job Information |
| "warn if an open work permit" | Work Permit Info |

Three entities. Therefore: a **cross-entity EC rule scenario** exposing Job Information as base with Compensation and Work Permit available; attached to `jobInfo` at **onSave** (because the decision depends on the final state of the record, not on a single field change).

Had you started with a Basic rule on Job Information, you would have written half the logic before discovering you cannot read the work permit.

---

## Common mistakes

- Choosing **Basic** for something needing cross-entity access, then rebuilding.
- Not checking **attachability** before building the logic.
- Assuming you can **change the scenario** later — you cannot; you rebuild.
- Using a **cross-entity scenario for simple single-entity logic**, making the rule heavier than it needs to be.
- Forgetting that rules are **effective-dated**, then wondering why a rule "did not exist" for a back-dated transaction.

---

## Interview-grade Q&A

- **What is a rule scenario? [HIGH]** The template chosen when creating a rule; it determines the base object, the additional objects available as parameters, where the rule can be attached, and which functions are offered.
- **Can you change a rule's scenario afterwards? [HIGH]** No — you rebuild the rule. That is why you choose deliberately and verify attachability before writing the logic.
- **What is a base object? [HIGH]** The primary record the rule reads and writes, normally the element the rule is attached to.
- **What is a cross-entity rule and when do you need one? [HIGH]** A rule whose scenario exposes several employee entities at once — used when a change in one entity must read or write another, such as deriving pay grade in Compensation from a job change in Job Information.
- **How do you decide which scenario to use? [HIGH]** Write the requirement naming every entity involved, list them, decide where the rule must attach, pick the scenario covering all of them, and verify you can attach a stub rule before building.
- **Are rules effective-dated? [MED]** Yes — you can introduce a rule from a future date, and rules evaluate against the effective date of the record being processed, which matters for retroactive and future-dated transactions.
- **Where can rules be attached? [MED]** HRIS elements and fields, MDF objects and fields, Position Management settings, workflow derivation, eligibility configuration in Benefits and Time, and alert definitions.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Business rules and scenarios](https://www.youtube.com/watch?v=90aPAtJbl9g)
