# MDF — Interview Questions & Answers

**[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## Fundamentals

**1. What is MDF? [HIGH]**
The Metadata Framework — SuccessFactors' configuration platform for defining objects with their own fields, associations, screens, business rules and security, entirely without code.

**2. Name standard objects that are MDF objects. [HIGH]**
Position, Workflow Configuration, Department, Division, Business Unit, Cost Centre, Location, Geozone, Job Classification, Job Function, Pay Grade, Pay Range, Pay Group, Time Type, Benefit, Payment Information.

**3. Object definition vs object data? [HIGH]**
The definition is the blueprint, created in **Configure Object Definitions**. The data is the records, maintained in **Manage Data**.

**4. What does MDF give you automatically once an object is defined? [MED]**
A Manage Data screen, import/export, optional effective dating with history, an OData entity (if API-visible), business rule support, RBP integration, reporting availability, and optional workflow.

**5. When would you build an MDF object rather than add a custom field? [HIGH]**
When the thing has several attributes of its own, needs its own history or parent-child structure, needs its own screen, workflow or security, or when one employee can have several of them over time.

**6. When is MDF the wrong answer? [HIGH]**
For a single attribute (use a custom field on the relevant HRIS element), for a simple code-and-label list (use a picklist), and for high-volume transactional data or complex cross-system calculation (that belongs outside the product).

**7. What governance would you apply to MDF? [MED]**
A naming convention (`cust_` prefix), documentation of every custom object and field in the configuration workbook, a decision on effective dating at creation time, and a clean-up of experimental objects before go-live.

---

## Object definitions

**8. What are the key header settings on an object definition? [HIGH]**
Object code, label, effective dating, API visibility, secured flag, permission category, status, and menu/todo categories.

**9. What are the effective-dating options? [HIGH]**
`None` (single current version), `Basic` (the object is effective-dated with time slices), and `From Parent` (inherits the parent's dating in a composite relationship).

**10. Why must effective dating be decided up front? [HIGH]**
Turning it on later for an object that already holds data is disruptive and may not be supported.

**11. What does API visibility control? [HIGH]**
Whether the object is exposed as an OData entity — without it, no integration, Integration Center report or external system can reach the object.

**12. What standard fields does every MDF object have? [MED]**
`externalCode`, `externalName`, audit fields (`createdBy`, `createdDate`, `lastModifiedBy`, `lastModifiedDate`), a record status, and `effectiveStartDate` when effective dating is enabled.

**13. Can you add custom fields to standard MDF objects like Position? [HIGH]**
Yes. Add fields prefixed `cust_`, add them to the Configuration UI so they render, permission them, and re-test the object's standard behaviour — especially for Position, where synchronisation logic touches many fields.

---

## Fields

**14. What field types does MDF support? [HIGH]**
String, Translatable String, Number/Integer, Decimal, Date, DateTime, Boolean, Picklist, Generic Object reference, User, Attachment, Auto Number, and foundation object references.

**15. Picklist or Generic Object reference? [HIGH]**
Picklist if the value is only a code and a label. Generic Object if the referenced thing has attributes of its own.

**16. How do you reference an employee on an MDF object? [HIGH]**
With a **User** field — it validates the reference, renders a person picker, and works with permissions. Never a plain string holding a user id.

**17. Boolean or a Yes/No picklist? [MED]**
A picklist for anything a user selects: it translates, reports cleanly and can be extended to a third state. Boolean for internal flags.

**18. How do you enforce uniqueness? [MED]**
The field-level unique setting, not a business rule.

**19. Which field changes are unsafe after go-live? [HIGH]**
Changing the field identifier, changing the data type, decreasing max length, and making an existing field required without checking every write path (UI, imports, OData).

**20. Where should validation live? [HIGH]**
At the highest declarative level that works: required/unique/max-length as field settings, then field criteria, then business rules for conditional or cross-field logic.

---

## Associations

**21. What types of association does MDF support? [HIGH]**
**Valid Values** — a reference that restricts what a field may contain. **Composite** — a parent-child ownership relationship where the child cannot exist independently, is maintained through the parent's screen, usually inherits the parent's effective dating, and is deleted with the parent.

**22. What multiplicities exist? [HIGH]**
One to One and One to Many.

**23. How would you build a hierarchy in MDF? [HIGH]**
A self-referencing valid-values association from the object to itself — exactly how the Position parent-position hierarchy works.

**24. How do you model many-to-many with attributes? [MED]**
A bridge object referencing both sides, holding the relationship's own attributes (for example an Employee–Project assignment carrying a percentage).

**25. What does an association give you beyond validation? [MED]**
Filtered dropdowns, navigation between related records, propagation of attributes into other fields, and the ability for reports to traverse the relationship.

**26. The association exists but the dropdown is empty. Why? [HIGH]**
The target records do not exist or are inactive, the association was never populated on existing records, or field criteria filter on a field that is still blank.

**27. What is the risk of a composite association? [MED]**
Deleting the parent deletes the children — so use it only where the child genuinely has no independent existence.

---

## Configuration UI and Manage Data

**28. What is Manage Data? [HIGH]**
The generic Admin Center screen for creating, editing, searching and inactivating records of any MDF object, including foundation objects.

**29. What is Configuration UI? [HIGH]**
The configurable screen layout for an MDF object — which fields appear, in what order, grouped how, editable or read-only — built in Manage Configuration UI.

**30. How do you show an MDF object on People Profile? [HIGH]**
Give the object a User reference to the employee, build a Configuration UI, add a custom MDF block in Configure People Profile pointing at the object and UI, then permission the object in RBP and test as employee, manager and HR.

**31. Insert vs Correct in Manage Data? [HIGH]**
The same semantics as employee data: Insert creates a new effective-dated slice for a genuine change; Correct overwrites the current slice to fix an error.

**32. A new field does not appear on screen. Why? [HIGH]**
It was added to the object definition but not to the Configuration UI — or it is not permissioned, or its visibility is set to not visible.

---

## Import, export and migration

**33. How do you bulk-load MDF data? [HIGH]**
Import and Export Data: download the template from the target instance, populate it using external codes for associations and picklists, save as UTF-8 CSV, import incrementally, monitor the job and read the result file, then spot-check.

**34. Incremental vs full purge? [HIGH]**
Incremental inserts or updates only the rows supplied; full purge replaces the object's entire data set and is for initial loads only.

**35. How do imports interact with effective dating? [HIGH]**
A new effective start date on an existing external code creates a new time slice; the same date overwrites the existing slice — so re-running a load can silently rewrite history.

**36. How do you move an MDF object between tenants? [HIGH]**
Instance Sync where the artefact is supported; otherwise export and import the object definition and then the data (referenced objects first), and separately migrate or rebuild the Configuration UI, business rules and attachments, picklists, alerts and RBP permissions.

**37. What does NOT travel with an object definition? [HIGH]**
Typically the Configuration UI, business rules and their attachments, picklists, alerts/notifications and permission roles.

**38. Does MDF version object definitions? [MED]**
No. Records can be effective-dated, but the definition has no history — so export definitions after every change, date them, and keep them with the configuration workbook.

**39. How do you load a self-referencing hierarchy such as positions? [MED]**
Two passes: create all records first, then update the parent references, because a parent must exist before it can be referenced.

**40. How would you audit MDF data quality? [MED]**
Export to CSV and pivot: missing associations, orphaned parents, inactive records still referenced by active ones, duplicates, records outside expected date ranges.

---

## Security and picklists

**41. How is access to an MDF object controlled? [HIGH]**
The `Secured` flag on the object definition brings it under RBP; then a permission role grants view/create/edit/delete/import, assigned to a permission group and optionally scoped by target population.

**42. What does `Secured = No` mean, and when is it acceptable? [HIGH]**
The object is not permission-controlled, so anyone who can reach Manage Data can work with it. Acceptable only for trivial internal configuration objects — never for anything holding personal or business-sensitive data.

**43. Would you grant Delete in production? [MED]**
Normally no — use an active/inactive status so history and references survive.

**44. Why is import/export a sensitive permission? [MED]**
It operates in bulk and can bypass screen-level validation, so it can overwrite large volumes of data quickly.

**45. How do you filter a picklist on an MDF field? [HIGH]**
Field criteria — filtering the field's values based on another field on the same record.

**46. Why must rules and imports use external codes rather than labels? [HIGH]**
Labels are display text, are localised and can change; only the external code is stable, so anything matching on a label will eventually fail silently.

---

## Scenario questions

**47. The customer wants to track company cars. Design it. [HIGH]**
An MDF object with fields for registration, model, issue and return dates and insurance expiry; a User field for the holder; an association or picklist for the provider; effective dating on; secured with a Fleet Admin role for maintenance and view-own for employees; a Configuration UI grouped into sections; a validation rule that return date follows issue date; an alert before insurance expiry; a People Profile block; and data loaded by import from the fleet provider's file.

**48. Someone built an object with `Secured = No` holding disciplinary data. What do you do? [HIGH]**
Treat it as an audit issue: set Secured to Yes, define an appropriate permission category, create a narrowly scoped permission role, verify by Proxy testing as several user types that nobody unintended has access, then check whether the data was ever exposed and report accordingly.

**49. The object works in DEV but is empty in TEST after migration. [HIGH]**
The definition migrated but not the data, or the referenced objects and picklists did not migrate, or the RBP permissions were not re-created in TEST, or the Configuration UI was not migrated so nothing renders. Work through the "what does not travel" checklist.

**50. HR wants an approval step before a new department is created. [MED]**
MDF objects support workflow: attach a workflow configuration to the object so record creation routes for approval, and restrict direct create permission to the approving role.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF and business rules
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [MDF end to end](https://www.youtube.com/watch?v=yEsquQA-MxU)
