# Lessons Learned — CAPTCHA Technologies Data Governance Lab

This document captures genuine technical findings and troubleshooting encountered while building this project — kept separate from the main configuration documentation because these are things that surprised or required investigation, not steps that went as planned on the first attempt. A real GRC/security analyst should expect and be able to work through exactly this kind of friction when standing up new tooling.

## 1. Consumer vs. organizational identity

The personal Microsoft account originally used to create the Azure subscription and Entra tenant was added into the tenant as an external-style identity (visible as a `#EXT#` suffix in its User Principal Name), and could not sign into the Microsoft 365 Admin Center — it returned *"Login is not supported for consumer users without business presence."* This is because a personal/consumer Microsoft account is fundamentally different from a proper organizational (work) identity, even when it technically holds a Member role in the tenant.

**Resolution:** created a dedicated, cloud-only administrative identity (`captcha.admin`) and assigned it the Global Administrator role, keeping it fully separate from any personal account going forward. This also reflects good practice independent of the technical requirement: administrative actions should be traceable to a dedicated identity, not a personal one.

## 2. Multiple billing account views can hide a newly purchased product

After activating the Microsoft 365 E5 trial ($0 cost, no card required), the new subscription did not appear under **Billing → Your products** at first. The cause was that the tenant has two separate billing account views ("MCA" personal billing account vs. "Default Directory / MOSA" billing account), and the trial had been provisioned under the one not currently selected. Switching the billing account view surfaced it correctly.

## 3. Purview sensitivity labels and DLP require mail-enabled security groups

Standard Microsoft Entra ID Security groups — the type created by default through the Entra admin center — are **not selectable** in Microsoft Purview's sensitivity label permission picker or in related access-control pickers used elsewhere in Purview. Only mail-enabled security groups (or Microsoft 365 groups) appear in these pickers.

**Resolution:** all five department groups were deleted and recreated as mail-enabled security groups via **Exchange Admin Center**, which is the only place this group type can currently be created (Entra ID's own group creation flow cannot produce it). This also surfaced a secondary requirement: Exchange requires a group's owner to be a licensed mail recipient, which meant the `captcha.admin` account needed a Microsoft 365 license assigned before it could be set as group owner — it had deliberately been left unlicensed up to that point.

## 4. The "Block external sharing" DLP action has an undocumented-in-UI structural requirement

When first configuring the DLP policy's action to "Block only people outside your organization," policy creation failed with a specific client error: the rule must include an explicit **"Content is shared with people outside your organization"** condition, joined with **AND**, as the first condition in the rule — simply selecting the blocking action is not sufficient on its own. Once identified, the fix was straightforward, and arguably produced a more precise, more clearly-scoped rule than the original attempt.

## 5. Propagation and reporting delays are inconsistent across Purview components, and each should be verified independently

Across this project, several distinct Purview components exhibited different propagation/reporting timelines, none of which matched their stated maximums exactly:

| Component | Stated maximum | Observed |
|---|---|---|
| Sensitivity label policy → visible in Office apps | Up to 24 hours | ~1 hour |
| DLP policy → synced to locations | Up to 2 hours (up to 24h for full user/group sync) | Confirmed complete at ~2 hours |
| Label usage analytics report | Hourly refresh (stated) | Still 0 items shortly after labeling — expected, given refresh cadence |
| DLP policy simulation "Items for review" / Alerts | Not explicitly stated | Still empty over an hour after a confirmed policy match (visible live in Outlook) |
| Unified Audit Log search | Not explicitly stated for new tenants | Returned "Failed to load data" for roughly the first day after tenant/license activation; became functional afterward without further intervention — confirmed by a successful search returning 5 real events, including a `captcha.admin` sign-in with source IP address |
| DLP admin incident report **email** (vs. the in-portal dashboard) | Not explicitly stated | Delivered within minutes of the triggering event, with full structured detail (Report Id, matched conditions, severity, sender/recipient) — notably faster and more complete than the Purview web UI's own "Items for review" / "Alerts" dashboard, which was still empty at the same point in time |
| Mailbox-level audit visibility for a specific licensed user (Lerato Khumalo) | Not explicitly stated | Once Unified Audit Log search became functional, a targeted search for this specific user (with and without a keyword filter) returned 0 items, despite tenant-level administrative activity (sign-ins, service principal changes) being clearly captured for other identities in the same window. This suggests tenant-level auditing (Entra ID sign-ins, admin actions) and mailbox-level auditing (Exchange Online message send/access events for a specific user) may be governed by different settings or have different enablement timelines, and were not both confirmed working within this project's timeline. |

**Practical takeaway:** none of these delays indicate misconfiguration by themselves. Before concluding something is broken, the correct troubleshooting sequence is: (1) confirm the underlying policy/configuration is actually correct and saved, (2) confirm permissions are not the blocker (verified directly via Purview's own "My permissions" view rather than assumed), and only then (3) attribute the gap to backend propagation delay and re-check later. Assuming failure too early — or re-configuring something that was already correct — wastes effort and can mask the real cause. This project's DLP testing specifically showed that an **email-based alert channel can be live and fully functional while the corresponding web UI dashboard for the same data is still empty** — a useful reminder that a single "monitoring" capability (DLP alerting, in this case) can be implemented through more than one delivery mechanism, each with its own independent reliability and latency characteristics, and an investigator should check all of them rather than relying on one view alone.

## 6. Empty or error states are still valid evidence

Several screenshots captured during this project deliberately show 0 results, empty dashboards, or explicit error messages (e.g., DLP simulation showing "0 matches found," Audit Search showing "Failed to load data"). These were kept as evidence rather than discarded, because they document the real operational timeline of a Microsoft Purview deployment — a distinction a real GRC analyst or auditor would want to see understood, not hidden.

## 7. Roster correction during setup

The IT department user was originally intended to be "Sipho Ndlovu" but an existing test account ("Tracy Madiba") was substituted in after an account-management mix-up during user creation. Rather than treating this as an error to hide, the substitution was carried through consistently across all subsequent configuration (groups, licensing, labeling, documentation) and is noted here for transparency.
