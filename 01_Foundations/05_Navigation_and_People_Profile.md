# Navigation and People Profile

This is the "where do I click" note. Learn it once and every later exercise gets faster. Exact wording varies slightly by release and by how the customer has branded the instance — the *structure* below is stable.

---

## The global header

| Element | What it is | Consultant use |
|---|---|---|
| **Home menu (top-left dropdown)** | Switches between Home, Employee Files, Admin Center, My Employee File, module landing pages | Your main navigation |
| **Action Search / global search** | Type an employee name *or an admin tool name* | The fastest way to work: type "Manage Data", "Import Employee Data", "Configure Business Rules" |
| **Notifications** | Approvals and alerts waiting for you | Where a workflow lands for the approver |
| **Name / photo menu** | Proxy Now, Settings, Sign out | **Proxy** is how you test another user's experience |
| **Support / help** | Contextual help, sometimes a chatbot | |

**Action Search is the single most useful habit.** Do not memorise menu paths; memorise *tool names* and search for them.

---

## The main areas

### 1. Home page

Cards and tiles: To-Do (approvals), quick actions, org chart, announcements, plus module tiles. Configured by admins (home page tiles / "Manage Home Page"). Not where configuration work happens.

### 2. Employee Files / People Profile

The employee record. One scrolling profile with sections; each section is a **block**.

Typical blocks in an EC-enabled instance:

- **Biographical Information** — person-level, not effective-dated
- **Personal Information** — effective-dated (name, marital status, etc.)
- **Addresses**, **Email**, **Phone**, **Social Accounts**, **National ID**, **Emergency Contact**, **Dependants**
- **Employment Information** — hire date, seniority, probation
- **Job Information** — the big one: position, department, manager, FTE, event/event reason
- **Compensation Information** — pay components, pay grade, frequency
- **Work Permit**, **Global Assignment**, **Job Relationships**
- Talent blocks: Background (education, work experience), competencies, career goals

Which blocks appear, and which fields inside them, comes from the **data model** plus **RBP**. If a user cannot see a block, the answer is one of those two, roughly 90% of the time.

### 3. Take Action menu

On an employee's profile, the **Take Action** button is where transactions start:

- Change Job and Compensation
- Manage Worker → Terminate, Rehire, Add Global Assignment, Manage Concurrent Employment
- Change Employee Status
- Generate Document
- Add / edit specific blocks

What appears here is driven by RBP permissions and by which sub-modules are enabled.

### 4. Admin Center

The configuration cockpit. Everything a customer admin or consultant can do without Provisioning. Grouped roughly into:

- **Company Processes & Cycles** — workflows, business rules, data models via Business Configuration UI
- **Employee Files** — import/export employee data, manage pending hires, data purge
- **Company Settings** — home page, logo, email notifications, password policy
- **Extensions** — MDF: Configure Object Definitions, Manage Data, Configure Configuration UI
- **Integration Tools** — Integration Center, API Center, OData audit log
- **Security** — Manage Permission Roles / Groups, RBP settings
- **Reporting** — Report Center, Story reports

Full lookup: [Admin tools cheat sheet](09_Admin_Tools_Cheat_Sheet.md).

---

## People Profile — how it is put together

People Profile (the modern profile) is assembled from:

1. **Sections** — the top-level tabs/areas of the profile
2. **Blocks** — the individual cards inside a section (an EC HRIS element, a background element, a custom MDF block, a live profile block)
3. **Field-level visibility** — from the data model (`visibility` on each `hris-field`) and from RBP field permissions

Configured through **Admin Center → Configure People Profile**. As a consultant you will spend real time here, because "the customer wants the org info at the top" is a genuine, frequent request.

**Debugging order when a field is missing:**

1. Is the field in the data model at all? (Business Configuration UI)
2. Is its `visibility` set to `both`/`view`/`edit`, not `none`?
3. Does the logged-in user's permission role grant the field?
4. Is the block placed on People Profile?
5. Is a business rule hiding it (rules can set visibility)?
6. Is it a country-specific field, and is the employee in that country?

---

## Proxy — testing as someone else

**Proxy Now** lets an authorised admin work *as* another user, to see exactly what they see. Essential for testing RBP and workflows, because "it works for me" is not a test.

- Proxy rights are granted in **Manage Proxies / Proxy Management**.
- Actions taken while proxying are logged as proxied — auditors care about that.
- Proxying is normal in DEV/TEST and tightly controlled in production.

---

## Org chart and Company Info

The **Company Info / org chart** view shows the manager hierarchy (and, if Position Management is on, the position hierarchy). Useful for spotting a broken hierarchy immediately — a person reporting to themselves, or an orphan branch, shows up visually before any report finds it.

---

## Step by step — your first five minutes in a new instance

1. Log in, open **Action Search**, type your own name, open your profile. See which blocks exist.
2. Open **Admin Center**. Skim the tool groups; do not click anything that writes.
3. Search the tool **Manage Business Configuration** — this is the data model in UI form. Look at `personalInfo` and `jobInfo`.
4. Search **Manage Data** — pick object `Legal Entity` and see the foundation data that exists.
5. Search **Manage Permission Roles** — open one role, look at its permission categories and its target population.
6. Open an employee, hit **Take Action**, and just *read* the options.

Do that in every new instance you touch. It tells you, in five minutes, how the client is configured.

---

## Interview-grade Q&A

- **Where does an admin configure the system?** Admin Center. Provisioning is partner-only and holds a smaller set of switches.
- **What is People Profile?** The unified employee profile made of sections and blocks, showing both EC data and talent data; configured in Configure People Profile.
- **A user says a field is missing. How do you troubleshoot?** Check data model presence and visibility, then RBP field permission, then People Profile placement, then rules, then country-specific applicability.
- **What is Proxy?** Working as another user to reproduce their exact experience — the standard way to test RBP and workflow routing.
- **Where do transactions start?** Take Action on the employee's profile (Change Job and Compensation, Manage Worker, Terminate, and so on).
- **What is Action Search?** Global search that finds both people and admin tools; the practical way to navigate.

---

## Further learning

- SAP Learning — [Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy)
- Video — [Navigation and architecture walkthrough](https://www.youtube.com/watch?v=qkmFdj4h4rA)
