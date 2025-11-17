# Torq Playground 3 – Challenge 3B - Enrich the Analyst’s Choice

## 1. Summary

This workflow takes an IOC selected by the analyst, determines its type, and routes it through the correct enrichment path using VirusTotal. Supported IOC types include file hashes (SHA256), domains, IP addresses, and URLs. Unsupported or uncategorized IOCs are explicitly handled and cause the workflow to exit cleanly.

This challenge builds on the previous “Capture and Present IOCs” workflow, which:

1. Accepted a free-form threat report.
2. Extracted IOCs via a nested workflow.
3. Displayed them in a table.
4. Captured the analyst’s chosen IOC and identified its type.

The selected IOC is enriched with VirusTotal.

---

## 2. Objectives

1. Use a Switch operator to route IOCs by type:

   * SHA256 file hash
   * Domain
   * IP address
   * URL
2. Integrate VirusTotal and configure:

   * Get File Report by Hash
   * Get Domain Information
   * Get IP Information
   * Scan URL + Get URL Analysis Report
3. Exit the workflow gracefully when an uncategorized IOC is selected.
4. Normalize enrichment responses into a single `enrichment_results` context variable.

---

## 3. Torq Implementation Steps (Condensed)

### A. Configure IOC Sorting Conditions

1. Open the workflow:

   * `Workflows → Hyperautomation Practitioner → Enrich IOCs → Edit`.
2. Enable the `Sort IOC` Switch operator:

   * Click the Eye icon to enable it.
   * Branches already exist: `SHA256`, `Domain`, `IP Address`, `URL`, and `Default`.
3. Set branch conditions using the output of the `Identify IOC Type` step:

* **SHA256 branch**

  * Condition 1:
    `{{ $.identify_ioc_type.results.0.type }} Equals File Hash`
  * Condition 2:
    `{{ $.identify_ioc_type.results.0.subType }} Equals SHA256`

* **Domain branch**

  * `{{ $.identify_ioc_type.results.0.type }} Equals hostname`

* **IP Address branch**

  * `{{ $.identify_ioc_type.results.0.type }} Equals IP Address`

* **URL branch**

  * `{{ $.identify_ioc_type.results.0.type }} Equals url`

The `Default` branch catches any IOC that does not match these conditions.

---

### B. Obtain a VirusTotal API Key

1. Go to `https://www.virustotal.com`.
2. Sign up and create an account.
3. Sign in and click your username → `API key`.
4. Copy the API key. You will use it to configure the Torq integration.

---

### C. Create VirusTotal Integration and Enrich File Hashes

1. In the Builderbox, search for `Get File Report by Hash` (VirusTotal Integration).

2. Drag `Get File Report by Hash` onto the `SHA256` branch of `Sort IOC`.

3. Click the step to open Properties → Integration dropdown → `+ Create new integration`.

4. New instance configuration:

   * Instance name: `virustotal`
   * VirusTotal API Key: `<your API key>`
   * Base URL: `https://www.virustotal.com/api/v3`

5. Click `Test Integration`, then `Add`.

6. Set the File hash parameter:

   * `File hash = {{ $.identify_ioc_type.results.0.value }}`

7. Test:

   * Run the workflow.
   * Use the prior challenge’s interaction form to select a SHA256 IOC.
   * Verify in the `Get File Report by Hash` Execution Log that the step returns an enrichment response.

---

### D. Enrich Domain Information

1. On the `Domain` branch of `Sort IOC`:

   * Add `Get Domain Information` (VirusTotal Integration).
2. Set:

   * `Domain = {{ $.identify_ioc_type.results.0.value }}`

---

### E. Enrich IP Address Information

1. On the `IP Address` branch of `Sort IOC`:

   * Add `Get IP Information` (VirusTotal Integration).
2. Set:

   * `IP address = {{ $.identify_ioc_type.results.0.value }}`

---

### F. Enrich URL Information

1. On the `URL` branch of `Sort IOC`:

   * Add `Scan URL` (VirusTotal Integration).

2. Set:

   * `Scan URL = {{ $.identify_ioc_type.results.0.value }}`

3. Add a `Wait` operator under `Scan URL`:

   * `Duration = 1 Seconds`.

4. Under the `Wait` operator, add `Get URL Analysis Report` (VirusTotal Integration).

5. Set:

   * `URL Address = {{ $.identify_ioc_type.results.0.value }}`

This sequence (Scan URL → Wait → Get URL Analysis Report) handles cases where VirusTotal provides URL reports asynchronously.

---

### G. Handle Uncategorized IOCs (Default Branch)

1. The `Default` branch of `Sort IOC` receives IOCs that do not match the other branch conditions.
2. An `Uncategorized IOC` Interaction step is already present on the `Default` branch.
3. Under `Uncategorized IOC Interaction`, add an `Exit` operator.
4. Keep default Exit settings:

   * Workflow stops after notifying the user that the IOC type is not supported for enrichment.

---

### H. Store Enrichment Results

For each enriched branch, normalize the VirusTotal response into a shared `enrichment_results` variable.

1. SHA256 branch:

   * Under `Get File Report by Hash`, add `Set Variable`:

     * Step name: `Enrichment Results`
     * Variable name: `enrichment_results`
     * Data type: `JSON`
     * Value json:
       `{{ $.get_file_report_by_hash.api_object.data }}`

2. Domain branch:

   * Under `Get Domain Information`, add `Set Variable`:

     * Step name: `Enrichment Results`
     * Variable name: `enrichment_results`
     * Data type: `JSON`
     * Value json:
       `{{ $.get_domain_information.api_object.data }}`

3. IP Address branch:

   * Under `Get IP Information`, add `Set Variable`:

     * Step name: `Enrichment Results`
     * Variable name: `enrichment_results`
     * Data type: `JSON`
     * Value json:
       `{{ $.get_ip_information.api_object.data }}`

4. URL branch:

   * Under `Get URL Analysis Report`, add `Set Variable`:

     * Step name: `Enrichment Results`
     * Variable name: `enrichment_results`
     * Data type: `JSON`
     * Value json:
       `{{ $.get_url_analysis_report.api_object.data }}`

Result: all branches converge on a common `enrichment_results` JSON blob, regardless of IOC type.

---

### I. Publish and Validate

1. Publish the workflow.
2. Test multiple times, each time choosing a different IOC type from the table:

   * SHA256 file hash
   * Domain
   * IP address
   * URL
3. Confirm:

   * The correct VirusTotal integration step runs for each IOC type.
   * `enrichment_results` is populated with the expected data.
   * Uncategorized IOC types hit the `Uncategorized IOC` interaction and exit cleanly.

---

## 4. Python Equivalent: VirusTotal Enrichment

Below is a Python module that mirrors the Torq behavior in a tool-agnostic way:

* Routes based on IOC type.
* Calls VirusTotal v3 API for:

  * File hash report
  * Domain information
  * IP information
  * URL scan + analysis
* Normalizes the result into a consistent `enrichment_results` dictionary.

Save as:

`virustotal_enrichment.py`

```python
import os
import time
import base64
import requests


VT_BASE_URL = "https://www.virustotal.com/api/v3"
VT_API_KEY = os.environ.get("VT_API_KEY")


class VirusTotalClient:
    def __init__(self, api_key: str | None = None):
        key = api_key or VT_API_KEY
        if not key:
            raise ValueError("VT_API_KEY environment variable is not set.")
        self.api_key = key
        self.session = requests.Session()
        self.session.headers.update({
            "x-apikey": self.api_key
        })

    def _get(self, path: str):
        url = f"{VT_BASE_URL}{path}"
        resp = self.session.get(url)
        resp.raise_for_status()
        return resp.json()

    def _post(self, path: str, data: dict):
        url = f"{VT_BASE_URL}{path}"
        resp = self.session.post(url, data=data)
        resp.raise_for_status()
        return resp.json()

    def get_file_report_by_hash(self, file_hash: str) -> dict:
        # v3 endpoint for files by hash
        return self._get(f"/files/{file_hash}")

    def get_domain_information(self, domain: str) -> dict:
        return self._get(f"/domains/{domain}")

    def get_ip_information(self, ip_address: str) -> dict:
        return self._get(f"/ip_addresses/{ip_address}")

    def scan_url(self, url_value: str) -> dict:
        # Returns an analysis object with an ID
        return self._post("/urls", data={"url": url_value})

    def get_url_analysis_report(self, url_value: str, wait_seconds: int = 1) -> dict:
        """
        Mirrors Torq's Scan URL + Wait + Get URL Analysis Report sequence.

        - First submit the URL for scanning.
        - Wait a bit (similar to the WAIT operator).
        - Then query the analysis report.
        """
        analysis = self.scan_url(url_value)
        analysis_id = analysis.get("data", {}).get("id")
        if not analysis_id:
            raise RuntimeError("No analysis ID returned from VirusTotal for URL scan.")

        time.sleep(wait_seconds)
        # Analysis reports are under /analyses/{id}
        return self._get(f"/analyses/{analysis_id}")


def enrich_ioc(ioc_value: str, ioc_type: str, client: VirusTotalClient | None = None) -> dict:
    """
    Enriches an IOC based on its type.

    Supported types (example mapping from the Torq step):
    - "File Hash" + subType "SHA256"  -> file report
    - "hostname"                      -> domain info
    - "IP Address"                    -> IP info
    - "url"                           -> URL analysis

    Returns a normalized dict: {"ioc": ..., "type": ..., "enrichment": {...}}.
    """
    vt = client or VirusTotalClient()

    # Normalize types to what Torq is using in the Sort IOC step
    # In Torq, you had:
    # - type = "File Hash" + subType "SHA256"
    # - type = "hostname"
    # - type = "IP Address"
    # - type = "url"
    enrichment_data: dict

    if ioc_type == "File Hash":
        # Assuming SHA256 as per the challenge
        enrichment_data = vt.get_file_report_by_hash(ioc_value)
    elif ioc_type == "hostname":
        enrichment_data = vt.get_domain_information(ioc_value)
    elif ioc_type == "IP Address":
        enrichment_data = vt.get_ip_information(ioc_value)
    elif ioc_type == "url":
        enrichment_data = vt.get_url_analysis_report(ioc_value)
    else:
        # Equivalent to the Default branch: unsupported / uncategorized IOC type
        return {
            "ioc": ioc_value,
            "type": ioc_type,
            "supported": False,
            "enrichment": {},
            "reason": "Unsupported IOC type for VirusTotal enrichment."
        }

    return {
        "ioc": ioc_value,
        "type": ioc_type,
        "supported": True,
        "enrichment": enrichment_data
    }


if __name__ == "__main__":
    # Simple CLI driver for local testing
    import argparse

    parser = argparse.ArgumentParser(description="Enrich a single IOC with VirusTotal.")
    parser.add_argument("ioc_value", help="IOC value (hash, domain, IP, or URL).")
    parser.add_argument(
        "ioc_type",
        help="IOC type as used in the Torq Sort IOC step "
             '(e.g. "File Hash", "hostname", "IP Address", "url").'
    )
    args = parser.parse_args()

    result = enrich_ioc(args.ioc_value, args.ioc_type)
    print(result)
```

Notes:

* This script expects `VT_API_KEY` as an environment variable.
* The flow mirrors Torq:

  * For URLs, it does `scan_url()` then waits and retrieves the analysis report, similar to the `Scan URL → Wait → Get URL Analysis Report` pattern with the WAIT operator.
* The return structure mirrors your `enrichment_results` variable: a single JSON object that downstream systems can consume.

---

## 5. Cross-SOAR Design: How This Maps to Other Platforms

This challenge is a canonical enrichment pattern you can replicate in other SOAR tools using roughly the same steps:

1. Determine IOC type.
2. Route to the correct enrichment action.
3. Normalize the response into a common structure.
4. Handle unsupported IOCs explicitly.

### Shuffle SOAR

* Trigger:

  * HTTP endpoint / form, or an upstream workflow that passes `ioc_value` and `ioc_type`.
* Routing:

  * Use a `Condition` node (or multiple) to route by IOC type.
* Enrichment:

  * VirusTotal app (if available) or a `Python` node using `virustotal_enrichment.py`.
* Storage:

  * Save normalized enrichment JSON into a workflow variable or a `storage` node.
* Default case:

  * If IOC type unsupported, add a `Notification` or `HTTP Response` node indicating “unsupported IOC” and terminate workflow.

Key point:
Shuffle’s Python-heavy design makes it straightforward to re-use the same `VirusTotalClient` code across workflows.

---

### Splunk SOAR (Phantom)

* Trigger:

  * Container with artifacts generated from an upstream investigation.
* IOC Routing:

  * `decision` block based on artifact fields or custom function output (e.g., `indicator_type`).
* Enrichment:

  * Built-in VirusTotal app actions:

    * `file reputation`
    * `domain reputation`
    * `ip reputation`
    * `url reputation` / `url scan`
* Storage:

  * Add normalized artifacts or tags (`set_artifact`, `set_label`, `update_artifact`).
* Default/Unsupported:

  * Branch of `decision` that logs a comment on the container and ends playbook.

Key point:
Phantom has first-class VirusTotal actions, so you mainly focus on indicator typing and branching logic.

---

### Cortex XSOAR (Demisto)

* Trigger:

  * Incident with IOC fields or Indicators attached.
* IOC Routing:

  * Use an `If` task or a playbook that branches on `indicator.type`.
* Enrichment:

  * Use the built-in `VirusTotal v3` integration commands:

    * `vt-get-file-report`
    * `vt-get-domain-report`
    * `vt-get-ip-report`
    * `vt-get-url-report` / analysis
* Storage:

  * Store results in incident context under a unified key, e.g. `Incident.EnrichmentResults`.
* Default/Unsupported:

  * “Uncategorized IOC” path that sets a context field and ends that branch.

Key point:
XSOAR’s context model and indicator types make this pattern very similar to Torq’s Switch + VirusTotal steps.

---

### Microsoft Sentinel / Logic Apps

* Trigger:

  * Logic App triggered by an incident, analytic rule, or HTTP endpoint.
* IOC Routing:

  * Use `Switch` or `Condition` actions in Logic Apps on IOC type.
* Enrichment:

  * HTTP actions calling VirusTotal v3 directly, or:

    * Azure Function with `virustotal_enrichment.py` deployed as a small API.
* Storage:

  * Full enrichment JSON stored in incident tags, comments, or an enrichment table (e.g., custom log table).
* Default/Unsupported:

  * Branch that posts a comment to the incident and terminates the Logic App.

Key point:
Sentinel has no native VirusTotal app; you rely on HTTP + Functions. The pattern is the same, just more plumbing.

---