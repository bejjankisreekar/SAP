# Provisioning vs Admin Center

## The two control rooms

SuccessFactors has **two** places where the system is configured, and knowing which is which is a genuine consultant skill.

| | **Provisioning** | **Admin Center** |
|---|---|---|
| Who can access | **SAP and certified implementation partners only** — never the customer | Customer admins and consultants with the right permissions |
| Reached at | A separate back-office URL per data centre | Inside the instance, from the Home menu |
| Purpose | Turn features **on**, create/manage tenants, upload some XML, set company-level switches | Day-to-day configuration and administration |
| Risk | High — few guardrails, some switches are irreversible | Lower — validated, permission-controlled, mostly auditable |
| Direction of travel | SAP keeps **moving settings out of it** into Admin Center | Keeps growing |

Analogy: Provisioning is the **electrical panel in the basement** — it decides which circuits exist. Admin Center is the **light switches in the rooms** — it decides how you use them. You do not send the tenant to the basement, and you do not rewire the building to turn on a lamp.

> **Practice-system rule:** do not attempt Provisioning tasks without the required implementation access. If a learning exercise says "in Provisioning…", read it, understand what the switch does, and then do the Admin Center part in your practice system.

---

## What lives in Provisioning (typical)

Exact contents shift release to release as things migrate to Admin Center, but historically:

- **Create / copy / refresh instances**, set company ID, data centre attributes
- **Enable modules and features** — Employee Central, Position Management, Time Off, MDF, Role-Based Permissions, Document Generation, Concurrent Employment, Global Assignment
- **Company Settings checkboxes** — a long list of feature toggles
- **Upload / download the data model XML files** — Succession Data Model, Corporate Data Model, and their country-specific counterparts
- **Manage admin (superuser) accounts** and password policies at company level
- **Enable API access / SFAPI**, set login exceptions
- **Turn on the Employee Central "V2" style features**, Business Configuration UI itself
- Housekeeping: purge, job scheduling at platform level

Two things about Provisioning that matter in practice:

1. **Some switches cannot be turned back off** (or turning them off breaks data). Enabling Concurrent Employment or a data-model change that removes fields are classic examples. Read the note, and test in DEV first.
2. **There is no undo and thin validation.** Uploading a malformed data model can leave the instance in a bad state. Always **download the current XML first** — that download *is* your backup.

---

## What lives in Admin Center

Everything you should be doing 95% of the time. Highlights by area:

| Area | Tools |
|---|---|
| **Data model (UI)** | Manage Business Configuration (BCUI) — edit HRIS elements and fields without touching XML |
| **Foundation data** | Manage Data (MDF FOs), Manage Organization/Pay and Job Structures (legacy FOs), Import and Export Data |
| **MDF** | Configure Object Definitions, Manage Data, Configure Configuration UI, Manage Object Definition Security |
| **Rules** | Configure Business Rules, Manage Business Configuration (to attach rules), rule execution log |
| **Workflows** | Manage Data → Workflow Configuration; Manage Workflow Requests; Manage Workflow Groups |
| **Security** | Manage Permission Roles, Manage Permission Groups, RBP settings, Manage Login Accounts |
| **Employee data** | Import Employee Data, Export Extracts, Manage Pending Hires, Data Retention Management |
| **Picklists** | Picklist Center (MDF picklists) / Manage Picklists (legacy) |
| **Profile & UI** | Configure People Profile, Manage Home Page, Manage Configuration UI, Theme Manager |
| **Integration** | Integration Center, API Center, OData API Data Dictionary, OData audit log |
| **Monitoring** | Check Tool, Execution Manager / scheduled job monitor, Admin Alerts |
| **Release** | Upgrade Center, What's New Viewer |

---

## The XML question everyone asks

**"Do I have to use Provisioning to change the data model?"**

Historically yes: download XML → edit → upload. Today, most day-to-day data-model changes are done in **Manage Business Configuration (BCUI)** in Admin Center — adding a field to `personalInfo`, changing a label, making a field required, attaching a rule.

You still need XML (and therefore a partner) for:

- Structural changes BCUI does not expose
- Bulk edits where hand-editing XML is genuinely faster
- Country-specific data models in some releases
- Reading the *current* configuration as one document, which is invaluable for documentation and comparison

**The professional habit:** even when you configure in BCUI, **export the XML afterwards** and keep it with the workbook, dated. It is your only real record of what the model looked like on any given day.

---

## Step by step — a safe data-model change

1. **Document the intent** — which element, which field, why, which release.
2. **Download the current XML** (Provisioning or BCUI export). Save it as `SDM_before_YYYYMMDD.xml`.
3. **Make the change in DEV**, preferably in BCUI.
4. **Validate**: open an employee record, confirm the field renders, is labelled correctly, and behaves (required/visibility).
5. **Check RBP** — a new field is invisible until permissioned.
6. **Export the new XML** as `SDM_after_YYYYMMDD.xml` and diff it against the before file. The diff is your change record.
7. **Update the configuration workbook.**
8. **Promote to TEST** via Instance Sync or XML upload; retest there.
9. Only then, **PROD**.

Steps 2 and 6 are the ones beginners skip and seniors never do.

---

## Interview-grade Q&A

- **What is Provisioning and who can use it?** The back-office tool for tenant management and feature enablement, restricted to SAP and certified implementation partners — never customers.
- **Give three things you can only do in Provisioning.** Create/copy instances, enable core features such as Employee Central or Concurrent Employment, and upload data-model XML files (release-dependent).
- **What is BCUI?** Manage Business Configuration — the Admin Center UI for editing HRIS elements and fields without editing XML.
- **What do you do before uploading a data model?** Download and archive the current XML. That is your backup and your diff baseline.
- **Why is Provisioning risky?** Little validation, limited undo, and some switches are effectively irreversible.
- **How would you document configuration?** A configuration workbook plus dated XML exports and MDF export files, updated the same day changes are made.

---

## Further learning

- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy)
- Video — [Navigation and architecture](https://www.youtube.com/watch?v=qkmFdj4h4rA)
