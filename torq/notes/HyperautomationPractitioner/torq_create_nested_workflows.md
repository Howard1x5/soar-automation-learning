# Nested Workflows in Torq

## 1. Summary

This module covers **nested workflows**—Torq’s mechanism for breaking complex processes into modular, reusable components. Nested workflows work like functions or subroutines in programming: you create a workflow once, then call it anywhere.

Nested workflows are foundational for large-scale automation, helping SOC teams maintain consistency, reduce duplicated logic, and scale out complex orchestration patterns.

Key concepts:

* What nested workflows are
* How they’re structured
* How to create and return structured outputs
* How parent workflows call and consume nested workflows
* Error handling and best practices
* How this model maps to XSOAR, Shuffle, and Splunk SOAR
* Python-equivalent logic for your GitHub portfolio

---

## 2. Concept Overview – What Are Nested Workflows?

Nested workflows are workflows **designed to be triggered only by other workflows**, not by external events. They serve as reusable logic blocks.

### Benefits

* **Reusability** – Build logic once, reuse across dozens of workflows.
* **Modularity** – Smaller, simpler workflows instead of one huge graph.
* **Consistency** – Standardized enrichment/normalization logic across the SOC.
* **Scalability** – Easier long-term maintenance and updates.

### Analogy

Nested workflows = **functions** in programming
Parent workflows = **main program**

---

## 3. Typical SOC Use Case – Threat Intelligence Management

Threat intel workflows often follow the same steps:

1. Extract IOCs (IPs, domains, hashes, URLs, emails).
2. Normalize and deduplicate.
3. Enrich them (VT, WHOIS, reputation API).
4. Upload to a TIP (OpenCTI, MISP, CrowdStrike Fusion, etc.).

Rather than rebuilding these steps for each workflow, Torq uses **nested workflows** to centralize IOC extraction & preparation.

This enables reuse across:

* Phishing workflows
* Malware sandbox results
* Hunting pipelines
* SIEM correlation logic
* DFIR case automation

---

## 4. Structure of a Nested Workflow

A nested workflow typically includes:

### 4.1 Trigger

Configured as:

* **"Nested only"**
* Meaning: it cannot be triggered manually or via webhook unless explicitly called by another workflow.

### 4.2 Inputs

Common inputs:

* `raw_text` (IOC text blob)
* `input_indicators` (JSON array)
* Credentials / integration parameters
* Control flags (e.g., validation mode)

### 4.3 Processing

Typical processing steps:

1. **Extract IOC types**

   * Regex
   * Built-in IOC extraction operators
   * jqFilter
2. **Normalize and clean data**

   * Trim whitespace
   * Lowercase domains
   * Deduplicate
   * Remove empty strings
3. **Structure output**

   * Construct a JSON payload such as:

```json
{
  "ip": ["1.1.1.1"],
  "domain": ["evil.com"],
  "sha256": [],
  "count": 2
}
```

4. **Error handling**

   * If no IOCs identified → return structured error object.

---

## 5. Output Handling & Error Control

Nested workflows usually end in an **Exit Operator**.

Two main exit paths:

### 5.1 No IOCs Found

```json
{
  "error_message": "No IOCs found",
  "error_code": 1
}
```

Parent workflow interprets this and either:

* Stops processing
* Notifies analyst
* Logs as informational event

### 5.2 Successful IOC Extraction

```json
{
  "error_code": 0,
  "ioc_count": 4,
  "iocs": {
    "ip": ["8.8.8.8", "4.4.4.4"],
    "domain": ["malware.com"],
    "sha256": ["<hash>"]
  }
}
```

Parent workflows can now:

* Loop over IPs to enrich with VirusTotal
* Push domains to OpenCTI
* Flag hashes in your EDR or blocklists
* Store full IOC bundle into a case management system

---

## 6. Calling a Nested Workflow from a Parent Workflow

### 6.1 Run Workflow Operator

In the parent workflow:

1. Drag **Run Workflow**
2. Select the nested workflow
3. Map input fields:

   * Parent variable → nested workflow variable
4. Capture output via:

   * `$.run_workflow_name.response`
   * Example:

```
{{ $.nested_extract_iocs.response.iocs.ip }}
```

### 6.2 Best Practices

* Escape strings if passing JSON as text.
* Document input → output mapping clearly.
* Call nested workflow at the earliest point after data ingestion.
* Always branch on error code:

```
IF $.nested_extract_iocs.response.error_code == 0
    → process IOCs
ELSE
    → log and exit
```

---

## 7. Publishing Behavior

The publishing rules matter:

* **Nested workflow must be published first.**
* Parent workflow will not be valid if nested workflow isn’t published.
* You can set nested workflow to **“nested only”** so users don’t accidentally run it manually.

---

## 8. Practical Scenario from Module

Workflow behavior:

### Analyst Input (Parent)

* Analyst pastes raw text containing potential IOCs.

### Nested Workflow:

1. Extract IP addresses using regex / parsing tool.
2. Deduplicate and strip whitespace.
3. Build JSON object with:

   * list of IPs
   * count
   * type
4. Return JSON to parent workflow.

### Parent Workflow

* Loops over extracted IOCs.
* Uploads them to OpenCTI (or any TIP).
* Logs completion.

### Result

All IOCs are structured, enriched, and uploaded automatically.

---

## 9. Additional Implementation Notes

### 9.1 Security

* Store API keys in Torq Secrets Vault.
* Limit variable exposure on nested workflows.
* Do not pass raw credentials through workflow inputs.

### 9.2 Testing

* Run nested workflow manually during development to verify parsing logic.
* Provide multiple IOC input samples:

  * Single-value
  * Multi-value
  * Mixed (URLs, domains, IPs)
  * Null/empty cases

### 9.3 Version Control

* Bump version numbers when modifying nested workflows.
* Document changes to keep parent workflows in sync.

### 9.4 Naming Conventions

Use predictable prefixes:

* `NW_Extract_IOCs`
* `NW_Parse_Email`
* `NW_Enrich_IP`
* `NW_Block_Indicators`

---

## 10. Cross-SOAR Comparison

### Shuffle SOAR

* Equivalent concept: *Subflows* or *Reusable Workflows*.
* Called via “Call Workflow” action.
* Similar JSON input/output requirements.
* Must explicitly define return fields.

### Cortex XSOAR

* Equivalent: **Sub-Playbooks**.
* Inputs: playbook inputs.
* Outputs: playbook outputs (context).
* Highly standardized for IOC extraction and enrichment.

### Splunk SOAR (Phantom)

* Equivalent: **Playbook Calls / Custom Functions**.
* Custom Functions often mirror nested workflows.
* Use `phantom.collect()` and `phantom.act()` for input/output handling.

Across all platforms:

* Nested workflows = modular, reusable building blocks
* Parent workflows orchestrate higher-level logic
* Error handling conventions are nearly identical

---

## 11. Python Equivalent Pattern (For Your Portfolio)

Below is a Python version of a nested IOC extraction workflow you can add to your repo for tool-agnostic documentation.

```python
import re
import json

def extract_iocs(raw_text: str):
    """
    Function equivalent of a nested workflow.
    Extracts IP addresses from raw text.
    """

    # Regex for IPv4
    ip_regex = r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
    ips = re.findall(ip_regex, raw_text)

    # Deduplicate
    ips = list(set(ips))

    if not ips:
        return {
            "error_code": 1,
            "error_message": "No IOCs found",
            "iocs": {}
        }

    return {
        "error_code": 0,
        "ioc_count": len(ips),
        "iocs": {
            "ip": ips
        }
    }


# Example parent workflow logic
def upload_to_tip(ioc_bundle):
    # Placeholder – simulate TIP upload
    for ip in ioc_bundle.get("iocs", {}).get("ip", []):
        print(f"[INFO] Uploading IP to TIP → {ip}")


if __name__ == "__main__":
    text_blob = """
    Suspicious outbound connections to:
    8.8.8.8 and 4.4.4.4
    """

    results = extract_iocs(text_blob)

    if results["error_code"] == 0:
        upload_to_tip(results)
    else:
        print("[INFO] No IOCs found.")
```

This script mirrors the Torq nested workflow pattern:

* Function = nested workflow
* JSON output = exit operator
* Parent code handles error conditions and takes action

---

## 12. Quick Reference Summary

* Nested workflows = reusable subroutines for automation logic.
* Ideal for IOC extraction, normalization, enrichment, and TIP uploads.
* Inputs → processing → structured JSON output.
* Parent workflows call nested workflows through the Run Workflow step.
* Both parent and nested workflows must be published.
* Nested workflows can be set to “nested only.”
* Follow consistent naming conventions and version control.
* Strong overlap with Sub-Playbooks (XSOAR), Subflows (Shuffle), and Custom Functions (Splunk SOAR).