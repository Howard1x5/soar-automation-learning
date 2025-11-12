
# Torq HyperSOC – Okta Suspicious Login Investigation

## 1. Summary

This practical lab simulated investigation of a **suspicious Okta login alert** within **Torq’s HyperSOC** platform.
The case centered around anomalous authentication activity for a user named **Kevin**, detected from multiple high-risk global locations.

As the SOC analyst, I investigated the event end-to-end using **Socrates (Torq’s AI SOC assistant)** to query logs, enrich observables, correlate related cases, communicate with the affected user, and ultimately resolve the case.

---

## 2. Objective

* Retrieve and review suspicious Okta login logs.
* Identify anomalous geographic access patterns.
* Enrich extracted IP observables using VirusTotal.
* Contact the user for activity validation.
* Classify and resolve the case appropriately.

---

## 3. Investigation Workflow (Steps Completed)

### Step 1. Case Review

* Located case: **“Okta suspicious login for Kevin.”**
* Opened the case in full view and reviewed the **Case Summary** for alert context (user, event source, detection time).
* Examined the **Runbook** tab to confirm the prescribed workflow: review Okta logs, validate observables, contact user, resolve outcome.

### Step 2. Case Assignment

* Clicked the **Owner icon → Assign to me**.
* Updated case status to **In Progress** to begin active investigation.

---

### Step 3. Query with Socrates – Okta Logs

* Opened the **Socrates** tab.
* Prompted:

  ```
  Retrieve Okta Logs for Kevin
  ```
* Observed the **Get Okta System Logs workflow** executing in the **Timeline** tab.
* After completion, returned to Socrates’ response:

  * Found multiple suspicious login attempts from **Beijing**, **Moscow**, **Istanbul**, and **Buenos Aires**.
  * The original alert listed only **Moscow**, revealing that this was part of a wider pattern of potentially automated access.

---

### Step 4. Document Findings

* Asked Socrates:

  ```
  Add a note summarizing your findings in the Okta logs
  ```
* Verified a new **Note** appeared listing the four IP addresses and their associated geolocations.
* Marked the note as **Key Note** (star icon) to surface it in the **Case Overview**.

---

### Step 5. Add and Enrich Observables

* Asked Socrates:

  ```
  Add all 4 IP addresses as observables
  ```
* Watched the **Observables tab** populate with the extracted IPs.

Then prompted:

```
Enrich all 4 IP addresses with VirusTotal
```

* Waited for enrichment workflows to complete.

If any enrichment results didn’t appear automatically:

```
Update each of the observables based on the VirusTotal enrichment
```

* Verified that each observable now displayed reputation data, threat classifications, and VirusTotal summary results.

Next, queried Socrates to correlate additional alerts:

```
Search for and link cases with the same observables
```

* Linked any discovered related cases under **Linked Cases**.

---

### Step 6. User Validation

* Executed **Contact User → Ask a question via IM**.
* Selected user **Kevin’s email address**.
* Sent inquiry:

  ```
  Did you travel to Moscow, Beijing, Istanbul, and Buenos Aires? We’re seeing anomalous activity from your account.
  ```
* Upon receiving the response (“This was VPN activity”), documented it automatically in the Timeline.

---

### Step 7. Document Resolution

* Added timeline comment:

  ```
  User indicated this is likely due to VPN activity, closing the case.
  ```
* Updated **Case Status → Resolved**.
* Selected **Resolution Reason: False Positive**.
* Ensured **Auto-generate Socrates Resolution Reason** was toggled on and regenerated the summary.
* Confirmed closure — case moved to **Resolved** section.

---

### Step 8. Mark Playground Case as Completed

* Ran the workflow **Mark the Playground case as completed**.
* Verified successful execution in the Timeline.

---

## 4. Outcome

* Validated multiple suspicious logins across several global regions.
* Confirmed benign explanation (VPN usage).
* Demonstrated complete case life-cycle: detection → enrichment → validation → resolution.
* Challenge marked complete and automatically evaluated by Torq Academy.

---

## 5. Key Learnings

1. **Multi-region authentication anomalies** often require contextual correlation with VPN or travel data.
2. **Socrates AI assistant** accelerates log retrieval, IOC extraction, and enrichment workflows.
3. **Automation-driven enrichment** ensures consistency and reduces analyst workload.
4. **User communication integration** (IM/Email within case) centralizes investigative context.
5. **AI-assisted resolution** maintains standardized documentation and audit readiness.

---

## 6. Optional Local Simulation (Python Example)

To illustrate how this case logic could be reproduced outside of Torq:

```python
import random

okta_logs = [
    {"user": "kevin", "ip": "203.0.113.10", "location": "Moscow"},
    {"user": "kevin", "ip": "203.0.113.11", "location": "Beijing"},
    {"user": "kevin", "ip": "203.0.113.12", "location": "Istanbul"},
    {"user": "kevin", "ip": "203.0.113.13", "location": "Buenos Aires"},
]

def enrich_ip(ip):
    # Fake VirusTotal enrichment
    return {"ip": ip, "reputation": random.choice(["Clean", "Suspicious"])}

enriched = [enrich_ip(log["ip"]) for log in okta_logs]

print("Enriched Observables:")
for e in enriched:
    print(e)

# Simulated user response
user_response = "VPN usage confirmed - false positive"
print(f"User Response: {user_response}")

final_resolution = {
    "status": "Resolved",
    "resolution_reason": "False Positive",
    "notes": "User confirmed VPN activity caused multiple geo logins."
}

print(final_resolution)
```

**Example Output**

```json
{
  "status": "Resolved",
  "resolution_reason": "False Positive",
  "notes": "User confirmed VPN activity caused multiple geo logins."
}
```

---

## 7. Folder Structure

```
/torq/automation-practitioner/okta-suspicious-login/
├── README.md
└── scripts/
    └── simulate_okta_case.py
```

---

## 8. Reflection

This exercise reinforced the analyst’s role in validating alerts within an automated SOAR workflow.
Torq’s **HyperSOC** provides a unified, AI-augmented process where investigation, enrichment, communication, and closure are tightly integrated — enabling rapid and consistent incident handling at scale.

---