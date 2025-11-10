# Torq HyperSOC – Phishing Playground Investigation

## 1. Summary

This practical exercise simulated a full **phishing alert investigation** using **Torq’s HyperSOC** environment.
The scenario involved a reported phishing email containing a suspicious URL designed to steal credentials.

As the SOC analyst, I handled the case end-to-end — from triage and enrichment to remediation and resolution — leveraging **AI-driven automation** (Socrates) and workflow-based response actions.

---

## 2. Objective

* Investigate a phishing alert from detection to resolution.
* Enrich observables (IP, sender email, URL, user account).
* Identify if the incident is part of a broader campaign.
* Prioritize the case and apply remediation actions.
* Document the investigation and close the case with an AI-assisted resolution.

---

## 3. Investigation Workflow (Steps Completed)

### Step 1. Case Assignment

* Navigated to **Cases → Tag: Playground 1 (Phishing)**.
* Located case: *Phishing alert received from [IT-Security-Team@your-company.com](mailto:IT-Security-Team@your-company.com)*.
* Assigned the case to myself and set the status to **In Progress**.
* Noted the **Case ID** for linking related incidents later.

### Step 2. Review the Runbook

* Opened the **Runbook** tab to understand the predefined process:

  * Validate indicators
  * Enrich IOCs
  * Identify campaign linkage
  * Contain affected users
  * Communicate and resolve

### Step 3. Find Related Cases

* Opened **Linked Cases** tab — no automatic links found.
* Returned to **Cases Grid**, filtered by:

  * Tag: *Phishing*
  * Assignee: *Unassigned*
* Selected relevant unassigned phishing cases.
* Linked them to my active case using the stored **Case ID**, relation type *Other*.
* Verified linked cases now appeared in my case’s **Linked Cases** tab.

### Step 4. IOC Enrichment

Performed enrichment on three observables:

| Observable Type | Action                           | Result                                                      |
| --------------- | -------------------------------- | ----------------------------------------------------------- |
| IP Address      | Used **Enrich IOC** quick action | Added geolocation, reputation score                         |
| Sender Email    | Used **Enrich IOC** quick action | Validated domain reputation and SPF alignment               |
| URL             | Used **Enrich IOC** quick action | Identified as potentially malicious / phishing landing page |

* Verified workflow execution and completion in the **Timeline** tab.

### Step 5. User Details Lookup

* Executed **User Action → Get user details** for recipient.
* Discovered the user was **Jeff Hayes, CFO (VIP)**.

### Step 6. Prioritization and Tagging

* Updated **Severity → Critical** due to VIP involvement.
* Added tag **VIP** to case metadata for visibility and tracking.

### Step 7. Query with Socrates (AI Analyst)

* Opened **Socrates Tab** (AI assistant).
* Prompted with:

  > Analyze the available raw email headers for phishing. Received/origin IP, identity mismatches (From/Return-Path/Reply-To), HELO/Message-ID, and obvious anomalies. Return: Verdict (Suspicious/Benign/Unclear), 3–5 evidence bullets with exact fields, Confidence 0–100, and 1–3 recommended actions.
* Socrates analyzed and returned a **Verdict: Suspicious** with supporting header anomalies (Return-Path mismatch, spoofed HELO, abnormal origin IP).

### Step 8. Remediation

Executed user-focused containment steps:

* **User Action → Revoke sessions & reset password**.
* Verified workflow success in Timeline.

### Step 9. User Communication

* Used **Comms Action → Email user**.
* Sent notification:

  ```
  We’re investigating a suspicious email and have revoked all your sessions and reset your password.
  ```
* Confirmed communication event in Timeline.

### Step 10. Case Resolution

* Updated **Case State → Resolved**.
* Selected **Resolution: True Positive – Benign** (per lab instruction).
* Enabled **Auto-generate Socrates Resolution Reason** and confirmed closure.

### Step 11. Mark Playground as Completed

* Executed workflow **Mark the Playground case as completed** to signal challenge completion.
* Verified automation in Timeline.

---

## 4. Outcome

* Case successfully investigated and resolved.
* All enrichment, remediation, and documentation steps validated by automation logs.
* Challenge marked complete for automated evaluation within Torq Academy.

---

## 5. Key Learnings

1. **End-to-End Visibility:** HyperSOC consolidates triage, enrichment, and containment into one interactive pane.
2. **AI Augmentation:** Socrates accelerates header analysis and summary documentation.
3. **Automation Efficiency:** Quick Actions (IOC Enrich, User Action, Comms Action) streamline repetitive SOC tasks.
4. **SOAR Best Practice Reinforcement:** The workflow mirrors real-world SOC playbooks—context preservation, enrichment before escalation, and verified containment before resolution.
5. **Audit Trail:** Every enrichment, decision, and action is logged in the Timeline for traceability.

---

## 6. Optional Local Reproduction (Python Example)

While this challenge runs in Torq’s environment, similar logic can be reproduced locally to simulate phishing enrichment or IOC correlation.

```python
import re
import requests

phishing_email = {
    "sender": "it-security-team@your-company.com",
    "recipient": "jeff.hayes@your-company.com",
    "url": "http://verify-account.your-company.com/login",
    "ip": "203.0.113.17"
}

def enrich_ip(ip):
    # Simulated enrichment
    reputation = "Suspicious" if ip.startswith("203.") else "Clean"
    return {"ip": ip, "reputation": reputation}

def enrich_url(url):
    is_phish = bool(re.search(r'verify|login|account', url))
    return {"url": url, "classification": "Phishing" if is_phish else "Benign"}

def remediate_user(user):
    print(f"Revoking sessions and resetting password for {user}")

ip_info = enrich_ip(phishing_email["ip"])
url_info = enrich_url(phishing_email["url"])
remediate_user(phishing_email["recipient"])

print({
    "ioc_results": [ip_info, url_info],
    "resolution": "True Positive - Phishing Campaign",
})
```

**Sample Output:**

```json
{
  "ioc_results": [
    {"ip": "203.0.113.17", "reputation": "Suspicious"},
    {"url": "http://verify-account.your-company.com/login", "classification": "Phishing"}
  ],
  "resolution": "True Positive - Phishing Campaign"
}
```

---

## 7. Folder Structure

```
/torq/automation-practitioner/phishing-playground/
├── README.md
└── scripts/
    └── simulate_phishing_case.py
```

---

## 8. Summary Reflection

This lab served as a realistic introduction to **case management, IOC enrichment, and AI-assisted investigation** inside Torq’s HyperSOC.
It demonstrated how automation accelerates analyst efficiency without sacrificing investigative rigor — a clear example of “hyperautomation in action.”

---
