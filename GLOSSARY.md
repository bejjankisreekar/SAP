# Glossary — SAP SuccessFactors Employee Central

Every recurring term in one place, so the topic notes don't have to keep re-explaining them. Alphabetical within groups.

---

## Products and platform

| Term | Meaning |
|---|---|
| **SAP SuccessFactors** | SAP's cloud HXM suite (SaaS) covering core HR, talent, learning and analytics |
| **HXM / HCM** | Human eXperience Management — the suite's name since 2019. Used interchangeably with HCM |
| **Employee Central (EC)** | The core HR module: the effective-dated system of record for person, employment, job, pay and org data |
| **EC Payroll (ECP)** | SAP payroll engine delivered in the cloud, fed by replication from EC |
| **SAP HCM (on-premise)** | The classic ECC/S4 HR modules: PA, OM, PY, PT. EC is the cloud successor to PA and OM |
| **Instance / tenant** | One isolated copy of the application with its own configuration and data |
| **Company ID** | The unique identifier of a tenant, entered at login |
| **Provisioning** | Partner-only back-office tool: creates tenants, enables features, uploads data-model XML. Customers have no access |
| **Admin Center** | The in-instance configuration cockpit used by admins and consultants |
| **Instance Sync** | Tool that copies supported configuration artefacts from one tenant to another |
| **Upgrade Center** | Admin Center tool listing optional and required release upgrades |
| **What's New Viewer** | SAP's searchable release-notes site |
| **Check Tool** | Configuration health-check tool; run it after every release and before go-live |
| **Proxy** | Working as another user to reproduce their exact experience |

---

## Data model

| Term | Meaning |
|---|---|
| **Corporate Data Model (CDM)** | XML defining foundation objects (organisation, job and pay structures) |
| **Succession Data Model (SDM)** | XML defining employee-level HRIS elements and background (talent) elements |
| **CSF-CDM / CSF-SDM** | Country/region-specific versions of the two models above |
| **HRIS element** | A block of employee data (`personalInfo`, `jobInfo`, `compInfo`…) |
| **HRIS field** | One field inside an HRIS element, with id, label, visibility, required flag |
| **`visibility`** | Field attribute: `both` (view+edit), `view`, `edit`, or `none` |
| **BCUI / Manage Business Configuration** | Admin Center UI for editing the data model without touching XML |
| **Background element** | A talent-profile list (education, work experience, certifications) defined in the SDM |
| **Effective dating** | Records valid from a start date until superseded, giving full history and "as of" reads |
| **Insert vs Correct** | Insert creates a new time slice (a real change); Correct overwrites the current slice (a data-entry fix) |
| **Time slice** | One effective-dated row, valid from its start date to the day before the next row |

---

## Employee data

| Term | Meaning |
|---|---|
| **Person** | The human being; keyed by `person_id_external` |
| **Employment** | One working relationship; keyed by `user_id` / `assignment_id_external` |
| **User record** | The suite-level login identity (username, status, locale, manager) |
| **`person_id_external`** | The person key used by all person-level data |
| **`user_id`** | The employment/user key used by job, pay, workflows and permissions |
| **Biographical Information (`personInfo`)** | Person-level, NOT effective-dated: date of birth, gender, place of birth |
| **Personal Information (`personalInfo`)** | Person-level, effective-dated: name, marital status |
| **Global Information** | Country-specific person data, effective-dated |
| **Employment Information (`employmentInfo`)** | Hire date, original start date, seniority date, termination details |
| **Job Information (`jobInfo`)** | Effective-dated: org assignment, job, manager, FTE, status, event/event reason |
| **Compensation Information (`compInfo`)** | Effective-dated header for pay: pay group, grade, currency, frequency |
| **Pay component** | One element of pay (base salary, allowance, bonus) — the equivalent of a wage type |
| **Pay component group** | An aggregation of pay components for totals (e.g. Total Base Pay) |
| **Recurring / non-recurring** | Paid every period vs a one-off with a payment date |
| **FTE** | Full-time equivalent (1.0 = full time) |
| **Compa-ratio** | Salary ÷ pay-range midpoint |
| **Job relationship** | A non-line-manager relationship (HR Manager, matrix manager) used for routing and permissions |
| **Dependant / Person Relationship** | A family member, used by Benefits, payroll allowances and documents |
| **Concurrent employment** | Two or more active employments for one person at the same time |
| **Global assignment** | A second employment for an international assignment, alongside the home employment |
| **Contingent worker** | A non-employee worker (contractor, agency staff) held in EC with a distinct worker type |

---

## Foundation and MDF

| Term | Meaning |
|---|---|
| **Foundation Object (FO)** | Shared organisational, job or pay structure data that employee records point at |
| **Legal Entity / Company** | The legal employer; drives country and country-specific behaviour |
| **Business Unit / Division / Department** | The organisational hierarchy above the employee |
| **Cost Centre** | The finance object the employee's cost is booked to |
| **Location / Location Group / Geozone** | Where the employee works; geozone groups locations for pay ranges |
| **Job Classification** | The catalogue job (job code) |
| **Job Function / Job Family** | Groupings of job classifications for career paths and reporting |
| **Pay Grade / Pay Range** | The grade, and its min/mid/max per geozone and currency |
| **Pay Group** | Which payroll run an employee belongs to |
| **Event / Event Reason** | The category of change and the specific reason for it, stored on the Job Information row |
| **MDF (Metadata Framework)** | The platform for defining objects, fields, associations, UIs and rules without code |
| **Generic Object (GO)** | A record of an MDF object |
| **Object Definition** | The MDF definition of an object's fields, keys, associations and security |
| **Association** | A defined relationship between objects (one-to-many, many-to-many) that constrains valid combinations |
| **Propagation** | Automatically copying a value from an associated object into a field |
| **Configuration UI** | The configurable screen layout for an MDF object |
| **Manage Data** | Admin tool for creating and editing MDF/foundation records |
| **Configure Object Definitions** | Admin tool for defining MDF objects |
| **Picklist** | A configurable list of values with external codes and translations |
| **External code** | The stable key of a picklist value or MDF record — what imports and integrations match on |
| **Field criteria** | Configuration that filters one field's values based on another field |

---

## Process and security

| Term | Meaning |
|---|---|
| **Business rule** | If/Then logic configured in the rules engine and attached to an element, field or object |
| **Rule scenario** | The template determining a rule's purpose and available base objects |
| **onInit / onChange / onSave / onPostSave** | Trigger points: screen load, field change, before save, after save |
| **Workflow** | A configured approval chain triggered by a data change |
| **Pending data** | A change awaiting approval; the live record is unchanged until it completes |
| **Approver type** | How a workflow step finds its approver: role, dynamic role, dynamic group, position relationship, job relationship |
| **CC role / contributor** | Workflow participants who are informed or asked to comment rather than approve |
| **Delegation / escalation** | Passing approval to another person, or auto-advancing after a timeout |
| **RBP (Role-Based Permissions)** | The security model: roles (what) granted to groups (who) over a target population (about whom) |
| **Permission group** | A set of users, defined statically or by dynamic criteria |
| **Permission role** | A set of permissions, granted to one or more groups |
| **Granted user** | The group receiving a role |
| **Target population** | The population the granted users may act on (all, a group, or a relationship such as manager hierarchy) |
| **ESS / MSS** | Employee self-service / manager self-service |

---

## Position Management

| Term | Meaning |
|---|---|
| **Position** | An MDF object representing a seat in the organisation, independent of who fills it |
| **Incumbent** | The employee currently holding the position |
| **To Be Hired** | Flag marking a vacant position that should be filled |
| **Position hierarchy** | Position-to-parent-position reporting structure, an alternative to the manager hierarchy |
| **Position–Job Info synchronisation** | Copying values between the position and the employee's Job Information, in one or both directions |
| **Position control / FTE control** | Restricting how many people (or how much FTE) a position may hold |
| **Mass copy / reclassification** | Bulk creation or change of positions |

---

## Integration and reporting

| Term | Meaning |
|---|---|
| **OData API** | The REST-style API exposing EC entities for read and write |
| **Entity** | An OData object such as `User`, `PerPerson`, `EmpJob`, `EmpCompensation`, `FOCompany` |
| **Upsert** | Insert-or-update — the standard OData write operation in SuccessFactors |
| **`$filter` / `$select` / `$expand`** | OData query options: filter rows, choose fields, include related entities |
| **`asOfDate` / `fromDate` / `toDate`** | Effective-dating parameters on OData reads |
| **Compound Employee API** | SOAP API returning a complete, nested employee record; used for payroll replication |
| **Delta / full transmission** | Sending only changes since the last run vs sending everything |
| **Integration Center** | Admin tool for building and scheduling file- and API-based integrations without middleware |
| **SAP Integration Suite / CPI** | SAP's middleware, used for packaged integration content such as EC-to-payroll replication |
| **Data Replication Monitor** | Tool for monitoring and re-running employee data replication to payroll |
| **Story report** | The current reporting tool in People Analytics |
| **API Center** | Admin tool for OAuth clients, API audit logs and the OData data dictionary |

---

## Delivery

| Term | Meaning |
|---|---|
| **Configuration workbook** | The written record of every configuration decision — your substitute for a transport request |
| **Unit / string / UAT testing** | Testing one change · a chain of changes end to end · the customer testing their own processes |
| **Cutover** | The sequenced go-live activity: final loads, integration switching, access opening |
| **Hypercare** | The intensive support period immediately after go-live |
| **Data Retention Management / purge** | Deleting or anonymising personal data according to policy |
