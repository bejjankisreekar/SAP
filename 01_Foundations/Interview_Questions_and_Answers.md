# Foundations — Interview Questions & Answers

Tagged by how often they come up: **[HIGH]** almost every interview · **[MED]** common · **[LOW]** occasional or senior-level.

---

## Product and positioning

**1. What is SAP SuccessFactors? [HIGH]**
SAP's cloud HXM suite — SaaS modules for core HR, talent, learning and analytics. Employee Central is the core HR module and the system of record.

**2. What is Employee Central? [HIGH]**
The core HR module: an effective-dated system of record for person data, employment data, job data, compensation data and the organisational structure (foundation objects). It is the cloud successor to SAP HCM's PA and OM.

**3. Is EC a payroll system? [HIGH]**
No. It holds pay *data* as pay components and passes it to a payroll engine — EC Payroll, SAP HCM Payroll on-premise, or a third-party provider.

**4. HCM vs HXM? [MED]**
The same suite. Renamed in 2019 to emphasise employee experience.

**5. Name the modules of the suite. [HIGH]**
Core HR: Employee Central, EC Payroll, Time Off/Time Sheet, Benefits, Position Management, Document Generation. Talent: Recruiting, Onboarding, Performance & Goals, Compensation & Variable Pay, Succession & Development, Learning. Plus Platform and People Analytics.

**6. Difference between Employee Profile and Employee Central data? [MED]**
Employee Profile is the talent record — background elements such as education and work experience, defined in the Succession Data Model. EC data is the effective-dated HR record (personal, job, compensation). Both appear in People Profile today.

**7. Can Recruiting or Learning run without EC? [MED]**
Yes. Many customers implement them standalone and load employee data by flat file. EC integration removes the re-keying.

---

## Cloud vs on-premise

**8. How is EC different from SAP HCM on-premise? [HIGH]**
No infrastructure to run, SAP upgrades twice a year, no ABAP — configuration only (XML data models, MDF, business rules, RBP), API-based integration instead of RFC/IDoc, and self-service/mobile built in.

**9. Map the main infotypes to EC. [HIGH]**
IT0000 → Event/Event Reason on Job Information; IT0001 → Job Information; IT0002 → Biographical + Personal Information; IT0006 → Addresses; IT0008 → Compensation Information plus recurring pay components; IT0009 → Payment Information; IT0105 → Email/Phone.

**10. What replaces dynamic actions, features and user exits? [HIGH]**
Business rules (onInit / onChange / onSave / onPostSave), plus MDF where new objects are needed. Anything beyond that goes to an SAP BTP extension outside the product.

**11. What replaces authorization objects? [HIGH]**
Role-Based Permissions: permission roles define *what*, permission groups define *who*, and the target population defines *about whom*.

**12. What replaces wage types? [MED]**
Pay components, organised into pay component groups.

**13. A customer keeps on-premise payroll. How does data reach it? [HIGH]**
Replication from EC using the Compound Employee API through middleware (SAP Integration Suite/CPI), with mapping from EC org objects to HCM's Personnel Area, Org Unit and Cost Centre.

---

## Architecture and landscape

**14. Describe the SuccessFactors landscape. [HIGH]**
Multi-tenant SaaS. Application code is shared per data centre; tenant data is isolated. Customers usually have DEV, TEST and PROD tenants, each with its own company ID.

**15. How do you move configuration between instances? [HIGH]**
No transport requests. Instance Synchronization for supported artefacts; export/import for data-model XML, MDF objects and picklists; manual re-configuration otherwise — always backed by a configuration workbook.

**16. What is a company ID? [LOW]**
The unique identifier of a tenant, entered at login.

**17. What are the risks of refreshing TEST from PROD? [MED]**
Existing TEST configuration is overwritten, and real personal data lands in a lower environment. Mask/anonymise data, restrict access, and disable outbound email first.

**18. How do users authenticate? [MED]**
Normally SAML 2.0 SSO to the customer's identity provider. API access uses OAuth 2.0 with SAML bearer assertion; basic auth only where still permitted.

**19. Why does the data centre matter? [LOW]**
Data residency obligations and release timing are both tied to it.

---

## Provisioning and Admin Center

**20. What is Provisioning? Who has access? [HIGH]**
The partner-only back-office tool used to create and manage tenants, enable features, and upload data-model XML. Customers never have access.

**21. Give examples of Provisioning-only tasks. [MED]**
Creating or copying an instance, enabling Employee Central / MDF / Position Management / Concurrent Employment, uploading the Succession and Corporate Data Models, enabling API access.

**22. What is BCUI? [HIGH]**
Manage Business Configuration — the Admin Center UI for editing HRIS elements and fields (labels, visibility, required, picklists, attached rules) without editing XML.

**23. What do you do before uploading a data model? [HIGH]**
Download and archive the current XML as your backup and diff baseline; make the change in DEV; validate; then promote.

**24. Name five admin tools you use daily. [MED]**
Manage Data, Configure Object Definitions, Manage Business Configuration, Configure Business Rules, Manage Permission Roles.

---

## Release management

**25. How often does SuccessFactors release? [HIGH]**
Twice a year, with a preview window on non-production tenants before the production release.

**26. Universal vs opt-in? [HIGH]**
Universal changes apply automatically to everyone; opt-in features are enabled deliberately by an admin, usually via Upgrade Center.

**27. Can a customer refuse a release? [MED]**
No — only the optional parts.

**28. How do you prepare for a release? [HIGH]**
Filter What's New Viewer to the customer's modules, classify each item as universal / opt-in / deprecation, regression-test in preview, run Check Tool, agree opt-ins, smoke-test production, then enable opt-ins one at a time.

**29. What typically breaks at release time? [MED]**
Integrations, rules that relied on undocumented behaviour, and custom UI or label overrides.

**30. What is Check Tool? [MED]**
A configuration health-check tool that flags inconsistencies; new checks arrive with releases, so run it after every upgrade and before go-live.

---

## Scenario questions

**31. A user says a field is missing from Job Information. Walk me through your diagnosis. [HIGH]**
Is the field in the data model? Is its visibility set to something other than `none`? Does the user's permission role grant the field? Is the block on People Profile? Is a rule hiding it? Is it country-specific and the employee is in another country?

**32. How do you document a configuration? [MED]**
A configuration workbook covering objects, fields, picklists, rules, workflows and roles, plus dated XML and MDF exports taken after each change.

**33. The customer's admin asks for Provisioning access. What do you say? [MED]**
It is not available to customers — it is restricted to SAP and certified implementation partners. Anything they need routinely should be delivered through Admin Center or requested from the partner.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy)
- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [SuccessFactors overview](https://www.youtube.com/watch?v=6xrD8Vkm0QI) · [Navigation and architecture](https://www.youtube.com/watch?v=qkmFdj4h4rA)
