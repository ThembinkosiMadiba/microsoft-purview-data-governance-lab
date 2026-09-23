# DLP Policy — CAPTCHA Technologies Customer Data Protection

## Business requirement

CAPTCHA Technologies must prevent unauthorized disclosure of sensitive customer information. Specifically, the organization needs visibility into — and eventually control over — attempts to share Confidential or Highly Confidential classified content, or content containing detectable sensitive data patterns (such as payment card numbers), with recipients outside the organization.

Sensitivity labels (see `data-classification/classification-model.md`) classify and mark data, but classification alone does not prevent misuse — a labeled Confidential spreadsheet can still be emailed to a personal address unless an active control inspects and acts on that activity. Data Loss Prevention (DLP) is that active control layer.

## Policy overview

| Property | Value |
|---|---|
| Policy name | CAPTCHA - Customer Data Protection |
| Scope type | Enterprise applications & devices |
| Locations | Exchange email, SharePoint sites, OneDrive accounts |
| Locations explicitly excluded | Teams chat/channel messages, Devices (Endpoint DLP), Instances (Defender for Cloud Apps), On-premises repositories |
| Deployment mode | Simulation mode, with policy tips enabled |
| Admin alerting | Enabled, severity: High, alerts sent to captcha.admin |

### Why Exchange, SharePoint, and OneDrive only

These three locations directly cover the realistic paths through which CAPTCHA Technologies' test data (Customer_Records.xlsx, stored in OneDrive and shared via email) could leave the organization. Teams, Endpoint DLP, and Defender for Cloud Apps ("Instances") were excluded deliberately:

- **Devices (Endpoint DLP)** and **Instances (non-Microsoft cloud apps)** triggered a pay-as-you-go Azure billing requirement and/or required additional Conditional Access and Edge for Business configuration not in scope for this lab
- **On-premises repositories** support was still in preview and required prerequisite steps not relevant to a cloud-only tenant

This is a deliberate, documented scoping decision, not an oversight — a real DLP rollout should always start narrow and expand coverage once the initial policy is validated.

## Rule logic

The policy contains a single rule, **"Detect and restrict Confidential customer data sharing,"** built with the following condition structure:

```
Content is shared from Microsoft 365 (with people outside my organization)
AND
  (
    Content contains any of these sensitivity labels:
      Confidential - CAPTCHA, Highly Confidential - CAPTCHA
    OR
    Content contains any of these sensitive info types:
      Credit Card Number, All Full Names
  )
```

**Why this structure:**
- The outer **AND** condition ("shared outside my organization") ensures the rule only fires on genuine external sharing attempts — internal collaboration on the same sensitive data does not trigger it, avoiding unnecessary noise
- The inner **OR** group means the rule catches sensitive content **two different ways**: either because a human correctly applied a Confidential/Highly Confidential sensitivity label, *or* because Purview independently detected sensitive data patterns in the content itself (e.g., a credit card number), even if no label was applied at all

This second detection path is deliberate: it demonstrates DLP's ability to catch sensitive content even when a user fails to classify a file correctly — a realistic and important scenario, since manual classification is never 100% reliable.

### A troubleshooting note

The policy's action, "Restrict access or encrypt content — block only people outside your organization," could not initially be saved. Microsoft Purview returned an explicit client-side validation error:

> *"To use Block External as an action, you must have 'Content is shared with people outside your organization' as the first condition along with operator 'AND' with other conditions or groups in your rule."*

This is a legitimate structural requirement, not a mistake: Purview will not let an admin configure an "external block" action without an explicit "shared outside the organization" condition present in the rule, since without it the action's intent would be ambiguous. The rule was restructured to add this condition as the first, top-level element, connected via AND to the existing sensitivity label / sensitive info type group. The resulting rule is more precise than the original attempt, not just a workaround.

## Actions configured

| Action | Configuration |
|---|---|
| Restrict access or encrypt content | Block only people outside the organization |
| User notifications | Enabled — users see an in-app policy tip explaining the match |
| Incident reports | Sent to Administrator (captcha.admin), severity High |
| Admin alerts | Sent on every matching activity (not threshold-based, given expected low test volume) |

## Why simulation mode

Per standard DLP rollout practice, this policy was deployed in **simulation mode with policy tips enabled**, rather than immediate enforcement. In this mode:

- No content is actually blocked — "your data won't be affected; the policy stays off while in simulation mode" (Purview's own description)
- Users still see the policy tip they would see in enforcement mode, allowing validation of both detection accuracy and the end-user experience before anything is actually restricted
- The 15-day auto-graduation-to-enforcement option was deliberately left **unchecked**, keeping full manual control over when (or whether) this policy moves to active enforcement

This mirrors how a real organization validates a new DLP policy: false positives at the enforcement stage disrupt legitimate business activity and erode trust in the control, so detection accuracy is proven in simulation first.

## Deployment timeline observed

| Stage | Expected delay (per Microsoft) | Observed |
|---|---|---|
| Sensitivity label policy propagation to Office apps | Up to 24 hours | ~1 hour |
| DLP policy sync to locations | Up to 2 hours (up to 24 hours for full user/group sync) | *(see evidence log)* |

This distinction — a stated maximum vs. actual observed behavior — is worth noting in any operational runbook: teams should plan around the documented maximum, not assume the best case will always occur.

## Test scenario

**Scenario:** Lerato Khumalo (Sales Analyst, owner of Customer_Records.xlsx) attempts to email the Confidential-labeled customer data file to a recipient outside the organization.

**Expected result (simulation mode):** A policy tip appears in the Outlook compose window warning that the message contains sensitive information; the message is not actually blocked from sending, since the policy remains in simulation mode.

Full test execution and results are documented in `dlp/test-scenarios.md` and `testing/test-plan.xlsx`, with supporting screenshots in `evidence/dlp/`.

## GRC takeaway

This policy demonstrates the core DLP principle of **defense in depth applied to data**: detection is not reliant on a single signal (a human-applied label) but combines classification-based and content-based detection, scoped precisely to the risk being addressed (external sharing), and deployed in a way that validates accuracy before enforcing restrictions — balancing security control against business disruption risk, which is exactly the trade-off a GRC analyst is expected to reason about and justify.
