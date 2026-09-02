# Business Rules — Interview Questions & Answers

**[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## Fundamentals

**1. What is a business rule in SuccessFactors? [HIGH]**
Configured If/Then/Else logic, built in Configure Business Rules and attached to an HRIS element, a field, an MDF object or a process, used to default, derive, validate, control visibility and required flags, and trigger workflows.

**2. What do business rules replace from SAP HCM? [HIGH]**
Dynamic actions, features (PINCH, ABKRS and similar) and small user exits — because there is no ABAP layer in the cloud.

**3. What can rules not do? [MED]**
Call external systems, perform unbounded loops or arbitrary computation. Those requirements need an integration or an extension on SAP BTP.

**4. What are the components of a rule? [HIGH]**
Rule id and name, description, scenario, base object and parameters, effective start date, and the If/Then/Else logic.

**5. Are business rules effective-dated? [MED]**
Yes. A rule can be introduced from a future date, and it evaluates against the effective date of the record being processed — which matters for retroactive and future-dated transactions.

**6. Why is null handling so important? [HIGH]**
Fields are frequently empty when a rule runs, especially at onInit and onChange before the user has completed the screen. Without null guards the conditions behave unexpectedly and defaults can overwrite existing values.

**7. Should a rule compare to a picklist label or its external code? [HIGH]**
The external code. Labels are display text, are localised, and can be changed — a rule written against a label will eventually fail silently.

**8. When would you NOT write a rule? [HIGH]**
When propagation, a required flag, a unique constraint, max length or field criteria achieve the same result declaratively — those are cheaper, faster and need no maintenance.

---

## Scenarios

**9. What is a rule scenario and why does it matter? [HIGH]**
The template chosen at creation; it fixes the base object, the additional entities available, where the rule may be attached, and which functions are offered. It cannot be changed later — a wrong choice means rebuilding the rule.

**10. What is a base object? [HIGH]**
The primary record the rule reads and writes, normally the element the rule is attached to.

**11. What is a cross-entity rule and when do you need one? [HIGH]**
A rule whose scenario exposes several employee entities at once — needed when a change in one entity must read or write another, such as deriving pay grade in Compensation from a job change in Job Information, or checking a work permit during a transfer.

**12. How do you choose a scenario? [HIGH]**
Write the requirement naming every entity involved, list them, decide where the rule must be attached, choose the scenario covering all of them, and verify a stub rule can actually be attached there before writing the logic.

---

## Trigger points

**13. Name the trigger points and when each fires. [HIGH]**
onInit when the screen or transaction initialises; onChange when a specific field's value changes; onSave when the user saves, before the record is written; onPostSave after the record is committed.

**14. Which trigger can block a save? [HIGH]**
onSave — it is the only one that can raise a blocking error.

**15. Why is onChange unreliable for mandatory logic? [HIGH]**
It fires only on UI field changes, so imports and API writes bypass it. Anything that must always be true belongs at onSave.

**16. What is the risk with onInit? [MED]**
Nearly all fields are empty at that point, and without a null guard the rule can overwrite values that were deliberately set.

**17. What is onPostSave for? [MED]**
Actions requiring the record to exist — creating related records or triggering follow-on processes. It cannot block anything.

**18. Do rules run during imports and API writes? [HIGH]**
Imports depend on the import's rule setting, and are usually run with rules off during migration so defaults do not overwrite migrated values. OData writes generally honour save-time validation but never fire onChange.

---

## Attaching and sequencing

**19. Where do you attach a rule to Job Information? [HIGH]**
Manage Business Configuration → the `jobInfo` HRIS element, choosing the trigger and sequence.

**20. Where do you attach a rule to an MDF object? [HIGH]**
Configure Object Definitions → the object (save/validate/initialize rules) or a specific field.

**21. Element-level vs field-level attachment? [HIGH]**
Element-level fires for the whole block at the chosen trigger and suits validation and cross-field logic; field-level fires when that specific field changes and suits dependent derivation.

**22. What determines the order rules run in? [HIGH]**
The configured sequence at each trigger. Derivation must be sequenced before any validation that depends on it, and event-reason derivation before workflow derivation.

**23. What happens if two rules write the same field? [HIGH]**
They execute in sequence and the last one wins, producing results that look random. The fix is to consolidate them into one rule.

**24. How does a rule trigger a workflow? [HIGH]**
A workflow-derivation rule at onSave returns a workflow configuration; the change then becomes pending data until approved. If no workflow is returned, the change applies immediately.

**25. My rule isn't firing — what do you check first? [HIGH]**
Whether it is attached at all, and at which element and trigger.

---

## Common rules

**26. Give three rules you have built. [HIGH]**
Event-reason derivation at onSave on Job Information; pay-range validation raising an error naming the actual amounts; pay-group derivation from legal entity and employee class.

**27. How would you validate a salary against a pay range? [HIGH]**
A cross-entity onSave rule on Compensation that looks up the pay range by grade, geozone and currency, raises an error outside min/max, optionally warns near the top of the range, and allows a role-based exception where the customer requires one.

**28. How do you default a probation end date? [HIGH]**
An onInit or onChange rule adding the country-specific period to the hire date, guarded so it fires only when the field is null.

**29. Why does the null guard matter on a defaulting rule? [HIGH]**
Without it, the rule overwrites deliberately set values — most damagingly during migration, where real seniority dates are replaced by hire dates and employees silently lose years of service.

**30. How would you handle a mapping with 40 combinations, such as pay group? [HIGH]**
A small configuration object holding the mapping plus a lookup rule, so HR can maintain new combinations without a consultant — not a 40-branch If/Else.

**31. How do you make a field conditionally mandatory? [MED]**
An onChange rule that sets the target field's required flag and visibility based on another field's value.

**32. How would you alert HR 90 days before a work permit expires? [HIGH]**
An alert/notification with a rule condition, run by a scheduled job — not an onSave rule, because nothing is being saved at that moment.

**33. How do you limit retroactive dating for managers? [MED]**
An onSave rule comparing the effective date against today minus the allowed window and raising an error, combined with RBP so HR retains the ability.

---

## Event reason derivation

**34. How do you derive an event reason? [HIGH]**
A single onSave rule on Job Information structured as a decision tree that compares the incoming values with the previous effective-dated record and sets event and reason from the most significant change, with a final ELSE so no row is left blank.

**35. Why does the order of conditions matter? [HIGH]**
Changes overlap. A promotion that also changes department must be recorded as a promotion, so the job/grade branch must be evaluated before the department branch.

**36. Should users choose the event reason themselves? [HIGH]**
Usually a hybrid: derive a default for consistency, allow HR to override within a cascading filtered list, and validate the override — because some distinctions cannot be inferred from the data.

**37. How do you stop "Promotion" being used without a job change? [HIGH]**
A validation rule raising an error when the reason is Promotion but neither job classification nor pay grade changed.

**38. What breaks when event reasons are wrong? [HIGH]**
Headcount and turnover reporting, promotion analytics, workflow routing, employee status, retro payroll behaviour and downstream replication.

**39. How do you test event reason derivation? [HIGH]**
A matrix of transaction types, including combination cases — a promotion that also changes department is the case that exposes ordering bugs.

---

## Debugging

**40. A rule is not working. Walk me through your diagnosis. [HIGH]**
Is it attached; at the right trigger; on the right element; is the condition actually true (nulls, code vs label, effective dates); is another rule or propagation overwriting the result; is the sequence correct; did the data arrive via a path where rules do not run; is the rule effective-dated out of range; is a field permission hiding what the rule reads; is the rule active.

**41. What tool shows which rules fired? [HIGH]**
The business rule execution log / rules trace, where available in the release; it lists which rules ran, in what order, with their outcomes.

**42. How do you isolate a faulty condition? [MED]**
Bisection — replace the condition with an always-true version to confirm attachment and trigger, then reintroduce conditions one at a time.

**43. A rule works in the UI but not overnight. Why? [HIGH]**
The overnight data arrives by import or API: imports may run with rules disabled, and onChange rules never fire for non-UI paths.

**44. A rule works for you but not for a manager. Why? [HIGH]**
Field-level permissions — a field the rule reads is not granted to that role, so it evaluates as null and the condition fails.

**45. What is the single most common cause of rule failure? [HIGH]**
The rule is not attached, or is attached at the wrong trigger.

---

## Scenario questions

**46. Requirement: when a manager transfers someone to a different legal entity, derive the event reason, default the pay group, and block the save if the salary is outside the new grade's range. Design it. [HIGH]**
A cross-entity EC scenario with Job Information as base and Compensation in scope. Three rules on `jobInfo`/`compInfo` at onSave, sequenced: (10) derive pay grade and pay group from the new company and job; (20) derive event and event reason from the company change; (30) validate the salary against the pay range for the new grade and geozone, raising an error with the actual figures. Then a workflow-derivation rule sequenced last so routing uses the derived reason. Test with a transfer that is also a promotion, and test the import path.

**47. The customer complains that 60% of Job Information rows are "Data Change". What do you do? [HIGH]**
Rationalise the event reason catalogue with HR and Payroll, build a derivation rule so the reason is inferred rather than chosen, make the field read-only for managers with HR override within a filtered list, add a validation rule on overrides, and accept that historical data cannot be reconstructed — flagging the cut-over date in reporting.

**48. A defaulting rule wiped out migrated seniority dates. What went wrong and how do you prevent it? [HIGH]**
The rule had no null guard and the migration ran with rules enabled. Fix by adding the guard, correcting the affected records from the migration source, and adopting the standard practice of running migration loads with rules off plus validating in the mapping instead.

**49. Two consultants each wrote a rule that sets the cost centre. What do you do? [MED]**
Identify both attachments and their sequence, decide which behaviour is correct with the customer, consolidate into a single rule, remove the other, retest, and record the decision in the configuration workbook.

**50. How would you keep 90 rules maintainable? [MED]**
One rule per purpose, a systematic naming convention encoding element/action/target, mandatory descriptions, a documented table of element/trigger/sequence/purpose, preference for propagation and field settings over rules, configuration objects instead of long branch chains, and a standing set of test employees run against every change.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Creating business rules for EC
- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy) — MDF and business rules
- Video — [Business rules](https://www.youtube.com/watch?v=90aPAtJbl9g)
