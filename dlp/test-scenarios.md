# DLP Test Scenarios — CAPTCHA Technologies

This document records DLP policy test execution against the **CAPTCHA - Customer Data Protection** policy (see `dlp/dlp-policy.md` for the policy's design and rule logic). All tests were performed against fictional data only.

---

## Test 1: External sharing of Confidential-labeled customer data via email

### Scenario

A Sales department user attempts to email a file containing customer data, labeled Confidential, to a recipient outside the organization.

### Objective

Validate that the DLP policy correctly detects sensitive content being shared externally, and — since the policy is deployed in simulation mode — that it warns the user without actually blocking the message.

### Preconditions

| Item | Status |
|---|---|
| DLP policy "CAPTCHA - Customer Data Protection" | Created, simulation mode, policy sync completed |
| Test file: Customer_Records.xlsx | Labeled "Confidential - CAPTCHA"; contains fictional customer names, emails, phone numbers, national ID numbers, and industry-standard test payment card numbers |
| Test user | Lerato Khumalo (Sales Analyst), licensed with Microsoft 365 E5 |

### Test steps

1. Signed in to Outlook on the web as Lerato Khumalo
2. Composed a new email:
   - **To:** an external (non-organizational) email address
   - **Subject:** "Customer data for review"
   - **Body:** "Hi, I have attached the customer records as discussed."
   - **Attachment:** Customer_Records.xlsx
3. Observed the compose window after the attachment finished uploading/scanning
4. Did not click Send — captured evidence, then discarded the draft

### Expected result

A policy tip should appear in the compose window, warning that the message conflicts with an organizational DLP policy, identifying the external recipient as the concern, and offering a corrective action — without blocking the Send button, consistent with simulation mode.

### Actual result: PASS

A policy tip appeared exactly as expected:

> *"Policy tip: Your email message conflicts with a policy in your organization. [recipient email]: Please verify the recipient(s) and the content, as it conflicts with the organization policy. [Remove recipient] View details about the information that appears sensitive."*

- The tip correctly identified the specific external recipient by address
- The tip offered a direct corrective action ("Remove recipient")
- The **Send** button remained active and clickable, confirming the policy did not block the action — correct behavior for simulation mode
- A lock icon was displayed alongside the tip, visually reinforcing that this was a security-related warning

**Evidence:** `evidence/dlp/50-dlp-policy-tip-triggered.png`

### Which condition triggered the match

The rule's condition structure is:
```
(Content shared outside organization)
AND
( (Sensitivity label = Confidential/Highly Confidential) OR (Sensitive info type = Credit Card Number) )
```

Since Customer_Records.xlsx was both labeled **Confidential - CAPTCHA** *and* contains fictional test credit card numbers, this test does not by itself distinguish which condition inside the OR group fired. A follow-up test (Test 2, below) isolates the content-based detection path specifically.

### Note on admin-side visibility (as of initial test execution)

At the time this test was run, the DLP policy's **"Items for review"** and **Alerts** tabs in the Purview admin console still showed no matches, despite the client-side policy tip having fired correctly in Outlook. This is consistent with a known reporting propagation delay observed elsewhere in this project (see `documentation/lessons-learned.md`, item 5) — the user-facing policy tip is evaluated and rendered by the client in near real time, while admin-facing reporting/alerting is a separate backend pipeline with its own refresh cadence. This gap will be re-checked and this document updated once admin-side alert data becomes available, to confirm the full detection → alert → investigation chain end to end.

**Status: pending update** — see `audit/investigation-report.md` for the follow-up once alert data is confirmed.

---

## Test 2: Content-only detection without a sensitivity label

### Scenario

A copy of the customer data file, with its sensitivity label removed (confirmed "Unlabeled" before testing), was emailed to the same external recipient used in Test 1.

### Objective

Demonstrate that the DLP policy's sensitive-info-type detection path functions independently of manual classification — i.e., that DLP still catches sensitive content even when a user fails to (or has not yet) applied the correct label. This directly supports the risk register's framing of DLP as a control against human classification error (see `risk-management/risk-register.xlsx`, R003).

### Test steps

1. Created a copy of Customer_Records.xlsx and confirmed via the Sensitivity button that it showed no label applied
2. Signed in to Outlook on the web as the same test user
3. Composed a new email to the same external address used in Test 1, with subject "Customer data - unlabeled copy test," and attached the unlabeled file
4. Observed the compose window after the attachment scan completed

### Expected result

If the DLP policy's content-based detection (Sensitive info type: Credit Card Number) functions independently of the sensitivity-label condition, a policy tip should still appear, specifically citing detected content rather than a label.

### Actual result: PASS

A policy tip appeared, with a detail panel stating explicitly:

> *"This message appears to contain the following sensitive information: **Credit Card Number**"*

Critically, this detail view made **no mention of a sensitivity label** — confirming the match was produced purely by the sensitive-info-type branch of the rule's OR condition, not the label branch. This is direct evidence that the policy protects sensitive data even when classification is missing or incorrect, which is the core risk-mitigation claim made for this control in `risk-management/risk-register.xlsx` (R003) and `risk-management/control-matrix.xlsx` (CTRL-002).

The interface also offered a **"Report"** option for the user to flag the match as a false positive if they believed the content wasn't actually sensitive — a meaningful UX detail supporting user trust in the control rather than pure silent enforcement.

**Evidence:** `evidence/dlp/54-unlabeled-test-file-confirmed.png`, `evidence/dlp/55-dlp-unlabeled-file-test-result.png`, `evidence/dlp/56-dlp-notification-email-received.png`

### Additional evidence: system-generated notification email

In addition to the real-time inline policy tip, Lerato Khumalo received a separate, automatically generated notification email confirming the same match after the fact:

> *"Your email message conflicts with a policy in your organization. Issues: Message is sent to people outside your organization. Message contains the following sensitive information: **All Full Names, Credit Card Number**."*

This confirms both sensitive info types configured in the rule's content-detection group (Credit Card Number and All Full Names) matched against this file, and validates that the policy's **"Notify users with email and policy tips"** action is functioning as two distinct, independent mechanisms: an inline compose-time warning, and a persistent email record the user can refer back to. This distinction matters operationally — the inline tip can be missed or dismissed quickly, while the email creates a durable, referenceable record of the policy match.

### Additional evidence: admin incident report email

Separately, `captcha.admin` received a structured **incident report email**, titled *"Rule detected - Detect and restrict Confidential customer data sharing,"* confirming the "Send incident reports to Administrator" and "Send alerts to Administrator" actions configured in the policy are fully functional. The email included:

- **Report Id:** a unique identifier (69a995b3-7aed-4c79-b5a9-074aae7911ac) suitable for referencing this specific incident
- **Service:** Exchange
- **Person sharing item:** Lerato Khumalo, full UPN
- **To:** the external recipient address, in full
- **Severity:** High (matching the configured severity)
- **False positive: No / Override: No**
- **Conditions matched to trigger the rule:** External recipients; Contains sensitive information Type — All Full Names, Credit Card Number

Notably, this admin notification **email** arrived within minutes of the test, while the Purview portal's own **"Items for review"** and **"Alerts"** dashboard views were still showing no data at the same point in time — demonstrating that the alerting/notification pipeline and the web UI's reporting dashboard are separate systems with independent latency (see `documentation/lessons-learned.md`, item 5). For incident investigation purposes, this means the admin incident email is currently the more reliable, faster evidence source for this DLP policy, and should not be assumed absent just because the dashboard shows nothing.

**Evidence:** `evidence/dlp/57-dlp-admin-incident-report-email.png`

---

## Summary

| Test | Result | Evidence |
|---|---|---|
| Test 1: External email with Confidential-labeled file | PASS (policy tip triggered correctly in simulation mode) | evidence/dlp/50-dlp-policy-tip-triggered.png |
| Test 2: Content-only detection (unlabeled file) | PASS (Credit Card Number detected independent of label) | evidence/dlp/55-dlp-unlabeled-file-test-result.png |

Both tests confirm the DLP policy's dual detection paths (label-based and content-based) each function correctly and independently, in simulation mode, without disrupting message send capability.

Full test tracking, including tests from other project phases, is consolidated in `testing/test-plan.xlsx`.
