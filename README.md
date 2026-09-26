# CAPTCHA Technologies: Enterprise Data Governance, Information Protection & DLP Lab

A hands-on Microsoft security/GRC portfolio project simulating the work of a Junior GRC / Security Analyst at a fictional company, **CAPTCHA Technologies**. Built entirely on a free Microsoft Entra ID tenant plus a Microsoft 365 E5 trial (no Azure paid resources, no pay-as-you-go billing enabled), this project takes a data governance program from zero to a documented, tested, evidenced state, following the same path a real analyst would.

> All company names, employees, and data in this project are fictional. No real personal information is used anywhere in this repository.

## Summary

> Designed and implemented a simulated enterprise data governance and protection environment using Microsoft Entra ID and Microsoft Purview, implementing data classification, least-privilege access controls, DLP policies, auditing, risk assessment, control mapping and security testing.

## Why this project exists

Most portfolio projects in this space are tutorials: click through a wizard, take a screenshot, move on. This one is built differently. Every configuration decision is tied to a specific business risk, every control is mapped to a risk register entry, every claim of "it works" is backed by an actual test with a PASS/FAIL/PARTIAL result, and critically, **every real problem encountered along the way is documented rather than hidden**. A genuinely broken feature (Audit Search failing for the first day of the tenant's life), a real technical constraint (Purview's mail-enabled security group requirement), and an honestly incomplete result (mailbox-level audit coverage still unresolved) are all recorded exactly as they happened. See [`documentation/lessons-learned.md`](documentation/lessons-learned.md) for the full list.

## The flow

```
Business risk -> Data governance -> Identity & access management -> Data classification
-> Information protection -> DLP -> Audit -> Investigation -> Risk assessment
-> Security controls -> Testing -> Evidence -> GRC recommendations
```

## Repository structure

```
├── README.md                              you are here
│
├── documentation/
│   ├── business-scenario.md               Company narrative, departments, data inventory
│   └── lessons-learned.md                 Real technical findings & troubleshooting
│
├── access-management/
│   ├── access-control-matrix.xlsx         Least-privilege matrix, mapped to real Entra groups
│   └── role-design.md                     Group-based IAM design & rationale
│
├── data-classification/
│   └── classification-model.md            4-tier sensitivity label model & configuration
│
├── dlp/
│   ├── dlp-policy.md                      DLP policy design, rule logic, troubleshooting
│   └── test-scenarios.md                  2 executed DLP tests, both PASS, full evidence
│
├── audit/
│   └── investigation-report.md            INC-001, full investigation plus addendum
│
├── risk-management/
│   ├── risk-register.xlsx                 5 risks, live-calculated scores, residual risk
│   └── control-matrix.xlsx                5 controls mapped to risks, tech, and evidence
│
├── testing/
│   └── test-plan.xlsx                     12 tests across every phase, live summary
│
├── sample-data/
│   ├── Marketing_Strategy.docx            Labeled Public - CAPTCHA
│   ├── IT_Procedure.docx                  Labeled Internal - CAPTCHA
│   ├── Customer_Records.xlsx              Labeled Confidential - CAPTCHA; used in DLP testing
│   └── Employee_Records.xlsx              Labeled Highly Confidential - CAPTCHA
│
└── evidence/
    ├── entra/       Screenshots: tenant, users, groups, licensing, Conditional Access, Access Reviews, PIM
    ├── purview/      Screenshots: sensitivity labels, policies
    ├── dlp/          Screenshots: policy config, test results, alerts
    └── audit/        Screenshots: audit search, investigation evidence
```

## What was built

### Identity & access management ([`access-management/`](access-management/))
Five fictional department users, each mapped 1:1 to a mail-enabled Entra ID security group (CAPTCHA-HR, -Finance, -Sales, -IT, -Security-GRC). Access is granted exclusively at the group level, never assigned directly to individuals, implementing least privilege by design. A dedicated, cloud-only Global Administrator account (`captcha.admin`) enforces separation of duties from any personal identity.

Beyond static group access, three additional identity security controls were implemented and tested:
- **Conditional Access** ("CAPTCHA - Require MFA for All Users"): a tenant-wide MFA requirement, deployed in **Report-only mode** first, with the break-glass admin account deliberately excluded to avoid a lockout risk.
- **Access Reviews** ("CAPTCHA-Sales Quarterly Access Review"): a recurring, quarterly recertification of CAPTCHA-Sales group membership, reviewed by Security/GRC with justification required and results auto-applied.
- **Privileged Identity Management (PIM)**: an **eligible** (not standing) Security Reader assignment for the GRC analyst, tested end-to-end including self-activation with a time-boxed expiry, demonstrating just-in-time privileged access.

The Access Review and PIM additions directly close out a recommendation originally written into the risk register (R005), turning a stated suggestion into an implemented, evidenced control that measurably reduced that risk's residual score. See [`risk-management/risk-register.xlsx`](risk-management/risk-register.xlsx) and the "Closing the loop" note below.

### Data classification ([`data-classification/`](data-classification/))
A four-tier sensitivity model (Public -> Internal -> Confidential -> Highly Confidential) implemented as Microsoft Purview sensitivity labels, with protection settings that scale proportionally to risk: no controls on Public content, content marking on Internal, and full access control (plus offline-access restriction at the top tier) on Confidential and Highly Confidential. Four realistic test documents were labeled by their appropriate department "owners."

### Data Loss Prevention ([`dlp/`](dlp/))
A custom DLP policy (**CAPTCHA - Customer Data Protection**) combining label-based and content-based detection with correct AND/OR condition nesting, deployed safely in **simulation mode**. Two independent tests both passed: one proving label-based detection, one proving the policy still catches sensitive content (credit card numbers, full names) even when a file carries **no label at all**, directly addressing the real-world risk of human classification error.

### Audit & investigation ([`audit/`](audit/))
A full incident investigation (**INC-001**) built around a controlled DLP test, using a three-artifact evidence chain (inline policy tip, user notification email, admin incident report email) after Purview's Unified Audit Log proved unavailable for roughly the first day of the tenant's life. A follow-up addendum documents what was later confirmed once audit search became functional, and what remained genuinely unresolved (mailbox-level audit coverage for a specific user).

### Risk & control management ([`risk-management/`](risk-management/))
A risk register scoring 5 real risks with live spreadsheet formulas (Likelihood x Impact), honestly reflecting residual risk where a control is only partially effective (e.g., a DLP policy still in simulation mode). A control matrix maps each control to its risk, technology, implementation status, and evidence, with two controls honestly marked "Partially implemented" / "Planned" rather than overstated.

### Testing ([`testing/`](testing/))
A consolidated test plan covering 12 tests across every phase of the project, with a live-calculated summary: **10 PASS, 0 FAIL, 2 PARTIAL**, including one test that was genuinely a FAIL for part of the project's timeline before being re-tested and reclassified as PARTIAL once new information came in, exactly as a real test log would evolve.

### Sample data ([`sample-data/`](sample-data/))
Four fictional test documents, one per classification tier, each labeled by a realistic department "owner" and referenced throughout `data-classification/classification-model.md` and `dlp/test-scenarios.md`. All content and personal data (names, ID numbers, payment card numbers) is fabricated; card numbers use industry-standard test ranges, never real data.

## Closing the loop: from recommendation to reduced risk

Most portfolio risk registers list recommendations that never go anywhere. This one is different for at least one risk: **R005 (excessive or unmanaged user privileges)** was scored, a control was implemented (group-based RBAC), and a recommendation was written: *"introduce periodic access reviews to confirm group membership remains appropriate over time."*

That recommendation was then actually implemented as a recurring Entra ID Access Review, and R005's residual score dropped accordingly. The updated recommendation went further, suggesting Privileged Identity Management (PIM) as a next step, which was then also implemented and tested end-to-end (an eligible role assignment, self-activated by the test user, time-boxed, with a working deactivation path). R005's residual score dropped again, to the lowest score in the entire register.

This is documented explicitly in the risk register's own notes, not just claimed in this README. See [`risk-management/risk-register.xlsx`](risk-management/risk-register.xlsx) and [`access-management/role-design.md`](access-management/role-design.md) for the full mechanics of both controls.

## Technologies used

- Microsoft Entra ID (Free tier + Microsoft 365 E5 trial)
- Microsoft Entra ID: Conditional Access, Access Reviews, Privileged Identity Management (PIM)
- Microsoft Purview: Information Protection (sensitivity labels)
- Microsoft Purview: Data Loss Prevention
- Microsoft Purview: Audit (Unified Audit Log)
- Exchange Admin Center (mail-enabled security groups)
- Microsoft 365 Admin Center

**Cost:** $0. No Azure paid resources were provisioned, and no pay-as-you-go billing was linked at any point (see `documentation/business-scenario.md`, Scope and constraints).

## Key findings worth highlighting

- Purview's sensitivity label and DLP permission pickers require **mail-enabled** security groups, not standard Entra security groups. A genuine, non-obvious constraint discovered and resolved via Exchange Admin Center.
- DLP's "Block external sharing" action has a specific structural requirement (an explicit "shared outside organization" condition, joined with AND) that isn't obvious from the UI alone until Purview's own validation error explains it.
- Different Purview components (label policy propagation, DLP sync, label analytics, DLP alerting, Audit Search) each have **independent, inconsistent latency**, and assuming a feature is broken before checking permissions and waiting is a mistake worth avoiding.
- A DLP alerting channel (incident report email) can be fully functional while its corresponding web UI dashboard is still empty. Investigators should check multiple evidence sources, not one dashboard.
- Privileged Identity Management was deliberately applied to a scoped, non-owner role (Security Reader) rather than the tenant's sole Global Administrator account. A real risk-based decision, since converting the only admin identity to eligible-only access risks a genuine lockout with no way to recover if activation fails.

Full detail on all of these: [`documentation/lessons-learned.md`](documentation/lessons-learned.md).

## Author's note

This project was built as a hands-on learning exercise following the completion of Microsoft SC-900, while studying toward CompTIA Security+. It is intended to demonstrate practical, evidenced familiarity with Microsoft Purview, Microsoft Entra ID, and core GRC concepts (least privilege, data classification, DLP, risk registers, control mapping, and incident investigation), not theoretical knowledge alone.
