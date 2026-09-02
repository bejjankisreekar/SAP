# What is SAP SuccessFactors?

## Plain definition

**SAP SuccessFactors** is SAP's cloud suite for managing people — hiring them, storing their HR record, paying them, measuring them, developing them, and reporting on them. It is delivered as **SaaS** (Software as a Service): SAP runs the servers, SAP applies the upgrades, and the customer pays a subscription per employee per year.

Analogy: an on-premise HR system is a **house you own** — you buy the land, build it, fix the roof, renovate when you like. SuccessFactors is a **serviced apartment in a large building** — furnished, already working, the building manager fixes the plumbing and repaints twice a year on *their* schedule, and you personalise the inside within the rules of the building. You trade *total control* for *no maintenance*.

---

## A short history (interviewers do ask this)

| Year | What happened | Why it matters to you |
|---|---|---|
| 2001 | SuccessFactors founded as a **performance management** vendor | The talent modules (Performance & Goals) are the oldest and most mature |
| 2011–12 | **SAP acquires SuccessFactors** (~USD 3.4bn) | SuccessFactors becomes SAP's cloud HR line |
| 2012–15 | **Employee Central** built out as core HR | EC is *younger* than the talent modules — which is exactly why its data model looks different from the old Employee Profile |
| 2019 | Suite renamed **HCM → HXM** ("Human eXperience Management") | Same product, marketing shift; the terms are used interchangeably |
| 2020s | **Metadata Framework (MDF)** becomes the extension platform; single modern UI, People Profile, Story reports | Modern configuration is MDF-first, XML second |

**One-line answer for an interview:** *"SuccessFactors is SAP's cloud HXM suite; Employee Central is its core HR module — effectively the cloud successor to SAP HCM's PA and OM."*

---

## On-premise vs cloud — the differences that change your job

| | **SAP HCM (ECC / S/4 on-prem)** | **SAP SuccessFactors (cloud)** |
|---|---|---|
| Where it runs | Your data centre | SAP's data centre, multi-tenant |
| Who upgrades | You, in projects, every few years | SAP, **twice a year**, for everyone |
| Customisation | ABAP — you can change almost anything | **Configuration only**: XML data models, MDF objects, business rules. No ABAP inside the product |
| Data model | Infotypes (IT0000, IT0001…) in PA/PB tables | HRIS elements + MDF Generic Objects |
| UI | SAP GUI, ESS/MSS on NetWeaver | Browser + mobile app, responsive |
| Cost model | Licence + hardware + Basis team | Subscription per employee per year |
| Integration | RFC / BAPI / IDoc | **OData API**, Compound Employee API, SFTP, Integration Center |

The single most important cultural difference: **you cannot code your way out of a gap.** If the standard object has no field for it, you add a field in MDF, you write a business rule, or you change the process. That constraint is why "configuration + business rules" *is* the job.

---

## What an EC consultant actually does

1. **Workshops** — sit with HR, capture how they hire, transfer and terminate today, and where it hurts.
2. **Design** — turn that into a *configuration workbook*: which foundation objects, which fields, which event reasons, which workflows, which roles.
3. **Build** — configure the instance: data models, MDF objects, picklists, business rules, RBP, workflows.
4. **Migrate** — load foundation data and employee data from the legacy system through import templates.
5. **Test** — unit test, string test, then **UAT** with the customer's HR team; log and fix defects.
6. **Integrate** — connect EC to payroll, Active Directory, benefits providers, ticketing.
7. **Cut over and support** — go live, hypercare, then hand over to the customer's admin team.

Notice how little of that is "clicking through screens". Most of the value is **translating an HR process into a data model plus rules**.

---

## Advantages

- **No infrastructure** — no servers, no Basis team, no upgrade projects.
- **Continuous innovation** — new features every six months, included in the subscription.
- **Global by design** — country-specific fields, address formats and legal requirements shipped by SAP for 100+ countries.
- **Integration-friendly** — practically everything in EC is exposed through OData; Integration Center builds files without middleware.
- **Self-service and mobile out of the box** — ESS/MSS is not a separate build.

## Disadvantages and limits

- **The release calendar is not yours** — some changes are *universal* and cannot be declined.
- **No ABAP escape hatch** — gaps close through MDF, rules, or process change (or an extension on SAP BTP, outside the product).
- **Configuration debt is real** — a badly designed data model or rule set is painful to unwind two years later.
- **Provisioning gating** — some switches are partner-only, so the customer depends on their implementation partner.
- **Reporting has layers** — Story reports, People Analytics and Integration Center each solve a different slice, and beginners mix them up.

---

## Real world example

A retail group with 22,000 staff across 9 countries runs SAP HCM on-premise. Payroll works fine, but every country keeps its own spreadsheet of positions, hiring takes three weeks of email approvals, and the CHRO cannot get one headcount number.

They move **core HR to Employee Central** and keep payroll on-premise:

- EC becomes the **system of record** for person, employment, job and pay data.
- Position Management gives one global position hierarchy.
- Workflows replace the email approvals — a promotion routes manager → HRBP → Compensation in two days.
- A nightly **Compound Employee API** replication feeds the on-premise payroll.
- Headcount comes from one report, because there is now one source.

Payroll never moved; the *system of record* did. That hybrid — "EC plus on-premise payroll" — is one of the most common landscapes you will meet in projects and in interviews.

---

## Interview-grade Q&A

- **What is SAP SuccessFactors?** SAP's cloud HXM suite: SaaS modules covering core HR, talent, learning and analytics, with Employee Central as the core HR system of record.
- **HCM vs HXM?** Same suite, renamed in 2019 to emphasise employee experience.
- **Is SuccessFactors the same as Employee Central?** No. SuccessFactors is the suite; EC is the core HR module inside it.
- **Can you customise SuccessFactors the way you customise ABAP?** No. You configure — XML data models, MDF objects, business rules, RBP. Anything beyond that lives outside the product, on SAP BTP.
- **How often is it upgraded?** Twice a year, with a preview window before production.
- **Why do customers move from SAP HCM to EC?** No infrastructure or upgrade burden, global standardisation, self-service and mobile, continuous features, and a modern API-based integration layer.
- **What is the biggest constraint versus on-premise?** No custom code inside the product, and no control over the upgrade calendar.
- **What is the consultant's real deliverable?** A configured instance *plus* the configuration workbook that explains and justifies it.

---

## Further learning

- SAP Learning — [SuccessFactors Platform Introduction Academy](https://learning.sap.com/courses/sap-successfactors-platform-introduction-academy)
- SAP Help — [SAP SuccessFactors platform documentation](https://help.sap.com/docs/SAP_SUCCESSFACTORS_PLATFORM)
- Video — [SuccessFactors overview](https://www.youtube.com/watch?v=6xrD8Vkm0QI)
