# Employee Data — Interview Questions & Answers

**[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## The data model

**1. Explain the Person / Employment / User structure. [HIGH]**
Person holds facts about the human being (`person_id_external`): biographical, personal, address, national ID, dependants. Employment holds one working relationship (`user_id` / assignment ID): hire date, seniority, termination. Job and Compensation Information are effective-dated children of the employment. The user record is the suite-level login identity tied to the employment.

**2. Give three cases where one person has several employments. [HIGH]**
Rehire, global assignment, concurrent employment.

**3. Which key do you use for person-level data vs employment-level data? [HIGH]**
`person_id_external` for person data; `user_id` / `assignment_id_external` for employment data.

**4. Where do you store date of birth, FTE, marital status and salary? [HIGH]**
Date of birth → Biographical (person, not effective-dated). Marital status → Personal Information (person, effective-dated). FTE → Job Information (employment, effective-dated). Salary → Compensation Information / pay components (employment, effective-dated).

**5. What is the difference between `user_id` and `username`? [MED]**
`user_id` is the internal key referenced by data, permissions and integrations; `username` is what the person types at login.

---

## Effective dating

**6. What is effective dating? [HIGH]**
Records are valid from a start date until superseded, giving full history and the ability to answer "as of" any date. End dates are implicit — the day before the next record starts.

**7. Insert vs Correct? [HIGH]**
Insert creates a new time slice for a real change on a real date. Correct overwrites the current slice to fix a data entry error. Choosing wrong either fabricates history or destroys it.

**8. Which elements are NOT effective-dated? [HIGH]**
Biographical Information, Email, Phone, National ID, Emergency Contact, Social Accounts.

**9. Can you enter a future-dated change? [HIGH]**
Yes, routinely. It applies on its effective date; reports default to "as of today" and will not show it until then; integrations pick it up on the effective date.

**10. What are the risks of retroactive changes? [HIGH]**
Retroactive payroll calculation, recalculated absence balances, already-published reports becoming wrong. Many customers limit how far back non-HR users may date changes, enforced by a rule.

**11. What happens if you delete a record in the middle of a history? [MED]**
The preceding record extends forward to cover the gap — there is no hole in the timeline.

---

## Personal and person-level data

**12. Biographical vs Personal Information? [HIGH]**
Both person-level. Biographical is not effective-dated (date of birth, gender, place of birth); Personal is effective-dated (name, marital status), so a name change creates history.

**13. Where does a name change on marriage go? [HIGH]**
A new effective-dated Personal Information record starting on the change date.

**14. What is Global Information? [HIGH]**
Country-specific person data, effective-dated, defined in the country/region-specific Succession Data Model. One person can hold it for several countries.

**15. Is the address effective-dated? Can someone have several? [HIGH]**
Yes and yes — several addresses of different types (Home, Mailing…), each with its own history.

**16. How does EC handle different address formats per country? [HIGH]**
Country-specific address configuration in the CSF Succession Data Model, with dependent picklists such as state filtered by country.

**17. Is National ID effective-dated? Can one person have several? [HIGH]**
Not effective-dated; and yes — several, across countries and card types (e.g. PAN and Aadhaar in India, or a passport ID plus an Emirates ID).

**18. How would you validate a national ID number? [MED]**
Country-specific validation where SAP ships it; otherwise a business rule checking format and length. Uniqueness across people is enforced by rule or during migration.

**19. Emergency contact vs dependant? [MED]**
Emergency contact is who to call; dependants drive benefits eligibility, some payroll allowances and documents. Different elements, often different people.

---

## Employment Information

**20. What is Employment Information? [HIGH]**
The record of one working relationship: hire date, original start date, seniority date, termination date, last date worked, primary flag, assignment type. Parent of Job and Compensation Information.

**21. Hire date vs original start date vs seniority date? [HIGH]**
Hire date starts this employment; original start date is the first ever start with the company across employments; seniority date drives entitlements and can be back-dated after an acquisition or rehire.

**22. How are contingent workers handled? [MED]**
As employments with a contingent worker type, usually without EC compensation data, carrying vendor and PO details, and excluded from headcount reporting.

**23. What happens to Employment Information on rehire? [HIGH]**
A new employment is created under the same person; original start date preserves prior service.

---

## Job Information

**24. What does Job Information contain? [HIGH]**
Organisational assignment (company, business unit, division, department, cost centre, location), job assignment (job classification, title, position, employee class), manager, working time (FTE, standard hours), employee status, and the event/event reason for the row.

**25. Where are Event and Event Reason stored? [HIGH]**
As fields on each Job Information row — EC has no separate action record.

**26. What drives employee status? [HIGH]**
The status configured on the event reason.

**27. How is the org chart built? [HIGH]**
From the manager field on Job Information, propagated to the user record; it also drives MSS target populations and workflow routing.

**28. How would you default the cost centre from the department? [HIGH]**
An onChange business rule on department, or propagation configured from the Department foundation object.

**29. What breaks when manager data is wrong? [HIGH]**
Org chart, MSS access through RBP, workflow routing, and every talent module reading the hierarchy.

**30. How do you keep job titles reportable? [MED]**
Derive the title from the job classification by rule; restrict who may override, and accept the reporting consequence if overrides are allowed.

---

## Compensation Information

**31. Describe the structure of Compensation Information. [HIGH]**
An effective-dated header (pay group, pay grade, pay range, currency, frequency, event reason) with recurring and non-recurring pay components underneath.

**32. Recurring vs non-recurring pay component? [HIGH]**
Recurring is paid each period and effective-dated (base salary, car allowance); non-recurring is a one-off with a payment date (bonus, sign-on).

**33. What is the SAP HCM equivalent of a pay component? [HIGH]**
A wage type; pay component groups aggregate them, and the payroll interface maps components to wage types.

**34. How is annual salary calculated? [HIGH]**
Amount × frequency per year, adjusted for FTE depending on whether amounts are stored as actual or full-time equivalent — a convention that must be decided and documented per customer.

**35. What is compa-ratio? [MED]**
Salary divided by the midpoint of the pay range for the employee's grade and geozone.

**36. How would you prevent a salary above the range maximum? [HIGH]**
An onSave rule comparing the amount against the pay range for the grade and geozone, raising an error (or a warning), optionally bypassed for a privileged role.

**37. How does the Compensation module write into EC? [MED]**
Publishing a compensation cycle creates effective-dated Compensation Information rows with a defined event reason; Variable Pay writes non-recurring components.

---

## Job relationships

**38. What are job relationships for? [HIGH]**
Non-line-manager relationships (HR Manager, matrix manager, custom) used for workflow routing and RBP target populations.

**39. How do you route a workflow to the employee's HRBP? [HIGH]**
Maintain an HR Manager job relationship, then use approver type *job relationship* in the workflow step.

**40. How do you avoid maintaining relationships by hand? [MED]**
Derive them with a business rule from organisational data, and report on exceptions.

---

## Imports and migration

**41. What is the correct import sequence? [HIGH]**
Foundation data → Basic User Info → Biographical → Personal/Address/Contact/National ID → Employment Info → Job History → Compensation and pay components → relationships and extras.

**42. Full purge vs incremental import? [HIGH]**
Full purge replaces the whole data set (initial loads only); incremental adds or updates the rows supplied.

**43. Should rules run during a migration load? [HIGH]**
Usually off, so defaulting rules do not overwrite migrated values — with validations enforced in the mapping instead.

**44. Name five reasons an import fails. [HIGH]**
Missing prerequisite record, date format mismatch, invalid picklist external code, effective date before hire date, duplicate effective dates, association violation, missing mandatory field, wrong key column.

**45. How does import match picklist values? [MED]**
By external code, not by label.

**46. How do you prove a migration succeeded? [HIGH]**
Reconciliation: record counts per entity, control totals such as headcount by legal entity and salary by pay group, plus a customer-signed sample of field-by-field checks.

---

## Scenario questions

**47. An employee's promotion "hasn't worked" according to their manager. Diagnose. [HIGH]**
Check the effective date (future?), whether the workflow is still pending (change sits as pending data), whether Correct was used instead of Insert, whether the manager is looking at an as-of date before the change, and whether RBP hides the field from them.

**48. A rehired employee has lost 6 years of service. What happened? [HIGH]**
A new employment was created and `original-start-date` / `seniority-date` were defaulted from the new hire date — either by a rule or by the transaction default. Fix the dates, and change the rule to respect existing values.

**49. Payroll says an employee's surname is wrong. Where do you look? [MED]**
Which element the interface maps — Biographical or Personal name fields — and whether the change was made as a Correct on an old slice rather than an Insert from the change date.

**50. HR wants to record a "works council representative" for each employee. How? [MED]**
A new job relationship type (picklist value), maintained manually or derived by rule, permissioned appropriately — not a custom field on Job Information, because the relationship points at a person and needs its own history.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [EC employee data](https://www.youtube.com/watch?v=6xrD8Vkm0QI) · [Employee data screens](https://www.youtube.com/watch?v=qkmFdj4h4rA)
