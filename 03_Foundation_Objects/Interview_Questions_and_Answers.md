# Foundation Objects — Interview Questions & Answers

**[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## Concepts

**1. What is a Foundation Object? [HIGH]**
Shared structural data — legal entity, department, cost centre, location, job classification, pay grade — defined once and referenced by employee records, so structure is consistent, maintainable, and able to drive derivation and validation.

**2. Why not store the department name directly on Job Information? [HIGH]**
You would lose consistency (spelling variants), maintainability (a rename would need a mass update), derivation (no cost centre to propagate) and validation (no way to reject invalid combinations).

**3. MDF Foundation Objects vs legacy Foundation Objects? [HIGH]**
Most FOs were migrated to MDF and are defined in Configure Object Definitions, maintained in Manage Data, freely extensible with custom fields and rules. A residual set — typically Pay Component, Pay Component Group, Frequency, Event Reason and Dynamic Role — is still defined in the Corporate Data Model and maintained in Manage Organization, Pay and Job Structures. Which is which varies by release, so check whether the object appears in Manage Data.

**4. Are Foundation Objects effective-dated? [HIGH]**
Yes. Renaming, re-parenting and retiring an FO are all effective-dated changes, so reports as of a past date show the structure as it then was.

**5. What is an external code? [HIGH]**
The stable key of an FO record, used by associations, imports, integrations and reports. Design the convention up front; never change it after go-live.

**6. How do you retire a Foundation Object? [HIGH]**
Set status to Inactive with an effective date. Never delete — historical employee records reference it.

---

## Organisational structure

**7. Name the organisational Foundation Objects. [HIGH]**
Legal Entity (Company), Business Unit, Division, Department, Cost Centre, Location, Location Group, Geozone.

**8. What does Legal Entity drive? [HIGH]**
Country — and therefore country-specific fields, address format and valid national ID types — plus currency, standard hours, legal registration numbers, payroll routing and document letterhead.

**9. Are Business Unit and Division mandatory? [HIGH]**
No. They are optional grouping levels. Create a level only if someone reports on it, someone maintains it, or it drives permissions or routing.

**10. What is a Geozone for? [HIGH]**
Grouping locations for compensation purposes — pay ranges are defined per grade, geozone and currency. It is about pay markets, not geography.

**11. How is Cost Centre normally populated? [HIGH]**
Propagated or derived from the department, with an override where Finance requires it. Cost centres are frequently replicated from the ERP, in which case EC is not the master.

**12. What is Alternative Cost Distribution? [MED]**
Splitting one employee's cost across several cost centres by percentage — used for shared services and project-funded roles.

**13. Department or position hierarchy — which is the organisational structure? [HIGH]**
A design decision. Department-based where Position Management is not implemented; position-hierarchy-based where it is. Mixing the two inconsistently is the usual cause of a wrong org chart.

**14. Where do you put remote workers? [MED]**
In an explicit "Remote — country" location. Never leave the location field blank, because holiday calendars, work schedules and some tax logic depend on it.

---

## Job structure

**15. Difference between Job Classification and Position? [HIGH]**
A job classification is a *kind of job* in the catalogue. A position is a single *seat* of that job in a specific part of the organisation, normally held by at most one incumbent.

**16. What are Job Function and Job Family for? [HIGH]**
Grouping job classifications for career paths, talent pools, compensation benchmarking, reporting and recruiting routing.

**17. A customer hands you 4,000 job titles. What do you do? [HIGH]**
Explain that titles are not jobs. Rationalise into a catalogue of a few hundred job classifications — ideally reusing an existing job architecture from a compensation exercise — and keep job title as a separate field, defaulted from the job code and overridable, with the reporting consequence stated explicitly.

**18. What does a Job Classification typically propagate? [HIGH]**
Pay grade, job title, FLSA/exempt status, standard hours and job level.

**19. How do you stop users picking the wrong job code? [MED]**
Field criteria filtering the picklist by job function, country or legal entity, plus validation rules.

**20. How many job classifications is reasonable? [MED]**
A few hundred for a mid-size company. Thousands means titles have been mistaken for jobs.

---

## Pay structure

**21. What is a pay component? [HIGH]**
One element of pay — base salary, allowance, bonus — the EC equivalent of an SAP HCM wage type, defined as a foundation object and mapped to a payroll wage type.

**22. Recurring vs non-recurring pay component? [HIGH]**
Recurring is paid every period and is effective-dated; non-recurring is a one-off with a payment date.

**23. What is a pay component group? [MED]**
An aggregation of pay components used for totals such as Total Base Pay or Total Target Cash.

**24. What is a pay range keyed by? [HIGH]**
Pay grade × geozone × currency, effective-dated, holding min, mid and max. Reloaded annually as a new time slice.

**25. What is compa-ratio? [MED]**
Salary divided by the pay-range midpoint for the employee's grade and geozone.

**26. Where does pay grade come from? [HIGH]**
Normally propagated from the Job Classification rather than typed.

**27. What is a Pay Group and how is it set? [HIGH]**
The payroll run an employee belongs to. Usually derived by business rule from legal entity, country and employee class; it routes the payroll replication.

**28. Which pay objects are still legacy? [MED]**
Typically Pay Component, Pay Component Group and Frequency, maintained in Manage Organization, Pay and Job Structures — verify per instance and release.

**29. How do you prevent an out-of-range salary? [HIGH]**
An onSave business rule comparing the entered amount against the pay range for the employee's grade and geozone, raising an error or a warning depending on the user's role.

**30. What does Frequency do? [MED]**
Defines how often an amount is paid and the factor used to annualise it (monthly = 12, bi-weekly = 26, and so on).

---

## Associations and propagation

**31. What is an association? [HIGH]**
A defined relationship between two objects that restricts valid combinations, filters picklists, enables navigation and makes propagation possible.

**32. Types of association? [HIGH]**
One-to-one, one-to-many and many-to-many. In MDF terms, valid-values associations (which restrict a field's values) and composite associations (parent–child ownership, where the child cannot exist without the parent).

**33. What is propagation? [HIGH]**
Automatically copying an attribute from an associated foundation object into an employee data field — department → cost centre, job classification → pay grade, location → geozone.

**34. Propagation vs business rule? [HIGH]**
Propagation is declarative, unconditional and cheap; use it when the value is simply an attribute of the selected object. Use a business rule when there are conditions, calculations or multiple source objects. Prefer propagation where both would work.

**35. If I change a department's cost centre, do existing employees update? [HIGH]**
No. Propagation applies at the moment of selection; existing Job Information rows hold the copied value and need a mass update to change.

**36. Can a user overwrite a propagated value? [MED]**
Yes by default — propagation supplies a default, not a lock. Remove edit rights or add a validation rule if it must be fixed.

**37. What are field criteria? [MED]**
Screen-level filtering of one field's values based on another field's value — job classifications filtered by job function, national ID types filtered by country, pay ranges filtered by grade and geozone.

---

## Managing and importing

**38. What order do you load foundation data in? [HIGH]**
Parents before children: legal entity → business unit → division → department → cost centre; geozone → location; job function → family → classification; pay grade → pay range → pay group; then frequency, pay components and pay component groups; then event reasons; then positions.

**39. Incremental vs full purge import? [HIGH]**
Incremental adds or updates only the rows in the file — the normal mode. Full purge replaces the object's entire data set and is reserved for initial loads.

**40. How do association columns work in an import file? [MED]**
They carry the **external code** of the parent record, not its name. A row whose parent does not exist fails association validation.

**41. How do you move foundation configuration between tenants? [MED]**
Instance Sync where the artefact is supported; otherwise export from the source and import into the target, keeping dated exports with the configuration workbook.

**42. What data-quality checks would you run on foundation data? [MED]**
FOs missing associations, active FOs with no employees, inactive FOs still referenced by active employees, duplicate names with different codes, missing pay ranges for live grade × geozone combinations, job classifications with no pay grade.

**43. Who owns foundation data after go-live? [MED]**
A governance decision: usually HR operations for org units, Finance for cost centres, Compensation for jobs and grades — with a controlled creation process, because uncontrolled creation degrades reporting within months.

---

## Scenario questions

**44. A user selects a legal entity and the Department picklist is empty. Diagnose. [HIGH]**
The departments are not associated to that legal entity (or to the intermediate division); or a field criterion filters on a field that is still blank; or the department records are inactive or outside their effective-date range.

**45. A field on Job Information keeps showing the wrong value after save. [HIGH]**
Something is writing it twice — typically a propagation and a business rule both targeting the same field, with the last writer winning. Remove one, or make the rule conditional.

**46. The customer wants to reorganise: three divisions become two, effective 1 April. How do you do it? [HIGH]**
Foundation-object work plus an employee mass update. Effective-date the changes on the division and department records (re-parenting departments, inactivating the retired division), then mass-update Job Information for affected employees effective 1 April with an appropriate event reason. Test the resulting org chart and check workflows and RBP target populations that referenced the old structure.

**47. Payroll reports an allowance is missing for 200 people. [MED]**
Check whether the pay component exists and is active, whether it is mapped to a wage type in the interface, whether the employees actually have the recurring component row, and whether the component is included in the relevant pay component group and in the extract's field list.

**48. The customer wants a new legal entity next month. What is involved? [MED]**
Far more than creating one record: country-specific data model enablement, address and national ID configuration, pay groups and pay components, event reasons if they differ, RBP groups and target populations, workflow routing, payroll interface configuration and mapping, document templates, and testing of a full hire-to-pay cycle in that entity.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring foundation objects
- Video — [Foundation objects](https://www.youtube.com/watch?v=yEsquQA-MxU)
