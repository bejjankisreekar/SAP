# Learning SAP SuccessFactors Employee Central

A personal learning repository for **SAP SuccessFactors Employee Central (EC)** — the cloud core-HR module, commonly described as *the cloud version of SAP HCM* — written as plain Markdown notes, organised **topic-wise**.

**No SAP background required.** Every note is written so that someone coming from outside SAP (or outside IT) can follow along. Terms are explained the first time they appear, and most topics open with a real-world analogy before the technical explanation.

**Start here:**

- **[TIMETABLE.md](TIMETABLE.md)** — a 6-week, 75-hour study plan with explicit **learning objectives** for every session, from scratch.
- **[ROADMAP.md](ROADMAP.md)** — the 8-phase path with milestones, telling you what to learn in what order and why.
- **[GLOSSARY.md](GLOSSARY.md)** — every recurring term in one place (HRIS element, MDF, RBP, pay component, effective dating…).
- **[30_Day_Plan_Mapping.md](30_Day_Plan_Mapping.md)** — the original day-wise plan, with every SAP Learning course link and video link, mapped onto these topic folders.

---

## How to read these notes

Each `.md` file is **one continuous read** that takes a topic from first principles to what an experienced consultant knows about it. There are no separate beginner/advanced tracks — the note simply keeps going:

1. **What is it?** — plain-language definition, usually with a real-world comparison.
2. **The detail** — every field, setting, or concept, in tables you can revise from.
3. **How it behaves** — the rules the system actually follows, including the ones that surprise people.
4. **Step by step** — what to click, in order, in a practice system.
5. **Design decisions** — the choices a consultant is asked to make, and the trade-offs.
6. **Real world example** — a short project story tying it together.
7. **Common mistakes** — the defects that show up in UAT.
8. **Interview-grade Q&A** — at the end of every file, tagged `[HIGH]` / `[MED]` / `[LOW]` by how often they come up.
9. **Further learning** — the exact SAP Learning course and video for that topic.

> **Practice-system rule (applies everywhere):** use the practice/learning system whenever the SAP course offers practice-system access. Do **not** attempt Provisioning tasks without the required implementation access — Provisioning is not available to customers and mistakes there are not self-serviceable.

---

## Learning path

| Category folder | Contains | Status |
|---|---|---|
| **[01_Foundations](01_Foundations/00_Foundations_Learning_Path.md)** | The suite, EC, EC vs SAP HCM, navigation, architecture, Provisioning vs Admin Center, releases, admin tools | ✅ complete |
| **[02_Employee_Data](02_Employee_Data/00_Employee_Data_Learning_Path.md)** | Person/Employment model, effective dating, personal data, job info, compensation, imports | ✅ complete |
| **03_Foundation_Objects** | Organisational, job and pay structures; associations and propagation | 🔜 |
| **04_Data_Models** | Corporate and Succession Data Models, country-specific models, HRIS elements, XML, picklists | 🔜 |
| **05_MDF** | Metadata Framework: object definitions, fields, associations, Configuration UI | 🔜 |
| **06_Business_Rules** | Rules engine, scenarios, trigger points, event-reason derivation, debugging | 🔜 |
| **07_Transactions_and_Workflows** | Events and event reasons, hire/transfer/promotion/termination/rehire, workflows | 🔜 |
| **08_Security_RBP** | Permission groups, roles, target populations, troubleshooting | 🔜 |
| **09_Position_Management** | Position object, hierarchy, synchronisation with Job Information | 🔜 |
| **10_Integrations** | Integration Center, OData, Compound Employee API, EC to ECP / SAP HCM | 🔜 |
| **11_Implementation_and_Testing** | SAP Activate, implementation sequence, migration, testing, cutover, troubleshooting | 🔜 |
| **12_Beyond_Core_EC** | Time Off, Benefits, Document Generation, reporting — awareness level | 🔜 |
| **Certifications** | EC Core certification guide | 🔜 |
| **Job Interviews** | Mock interview script, project explanation, scenario questions | 🔜 |

---

### 01_Foundations — what this product is and where everything lives

- [What is SAP SuccessFactors](01_Foundations/01_What_is_SAP_SuccessFactors.md) — the suite, its history, SaaS vs on-premise, and what a consultant actually does
- [The HXM Suite — module map](01_Foundations/02_HXM_Suite_Modules.md) — every module, and which ones consume EC data
- [What is Employee Central](01_Foundations/03_What_is_Employee_Central.md) — the five pillars, the three-layer data structure, one promotion end to end
- [EC vs SAP HCM on-premise](01_Foundations/04_EC_vs_SAP_HCM_On_Premise.md) — the infotype → HRIS element translation table, and what has no equivalent
- [Navigation and People Profile](01_Foundations/05_Navigation_and_People_Profile.md) — Action Search, blocks, Take Action, Proxy, and the missing-field debugging order
- [Architecture, instances and landscape](01_Foundations/06_Architecture_Instances_and_Landscape.md) — multi-tenancy, DEV/TEST/PROD, Instance Sync, refresh risks
- [Provisioning vs Admin Center](01_Foundations/07_Provisioning_vs_Admin_Center.md) — the two control rooms, and a safe data-model change procedure
- [Release cycle and Upgrade Center](01_Foundations/08_Release_Cycle_and_Upgrade_Center.md) — half-yearly releases, universal vs opt-in, the four-week release routine
- [Admin tools cheat sheet](01_Foundations/09_Admin_Tools_Cheat_Sheet.md) — "I want to do X → open tool Y"
- **[Foundations Q&A](01_Foundations/Interview_Questions_and_Answers.md)** — 33 questions, tagged by frequency

### 02_Employee_Data — how one human being is stored

- [Person, User and Employment](02_Employee_Data/01_Person_User_Employment_Model.md) — the #1 EC concept, and what happens to IDs in every scenario
- [Effective dating and history](02_Employee_Data/02_Effective_Dating_and_History.md) — time slices, **Insert vs Correct**, future and retroactive dating
- [Biographical and Personal Information](02_Employee_Data/03_Biographical_and_Personal_Information.md) — `personInfo` vs `personalInfo`, global information, name formats, privacy
- [Addresses, contact and National ID](02_Employee_Data/04_Addresses_Contact_and_National_ID.md) — country-specific address formats, multiple national IDs, dependants
- [Employment Information](02_Employee_Data/05_Employment_Information.md) — hire/original start/seniority dates, terminations, contingent workers
- [Job Information](02_Employee_Data/06_Job_Information.md) — the most important element in EC, field group by field group
- [Compensation Information](02_Employee_Data/07_Compensation_Information.md) — pay components, recurring vs non-recurring, FTE and annualisation, compa-ratio
- [Job relationships and dependants](02_Employee_Data/08_Job_Relationships_and_Dependants.md) — how HRBP routing and matrix reporting actually work
- [Employee data imports](02_Employee_Data/09_Employee_Data_Imports.md) — the import sequence, why loads fail, and how to prove a migration worked
- **[Employee Data Q&A](02_Employee_Data/Interview_Questions_and_Answers.md)** — 50 questions, tagged by frequency

---

## Source material

Everything in this repo is written to be read **alongside** the official SAP material:

| Resource | Link |
|---|---|
| SAP Learning — SuccessFactors Platform Introduction Academy | https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy |
| SAP Learning — Employee Central Core Academy | https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy |
| SAP Learning — EC Position Management Academy | https://learning.sap.com/courses/sap-successfactors-employee-central-position-management-academy |
| SAP Learning Journey — Configuring EC Core and Position Management | https://learning.sap.com/learning-journeys/configuring-sap-successfactors-employee-central-core-and-position-management |
| SAP Help — Recommended implementation sequence | https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core/recommended-implementation-sequence |
| SAP Help — Implementing Employee Central Core | https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core |

**Videos** used throughout the notes:

| Ref | Topic | Link |
|---|---|---|
| V1 | SuccessFactors / EC overview | https://www.youtube.com/watch?v=6xrD8Vkm0QI |
| V2 | Navigation, architecture, transactions | https://www.youtube.com/watch?v=qkmFdj4h4rA |
| V3 | Foundation objects, data models, MDF | https://www.youtube.com/watch?v=yEsquQA-MxU |
| V4 | Business rules | https://www.youtube.com/watch?v=90aPAtJbl9g |
| V5 | Role-based permissions | https://www.youtube.com/watch?v=T4yXgdkpxz8 |

---

## A note on accuracy

SuccessFactors changes twice a year. Tool names, the exact split between Provisioning and Admin Center, and which foundation objects are MDF-based have all moved over time, and will move again. Where something is release-dependent, these notes say so — **always verify against the release running in your instance** and against current SAP Help before quoting a detail in a client meeting.
