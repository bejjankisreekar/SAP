# MDF associations

Associations are how MDF objects relate to each other. There are two kinds and they do very different things — this is one of the most reliably asked MDF interview questions.

---

## The two types

| | **Valid Values association** | **Composite association** |
|---|---|---|
| Meaning | "This field's value must come from that object" | "That object is a **child** of this one and cannot exist without it" |
| Relationship | A **reference** | **Ownership** |
| Child exists alone? | Yes — the referenced record is independent | **No** — delete the parent, the children go too |
| Typical use | Department → Division; Position → Job Classification | Pay Range as a child of Pay Grade; a set of lines belonging to a header |
| Effective dating | Independent | Child usually inherits from parent (`From Parent`) |
| In the UI | A dropdown or object picker | A sub-table on the parent's screen |

Analogy: a **valid values** association is *"my employer is that company"* — the company exists whether or not I do. A **composite** association is *"these are my invoice lines"* — the lines have no meaning without the invoice, and deleting the invoice deletes them.

---

## Multiplicity

| Multiplicity | Meaning | Example |
|---|---|---|
| **One to One** | Each record links to at most one record of the other object | A location has one geozone |
| **One to Many** | One record links to many | A pay grade has many pay ranges; a business unit has many divisions |

Note that "many to many" in EC is generally achieved by modelling an intermediate object, or by allowing the same target to be selected on many source records.

---

## What an association buys you

1. **Restricted values** — the field can only contain a valid record of the target object.
2. **Filtering** — combined with field criteria, the dropdown shows only relevant records.
3. **Navigation** — from the record to its related records in Manage Data and reports.
4. **Propagation** — copying an attribute from the associated record into another field. See [Associations and propagation](../03_Foundation_Objects/05_Associations_and_Propagation.md).
5. **Validation** — invalid combinations are rejected on save.
6. **Reporting** — Story reports can traverse the association to bring in related fields.

---

## Creating a valid-values association — step by step

Requirement: a `cust_CompanyCar` must reference a `cust_CarProvider`.

1. **Configure Object Definitions** → open `cust_CompanyCar`.
2. Scroll to **Associations** → *Details* → add a new association.
3. Set:
   - **Name**: `cust_provider`
   - **Multiplicity**: `One to One` (each car has one provider)
   - **Destination Object / Type**: `cust_CarProvider`
   - **Association Type**: **Valid Values**
   - Optionally, a **label**
4. Save the definition.
5. In **Manage Configuration UI**, add the association field to the screen if it does not appear automatically.
6. **Manage Data** → create a car record; the provider now appears as a picker restricted to existing providers.
7. Optionally configure **field criteria** so the provider list is filtered further (e.g. by country).
8. Test with a valid and an invalid combination.

---

## Creating a composite association — step by step

Requirement: a `cust_CarService` record (service history) belongs to a car and has no independent existence.

1. Create the child object `cust_CarService` with **Effective Dating = From Parent**.
2. On the **parent** (`cust_CompanyCar`), add an association:
   - **Multiplicity**: `One to Many`
   - **Destination**: `cust_CarService`
   - **Association Type**: **Composite**
3. Save.
4. In **Manage Configuration UI** for the car, the service records appear as an editable **sub-table** on the car's screen.
5. Records are created *within* the parent, not independently.
6. Deleting the car deletes its service records.

**When to choose composite:** the child is meaningless alone, is always maintained through the parent's screen, and should disappear with the parent. If any of those is untrue, use a valid-values association instead.

---

## Standard EC associations you will meet

| Source | Target | Type | Effect |
|---|---|---|---|
| Department | Division | Valid values | Filters the department list by division |
| Division | Business Unit | Valid values | |
| Location | Geozone | Valid values | Enables pay-range lookup |
| Position | Job Classification | Valid values | The position's job |
| Position | Parent Position | Valid values (self-referencing) | Builds the **position hierarchy** |
| Job Classification | Job Family | Valid values | |
| Pay Range | Pay Grade + Geozone | Valid values | Keys the range |
| Workflow Configuration | Workflow steps | Composite | Steps belong to the workflow |

The **self-referencing association on Position** is worth calling out: an object can associate to itself, and that is exactly how hierarchies are built.

---

## Field criteria — the partner mechanism

An association says what is *valid*. **Field criteria** decide what is *shown* on a particular screen, based on another field's value.

Example on `cust_CompanyCar`:

- Association: provider must be a `cust_CarProvider`.
- Field criteria: show only providers whose `cust_country` equals the car's country.

Without the criteria, the dropdown shows every provider in every country and users pick the wrong one. **Configure both**, and when a dropdown misbehaves, check both.

---

## Modelling patterns

**Pattern 1 — Reference data**
Object A references object B, B is maintained independently. *Valid values, One to One.* (Car → Provider.)

**Pattern 2 — Header and lines**
Object A owns a set of B records maintained on A's screen. *Composite, One to Many, child effective-dated From Parent.* (Workflow → steps.)

**Pattern 3 — Hierarchy**
Object A references itself as parent. *Valid values, One to One, self-referencing.* (Position → parent Position; Department → parent Department in some designs.)

**Pattern 4 — Many to many via a bridge**
Objects A and B need a many-to-many link with its own attributes. Create object C referencing both. (Employee ↔ Project, with a percentage allocation on C.)

Being able to name those four patterns is a strong senior-level answer.

---

## Common mistakes

- **Using composite where valid values was meant**, so deleting a parent silently deletes independent data.
- **Association declared but records not linked** → empty dropdowns.
- **No field criteria**, so the dropdown lists everything.
- **Deep association chains** (A → B → C → D) that are slow and hard to report on.
- **Forgetting the child inherits effective dating** in a composite relationship, then being surprised by the date behaviour.
- **Building a many-to-many with two one-to-many associations** instead of a bridge object, then having nowhere to store the relationship's own attributes.
- **Adding the association but not adding it to the Configuration UI**, so nobody can set it.

---

## Interview-grade Q&A

- **What types of association does MDF support? [HIGH]** Valid Values (a reference restricting a field's values) and Composite (a parent-child ownership relationship where the child cannot exist without the parent).
- **Difference between them? [HIGH]** Valid values is a reference to an independent record; composite means ownership — the child is maintained through the parent's screen, usually inherits its effective dating, and is deleted with it.
- **What multiplicities are available? [HIGH]** One to One and One to Many.
- **How would you build a hierarchy in MDF? [HIGH]** A self-referencing valid-values association from the object to itself — exactly how Position's parent-position hierarchy works.
- **How do you model many-to-many with attributes? [MED]** Create a bridge object referencing both sides, holding the relationship's own attributes.
- **What does an association give you beyond validation? [MED]** Filtered dropdowns, navigation between records, propagation of attributes, and the ability for reports to traverse the relationship.
- **The association exists but the dropdown is empty. Why? [HIGH]** The target records do not exist or are inactive, the link on existing records was never populated, or field criteria filter on a field that is still blank.
- **What is the risk of a composite association? [MED]** Deleting the parent deletes the children — so use it only where the child genuinely has no independent existence.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF associations](https://www.youtube.com/watch?v=yEsquQA-MxU)
