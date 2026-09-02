# Admin tools cheat sheet — "I want to do X → open tool Y"

Everything below is reached by typing the tool name into **Action Search** or finding it in **Admin Center**. Tool names occasionally change with releases; search by keyword rather than by menu path.

---

## Data model and business configuration

| I want to… | Tool |
|---|---|
| Add a field to Personal Info / Job Info, change a label, make a field required | **Manage Business Configuration** (BCUI) |
| Attach a business rule to an HRIS element or field | **Manage Business Configuration** |
| See or edit the raw Succession / Corporate Data Model XML | **Provisioning** (partner only) — download, edit, upload |
| Change which blocks appear on the employee profile | **Configure People Profile** |
| Change the layout of an MDF object screen | **Manage Configuration UI** |

## Foundation data

| I want to… | Tool |
|---|---|
| Create a Legal Entity, Business Unit, Department, Cost Centre, Location, Job Classification, Pay Grade | **Manage Data** (they are MDF objects) |
| Maintain Pay Component, Pay Component Group, Frequency, Event Reason, Dynamic Role (legacy FOs) | **Manage Organization, Pay and Job Structures** |
| Bulk-load foundation data | **Import and Export Data** (MDF) / **Import Foundation Data** (legacy) |
| Check what associations exist between FOs | **Configure Object Definitions** (MDF) or the Corporate Data Model (legacy) |

## Employee data

| I want to… | Tool |
|---|---|
| Load employees in bulk (hires, job data, comp data) | **Import Employee Data** |
| Export employee data as a file | **Import Employee Data → Export** / **Employee Export** |
| See hires waiting to be completed from Recruiting/Onboarding | **Manage Pending Hires** |
| Delete or anonymise employee data for GDPR | **Data Retention Management / Data Purge** |
| Fix a wrong record | The employee's block → **History → Correct** (or Insert) |

## Rules and process

| I want to… | Tool |
|---|---|
| Create or edit a business rule | **Configure Business Rules** |
| See why a rule did or did not fire | **Business Rule Execution Log / Rules Trace** (naming varies by release) |
| Create an approval workflow | **Manage Data → Workflow Configuration** |
| See or reassign pending approvals | **Manage Workflow Requests** |
| Group workflow steps / mass-manage approvers | **Manage Workflow Groups** |
| Alert HR when something happens (e.g. probation ending) | **Manage Data → Alerts & Notifications / Intelligent Services** |

## MDF

| I want to… | Tool |
|---|---|
| Create a custom object | **Configure Object Definitions** |
| Create or edit records of an object | **Manage Data** |
| Design the screen for an object | **Manage Configuration UI** |
| Import/export object definitions and records | **Import and Export Data** |
| Secure an object | **Configure Object Definitions** → set security, then permission it in RBP |
| Manage picklists | **Picklist Center** (MDF picklists) |

## Security

| I want to… | Tool |
|---|---|
| Create a group of users | **Manage Permission Groups** |
| Grant permissions | **Manage Permission Roles** |
| See what a specific user can do / why they cannot see something | **View User Permission** (RBP troubleshooting) |
| Create login accounts / reset passwords | **Manage Login Accounts / Reset User Passwords** |
| Let one user act as another | **Manage Proxies** |

## Integration and reporting

| I want to… | Tool |
|---|---|
| Build a scheduled outbound file (CSV/XML/JSON) from EC data | **Integration Center** |
| Look up which OData entity holds a field | **OData API Data Dictionary** |
| Test an OData call, manage OAuth clients, check API audit logs | **API Center** |
| Build a report | **Report Center** → Story report / Table report |
| Check scheduled job status | **Scheduled Job Manager / Execution Manager** |

## Health and release

| I want to… | Tool |
|---|---|
| Check the instance for configuration problems | **Check Tool** |
| See and enable optional new features | **Upgrade Center** |
| Read release notes | **What's New Viewer** (external) |
| Copy configuration between instances | **Instance Synchronization** |

---

## The five tools you will use every single day

1. **Manage Data** — foundation and MDF records
2. **Configure Object Definitions** — object structure
3. **Manage Business Configuration** — the employee data model
4. **Configure Business Rules** — the logic
5. **Manage Permission Roles** — why nobody can see anything

If you master those five and Import Employee Data, you can do most of an EC build.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy)
- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy)
- Video — [Navigation and admin tools](https://www.youtube.com/watch?v=qkmFdj4h4rA)
