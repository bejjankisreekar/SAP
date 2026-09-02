# Data Models — Interview Questions & Answers

**[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## The landscape

**1. Name the data models in SuccessFactors. [HIGH]**
Corporate Data Model (CDM), Country/Region-Specific Corporate Data Model (CSF-CDM), Succession Data Model (SDM), Country/Region-Specific Succession Data Model (CSF-SDM).

**2. What does each govern? [HIGH]**
CDM: the organisation — legacy foundation object elements, fields and associations. SDM: the employee — standard/user elements, EC HRIS elements, talent background elements. The CSF versions add country-specific fields to each.

**3. Why is it called the "Succession" Data Model? [MED]**
Historical accident. The file predates Employee Central and originally defined the talent profile. It has nothing to do with succession planning.

**4. What is in a modern Corporate Data Model? [HIGH]**
Much less than historically. Most org, job and pay foundation objects migrated to MDF and are now defined in Configure Object Definitions. What typically remains is pay component, pay component group, frequency, event reason and dynamic role.

**5. Which model would you open to add a field to Department? [HIGH]**
None of them — Department is an MDF object, so **Configure Object Definitions**.

**6. Where do you add a field for French employees only? [HIGH]**
The country-specific Succession Data Model for France, on the appropriate HRIS element.

---

## Elements and fields

**7. What is an `hris-element`? An `hris-field`? [HIGH]**
An element is a block of data — an object or a section of the employee record. A field is one attribute inside it, with an id, label, visibility and required flag.

**8. What values can `visibility` take? [HIGH]**
`both` (view and edit), `view`, `edit`, `none`.

**9. Difference between `visibility="none"` and an RBP restriction? [HIGH]**
`visibility` hides the field from everyone, globally. RBP hides it from specific roles. Use RBP whenever visibility depends on who is looking.

**10. What are the risks of `required="true"`? [HIGH]**
Every write path must supply the field — the UI, import templates, OData integrations, Recruiting-to-EC and Onboarding hires. Interfaces that do not send it start failing, often silently at first.

**11. How do you add a custom field? [HIGH]**
Claim an unused custom placeholder (`custom-string1`, `custom-date2`…) on the element, set label, visibility, required, max length and picklist — in BCUI — then permission it in RBP, test the UI, the import template and OData, and record the placeholder's meaning in the configuration workbook.

**12. Why must you never reuse a custom placeholder that held data? [HIGH]**
The existing values remain in the database and are silently re-labelled as the new field's meaning — a defect that is very hard to detect afterwards.

**13. Why must you never change a field `id`? [HIGH]**
Business rules, import templates, integrations and reports all reference it. Changing it breaks all of them.

**14. List the person-level HRIS elements. [HIGH]**
`personInfo`, `personalInfo`, `globalInfo`, `homeAddress`, `emailInfo`, `phoneInfo`, `nationalIdCard`, `emergencyContactPrimary`, `personRelationshipInfo`, `imInfo`, `socialAccountInformation`.

**15. List the employment-level HRIS elements. [HIGH]**
`employmentInfo`, `jobInfo`, `compInfo`, `payComponentRecurring`, `payComponentNonRecurring`, `jobRelationsInfo`, `workPermitInfo`, `globalAssignmentInfo`.

**16. Which of those are effective-dated? [HIGH]**
`personalInfo`, `globalInfo`, `homeAddress`, `jobInfo`, `compInfo`, `payComponentRecurring`, `jobRelationsInfo`. Not `personInfo`, `emailInfo`, `phoneInfo`, `nationalIdCard`, `emergencyContactPrimary`.

**17. What are background elements? [MED]**
Talent-profile lists defined in the SDM — education, work experience, certifications, languages. They are simple lists of rows, not effective-dated, and they render on People Profile next to EC blocks.

**18. How do you tell an HRIS element from a background element in the UI? [MED]**
HRIS elements have effective-dated History with event reasons; background elements are lists you add rows to.

**19. What are standard/user elements and why do they matter? [MED]**
The pre-EC user-record fields (department, division, location, job code as plain text). Talent modules and some reports read them, and they are populated from EC — so a misconfigured sync makes the same attribute show two different values in two places.

---

## Country-specific models

**20. What do the CSF models do? [HIGH]**
Layer country-specific fields on top of the base models so employees and organisational objects show only the fields relevant to their country.

**21. How does the system decide which country's fields to show? [HIGH]**
By the country on the record itself — the address's country, the national ID row's country, the Global Information block's country — and, for job and compensation fields, by the country of the legal entity on Job Information.

**22. Give examples of country-specific fields. [HIGH]**
India: PAN, Aadhaar, UAN, PF number. Germany: tax ID, social insurance number, church tax indicator. USA: SSN, state, FLSA status, EEO category. UK: National Insurance number. Brazil: CPF, PIS.

**23. Does SAP ship country content, or do you build it? [HIGH]**
SAP ships content for 100+ countries — required fields, address formats, national ID types and validations. You enable the country, review the fields with local HR and payroll, hide what is unused, and add what is missing.

**24. How does a global assignment interact with country data? [MED]**
The person carries country blocks, national IDs and addresses for both home and host countries, across two employments whose legal entities differ.

**25. What must you test when adding a country? [HIGH]**
That the new fields appear for that country, that they do **not** appear for existing countries (regression), and that a full hire-to-payroll cycle works in the new legal entity.

**26. Where do country-specific fields need to end up? [HIGH]**
In the payroll interface mapping and statutory reporting. Capturing them in EC without mapping them onward is the classic gap.

---

## XML and editing practice

**27. Do you still edit data models as XML? [HIGH]**
Most changes go through Manage Business Configuration. XML is used for what BCUI does not expose, for bulk edits, and always for documentation and environment comparison.

**28. What do you do before uploading a data model? [HIGH]**
Download and archive the current version — it is both the backup and the diff baseline — and work in DEV first.

**29. What breaks XML most often? [MED]**
Unclosed tags, overlapping nesting, and unescaped special characters such as `&` in a label like "R&D".

**30. How do you retire a field? [MED]**
Set `visibility="none"`. Do not delete it — the data remains and a deleted-then-reused definition is worse than a hidden field.

**31. How would you find what changed in the data model last month? [MED]**
Diff dated XML exports — which is why you export after every change.

**32. You inherit an unfamiliar instance. How do you learn its configuration quickly? [MED]**
Export the models and review: which elements are in use, which custom placeholders exist and how they are labelled, which fields are required, which are hidden, and which rules are attached at which triggers.

---

## Business Configuration UI

**33. What is BCUI? [HIGH]**
Manage Business Configuration — the Admin Center tool for editing the Succession Data Model through a UI: elements, fields, labels, visibility, required, picklists, field criteria and rule attachments.

**34. What can you not do in BCUI? [HIGH]**
Enable features or upload a whole data model (Provisioning), edit MDF object definitions (Configure Object Definitions), create picklist values (Picklist Center), or control who sees a field (RBP).

**35. How do you attach a business rule to Job Information? [HIGH]**
Create the rule in Configure Business Rules, then in BCUI open `jobInfo`, choose element or field level, select the trigger (onInit / onChange / onSave / onPostSave), and attach the rule.

**36. You added a field in BCUI and cannot see it. Why? [HIGH]**
It is not permissioned in RBP, or its visibility is `none`, or the block is not placed on People Profile.

---

## Picklists

**37. What is a picklist, and what is the external code? [HIGH]**
A configurable list of values behind a dropdown. The external code is the stable stored key used by imports, integrations, rules and reports; the label is the display text and may be changed freely.

**38. Can you delete a picklist value? [HIGH]**
Not if it is in use. Set it inactive — existing records keep and display the value, and new records cannot select it.

**39. What is a cascading picklist? Give the EC example. [HIGH]**
A picklist filtered by a parent field's value. The canonical EC example is **Event → Event Reason**, so a user choosing Termination sees only termination reasons.

**40. Cascading picklist vs field criteria? [MED]**
Cascading filters on a parent picklist value; field criteria filter on another field's value, including foundation objects — such as job classifications filtered by job function, or pay ranges by grade and geozone.

**41. Legacy vs MDF picklists? [MED]**
Legacy picklists were maintained by CSV import and are not effective-dated. MDF picklists are managed in Picklist Center, are effective-dated and support richer configuration; MDF is the current model.

**42. When do you use a picklist rather than an object? [HIGH]**
When the value is just a code and a label. If it needs attributes, associations, effective-dated history or the ability to drive derivation, it must be a foundation object or MDF object.

**43. A rule that checks marital status never fires. Why? [MED]**
It was probably written against the label rather than the external code, or against a value that has since been superseded.

---

## Scenario questions

**44. HR wants a new field capturing "Notice Period (weeks)" that payroll also needs. Walk me through it. [HIGH]**
Decide the element (`employmentInfo` or `jobInfo`, depending on whether it must be effective-dated), decide base vs country-specific, claim a `custom-long`/`custom-double` placeholder in BCUI, set label and visibility, add a validation rule for sensible ranges, permission it in RBP, test the UI and import template, add it to the payroll interface mapping, then export the XML, diff, and document the placeholder in the workbook.

**45. Adding a country: what does it actually involve? [HIGH]**
Country enablement, a legal entity, CSF-SDM field review, address format, national ID types with validations, pay components, pay group and pay ranges, any country-specific event reasons, RBP groups and target populations, payroll interface mapping, document templates, a full hire-to-pay test, and a regression test that existing countries are unchanged.

**46. A field shows on People Profile for some users but not others. Is that a data model issue? [HIGH]**
No — global visibility lives in the model; per-user visibility is RBP field-level permission. Check the users' roles, then the People Profile configuration, then the model.

**47. The nightly hire interface started failing after your change. What did you do? [HIGH]**
Most likely made a field `required="true"` that the interface does not supply — or changed a picklist so the codes it sends no longer resolve, or reduced a `max-length` below what it sends.

**48. The customer asks to rename event reason codes to be "cleaner". [MED]**
Explain that external codes are permanent: renaming orphans every historical record, integration mapping and rule. The safe route is new values plus inactivation of the old ones, accepting a mixed history — and that this is exactly why codes are designed on day one.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Data models, picklists and configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
