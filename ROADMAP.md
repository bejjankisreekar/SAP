# SAP SuccessFactors Employee Central — Roadmap (Zero to Job-Ready)

A single ordered path from *"I have never seen an HR system"* to *"I can configure Employee Central, pass the EC Core certification, and hold my own in a consultant interview."*

This is the **map**. The lessons live in the [category folders](README.md); this file tells you **what to learn, in what order, why, and how to know you are ready**. Every ✅ topic already has notes in this repo. Every 🔜 topic is planned but not yet written.

For the calendar version with hours per day, see [TIMETABLE.md](TIMETABLE.md). For the original 30-day plan and every source link, see [30_Day_Plan_Mapping.md](30_Day_Plan_Mapping.md).

> **How to use this file:** work top to bottom. Do not skip a phase because it looks easy — each is a prerequisite for the next. Every phase ends with a **milestone**: something you can *do*, not just *know*. If you cannot hit the milestone, stay in the phase.

---

## What does an EC consultant actually do?

You translate **an HR process into a data model plus rules**, inside a system you cannot write code in. Day to day:

- **Gather** requirements from HR in workshops, and challenge the ones that will not work.
- **Design** the organisational structures, the employee data model, the events, the approvals and the roles.
- **Configure** foundation objects, data models, MDF objects, picklists, business rules, workflows and permissions.
- **Migrate** legacy employee data through import templates, and prove it reconciles.
- **Integrate** EC with payroll, identity management and third parties.
- **Test** with the customer, fix defects, cut over, and support hypercare.

The toolset: **Admin Center, Manage Data, Configure Object Definitions, Manage Business Configuration, Configure Business Rules, Manage Permission Roles, Import Employee Data, Integration Center** — plus XML literacy and enough OData to debug an interface.

---

## The 8 phases at a glance

| Phase | Theme | You will be able to… |
|---|---|---|
| **0** | Orientation | Explain the suite, EC's role, and where every admin tool lives |
| **1** | The employee record | Describe how one human is stored, and manipulate effective-dated data correctly |
| **2** | Organisational structures | Design and build the foundation objects a company runs on |
| **3** | The data models | Read and change the XML that defines the whole system |
| **4** | MDF | Extend the system with your own objects, fields and screens |
| **5** | Logic and process | Make the system behave: rules, events, transactions, workflows, security |
| **6** | Position Management | Run a position-controlled organisation |
| **7** | Integration and delivery | Get data out, test the build, and go live |
| **8** | Certification and interviews | Prove it |

---

# Phase 0 — Orientation

**Goal:** know what the product is and where everything lives, before configuring anything.

- ✅ [What is SAP SuccessFactors](01_Foundations/01_What_is_SAP_SuccessFactors.md)
- ✅ [The HXM suite — module map](01_Foundations/02_HXM_Suite_Modules.md)
- ✅ [What is Employee Central](01_Foundations/03_What_is_Employee_Central.md)
- ✅ [EC vs SAP HCM on-premise](01_Foundations/04_EC_vs_SAP_HCM_On_Premise.md)
- ✅ [Navigation and People Profile](01_Foundations/05_Navigation_and_People_Profile.md)
- ✅ [Architecture, instances and landscape](01_Foundations/06_Architecture_Instances_and_Landscape.md)
- ✅ [Provisioning vs Admin Center](01_Foundations/07_Provisioning_vs_Admin_Center.md)
- ✅ [Release cycle and Upgrade Center](01_Foundations/08_Release_Cycle_and_Upgrade_Center.md)
- ✅ [Admin tools cheat sheet](01_Foundations/09_Admin_Tools_Cheat_Sheet.md)
- ✅ [Foundations Q&A](01_Foundations/Interview_Questions_and_Answers.md)

**Milestone:** find any admin tool in under ten seconds using Action Search, and explain to a non-technical person what EC is and how it differs from SAP HCM.

---

# Phase 1 — The employee record

**Goal:** the single most important body of knowledge in EC. Everything else references it.

- ✅ [Person, User and Employment](02_Employee_Data/01_Person_User_Employment_Model.md)
- ✅ [Effective dating and history](02_Employee_Data/02_Effective_Dating_and_History.md)
- ✅ [Biographical and Personal Information](02_Employee_Data/03_Biographical_and_Personal_Information.md)
- ✅ [Addresses, contact and National ID](02_Employee_Data/04_Addresses_Contact_and_National_ID.md)
- ✅ [Employment Information](02_Employee_Data/05_Employment_Information.md)
- ✅ [Job Information](02_Employee_Data/06_Job_Information.md)
- ✅ [Compensation Information](02_Employee_Data/07_Compensation_Information.md)
- ✅ [Job relationships and dependants](02_Employee_Data/08_Job_Relationships_and_Dependants.md)
- ✅ [Employee data imports](02_Employee_Data/09_Employee_Data_Imports.md)
- ✅ [Employee Data Q&A](02_Employee_Data/Interview_Questions_and_Answers.md)

**Milestone:** whiteboard the full record of someone hired 2019, promoted 2022, terminated 2023, rehired 2025 — naming every element involved, which IDs change, and which rows get created. Then do the same transactions in a practice system.

---

# Phase 2 — Organisational structures (Foundation Objects)

**Goal:** design the structures every employee record points at.

- 🔜 What are Foundation Objects
- 🔜 Organisational structure FOs — legal entity, business unit, division, department, cost centre, location, geozone
- 🔜 Job structure FOs — job function, job classification, job family
- 🔜 Pay structure FOs — pay grade, pay range, pay group, pay component, pay component group
- 🔜 Associations and propagation
- 🔜 Managing and importing foundation data
- 🔜 Foundation Objects Q&A

**Milestone:** given a company's org chart and job catalogue on paper, choose the right objects, draw the associations, and build one country of it in a practice system.

---

# Phase 3 — The data models

**Goal:** read and change the configuration that defines the whole instance.

- 🔜 The data model landscape — which model governs what
- 🔜 Corporate Data Model
- 🔜 Succession Data Model
- 🔜 Country/region-specific data models
- 🔜 HRIS elements and fields reference
- 🔜 XML syntax and safe editing rules
- 🔜 Business Configuration UI
- 🔜 Picklists (including cascading picklists and field criteria)
- 🔜 Data Models Q&A

**Milestone:** open an unfamiliar data-model XML and explain every element, then add a custom field end to end — model, permission, test, export, document.

---

# Phase 4 — MDF (Metadata Framework)

**Goal:** extend the product without code.

- 🔜 MDF fundamentals
- 🔜 Object definitions
- 🔜 Fields and field types
- 🔜 Associations
- 🔜 Configuration UI and Manage Data
- 🔜 Import, export and versioning
- 🔜 MDF security and picklists
- 🔜 MDF Q&A

**Milestone:** build a custom MDF object with an association and a configured screen, populate it by import, secure it in RBP, and drive one field with a business rule.

---

# Phase 5 — Logic and process

**Goal:** make the system behave, route, and stay secure.

**Business rules**
- 🔜 Rules engine fundamentals
- 🔜 Rule scenarios and contexts
- 🔜 Trigger points — onInit, onChange, onSave, onPostSave
- 🔜 Attaching rules
- 🔜 Common EC rule recipes
- 🔜 Event reason derivation
- 🔜 Debugging rules
- 🔜 Business Rules Q&A

**Transactions and workflows**
- 🔜 Events and Event Reasons
- 🔜 Hire and rehire
- 🔜 Job change, transfer, promotion
- 🔜 Termination
- 🔜 Global assignment and concurrent employment
- 🔜 Workflow fundamentals
- 🔜 Workflow configuration step by step
- 🔜 Workflow administration and troubleshooting
- 🔜 Transactions Q&A

**Security**
- 🔜 RBP fundamentals
- 🔜 Permission groups
- 🔜 Permission roles and categories
- 🔜 Target population
- 🔜 RBP design and troubleshooting
- 🔜 RBP Q&A

**Milestone:** hire someone, promote them through a two-step workflow with a derived event reason, terminate them, and rehire them — then explain every record, rule and approval that fired, and prove with Proxy that each role sees only what it should.

---

# Phase 6 — Position Management

**Goal:** run a position-controlled organisation.

- 🔜 Position Management fundamentals
- 🔜 The Position object and its configuration
- 🔜 Position hierarchy and org chart
- 🔜 Synchronisation between Position and Job Information
- 🔜 Position Management in transactions
- 🔜 Position Management Q&A

**Milestone:** configure position management settings, create a position hierarchy, hire into a vacant position, and explain what happens to the position when the incumbent is promoted out of it.

---

# Phase 7 — Integration and delivery

**Goal:** get data out of EC, prove the build works, and go live.

**Integration**
- 🔜 The integration landscape
- 🔜 Integration Center
- 🔜 OData API
- 🔜 Compound Employee API
- 🔜 EC to EC Payroll
- 🔜 EC to SAP HCM on-premise
- 🔜 Third-party integration and middleware
- 🔜 Integration Q&A

**Delivery**
- 🔜 Implementation methodology (SAP Activate)
- 🔜 Recommended implementation sequence
- 🔜 Data migration
- 🔜 Testing strategy
- 🔜 Troubleshooting playbook
- 🔜 Cutover and hypercare
- 🔜 Implementation Q&A

**Beyond core EC (awareness level)**
- 🔜 Time Off and Time Sheet
- 🔜 Benefits, Document Generation, Alerts & Notifications
- 🔜 Reporting and analytics

**Milestone:** build a scheduled Integration Center extract, write an OData query with effective-dating parameters, draw the EC-to-payroll replication architecture, and write a full test script for the employee lifecycle.

---

# Phase 8 — Certification and interviews

**Goal:** prove it, and be able to talk about it.

- 🔜 EC Core certification guide — exam structure, topic weightings, preparation plan
- 🔜 Mock interview script
- 🔜 Project explanation template
- 🔜 Scenario-based questions
- 🔜 Consultant skills — workshops, documentation, stakeholder management

**Milestone:** a 30-minute recorded mock interview you would let a hiring manager hear, plus a three-minute project story covering scope, your role, one hard problem and the outcome.

---

## Progress tracker

| Phase | Status |
|---|---|
| 0 — Orientation | ✅ notes complete |
| 1 — The employee record | ✅ notes complete |
| 2 — Foundation Objects | 🔜 |
| 3 — Data models | 🔜 |
| 4 — MDF | 🔜 |
| 5 — Logic and process | 🔜 |
| 6 — Position Management | 🔜 |
| 7 — Integration and delivery | 🔜 |
| 8 — Certification and interviews | 🔜 |
