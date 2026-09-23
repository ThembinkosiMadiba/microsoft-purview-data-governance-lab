# Investigation Report • INC-001

## Incident summary

| Field | Detail |
|---|---|
| Incident ID | INC-001 |
| Title | Potential Customer Data Exposure — External Email Sharing |
| Date/time detected | 22 September 2026, ~13:45–13:49 SAST (per notification and incident report timestamps) |
| Severity | High |
| Status | **Closed — controlled lab test, no actual data exposure occurred** |
| Investigated by | Ayanda Maseko (GRC Analyst) / CAPTCHA Admin |

> **This incident was a deliberate, controlled test performed as part of this project's Day 4 DLP validation exercise, not a genuine security event.** It is documented here in full investigation format to demonstrate the detection → alert → investigation workflow a real incident would follow. No message was actually sent, and no real customer data exists in this environment — all data is fictional (see `documentation/business-scenario.md`).

## What happened

A user in the Sales department (Lerato Khumalo) composed two test emails addressed to an external recipient, each with an attachment containing fictional customer data:

1. **Attempt 1:** Customer_Records.xlsx, carrying the sensitivity label "Confidential - CAPTCHA"
2. **Attempt 2:** an unlabeled copy of the same file (label deliberately removed), to test detection independent of classification

Both attempts were intercepted by the **CAPTCHA - Customer Data Protection** DLP policy before the message was sent (the policy operates in simulation mode, so detection and alerting occurred, but the message itself was not technically blocked from sending — see `dlp/dlp-policy.md` for why simulation mode was chosen).

## User

**Lerato Khumalo** — Sales Analyst, member of the CAPTCHA-Sales security group, licensed under Microsoft 365 E5. Lerato is the legitimate owner of the tested file (she labeled it Confidential as part of Day 3 activity) and had a legitimate business reason to be handling it; the "sharing" action in this test was deliberately performed as a controlled exercise, not a genuine attempt to exfiltrate data.

## Resource

**Customer_Records.xlsx** (and an unlabeled copy of the same file) — a fictional customer data spreadsheet containing customer names, emails, phone numbers, national ID numbers, and industry-standard test payment card numbers. Stored in Lerato Khumalo's OneDrive. See `data-classification/classification-model.md` for its classification history.

## Activity

Attempted external email sharing via Outlook on the web (Exchange Online), addressed to a recipient outside the organization's domain.

## Detection method

The DLP policy's rule condition matched on two independent paths across the two test attempts:

- **Attempt 1:** matched via the sensitivity label condition (Confidential - CAPTCHA)
- **Attempt 2:** matched via the sensitive info type condition (Credit Card Number, All Full Names), with no label present — confirming detection does not depend on correct manual classification

Full technical detail of the rule logic is in `dlp/dlp-policy.md`; full test execution detail is in `dlp/test-scenarios.md`.

## Policy

**CAPTCHA - Customer Data Protection**, deployed in simulation mode with user notifications and admin incident alerting enabled. Policy sync confirmed completed prior to testing.

## Evidence collected

Three independent, corroborating artifacts were captured for this incident:

1. **Inline policy tip** (real-time, shown to the sender in the Outlook compose window) — `evidence/dlp/50-dlp-policy-tip-triggered.png`, `evidence/dlp/55-dlp-unlabeled-file-test-result.png`
2. **User notification email** (sent to the sender after the fact, confirming the specific policy issues detected) — `evidence/dlp/56-dlp-notification-email-received.png`
3. **Admin incident report email** (sent to captcha.admin, containing a formal Report ID, matched conditions, severity, and full sender/recipient detail) — `evidence/dlp/57-dlp-admin-incident-report-email.png`

## Investigation notes

A notable finding during this investigation: the Purview web portal's own "Items for review" and "Alerts" dashboards (under Solutions → Data Loss Prevention → Policies → [policy] → Items for review / Alerts) remained empty well over an hour after both test events, despite the notification and incident report **emails** having already been delivered within minutes. This confirmed that DLP's alerting/notification pipeline and its web UI reporting dashboard are separate backend systems with independent latency — a real investigator working this incident would have needed to rely on the incident report email as the primary evidence source, since the dashboard alone would have (incorrectly) suggested nothing had happened. This finding is recorded in full in `documentation/lessons-learned.md`.

Separately, Microsoft Purview's centralized **Unified Audit Log search** was found to be non-functional at the time of this investigation ("Failed to load data" / "trouble figuring out if activity is being recorded"), on a tenant created the previous day. Administrative permissions were confirmed correct (captcha.admin verified as a member of Organization Management via Purview's own "My permissions" view) before concluding this was a backend provisioning delay rather than a misconfiguration. This means the investigation for INC-001 was conducted **without** access to the centralized audit trail that would normally corroborate DLP alert data — a real limitation, documented honestly here rather than glossed over.

### Addendum — Audit log follow-up (added after initial investigation)

Approximately one day after this investigation was originally conducted, Unified Audit Log search was re-tested and found to be functional: a broad search successfully returned 5 real events, including a verified sign-in by `captcha.admin` with a source IP address. This confirms the earlier failure was indeed a temporary backend provisioning delay, not a permanent misconfiguration, consistent with the hypothesis recorded above.

However, a follow-up targeted search — scoped specifically to Lerato Khumalo's account, both with and without a keyword filter, across the same date range as the original incident — returned **0 results**, despite tenant-level administrative activity being clearly captured for other identities in the same window. This suggests that Unified Audit Log's coverage of **tenant-level/administrative activity** (sign-ins, directory changes) and its coverage of **mailbox-level activity** (message send/access events for a specific licensed user) may be independently governed, and the latter was not confirmed working within this project's timeline. This is recorded as an open item rather than resolved, and is reflected honestly in `testing/test-plan.xlsx` (T10, updated to PARTIAL rather than PASS or FAIL).

**Practical implication:** even after Audit Search became generally functional, it could not be used to directly corroborate the specific DLP test activity described in this incident. The three-artifact evidence chain (inline policy tip, user notification email, admin incident report email) documented above remains the authoritative evidence for INC-001, not the audit log.

## Result

**Confirmed:** the DLP policy correctly detected both a labeled and an unlabeled attempt to share sensitive customer data externally, generated appropriate user-facing and admin-facing alerts, and — because the policy remains in simulation mode — did not disrupt the sender's ability to complete the action, consistent with the deliberate rollout strategy documented in `dlp/dlp-policy.md`.

No actual data left the organization: both test emails were discarded as drafts and never sent.

## Recommended action

1. **Review DLP simulation results after a longer observation period** (recommend at least 1–2 weeks in a real deployment) to confirm a low false-positive rate before moving the policy from simulation to active enforcement (Block).
2. **Investigate mailbox-level audit coverage specifically** — tenant-level Unified Audit Log is now confirmed functional, but a targeted search for a specific licensed user's mailbox activity returned no results during this project. Confirm whether mailbox auditing requires separate explicit enablement (e.g., via Exchange Online mailbox audit settings) beyond the tenant-wide audit toggle.
3. **Do not rely solely on the Purview DLP dashboard** for real-time incident awareness; ensure admin incident report emails are monitored (or forwarded to a security mailbox/ticketing system in a production deployment) as the more timely alert channel.
4. **Formalize this test as a repeatable validation step**: re-run equivalent label-based and content-based DLP tests after any future policy change, to confirm both detection paths continue to function as designed.

## GRC takeaway

This investigation demonstrates that alert *generation* and alert *visibility* are not the same thing — a control can be technically firing correctly while its administrative dashboard lags behind, and an investigator needs to know to check multiple evidence channels rather than trusting a single pane of glass. This is a realistic, defensible finding to bring to an interview: it shows investigative judgment, not just tool operation.
