# Torq HyperAutomation Practitioner – Playground 2B

## Organize and Structure IOCs for Analysis

### 1. Summary

This challenge extends the IOC intake workflow from Playground 2A.
The goal is to turn extracted Indicators of Compromise (IOCs) into a **normalized, structured JSON summary** that’s ready for downstream enrichment, correlation, or case management.

Core tasks:

* Loop through extracted IOCs
* Sort IOCs by type using a Switch operator
* Collect indicators into grouped arrays
* Use a Transform step (JQ) to flatten and normalize
* Return a final, clean JSON object from an Exit step

Along the way, the biggest pain point was the **Transform operator using AI-generated instructions**, which produced subtle but fatal schema mismatches. The final solution required **hand-editing the transformation plan in YAML** to get deterministic, grader-approved output.

---

### 2. Objective

* Loop over the IOC results from the “Extract IOCs” step.
* Sort IOCs into:

  * SHA256 hashes
  * Domains
  * IPv4 addresses
  * URLs
  * Uncategorized (anything that did not match the above)
* Collect each category into its own list.
* Use a Transform step to:

  * Flatten all categories into a single unified list
  * Standardize field names (`ioc_type`, `ioc_value`, and `ioc_subtype` for the uncategorized case)
  * Wrap everything into a top-level object called `iocs`
* Exit with:

  ```json
  {
    "iocs": { "iocs": [...] },
    "error_code": 0
  }
  ```

---

### 3. Workflow Logic (Torq)

#### 3.1. Loop Over IOCs

* On the TRUE branch of `If IOCs Exist`, added a **Loop** operator:

  * Name: `Loop over all IOCs`
  * List: `{{ $.extract_iocs.results }}`
  * Value name: `ioc_value`
  * Key name: `ioc_key`

Each iteration exposes the current IOC object via `$.ioc_value`.

---

#### 3.2. Sort by IOC Type (Switch)

* Added a **Switch** operator under the loop:

  * Name: `Sort by IOC Type`

Branches:

* **SHA256 branch**

  * Condition 1: `{{ $.ioc_value.type }} Equals File Hash`
  * Condition 2: `{{ $.ioc_value.subType }} Equals SHA256`

* **Hostname branch**

  * Condition: `{{ $.ioc_value.type }} Equals Hostname`

* **IP Address branch**

  * Condition: `{{ $.ioc_value.type }} Equals IP Address`

* **URLs branch**

  * Condition: `{{ $.ioc_value.type }} Equals URL`
    (Capitalization matters.)

* **Default branch**

  * No explicit condition; catches all remaining IOCs
  * Used to preserve full objects (type, subtype, value) for anything “uncategorized” (e.g., email address).

---

#### 3.3. Collect IOCs by Type

Attached **Collect** operators under each branch:

* **Collect SHA256 Hashes**

  * Input: `{{ $.ioc_value.value }}`

* **Collect Domains**

  * Input: `{{ $.ioc_value.value }}`

* **Collect IPv4 Addresses**

  * Input: `{{ $.ioc_value.value }}`

* **Collect URLs**

  * Input: `{{ $.ioc_value.value }}`

* **Collect Uncategorized IOCs**

  * Input: `{{ $.ioc_value }}`
  * This collector intentionally stores the *entire IOC object* so that subtype and type can be used later in the transform.

All collectors were validated via their Execution Logs.

---

#### 3.4. Build Transform Input

Added a **Transform** operator on the TRUE branch of `If IOCs Exist`, after the loop:

* Name: `Build Final Output`
* Input (as JSON):

```json
{
  "sha256": {{ $.collect_sha256_hashes }},
  "domains": {{ $.collect_domains }},
  "ip_addresses": {{ $.collect_ipv4_addresses }},
  "urls": {{ $.collect_urls }},
  "uncategorized": {{ $.collect_uncategorized_iocs }}
}
```

This object is exposed inside the Transform step as `INPUT_JSON`.

---

### 4. Troubleshooting: Why the AI Transform Kept Failing

Initially, I followed the lab instructions and used **Socrates (Torq’s AI)** to generate JQ transformations via natural-language prompts:

1. “Transform the data into json table with columns ioc_type, ioc_value…”
2. “For items where ioc_type is equal to uncategorized rename ioc_type to ioc_subtype.”
3. “Where subtype exists remove it from the ioc_value and give it a column of its own.”
4. “Put this table in an object called iocs.”

Problems that appeared:

* The email IOC row was being flattened incorrectly, producing:

  ```json
  {
    "ioc_type": "ioc_subtype",
    "ioc_value": "jefferson@airplane.com",
    "subtype": "Email Address"
  }
  ```
* The lab grader wanted a row with **`ioc_subtype`**, not `"ioc_subtype"` as a value:

  ```json
  {
    "ioc_subtype": "Email Address",
    "ioc_value": "jefferson@airplane.com"
  }
  ```
* Occasional presence of `ioc_type: null` in the uncategorized row.
* The AI-generated `TRANSFORMATION_PLAN` in YAML became large and opaque, and small tweaks in prompts led to different underlying JQ, making debugging painful.

In short: **the AI helper was non-deterministic and slightly wrong in exactly the way the grader cared about**.

---

### 5. Final Fix: Manual JQ in YAML

To eliminate the AI’s inconsistency, I edited the Transform step’s YAML directly and replaced `TRANSFORMATION_PLAN` with a simple, deterministic JQ pipeline.

Conceptually, the JQ does this:

1. Convert the grouped buckets (`sha256`, `domains`, `ip_addresses`, `urls`, `uncategorized`) into `{ key, value }` pairs.
2. For each bucket, explode the `.result` array into individual rows.
3. For standard buckets:

   * Emit `{ ioc_type: <bucket_name>, ioc_value: <indicator> }`.
4. For the uncategorized bucket:

   * The stored objects look like:
     `{ type: "Email Address", subType: "Email Address", value: "jefferson@airplane.com", ... }`
   * Emit `{ ioc_subtype: .subType (or .type), ioc_value: .value }`.
5. Wrap everything in `{ iocs: [...] }`.

After this was in place, the Transform step output looked like:

```json
{
  "output": {
    "iocs": [
      { "ioc_type": "domains", "ioc_value": "statsrvv.com" },
      { "ioc_type": "domains", "ioc_value": "1312services.ru" },
      { "ioc_type": "domains", "ioc_value": "kaspersky-secure.ru" },
      { "ioc_type": "domains", "ioc_value": "werdotx.shop" },
      { "ioc_type": "domains", "ioc_value": "jsonlint.com" },
      { "ioc_type": "domains", "ioc_value": "airplane.com" },
      { "ioc_type": "ip_addresses", "ioc_value": "94.152.193.247" },
      { "ioc_type": "ip_addresses", "ioc_value": "211.101.234.142" },
      { "ioc_type": "ip_addresses", "ioc_value": "114.96.92.155" },
      { "ioc_type": "sha256", "ioc_value": "226a7..." },
      { "ioc_type": "sha256", "ioc_value": "09645..." },
      { "ioc_subtype": "Email Address", "ioc_value": "jefferson@airplane.com" },
      { "ioc_type": "urls", "ioc_value": "https://jsonlint.com/" }
    ]
  },
  "step_status": { ... }
}
```

The `step_status` envelope is added by Torq internally and is ignored by the Exit step.

---

### 6. Final Exit Step

The final **Exit with Results** step was configured with:

```json
{
  "iocs": {{ $.build_final_output.output }},
  "error_code": 0
}
```

This strips off the `step_status` wrapper and returns only:

```json
{
  "iocs": {
    "iocs": [
      ...
    ]
  },
  "error_code": 0
}
```

The Playground checker accepted this structure.

---

### 7. Python Equivalent – IOC Organization & Flattening

Below is a Python script that mimics the same logic outside Torq:

* Extract IOCs from a raw report
* Group them by type
* Handle “uncategorized” (e.g., email) with subtype
* Flatten into the same JSON structure the Torq workflow produced

```python
import re
from typing import Dict, List, Any


def extract_iocs(text: str) -> Dict[str, List[Any]]:
    """Rudimentary IOC extractor to simulate Torq's Extract IOCs utility."""
    ips = re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', text)
    urls = re.findall(r'https?://[^\s]+', text)
    emails = re.findall(r'[\w\.-]+@[\w\.-]+\.\w+', text)
    domains = re.findall(r'\b[a-zA-Z0-9-]+\.[a-zA-Z]{2,}\b', text)

    # crude SHA256 detection
    hashes = re.findall(r'\b[a-fA-F0-9]{64}\b', text)

    # remove domains that are clearly part of URLs
    url_domains = {re.sub(r'^https?://', '', u).split('/')[0] for u in urls}
    domains_only = [d for d in domains if d not in url_domains]

    return {
        "ip_addresses": ips,
        "urls": urls,
        "domains": domains_only,
        "sha256": hashes,
        "uncategorized": [
            {
                "type": "Email Address",
                "value": e
            }
            for e in emails
        ]
    }


def organize_iocs(raw_iocs: Dict[str, List[Any]]) -> Dict[str, Any]:
    """Mimic the Torq Loop + Switch + Collect + Transform behavior."""
    flattened: List[Dict[str, Any]] = []

    # standard buckets
    for domain in raw_iocs.get("domains", []):
        flattened.append({"ioc_type": "domains", "ioc_value": domain})

    for ip in raw_iocs.get("ip_addresses", []):
        flattened.append({"ioc_type": "ip_addresses", "ioc_value": ip})

    for h in raw_iocs.get("sha256", []):
        flattened.append({"ioc_type": "sha256", "ioc_value": h})

    for url in raw_iocs.get("urls", []):
        flattened.append({"ioc_type": "urls", "ioc_value": url})

    # uncategorized: treat as subtype-based entries
    for obj in raw_iocs.get("uncategorized", []):
        subtype = obj.get("type", "Uncategorized")
        value = obj.get("value")
        flattened.append({
            "ioc_subtype": subtype,
            "ioc_value": value
        })

    return {
        "iocs": {
            "iocs": flattened
        },
        "error_code": 0
    }


if __name__ == "__main__":
    sample_report = """
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
    """

    iocs_raw = extract_iocs(sample_report)
    final_output = organize_iocs(iocs_raw)

    import json
    print(json.dumps(final_output, indent=2))
```

This script could be:

* A standalone helper run from a SOAR platform via Python
* The logic inside a custom playbook step in XSOAR / Splunk SOAR / Shuffle
* A unit-testable equivalent of what the Torq pipeline is doing visually

---

### 8. How This Maps to Other SOAR Tools

If this were implemented in other SOAR platforms:

* **Shuffle**

  * Use HTTP or Python app to ingest the raw report.
  * Add a Python step (or function) that:

    * Extracts indicators via regex or external services.
    * Produces a unified JSON similar to `final_output` above.
  * Store `final_output["iocs"]["iocs"]` as artifacts or case data.

* **Splunk SOAR (Phantom)**

  * Use custom functions or a Python “playbook block” to do IOC extraction and normalization.
  * Output artifacts with fields `ioc_type`, `ioc_value`, and optionally `ioc_subtype`.

* **Cortex XSOAR**

  * Use a Python automation or script to:

    * Parse the input text
    * Create incident fields or context keys with `iocs = [...]`
  * Use that normalized list for enrichment sub-playbooks and correlation rules.

The core pattern is identical across tools:
**extract → group → normalize → flatten → return a clean IOC list for downstream logic.**

---

### 9. Reflection

This challenge was less about clicking through UI steps and more about:

* Understanding how IOC data should be structured for real SOC workflows
* Recognizing the limits of “AI helpers” in deterministic pipelines
* Dropping down a level (YAML + JQ) to take control of the logic

When I build my own open-source SOAR stack (Shuffle + detections feeding into it), this pattern will be reused:

* Consolidate noisy, scattered IOCs
* Normalize formats
* Feed standardized structures into enrichment, alert deduplication, and auto-closure logic

This lab was a good forcing function to get comfortable with that level of control.