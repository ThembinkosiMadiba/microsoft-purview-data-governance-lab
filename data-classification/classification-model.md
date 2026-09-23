# Data Classification Model — CAPTCHA Technologies

## Purpose

CAPTCHA Technologies processes several categories of business-sensitive information across its HR, Finance, Sales, IT, and Security/GRC departments, including customer data, employee records, financial information, and internal operational documentation. Without a consistent classification scheme, staff have no reliable way to judge how a given piece of information should be handled, shared, or protected — which creates real risk of accidental disclosure, inconsistent access control, and non-compliance with data protection expectations.

This document defines the classification model adopted for CAPTCHA Technologies and describes how it was implemented technically using Microsoft Purview Information Protection (sensitivity labels).

## Business risk addressed

| Risk | Description |
|---|---|
| Inconsistent handling of sensitive data | Without labels, employees rely on personal judgment to decide what's safe to share externally |
| Accidental external disclosure | Sensitive files could be emailed or shared outside the organization with no warning or control |
| Lack of audit trail for sensitive content | No way to track where classified information exists or how it's being used |
| Over- or under-protection of data | Applying uniform controls regardless of sensitivity wastes effort on low-risk data and under-protects high-risk data |

## The classification model

Four tiers were defined, in increasing order of sensitivity:

### 1. Public
**Definition:** Information approved for public release. No restrictions on sharing.

**Examples:** Marketing material, public website content, press releases.

**Protection settings applied:** None. Public data carries no confidentiality requirement, so no access control, content marking, or restriction was configured. This is a deliberate decision, not an oversight — applying protective controls to genuinely public content would be disproportionate to the risk it carries.

### 2. Internal
**Definition:** For internal company use only. Not intended for external sharing, but not access-restricted within the organization.

**Examples:** Internal procedures, internal company documentation.

**Protection settings applied:** Content marking (footer: "CAPTCHA Technologies - Internal Use Only"). No access control was applied — Internal data is a policy/behavioral boundary ("staff shouldn't share this externally") rather than a technically enforced one. The footer serves as a visible, persistent reminder of sensitivity to anyone viewing the document, regardless of where it ends up.

### 3. Confidential
**Definition:** Sensitive business or customer information requiring restricted distribution.

**Examples:** Customer information, internal reports.

**Protection settings applied:**
- Content marking (footer: "CAPTCHA Technologies - Confidential")
- Access control (Assign permissions now): explicit user/group-level permission assignment, restricting who can open, edit, or view labeled content
- Access to content: Never expires; offline access: Always allowed

This is the first tier where classification becomes a technical control rather than only a visual signal — only specifically permitted identities can meaningfully interact with Confidential content.

### 4. Highly Confidential
**Definition:** Highly sensitive information requiring the organization's strictest access control.

**Examples:** Employee records, financial records, security investigations.

**Protection settings applied:**
- Content marking (footer: "CAPTCHA Technologies - HIGHLY CONFIDENTIAL", plus a visible watermark: "HIGHLY CONFIDENTIAL")
- Access control (Assign permissions now), scoped more tightly than Confidential
- Access to content: Never expires; **offline access: Never allowed** — unlike Confidential, this tier restricts caching for offline viewing, reflecting that highly sensitive data (e.g. employee financial data, active security investigations) should not persist unmanaged on an endpoint

## Priority ordering

Labels were created with ascending priority values (Public = 0, Internal = 1, Confidential = 2, Highly Confidential = 3), consistent with Purview's convention that higher priority values represent higher sensitivity and take precedence in any conflict (e.g., label inheritance from attachments).

## Access control mapping

The access control settings on Confidential and Highly Confidential labels are intended to technically reinforce the access-control matrix defined separately in `access-management/access-control-matrix.xlsx`. For example, the Confidential label's permission assignment reflects that Sales owns and edits customer data, while Security/GRC retains read-only oversight visibility — mirroring the least-privilege design established during the Day 2 access management phase.

**Known limitation:** Microsoft Purview's sensitivity label permission picker only supports mail-enabled security groups (or Microsoft 365 groups) — not standard Entra ID security groups. CAPTCHA Technologies' access groups were initially created as standard security groups and required recreation as mail-enabled security groups via Exchange Admin Center before they could be used directly in label permission assignments. This is documented further in `documentation/lessons-learned.md`.

## Label policy publication

All four labels were published to the organization via a single label policy ("CAPTCHA Technologies - Sensitivity Label Policy"), scoped to all users and groups (Exchange email / All users & groups), with the following organization-wide settings:

- No default label enforced — classification is a deliberate user action, not an automatic default
- Justification required to remove or downgrade a label (supports accountability and audit trail)
- Label inheritance from attachments enabled, in "recommend" (not automatic) mode — if a user attaches a higher-sensitivity file to a lower-labeled email, they are prompted to reclassify rather than having the system silently override their choice

Purview documented an expected propagation delay of up to 24 hours for labels to become available in users' Office apps after policy publication; in practice, labels were visible in Word/Excel for the web within roughly one hour.

## Applied to test data

Four fictional test files were created and labeled by realistic department "owners" to demonstrate the classification model in practice:

| File | Classification applied | Labeled by |
|---|---|---|
| Marketing_Strategy.docx | Public - CAPTCHA | Nomsa Dlamini |
| IT_Procedure.docx | Internal - CAPTCHA | Tracy Madiba |
| Customer_Records.xlsx | Confidential - CAPTCHA | Lerato Khumalo |
| Employee_Records.xlsx | Highly Confidential - CAPTCHA | Nomsa Dlamini |

Screenshots of each labeled file are captured under `evidence/purview/`.

## GRC takeaway

This exercise demonstrates the core information protection principle that **controls should be proportionate to data sensitivity**: no protection was applied where none was warranted (Public), while increasingly strict technical controls — content marking, access restriction, and offline access limits — were layered on as sensitivity increased. This mirrors how a real organization would design and justify a classification scheme to an auditor: every control decision maps back to a specific, statable risk.
