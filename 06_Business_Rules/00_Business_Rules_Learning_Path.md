# 06 Business Rules — Learning Path

Business rules are **the logic layer of Employee Central**. They are what replaced dynamic actions, features and user exits from SAP HCM — and because there is no ABAP in the cloud, they are the *only* place conditional behaviour lives.

An EC consultant who cannot write and debug rules is not an EC consultant.

| # | File | What you get out of it |
|---|---|---|
| 1 | [Rules engine fundamentals](01_Rules_Engine_Fundamentals.md) | If/Then structure, the rule editor, functions and operators |
| 2 | [Rule scenarios and contexts](02_Rule_Scenarios_and_Contexts.md) | Choosing the right scenario — the decision that determines everything else |
| 3 | [Trigger points](03_Trigger_Points_onInit_onChange_onSave.md) | onInit, onChange, onSave, onPostSave — when each fires and why it matters |
| 4 | [Attaching rules](04_Attaching_Rules.md) | Element level, field level, MDF objects, and rule sequencing |
| 5 | [Common EC rule recipes](05_Common_EC_Rule_Recipes.md) | The dozen rules every project builds, written out |
| 6 | [Event reason derivation](06_Event_Reason_Derivation.md) | The single most-asked rule requirement, in depth |
| 7 | [Debugging rules](07_Debugging_Rules.md) | Why a rule did not fire, in a defined order of checks |
| — | [Interview Q&A](Interview_Questions_and_Answers.md) | Questions covering this whole folder |

**Milestone:** given a requirement in plain English ("when a manager transfers someone to a different legal entity, derive the event reason, default the pay group, and stop them saving if the salary is outside the new grade's range"), choose the scenarios, write the rules, attach them at the right triggers, and prove they fire — and prove they *don't* fire when they shouldn't.

**Source material for this folder**

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for Employee Central
- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF and business rules
- Video — [Business rules](https://www.youtube.com/watch?v=90aPAtJbl9g)

*(Original plan: Days 19–20.)*
