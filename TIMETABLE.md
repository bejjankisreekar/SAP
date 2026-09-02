# Timetable and Learning Objectives — SAP SuccessFactors Employee Central from scratch

A **6-week, topic-wise timetable** at **2 hr 30 min per day, 5 days a week** (weekends kept free for catch-up and revision) — the same 75-hour budget as the original 30-day plan, re-cut so that prerequisites come before the things that depend on them.

- **Total:** 30 study days × 2h30 = **75 hours**
- **Assumes:** no prior SAP knowledge
- **Requires:** a practice/learning system wherever the SAP course provides one

**How each session is structured** (the 2h30 block):

| Minutes | Activity | Why |
|---|---|---|
| 0–15 | Recall yesterday **without looking** — say it out loud or write it | Retrieval, not re-reading, is what makes it stick |
| 15–60 | SAP Learning course lesson | The authoritative source |
| 60–105 | Video + notes from this repo, side by side | Seeing the screens makes the theory concrete |
| 105–135 | **Hands-on in the practice system** (or written exercise where hands-on is not possible) | You do not know a topic until you have configured it |
| 135–150 | Write 5 bullet points and 2 interview questions in your own words | Your revision material writes itself |

> **Practice-system rule:** use the practice/learning system whenever the SAP course offers practice-system access. Do **not** attempt Provisioning tasks without the required implementation access — read what the switch does instead, and do the Admin Center part.

---

## The 6 weeks at a glance

| Week | Theme | Topic folders | Days | Hours |
|---|---|---|---|---|
| **1** | Orientation and the employee record | 01 Foundations, 02 Employee Data | 1–5 | 12.5 |
| **2** | Structures and the data model | 02 Employee Data, 03 Foundation Objects | 6–10 | 12.5 |
| **3** | Data models and picklists | 04 Data Models | 11–15 | 12.5 |
| **4** | MDF and business rules | 05 MDF, 06 Business Rules | 16–20 | 12.5 |
| **5** | Transactions, workflows, security, positions | 07 Transactions, 08 RBP, 09 Position Management | 21–25 | 12.5 |
| **6** | Integration, testing, certification, interviews | 10 Integrations, 11 Implementation, Certifications, Interviews | 26–30 | 12.5 |

---

# Week 1 — Orientation and the employee record

**Week objective:** explain what EC is, navigate the system confidently, and describe how one human being is stored.

### Day 1 — What SuccessFactors and EC are
**Read:** [01_Foundations 01–03](01_Foundations/01_What_is_SAP_SuccessFactors.md) · **Course:** [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) · **Video:** [V1](https://www.youtube.com/watch?v=6xrD8Vkm0QI)

*By the end you can:*
- [ ] Define SuccessFactors, HXM and Employee Central in one sentence each
- [ ] Name at least eight modules of the suite and say which consume EC data
- [ ] State three things you can do in on-premise SAP HCM that you cannot do in EC
- [ ] Explain why EC is called "the cloud version of SAP HCM" and where that phrase is inaccurate

### Day 2 — Navigation, architecture and the admin tools
**Read:** [01_Foundations 04–09](01_Foundations/05_Navigation_and_People_Profile.md) · **Course:** Platform Introduction Academy — platform basics · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Find any admin tool using Action Search, without menu paths
- [ ] Describe People Profile as sections and blocks, and say what controls each
- [ ] Explain Provisioning vs Admin Center and who may access each
- [ ] Describe the DEV/TEST/PROD landscape and how configuration moves between tenants
- [ ] Explain the half-yearly release cycle and the difference between universal and opt-in
- [ ] Map at least ten infotypes to their EC equivalents

### Day 3 — Person, User, Employment
**Read:** [02_Employee_Data 01](02_Employee_Data/01_Person_User_Employment_Model.md) · **Course:** [EC Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — getting started · **Video:** [V1](https://www.youtube.com/watch?v=6xrD8Vkm0QI)

*By the end you can:*
- [ ] Draw the Person → Employment → Job Information structure from memory
- [ ] Say which key (`person_id_external` vs `user_id`) belongs to which layer
- [ ] Give three scenarios producing multiple employments for one person
- [ ] Decide, for any given field, whether it belongs at person or employment level

### Day 4 — Effective dating, Biographical and Personal Information
**Read:** [02_Employee_Data 02–03](02_Employee_Data/02_Effective_Dating_and_History.md) · **Course:** EC Core Academy — configuring employee data · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Explain effective dating and how a record's end date is determined
- [ ] Choose correctly between **Insert** and **Correct** for any given scenario
- [ ] List which EC elements are effective-dated and which are not
- [ ] Explain what a future-dated and a retroactive change do to payroll and reporting
- [ ] State the difference between `personInfo` and `personalInfo` and where a name change goes

**Hands-on:** insert a record, correct a record, and view history on one test employee.

### Day 5 — Addresses, contact data, national IDs, Employment Information
**Read:** [02_Employee_Data 04–05](02_Employee_Data/04_Addresses_Contact_and_National_ID.md) · **Course:** EC Core Academy — configuring employee data

*By the end you can:*
- [ ] Explain why address layouts differ per country and where that is configured
- [ ] Say why one person may hold several national IDs
- [ ] Distinguish hire date, original start date and seniority date
- [ ] Explain how contingent workers are stored and why they are excluded from headcount

**Week 1 milestone:** whiteboard the record of someone hired 2019, promoted 2022, terminated 2023, rehired 2025 — naming which IDs change and which do not.

---

# Week 2 — Structures and how they connect

**Week objective:** explain everything an employee record *points at*, and configure it.

### Day 6 — Job Information
**Read:** [02_Employee_Data 06](02_Employee_Data/06_Job_Information.md) · **Course:** EC Core Academy · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Name and explain fifteen Job Information fields
- [ ] Explain how Event and Event Reason sit *on* the job row
- [ ] Describe how manager, position, department and cost centre interact
- [ ] Explain what FTE, standard hours and employee status control

### Day 7 — Compensation Information
**Read:** [02_Employee_Data 07–08](02_Employee_Data/07_Compensation_Information.md) · **Course:** EC Core Academy · **Video:** [V1](https://www.youtube.com/watch?v=6xrD8Vkm0QI)

*By the end you can:*
- [ ] Explain the header-plus-pay-components structure
- [ ] Distinguish recurring from non-recurring pay components
- [ ] Explain annualisation, FTE and frequency, and compute an annual salary from a monthly amount
- [ ] Say how pay grade, pay range and compa-ratio relate

### Day 8 — Foundation Objects: organisational structure
**Read:** [03_Foundation_Objects 01–02](03_Foundation_Objects/01_What_are_Foundation_Objects.md) · **Course:** EC Core Academy — configuring foundation objects · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Define a Foundation Object and say why FOs exist
- [ ] Explain Legal Entity, Business Unit, Division, Department, Cost Centre, Location, Geozone
- [ ] Draw a customer's org structure using the right objects
- [ ] Create a Legal Entity and a Department in Manage Data

### Day 9 — Foundation Objects: job and pay structures
**Read:** [03_Foundation_Objects 03–04](03_Foundation_Objects/03_Job_Structure_FOs.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain Job Function, Job Classification, Job Family, Pay Grade, Pay Range, Pay Group, Pay Component, Pay Component Group
- [ ] Say which FOs are MDF-based and which remain legacy
- [ ] Build a small job catalogue and pay structure for one country

### Day 10 — Associations, propagation, importing FOs
**Read:** [03_Foundation_Objects 05–06](03_Foundation_Objects/05_Associations_and_Propagation.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain a one-to-many and a many-to-many association with an EC example
- [ ] Explain **propagation** and how it differs from a business rule
- [ ] Import foundation data in the correct dependency order

**Week 2 milestone:** design, on paper, the complete foundation structure for a two-country company, then build one country of it in the practice system.

---

# Week 3 — The data models

**Week objective:** read and change the XML that defines the whole system.

### Day 11 — The data model landscape and the Corporate Data Model
**Read:** [04_Data_Models 01–02](04_Data_Models/01_Data_Model_Landscape.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Name the four data models and say what each governs
- [ ] Read a Corporate Data Model XML fragment and explain every line
- [ ] Say what happens if you upload a malformed data model — and what you do first

### Day 12 — HRIS elements, HRIS fields and XML syntax
**Read:** [04_Data_Models 05–06](04_Data_Models/05_HRIS_Elements_and_Fields_Reference.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain `hris-element`, `hris-field`, `visibility`, `required`, `label`, `picklist`
- [ ] Add a custom field to an HRIS element and permission it
- [ ] List the person-level and employment-level HRIS elements from memory

### Day 13 — The Succession Data Model
**Read:** [04_Data_Models 03](04_Data_Models/03_Succession_Data_Model.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] State exactly what the SDM contains and how it differs from the CDM
- [ ] Explain what background elements are and why they exist
- [ ] Edit the SDM through Manage Business Configuration and know when XML is still needed

### Day 14 — Country/region-specific data models
**Read:** [04_Data_Models 04](04_Data_Models/04_Country_Specific_Data_Models.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain CSF-CDM and CSF-SDM and give two examples of each
- [ ] Say how the system decides which country block to show
- [ ] Explain how country-specific fields reach payroll

### Day 15 — Picklists
**Read:** [04_Data_Models 07–08](04_Data_Models/08_Picklists.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Create a picklist in Picklist Center with external codes and translations
- [ ] Build a **cascading/dependent** picklist and explain field criteria
- [ ] Explain why you never delete a picklist value that is in use
- [ ] Say when to use a picklist versus a foundation object versus an MDF object

**Week 3 milestone:** read an unfamiliar data-model XML and, without help, explain what every element does and what would break if you removed a given field.

---

# Week 4 — MDF and business rules

**Week objective:** extend the system and make it behave.

### Day 16 — MDF fundamentals
**Read:** [05_MDF 01](05_MDF/01_MDF_Fundamentals.md) · **Course:** [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain what MDF is and what it replaced
- [ ] Name five standard objects that are MDF objects
- [ ] Explain Generic Object vs Generic Object Definition

### Day 17 — Object definitions and field types
**Read:** [05_MDF 02–03](05_MDF/02_Object_Definitions.md) · **Course:** Platform Introduction Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Create a custom MDF object end to end
- [ ] Choose the right field type and explain effective dating on MDF objects
- [ ] Explain API visibility, security settings and object status

### Day 18 — MDF associations, Configuration UI, import/export
**Read:** [05_MDF 04–07](05_MDF/04_Associations.md) · **Course:** Platform Introduction Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Build valid-values and composite associations, and explain the difference
- [ ] Design a screen with Configure Configuration UI
- [ ] Export an object definition and its data, and import it into another tenant

### Day 19 — Business rules: fundamentals
**Read:** [06_Business_Rules 01–03](06_Business_Rules/01_Rules_Engine_Fundamentals.md) · **Course:** EC Core Academy — creating business rules · **Video:** [V4](https://www.youtube.com/watch?v=90aPAtJbl9g)

*By the end you can:*
- [ ] Explain If/Then structure, rule scenarios and base objects
- [ ] Choose the right scenario for a requirement
- [ ] Write a simple defaulting rule and a simple validation rule

### Day 20 — Triggers, attachment points, event-reason derivation
**Read:** [06_Business_Rules 04–07](06_Business_Rules/04_Attaching_Rules.md) · **Course:** EC Core Academy · **Video:** [V4](https://www.youtube.com/watch?v=90aPAtJbl9g)

*By the end you can:*
- [ ] Explain onInit, onChange, onSave, onPostSave and when each fires
- [ ] Attach a rule to an HRIS element, a field, and an MDF object
- [ ] Write an event-reason derivation rule
- [ ] Debug a rule that is not firing, in a defined order of checks

**Week 4 milestone:** build a custom MDF object with an association, put it on a screen, and drive one of its fields with a business rule.

---

# Week 5 — Transactions, workflows, security, positions

**Week objective:** run the employee lifecycle end to end, safely.

### Day 21 — Events and Event Reasons
**Read:** [07_Transactions 01](07_Transactions_and_Workflows/01_Events_and_Event_Reasons.md) · **Course:** EC Core Academy — configuring transactions · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain Event vs Event Reason and give five examples of each
- [ ] Explain what employee status an event reason drives
- [ ] Design an event-reason catalogue for a customer

### Day 22 — The lifecycle: hire, transfer, promotion, termination, rehire
**Read:** [07_Transactions 02–05](07_Transactions_and_Workflows/02_Hire_and_Rehire.md) · **Course:** EC Core Academy · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Execute every lifecycle transaction in the practice system
- [ ] Say exactly which records each transaction creates or changes
- [ ] Explain global assignment and concurrent employment

### Day 23 — Workflows
**Read:** [07_Transactions 06–08](07_Transactions_and_Workflows/06_Workflow_Fundamentals.md) · **Course:** EC Core Academy · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Build a two-step workflow with a CC role
- [ ] Explain approver types, delegation, escalation and pending data
- [ ] Troubleshoot a stuck workflow

### Day 24 — Role-Based Permissions
**Read:** [08_Security_RBP](08_Security_RBP/00_RBP_Learning_Path.md) · **Course:** Platform Introduction Academy — permissions · **Video:** [V5](https://www.youtube.com/watch?v=T4yXgdkpxz8)

*By the end you can:*
- [ ] Explain permission group, permission role, granted user and target population
- [ ] Build an ESS role, an MSS role and an HR Admin role
- [ ] Diagnose "I cannot see this field" in a defined order

### Day 25 — Position Management
**Read:** [09_Position_Management](09_Position_Management/00_Position_Management_Learning_Path.md) · **Course:** [Position Management Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-position-management-academy) · **Videos:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA) · [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Explain position-controlled vs job-controlled organisations
- [ ] Configure the Position object and Position Management Settings
- [ ] Explain synchronisation between Position and Job Information, in both directions
- [ ] Explain To Be Hired, FTE control and the position hierarchy

**Week 5 milestone:** hire someone into a position, promote them through a workflow, terminate them, rehire them — and explain every record that changed.

---

# Week 6 — Integration, delivery, certification, interviews

**Week objective:** get data out, prove the build works, and talk about it convincingly.

### Day 26 — Integration Center
**Read:** [10_Integrations 01–02](10_Integrations/02_Integration_Center.md) · **Course:** [Learning journey](https://learning.sap.com/learning-journeys/configuring-sap-successfactors-employee-central-core-and-position-management) · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Build and schedule a CSV extract with filters and calculated fields
- [ ] Say when Integration Center is the right tool and when it is not

### Day 27 — OData and Compound Employee API
**Read:** [10_Integrations 03–04](10_Integrations/03_OData_API.md) · **Course:** Learning journey · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Write an OData query with `$filter`, `$select`, `$expand` and effective-dating parameters
- [ ] Explain upsert and why EC has no plain insert
- [ ] Explain when to use Compound Employee API instead of OData

### Day 28 — EC to payroll: ECP and on-premise SAP HCM
**Read:** [10_Integrations 05–07](10_Integrations/05_EC_to_Employee_Central_Payroll.md) · **Source:** [Recommended implementation sequence](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core/recommended-implementation-sequence) · **Video:** [V2](https://www.youtube.com/watch?v=qkmFdj4h4rA)

*By the end you can:*
- [ ] Draw the replication architecture from EC to ECP and to on-premise HCM
- [ ] Explain org-assignment mapping and where replication errors are monitored
- [ ] Explain what flows back from payroll and what does not

### Day 29 — Implementation sequence, testing, troubleshooting
**Read:** [11_Implementation_and_Testing](11_Implementation_and_Testing/00_Implementation_Learning_Path.md) · **Course:** EC Core Academy · **Video:** [V3](https://www.youtube.com/watch?v=yEsquQA-MxU)

*By the end you can:*
- [ ] Recite the recommended implementation sequence and justify the order
- [ ] Write test scenarios for the full lifecycle, with expected results
- [ ] Troubleshoot the ten most common EC defects in a defined order
- [ ] Describe a realistic cutover plan

### Day 30 — Certification roadmap and mock interview
**Read:** [Certifications](Certifications/01_EC_Core_Certification_Guide.md) · [Job Interviews](Job%20Interviews/01_Mock_Interview_Script.md) · **Course:** Learning journey · **Video:** [V1](https://www.youtube.com/watch?v=6xrD8Vkm0QI)

*By the end you can:*
- [ ] State which certification you are aiming at and its exam structure
- [ ] Deliver a 3-minute project story covering scope, your role, one hard problem and the outcome
- [ ] Answer 20 scenario questions without notes

**Week 6 milestone:** a recorded 30-minute mock interview you would be willing to let a hiring manager hear.

---

## If you only have 2 weeks

Compress to the exam-critical core, 4 hours a day:

| Day | Topics |
|---|---|
| 1 | Foundations (all) |
| 2 | Person/Employment, effective dating |
| 3 | Personal, Address, Employment Information |
| 4 | Job Information, Compensation |
| 5 | Foundation Objects (all) |
| 6 | Data models (all) |
| 7 | Picklists + MDF fundamentals |
| 8 | MDF objects, associations, Configuration UI |
| 9 | Business rules (all) |
| 10 | Events/event reasons + lifecycle transactions |
| 11 | Workflows + RBP |
| 12 | Position Management |
| 13 | Integrations (all) |
| 14 | Testing, troubleshooting, interview Q&A |

---

## Tracking your progress

Copy this into a note and tick as you go. The **milestone** matters more than the tick — if you cannot hit the milestone, repeat the week rather than moving on.

- [ ] Week 1 — Foundations and the employee record
- [ ] Week 2 — Job/Comp data and foundation objects
- [ ] Week 3 — Data models and picklists
- [ ] Week 4 — MDF and business rules
- [ ] Week 5 — Transactions, workflows, RBP, positions
- [ ] Week 6 — Integrations, testing, certification, interviews

See also: [ROADMAP.md](ROADMAP.md) for the phase view, and [30_Day_Plan_Mapping.md](30_Day_Plan_Mapping.md) for the original day-wise plan with all source links.
