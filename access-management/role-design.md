# Role Design - CAPTCHA Technologies Access Management

## Purpose

This document explains the identity and access management (IAM) design decisions behind CAPTCHA Technologies' Microsoft Entra ID configuration: why groups were used instead of direct user permissions, how least privilege was implemented, and the real technical constraints encountered while building it.

## Business risk addressed

| Risk | Description |
|---|---|
| Excessive user privileges | Employees granted more access than their role requires, increasing the impact of a compromised account or insider mistake |
| Unauditable access | Access granted ad hoc, with no clear record of who has access to what, or why |
| Access that doesn't scale | Manually managing permissions per user does not scale past a handful of people and is highly error-prone |
| Orphaned access | Access that isn't revoked when someone changes role or leaves, because it was never tracked centrally |

## Design principle: group-based access, not direct user assignment

Every access decision in this project is expressed through **group membership**, never through permissions assigned directly to an individual user. This is a deliberate application of a core IAM best practice, for three concrete reasons:

1. **Auditability.** "Who can access customer data?" is answered by looking at one group's membership (CAPTCHA-Sales), not by checking every resource's permission list individually.
2. **Scalability.** Onboarding a new Sales Analyst means adding them to one group; their access to every resource that group is entitled to follows automatically. Direct assignment would require touching every resource individually, for every new hire.
3. **Revocability.** Offboarding, or a role change, is a single action — remove the person from the group — rather than hunting down every place they were separately granted access.

## Identity structure

Five department security groups were created, each mapped 1:1 to a fictional department user in this lab (a real organization would have many users per group):

| Group | Member(s) | Department |
|---|---|---|
| CAPTCHA-HR | Nomsa Dlamini | HR |
| CAPTCHA-Finance | Thabo Mokoena | Finance |
| CAPTCHA-Sales | Lerato Khumalo | Sales |
| CAPTCHA-IT | Tracy Madiba | IT |
| CAPTCHA-Security-GRC | Ayanda Maseko | Security/GRC |

Membership type was set to **Assigned** (manual) rather than **Dynamic**. For a lab of this size, manual assignment is easier to demonstrate and reason about directly. In a real enterprise deployment, dynamic membership rules (e.g., `department -eq "Sales"`) would be the more scalable choice, automatically adding/removing users as their directory attributes change — this is a known "next step" for scaling this design beyond a lab.

## Separation of duties: dedicated admin identity

A dedicated cloud-only account, `captcha.admin`, was created and assigned the Global Administrator role, separate from the personal Microsoft account originally used to create the Entra tenant.

**Why this matters:** the personal account that provisions a tenant is often, by default, treated as an owner/admin identity — but it is a consumer identity, not a proper organizational one, and mixing personal and administrative access is poor practice. Real security programs typically require a dedicated, auditable administrative identity (sometimes a "break-glass" account reserved for emergencies), kept separate from any individual's personal or daily-use account. This also directly enables **accountability**: actions taken by `captcha.admin` in audit logs are unambiguously tied to a deliberate administrative identity, not conflated with personal account activity.

## Known technical constraint: mail-enabled security groups

All five department groups were initially created as standard Entra ID **Security groups**. During Day 3 (Information Protection), it became clear that Microsoft Purview's sensitivity label permission picker — and, as confirmed again in Day 4, the DLP-adjacent tooling — only supports **mail-enabled security groups** (or Microsoft 365 groups) for permission assignment. Standard security groups, while fully valid for Entra ID access control (e.g., app role assignments, Conditional Access targeting), are not selectable in these specific Purview pickers.

**Resolution:** all five groups were deleted and recreated as mail-enabled security groups via **Exchange Admin Center** (Entra ID's group creation flow cannot produce this group type directly), preserving the same names, descriptions, and membership. This required the `captcha.admin` account itself to hold a Microsoft 365 license, since Exchange requires group owners to be valid mail recipients — an unlicensed administrative account could not be assigned as group owner.

This is documented here as a genuine finding, not a design flaw discovered too late: it reflects a real, non-obvious constraint in how Entra ID and Purview interact, and the kind of troubleshooting a GRC/security analyst would realistically need to work through when standing up a new tenant's protection controls.

## License assignment

Microsoft 365 E5 trial licenses were assigned to the five department users, enabling Purview capability (sensitivity labels, DLP scanning) on their mailboxes and OneDrive. `captcha.admin` was initially left unlicensed (administrative accounts do not need a mailbox to administer the tenant) and was only licensed once the group ownership requirement above required it. The original personal-linked account (`Thembinkosi Madiba`) was deliberately left unlicensed and excluded from the department group structure, since it is not part of the fictional CAPTCHA Technologies organization.

## Mapping to the access control matrix

The group structure defined here directly implements the resource-level permissions documented in `access-management/access-control-matrix.xlsx`. For example, the matrix's "Customer Data: Sales = Read/Write, Security/GRC = Read" mapping is enforced two ways in this project:
- Technically, via the Confidential sensitivity label's access control settings (see `data-classification/classification-model.md`)
- Structurally, via the CAPTCHA-Sales and CAPTCHA-Security-GRC group definitions documented here

## GRC takeaway

This design demonstrates that least privilege is not just a policy statement — it is an architecture: identities are grouped by role, access is granted to the group (not the individual), and every subsequent control (sensitivity labels, and later DLP) references those same groups rather than re-defining access from scratch. This consistency is exactly what an auditor looks for when assessing whether an access control program is genuinely enforced or only documented on paper.
