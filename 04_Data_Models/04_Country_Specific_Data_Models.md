# Country/region-specific data models (CSF)

## The problem they solve

One instance serves every country the customer operates in. But:

- A German employee needs a **tax ID and a church-tax indicator**; a British one does not.
- A US address has a **state** and a ZIP; a Japanese one has a prefecture and a block number.
- India needs **PAN, Aadhaar, UAN and PF number**; Brazil needs CPF and PIS.
- France needs collective-agreement fields; the UK needs a NI number and a student-loan indicator.

Putting all of that on one global Personal Information screen would produce a form with 200 mostly-empty fields. **Country/region-specific (CSF) data models** solve it by layering country fields on top of the base model, so an employee sees only the fields relevant to their country.

Analogy: a passport form on a government website. Everyone fills in name and date of birth; the moment you choose a country of residence, a different set of extra questions appears.

---

## The two CSF models

| Model | Extends | Typical contents |
|---|---|---|
| **CSF Corporate Data Model (CSF-CDM)** | Foundation objects | Country-specific fields on legal entity, location and other organisational objects — local registration numbers, statutory codes, local address formats |
| **CSF Succession Data Model (CSF-SDM)** | Employee data | Country-specific fields on `personalInfo`, `homeAddress`, `nationalIdCard`, `globalInfo`, `jobInfo`, `compInfo` and others |

Both are XML, both are uploaded through Provisioning (partner only), and much of the employee side is now also editable through **Manage Business Configuration**.

---

## How the system decides which country applies

This is the mechanism to be able to explain:

| Data | Country determined by |
|---|---|
| **Address** | The **country of the address itself** — an employee can hold a UK home address and a French mailing address, each rendered in its own format |
| **National ID** | The **country selected on the ID row** — one person can hold IDs for several countries |
| **Global Information** | The **country of the Global Information record** — a person can hold blocks for several countries |
| **Job / Compensation country fields** | The country of the **Legal Entity** on Job Information |
| **Personal Information country fields** | Usually the employee's country, derived from the legal entity |

The important consequence: **country-specific data is keyed by country, not by employee.** A person on a global assignment can legitimately carry German *and* Brazilian country blocks at the same time, which is precisely why the design works this way.

---

## What SAP ships versus what you build

SAP delivers **standard country content** for 100+ countries: the legally required fields, address formats, national ID types and their validations. In practice:

1. **Enable the country** (Provisioning / country enablement).
2. **Review the shipped fields** — they are usually close to what payroll needs.
3. **Hide what the customer does not use** — set `visibility="none"`; do not delete.
4. **Add what is missing** as custom fields in the CSF model.
5. **Label them in the local language** as well as English.
6. **Permission them** — country fields are frequently the most sensitive in the instance.

You almost never build a country from scratch. You curate what SAP ships.

---

## Examples of country content

| Country | Typical country-specific fields |
|---|---|
| **India** | PAN, Aadhaar, UAN, PF number, ESI number; state in address; PIN code |
| **Germany** | Steuer-ID (tax ID), social insurance number, church-tax indicator, disability status, works council fields |
| **USA** | SSN; state and county in address; FLSA/exempt status; EEO job category; ethnicity and veteran status |
| **UK** | National Insurance number; student loan plan; NHS-related fields in some designs |
| **Brazil** | CPF, PIS/PASEP, CTPS work card number, RG |
| **Japan** | My Number; Kana name fields; prefecture-based address |
| **France** | INSEE/NIR number, collective agreement (convention collective), coefficient |

You are not expected to memorise these. You *are* expected to say: **"SAP ships country content; I enable the country, review the shipped fields with local HR and payroll, hide what is unused, and add what is missing."**

---

## Where country-specific data ends up

Country fields exist mostly because **payroll and statutory reporting need them**. So the design does not end at the screen:

1. Field added in CSF-SDM →
2. Permissioned in RBP →
3. Mapped in the **payroll replication** (ECP, on-prem HCM or a third-party provider) →
4. Included in **statutory reports** →
5. Covered by the **data retention/purge** policy, because most of it is sensitive personal data.

Skipping step 3 is the classic defect: the field is captured beautifully in EC and never reaches payroll.

---

## Global assignments and multiple country blocks

An employee sent from Germany to Brazil for two years:

- Keeps their **German** Global Information block (home country).
- Gains a **Brazilian** Global Information block (host country) with CPF and PIS.
- Holds **two national IDs**, one per country.
- May hold **two addresses**, one per country, each in its own format.
- Has **two employments** — home and assignment — with different legal entities, so their job-level country fields differ too.

If an interviewer asks "how does EC handle a global assignment's country data?", that list is the answer.

---

## Step by step — enable and configure a country

1. **Confirm the legal requirement** with local HR and payroll — do not design from the field list alone.
2. **Enable the country** in the instance (Provisioning, partner task).
3. **Export the current CSF-SDM and CSF-CDM** as backups.
4. In **Manage Business Configuration**, open the country section for the relevant HRIS elements and review the shipped fields.
5. Set `visibility` and `required` per field, in consultation with payroll.
6. Add any missing fields using custom placeholders in the country block.
7. Add **local-language labels**.
8. Configure the **address format** and **national ID card types** for that country.
9. Set **field criteria** so dependent picklists filter (state by country, ID type by country).
10. **Permission** the fields to the roles that need them — usually local HR and payroll only.
11. **Test with an employee in that legal entity**: hire, check the fields appear, save, check history.
12. **Test with an employee in another country** — confirm the fields do *not* appear.
13. Map the fields into the **payroll interface** and test the extract.
14. Export the XML, diff, document.

Step 12 is the one people forget, and it is exactly how a German field ends up on every British employee's screen.

---

## Real world example

A customer goes live in the UK, then adds India six months later.

What "adding India" actually involved:

| Area | Work |
|---|---|
| Country enablement | Provisioning switch |
| Legal entity | New Indian entity, country = IN, INR currency |
| CSF-SDM | PAN, Aadhaar, UAN, PF number enabled; ESI added as a custom field |
| Address | Indian format with state picklist and PIN code |
| National ID | Four card types configured, with format validation rules |
| Pay | Indian pay components, pay group `IN-MTH`, INR pay ranges by geozone |
| Event reasons | Two India-specific termination reasons required by local practice |
| RBP | India HR permission group and role, with target population limited to the Indian entity |
| Payroll | Mapping of all new fields into the payroll interface |
| Testing | A full hire-to-pay cycle in India, plus a regression test that UK screens were unchanged |

The last line is the point: **adding a country is a project, not a configuration task**, and the regression test on existing countries is mandatory.

---

## Common mistakes

- Putting country fields in the **base model**, so every country sees them.
- **Deleting** shipped country fields instead of hiding them.
- English-only labels for a workforce that does not work in English.
- Forgetting the **payroll mapping**, so the data never leaves EC.
- Not configuring **field criteria**, so a US employee can pick a German ID type.
- Assuming country is derived from the *address* for job-level fields — it is derived from the **legal entity**.
- No **regression test** on existing countries after adding a new one.

---

## Interview-grade Q&A

- **What are the country/region-specific data models? [HIGH]** CSF-CDM and CSF-SDM: XML overlays that add country-specific fields to foundation objects and to employee data, so employees see only the fields relevant to their country.
- **How does the system know which country's fields to show? [HIGH]** By the country on the record — the address's own country, the national ID row's country, the Global Information block's country — and, for job and compensation fields, by the country of the legal entity on Job Information.
- **Give examples of country-specific fields. [HIGH]** India: PAN, Aadhaar, UAN, PF. Germany: tax ID, social insurance number, church tax. USA: SSN, state, FLSA status, EEO category. UK: National Insurance number.
- **Does SAP ship country content? [HIGH]** Yes, for 100+ countries — legally required fields, address formats, national ID types and validations. You enable the country, curate the fields with local HR and payroll, and add anything missing.
- **How does a global assignment work with country data? [MED]** The person holds country blocks, national IDs and addresses for both home and host countries, with two employments whose legal entities differ.
- **Where do country-specific fields need to end up? [HIGH]** In the payroll interface and statutory reporting — capturing them in EC without mapping them onward is the classic gap.
- **What must you test when adding a new country? [HIGH]** That the new fields appear for that country, that they do **not** appear for existing countries (regression), and that a full hire-to-payroll cycle works in the new entity.
- **Would you put a German works-council field on `personalInfo` globally? [MED]** No — it belongs in the German country-specific model, on the element whose history you need (often `jobInfo`), permissioned to German HR only.

---

## Further learning

- SAP Learning — [Employee Central Core Academy](https://learning.sap.com/courses/sap-successfactors-employee-central-core-academy) — Configuring employee data
- SAP Help — [Implementing Employee Central Core](https://help.sap.com/docs/successfactors-employee-central/implementing-employee-central-core)
- Video — [Country-specific configuration](https://www.youtube.com/watch?v=yEsquQA-MxU)
