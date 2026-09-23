# Business Scenario • CAPTCHA Technologies

## Company overview

**CAPTCHA Technologies** is a fictional cloud security and IT consulting company, created for this project to simulate the data governance, identity, and information protection challenges faced by a real mid-sized organization. All employees, customers, and data referenced throughout this project are fictional; no real personal information is used anywhere in this repository.

CAPTCHA Technologies is organized into five departments:

| Department | Function |
|---|---|
| HR | Manages employee records, recruitment, and onboarding |
| Finance | Manages financial records, budgeting, and reporting |
| Sales | Manages customer relationships and customer data |
| IT | Manages infrastructure, device provisioning, and technical operations |
| Security/GRC | Manages security posture, compliance, risk, and governance oversight |

## Fictional personnel

| Name | Department | Job title |
|---|---|---|
| Nomsa Dlamini | HR | HR Manager |
| Thabo Mokoena | Finance | Finance Analyst |
| Lerato Khumalo | Sales | Sales Analyst |
| Tracy Madiba | IT | IT Administrator |
| Ayanda Maseko | Security/GRC | GRC Analyst |

(Note: the IT department user was originally planned as "Sipho Ndlovu" but was consolidated into an existing test account, "Tracy Madiba," during hands-on account management — see `documentation/lessons-learned.md`.)

## The business problem

Like many growing organizations, CAPTCHA Technologies began with **no formal data governance program**. Its Microsoft 365 tenant started with a single Entra ID Free license, no sensitivity labels, no DLP policies, and no structured access control beyond default settings — a realistic starting point for a company that has grown faster than its security practices.

This creates concrete, statable business risk:

- **No consistent way to classify data** by sensitivity, so staff cannot reliably judge what is safe to share
- **No technical enforcement of least privilege** — access decisions, if made at all, are ad hoc rather than group-based and auditable
- **No detection or prevention of sensitive data leaving the organization** — a Sales Analyst emailing a customer spreadsheet externally, deliberately or by mistake, would go completely unnoticed
- **No audit trail** to investigate a suspected data exposure incident after the fact
- **No documented risk register or control mapping** — the organization has no way to demonstrate to a customer, regulator, or auditor what controls exist and why

## Data inventory

CAPTCHA Technologies processes the following categories of information:

| Data category | Examples | Owning department(s) | Sensitivity |
|---|---|---|---|
| Employee information | Employee records, national ID numbers, salaries, bank details | HR | Highly Confidential |
| Customer information | Customer names, emails, phone numbers, payment card data | Sales | Confidential |
| Financial information | Financial records, budgets, reporting | Finance | Highly Confidential |
| Internal business information | Internal procedures, internal reports | All departments | Internal / Confidential |
| IT information | IT procedures, infrastructure documentation | IT | Internal |
| Security and compliance information | Security investigations, risk assessments, audit findings | Security/GRC | Highly Confidential |
| Marketing / public information | Marketing material, public website content | Sales/Marketing | Public |

## Project objective

This project simulates the work of a Junior GRC / Security Analyst tasked with designing and implementing a foundational data governance and protection program for CAPTCHA Technologies, using Microsoft Entra ID and Microsoft Purview. The project follows the natural progression a real analyst would follow:

**Business risk → Data governance → Identity & access management → Data classification → Information protection → DLP → Audit → Investigation → Risk assessment → Security controls → Testing → Evidence → GRC recommendations**

Each phase of this project is documented not just as "what was clicked," but as a deliberate control decision made in response to a specific, statable business risk — consistent with how a real GRC function would justify its program to leadership or an auditor.

## Scope and constraints

This lab was built entirely within a Microsoft 365 E5 trial license (30 days, activated at $0 cost) layered on top of a free Microsoft Entra ID tenant. No Azure paid resources (VMs, storage accounts, SQL databases) were provisioned, and no Azure pay-as-you-go billing was enabled — deliberately, to keep the project cost-free and reproducible by anyone with access to an M365 trial. Where a Purview feature required Azure billing (e.g., Endpoint DLP, Defender for Cloud Apps instances), it was explicitly scoped out and documented as a known limitation rather than enabled.
