# MDF fundamentals

## What MDF is

The **Metadata Framework (MDF)** is SuccessFactors' built-in platform for **defining your own data objects** — their fields, keys, relationships, screens, validations and security — entirely through configuration.

Analogy: MDF is a **database table designer, a form builder and a rules engine in one**, for people who cannot write code. In SAP HCM terms, it does in an afternoon what used to take a Z-table, a maintenance view, a screen and an authorisation object.

The core insight: **MDF objects are not "custom add-ons"**. Enormous parts of standard SuccessFactors *are* MDF objects. Position, Workflow Configuration, Department, Location, Job Classification, Pay Grade, Benefit, Time Type — all MDF. Learning MDF is therefore not optional, and it is not "the advanced topic". It is the fabric.

---

## What MDF replaced

| Before MDF | With MDF |
|---|---|
| Foundation objects defined only in the Corporate Data Model XML, extensible only by SAP-provided placeholders | Objects defined in a UI, extensible with as many fields as you need |
| No way to build a new object without a product enhancement | Build your own object in an hour |
| Screens fixed by the product | Configuration UI: you design the layout |
| Validation only in code | Business rules attached to objects and fields |
| Security baked into the product | Object-level security you define, enforced through RBP |

---

## The vocabulary

| Term | Meaning |
|---|---|
| **Generic Object (GO)** | An MDF object — the general term |
| **Object Definition** | The *design*: fields, keys, associations, security, effective dating. Created in **Configure Object Definitions** |
| **Object / record / data** | An *instance* of the definition — one department, one position. Maintained in **Manage Data** |
| **Configuration UI** | The screen layout for an object, built in **Manage Configuration UI** |
| **MDF picklist** | A picklist managed in Picklist Center, usable by MDF fields |
| **Composite association** | A parent-child relationship where the child cannot exist alone |
| **Valid values association** | A relationship that restricts what a field may contain |

**The distinction to keep straight:** *Object Definition = the blueprint. Manage Data = the building.* Confusing them is the most common beginner mistake ("I created the object but there's no data" — because you created the definition).

---

## Where MDF already lives in your instance

Open **Configure Object Definitions** in any EC instance and you will find dozens of standard objects. The important ones:

| Area | MDF objects |
|---|---|
| Organisation | Legal Entity, Business Unit, Division, Department, Cost Centre, Location, Location Group, Geozone |
| Job & pay | Job Function, Job Classification, Pay Grade, Pay Range, Pay Group |
| Position Management | **Position**, Position Management Settings |
| Process | **Workflow Configuration**, Workflow Group, Dynamic Role, Alert/notification objects |
| Time | Time Type, Time Profile, Holiday Calendar, Work Schedule, Absence-related objects |
| Benefits | Benefit, Benefit Enrolment, Benefit Program — Benefits is built almost entirely on MDF |
| Payments | Payment Information |

Practical consequence: **learning MDF gives you leverage over half the product**, not just over custom objects.

---

## When to use MDF — and when not to

This is a design judgement interviewers probe, because the wrong answer creates permanent mess.

| Requirement | Solution |
|---|---|
| One extra attribute on the employee, effective-dated with the job | **Custom field on `jobInfo`** — not MDF |
| A simple list of allowed values | **Picklist** — not MDF |
| A thing with several attributes of its own, referenced by employees | **MDF object** |
| A thing with its own history, parent-child structure, or its own screen | **MDF object** |
| Records that need their own approval workflow | **MDF object** (MDF supports workflows) |
| Complex calculations across many entities | Usually **outside** the product — a BTP extension or a reporting layer |

**Worked judgement:** *"Track company cars issued to employees — registration, model, issue date, return date, insurance expiry."*

- Not a custom field: there are six attributes, and one employee may have several cars over time.
- Not a picklist: cars have attributes.
- **An MDF object** `Company_Car`, with an association to the employee, its own Configuration UI, an alert on insurance expiry, and RBP security for Fleet Admin. Correct answer.

Contrast: *"Record whether an employee has signed the code of conduct this year."* That is **one field**, not an object. Building an MDF object for it is over-engineering, and you will be maintaining a screen and a permission role for a checkbox.

---

## What MDF gives you out of the box

Once you define an object, you get — without further work:

- A **Manage Data** screen for creating, editing and searching records
- **Effective dating** with full history, if you enable it
- **Import and export** through Import and Export Data
- An **OData entity** (if API visibility is enabled), so integrations can read and write it
- **Business rule** support — validation, defaulting, and rules that run on save
- **RBP integration** — object-level permissions in permission roles
- **Reporting** availability in Story reports
- Optional **workflow** on record changes

That list is why MDF is powerful: one definition, and the whole platform picks it up.

---

## Limits and cautions

- **Performance.** Objects with very large record counts, deep association chains and many rules can be slow. Position and Location are fine; a custom object with millions of rows deserves design thought.
- **Not a general-purpose database.** MDF is for HR-shaped configuration and master data, not for transactional volumes.
- **Governance.** MDF makes it *easy* to create objects. Instances accumulate abandoned objects created "just to try". Name them properly (`cust_` prefix), document them, and delete unused ones before go-live.
- **Effective dating is a decision at creation time.** Enabling it later on an object with existing data is disruptive — decide up front.
- **Deleting an object definition with data** is destructive and often blocked. Design deliberately.

---

## Real world example

A hospital group needs to track **professional registrations** — nursing council numbers, medical licences, expiry dates, issuing bodies — with alerts before expiry, visible to HR and the compliance team, and reportable.

Why an MDF object:

| Requirement | MDF feature |
|---|---|
| Multiple registrations per employee | Object with an association to the employee, multiple records per person |
| Registration body, number, issue and expiry dates, status | Fields of appropriate types |
| Only valid bodies may be chosen | MDF picklist, or an association to a `Registration_Body` object |
| Alert 90 days before expiry | Alerts/notifications on the object |
| Compliance team maintains, managers view | Object-level security + RBP roles |
| Must be reportable and extractable | OData visibility + Story reports + Integration Center |

One object definition, one Configuration UI, one picklist, one alert rule, two permission roles. No code. That is the case for MDF in a sentence.

---

## Interview-grade Q&A

- **What is MDF? [HIGH]** The Metadata Framework — SuccessFactors' configuration platform for defining objects with their own fields, associations, screens, rules and security, without code.
- **Give five standard objects that are MDF objects. [HIGH]** Position, Workflow Configuration, Department, Location, Job Classification, Pay Grade, Time Type, Benefit — most foundation objects migrated to MDF.
- **Object definition vs object data? [HIGH]** The definition is the blueprint, created in Configure Object Definitions. The data is the records, maintained in Manage Data.
- **When would you build an MDF object rather than add a custom field? [HIGH]** When the thing has several attributes of its own, needs its own history or parent-child structure, needs its own screen, workflow or security, or when an employee can have several of them.
- **When is MDF the wrong answer? [HIGH]** For a single attribute (use a custom field), for a simple list of values (use a picklist), or for high-volume transactional data and complex cross-entity calculation (that belongs outside the product).
- **What do you get automatically when you define an MDF object? [MED]** A Manage Data UI, import/export, optional effective dating, an OData entity, business rule support, RBP integration, reporting availability and optional workflow.
- **What replaced the Corporate Data Model for foundation objects? [MED]** MDF — most foundation objects are now object definitions maintained in Manage Data rather than XML elements.
- **What governance would you put around MDF? [MED]** A naming convention, documentation of every custom object and field in the configuration workbook, a review before go-live to remove experiments, and a decision on effective dating at creation time.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF and business rules
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF concepts](https://www.youtube.com/watch?v=yEsquQA-MxU)
