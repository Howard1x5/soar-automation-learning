# Torq HyperAutomation Practitioner – Academy Playground #2 (Challenge A: Build an IOC Intake Workflow)

## 1. Summary

This lab introduced **data extraction and context logic** by building an **IOC (Indicator of Compromise) Intake Workflow** in Torq.
The workflow receives a raw threat intelligence report, stores it in a variable, extracts IOCs using Torq’s built-in utility, and validates whether meaningful indicators exist.
This practical scenario mirrors real-world SOAR use cases — transforming unstructured text into actionable, structured intelligence.

---

## 2. Objective

* Capture free-form text from an On-Demand trigger input.
* Store it temporarily in a workflow variable for processing.
* Use Torq’s **Extract IOCs** utility to parse indicators from the text.
* Implement conditional logic to handle both successful and empty results.
* Return structured output or a controlled failure message based on IOC presence.

---

## 3. Workflow Design Steps

### Step 1. Create New Workflow

* Navigated to **Workflows → New blank workflow**.
* **Name:** `Automated IOC Intake`
* **Section:** `Playground Workflows`
* **Trigger:** On-Demand Trigger → **Create Workflow**.

---

### Step 2. Add Input Parameter

* In the On-Demand trigger, added:

  * **Type:** Long text
  * **Name:** `raw_intelligence_report`
  * **Required:** Enabled
* Saved configuration.

This allows an analyst to paste an entire raw threat report as text at runtime.

---

### Step 3. Store the Input in a Workflow Variable

* In **Builderbox → Integrations**, searched **Set Variable** and dragged it below the trigger.
* Renamed the step: **Incoming Threat Intelligence Report**.
* Configured:

  * **Variable name:** `report`
  * **Data type:** string
  * **Value string:**

    ```text
    {{ $.event.raw_intelligence_report }}
    ```

This step temporarily stores the incoming text for reference in later steps.

---

### Step 4. Add Test Data for Validation

* Right-clicked canvas → **Add Annotation**.
* Pasted the sample report text:

  ```
  Subject: IOC intake from recent intelligence report

  At 07:22 UTC, suspicious activity was detected involving connections to external IPs and domains not commonly seen in the environment.

  Observed domains in DNS logs:
  statsrvv.com
  1312services.ru
  kaspersky-secure.ru
  werdotx.shop

  IP addresses noted in firewall logs:
  94.152.193.247
  211.101.234.142
  114.96.92.155

  Suspicious URLs identified in proxy logs:
  https://jsonlint.com/

  File hashes from EDR quarantine:
  SHA256: 226a723ffb4a91d9950a8b266167c5b354ab0db1dc225578494917fe53867ef2
  SHA256: 096451c8d271c5c5768462a9273ca336cba6396b8a3fa0add9d57147cb809511

  Email artifacts:
  jefferson@airplane.com
  ```
* This serves as both sample data and inline workflow documentation.

---

### Step 5. Test Variable Storage

* Ran the workflow once and confirmed in the **Execution Log** of the “Incoming Threat Intelligence Report” step that the full text was stored under `report`.

---

### Step 6. Add the IOC Extraction Utility

* From **Builderbox → Utilities**, searched for **Extract IOCs** and dragged it under the variable step.

* Configured **Input parameter** to:

  ```text
  {{ $.incoming_threat_intelligence_report.report }}
  ```

* Clicked **Evaluate expressions** to verify the context contained the raw text from the last run.

* Executed the step to confirm IOCs were extracted under structured categories such as:

  * **Email**
  * **Hostname (Domains)**
  * **IP Address**
  * **URL**
  * **File Hashes**

---

### Step 7. Add Conditional Logic

* From **Builderbox → Operators**, dragged an **IF** operator under the Extract IOCs step.
* Renamed it to **If IOCs Exist**.
* Configured:

  * **Value:** `{{ $.extract_iocs.results }}`
  * **Condition Operator:** `Not Empty`

This ensures the workflow can take different paths based on whether IOCs were successfully extracted.

---

### Step 8. Handle Empty Results with an Exit Operator

* Dragged an **Exit** operator onto the **FALSE branch** of the IF operator.
* Renamed it **Exit with No Results**.
* Configured:

  * **Workflow Status:** Failed
  * **Parameters:**

    | Type       | Name          | Value          |
    | ---------- | ------------- | -------------- |
    | Short text | error_message | IOCs not Found |
    | Number     | error_code    | 1              |

This step gracefully ends execution if no indicators were found, returning a structured error payload.

---

### Step 9. Validate Conditional Branches

* **Positive Test:** Used the full annotated report → IOCs found → IF = TRUE branch.
* **Negative Test:** Pasted sample text with no indicators.

  * Verified in **If IOCs Exist** step log → `Condition: False`.
  * Checked **Exit with No Results** execution log →

    ```json
    {
      "error_message": "IOCs not Found",
      "error_code": 1
    }
    ```

---

## 4. Example Workflow Outputs

### **Run 1 – IOC Report with Indicators**

**Output:**

```json
{
  "Extracted_IOCs": {
    "IP": ["94.152.193.247", "211.101.234.142", "114.96.92.155"],
    "Domain": ["statsrvv.com", "1312services.ru", "kaspersky-secure.ru", "werdotx.shop"],
    "Email": ["jefferson@airplane.com"],
    "URL": ["https://jsonlint.com/"],
    "Hash": [
      "226a723ffb4a91d9950a8b266167c5b354ab0db1dc225578494917fe53867ef2",
      "096451c8d271c5c5768462a9273ca336cba6396b8a3fa0add9d57147cb809511"
    ]
  }
}
```

### **Run 2 – No IOC Input**

**Output:**

```json
{
  "error_message": "IOCs not Found",
  "error_code": 1
}
```

---

## 5. Key Learnings

1. **Workflow Variables** – store temporary runtime data for downstream actions.
2. **Context References (`$.event.*`)** – allow dynamic access to user input and workflow memory.
3. **IOC Extraction Utility** – transforms free text into actionable data objects for enrichment.
4. **Conditional Logic (IF Operator)** – enables branching for error handling and logic control.
5. **Structured Exit Handling** – standardizes error reporting and integrates smoothly with chained workflows.

---

## 6. Optional Local Simulation (Python Equivalent)

```python
import re

def extract_iocs(text):
    iocs = {
        "ips": re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', text),
        "domains": re.findall(r'\b[a-zA-Z0-9-]+\.[a-zA-Z]{2,}\b', text),
        "emails": re.findall(r'[\w\.-]+@[\w\.-]+', text),
        "urls": re.findall(r'https?://[^\s]+', text),
        "hashes": re.findall(r'[A-Fa-f0-9]{64}', text)
    }
    return {k: v for k, v in iocs.items() if v}

def run_workflow(text):
    iocs = extract_iocs(text)
    if not iocs:
        return {"error_message": "IOCs not Found", "error_code": 1}
    return {"Extracted_IOCs": iocs}

# Example run
sample = "Suspicious host statsrvv.com IP 94.152.193.247 contact jefferson@airplane.com"
print(run_workflow(sample))
```

**Output Example**

```json
{
  "Extracted_IOCs": {
    "ips": ["94.152.193.247"],
    "domains": ["statsrvv.com"],
    "emails": ["jefferson@airplane.com"]
  }
}
```

---

## 7. Folder Structure

```
/torq/automation-practitioner/academy-playground-2-ioc-intake-workflow/
├── README.md
└── scripts/
    └── automated_ioc_intake.py
```

---

## 8. Reflection

This lab taught how to **bridge unstructured and structured data** inside an automated SOAR workflow — a critical skill for real-world security automation.
It reinforced fundamental design patterns: dynamic variable referencing, contextual validation, and conditional workflow branching.
In practice, this approach can be extended to enrich threat intelligence, correlate across detections, and auto-close low-value alerts in production SOC environments.