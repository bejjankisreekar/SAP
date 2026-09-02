# EC vs SAP HCM on-premise — the translation table

Employee Central is often introduced as **"the cloud version of SAP HCM"**. That is a useful headline and a dangerous shortcut. This note gives you the honest mapping — what genuinely corresponds, what has no equivalent, and what to say when an interviewer with 15 years of ABAP behind them tests you.

---

## The headline

| SAP HCM on-premise | Employee Central |
|---|---|
| **PA** — Personnel Administration | Employee data (Person, Employment, Job Information, Compensation Information) |
| **OM** — Organizational Management | Foundation Objects + Position Management |
| **PY** — Payroll | *Not in EC.* Either EC Payroll (same engine, cloud-hosted) or the existing on-prem payroll, fed by replication |
| **PT** — Time Management | Time Off / Time Sheet (separate EC sub-modules) |
| **ESS/MSS on NetWeaver Portal** | Built into the platform — no separate build |

**Say this in an interview:** *"EC replaces PA and OM. Payroll and time are separate modules; EC on its own is not a payroll system."*

---

## Infotype → HRIS element mapping

The mapping every ex-HCM person wants. It is approximate — the concepts line up, the field lists do not always.

| Infotype | Name | EC equivalent (HRIS element) | Notes |
|---|---|---|---|
| IT0000 | Actions | **Event / Event Reason** on Job Information | In EC this is not a separate record — it is fields *on* the job row |
| IT0001 | Organizational Assignment | **Job Information** (`jobInfo`) | Company, business unit, department, cost centre, position, manager |
| IT0002 | Personal Data | **Biographical Info** (`personInfo`) + **Personal Info** (`personalInfo`) | EC splits never-changing biographical facts from effective-dated personal data (e.g. name change on marriage) |
| IT0006 | Addresses | **Addresses** (`homeAddress`) | Address type is a picklist; format is country-specific |
| IT0008 | Basic Pay | **Compensation Info** (`compInfo`) + **Recurring Pay Components** | Salary is a pay component, not a single field |
| IT0009 | Bank Details | **Payment Information** | MDF-based in current releases |
| IT0014 / IT0015 | Recurring / Additional Payments | **Recurring / Non-Recurring Pay Components** | Same distinction, same purpose |
| IT0016 | Contract Elements | Fields on **Employment Information** / Job Information | Probation period, notice period |
| IT0021 | Family / Dependants | **Dependants** (Person Relationship) | Person-level |
| IT0032 | Internal Data | Custom fields on the relevant element | |
| IT0041 | Date Specifications | Fields on **Employment Information** (seniority date, original start date) | |
| IT0105 | Communication | **Email Info**, **Phone Info**, plus `username` on the user record | |
| IT0185 | Personal IDs | **National ID Card** | Country-specific card types |
| IT0709 | Person ID | **`person_id_external`** | The person key |

Structural mapping:

| SAP HCM | EC |
|---|---|
| Personnel Area / Subarea | Legal Entity + Business Unit / Division / Department (design choice) |
| Employee Group / Subgroup | Employee Class, Employment Type, Pay Group (design choice) |
| Org Unit (O) | Department (Foundation Object) — *and/or* the position hierarchy |
| Position (S) | **Position** (MDF object) |
| Job (C) | **Job Classification** (Foundation Object) |
| Cost Centre | **Cost Centre** (Foundation Object, or replicated from Finance) |
| Pay Scale Group / Level | **Pay Grade / Pay Range** |
| Wage Type | **Pay Component** |
| Action / Reason for Action | **Event / Event Reason** |
| Dynamic Actions | **Business Rules** (onSave / onChange) |
| Features (PINCH, ABKRS…) | **Business Rules** |
| User Exits / BAdIs / ABAP | **Business Rules**, MDF, or an SAP BTP extension |
| Authorization objects (P_ORGIN…) | **Role-Based Permissions** (roles + groups + target population) |
| Report Painter / ad hoc query | **Story reports / People Analytics** |
| ALE / IDoc | **OData API**, Compound Employee API, Integration Center |

---

## Concepts that genuinely have no counterpart

**In EC but not in classic HCM**

- **MDF (Metadata Framework)** — build your own object with fields, associations, UI and rules, without a developer. The closest ABAP analogue is "create a Z-table plus a maintenance view plus a screen", except it takes an afternoon.
- **Foundation Object associations** — the *object model itself* enforces which departments belong to which business unit; no derivation code required.
- **Configuration UI** — screen layout as configuration.
- **Integration Center** — build and schedule a file-based interface from the UI, with no middleware.

**In HCM but not in EC**

- **Payroll schema and PCRs** — nothing in EC resembles them. That logic lives in ECP or the on-prem system.
- **Infotype screen enhancements in ABAP** — replaced by Configuration UI and rules, with a hard ceiling.
- **Full authorisation flexibility of structural + context authorisations** — RBP is powerful but different in shape.
- **Direct table access (SE16)** — you have OData and reports, not tables.

---

## What actually surprises HCM veterans

1. **There is no "action" record.** Event and Event Reason are *fields on the Job Information row*, so the history of actions and the history of job data are one and the same list.
2. **Salary is a child record.** Compensation Information is a header (pay group, target pay, frequency) with **pay components** underneath, more like IT0008 subtypes than a single amount.
3. **Time-constraint behaviour differs.** EC records are effective-dated with a start date; the end date is implied by the next record. Delimiting is implicit, not a field you type.
4. **Correcting versus inserting matters enormously.** "Insert new record" creates history; "Correct" (Make Correction) rewrites the existing row. Choosing wrong is the most common data defect in a project.
5. **You cannot hire into structures that do not exist.** Foundation data is a prerequisite, always loaded first.
6. **Config is data.** Data models are XML you upload; MDF objects are records you can import. Transport happens via Instance Sync or manual re-import, not via a transport request. See [Provisioning vs Admin Center](07_Provisioning_vs_Admin_Center.md).

---

## Real world example — a hybrid landscape

A manufacturer keeps SAP HCM payroll for Germany (works council, complex collective agreements) but moves core HR to EC for all 14 countries.

- EC is master for person, employment, job and org data.
- Employee master data replicates to on-prem HCM through **SAP Integration Suite** using the **Compound Employee API**.
- Org assignment mapping translates EC's Legal Entity/Department/Cost Centre into HCM's Personnel Area/Org Unit/Cost Centre.
- Payroll results flow *back* to EC only as reporting data, not as master data.
- Nobody maintains employees in HCM any more; the PA transactions are locked down.

The mapping table above is literally the design document for that interface. This is why interviewers care about it.

---

## Interview-grade Q&A

- **Is EC the cloud version of SAP HCM?** It is the cloud successor to PA and OM. Payroll and time are separate modules, and there is no ABAP layer.
- **Map IT0001 and IT0008 to EC.** IT0001 → Job Information; IT0008 → Compensation Information plus recurring pay components.
- **What replaces dynamic actions and features?** Business rules — onChange/onSave for dynamic-action behaviour, and rules for defaulting where features were used.
- **What replaces authorisation objects?** Role-Based Permissions: permission roles (what) granted to permission groups (who) over a target population (about whom).
- **What replaces wage types?** Pay components, grouped into pay component groups.
- **How does EC handle actions?** Event and Event Reason fields on each effective-dated Job Information row — there is no separate action infotype.
- **A customer keeps on-premise payroll. How does data get there?** Replication from EC via the Compound Employee API through middleware (SAP Integration Suite), with org-assignment mapping.

---

## Further learning

- SAP Help — [Employee Central Core implementation](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- SAP Help — [Recommended implementation sequence](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core/recommended-implementation-sequence)
- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [EC data and navigation walkthrough](https://www.youtube.com/watch?v=qkmFdj4h4rA)
