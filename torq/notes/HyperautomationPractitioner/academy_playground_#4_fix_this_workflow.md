# Hyperautomation Practitioner – Playground: Fix This Workflow (CVE Grouping)

### 1. Scenario

This playground shipped with a deliberately broken Torq workflow that:

* Ingests a JSON CVE feed (NVD format).
* Loops over the vulnerabilities and groups them by **CVSS v3.1 base severity** (Critical / High / Medium / Low).
* Normalizes any **empty severity groups** so the output shape is stable.
* Prepares a final output with:

  * `total_results` (expected: 50 CVEs)
  * `quiz_answer` (`"hyperautomation!"`)

My job was to reverse-engineer the broken logic, fix all errors, and get the workflow to return **exactly 50 CVEs grouped by severity** with a valid quiz answer.

---

### 2. Final Torq Workflow Logic

High-level flow:

1. **CVE List (Variable)**

   * Data type: JSON.
   * Holds the NVD CVE payload under `data`.
2. **Loop over CVEs (Loop operator)**

   * `List`: `{{ $.cve_list.data.vulnerabilities }}`
   * `Value name`: `cve_value`
   * `Key name`: `cve_key`
3. **Place CVEs into Severities (Switch operator)**

   * Branch per severity, evaluated against the CVSS v3.1 base severity:

     * `cvssV3 Critical`
     * `cvssV3 High`
     * `cvssV3 Medium`
     * `cvssV3 Low`
   * Each branch drops the CVE ID into a collector:

     * `Collect Critical` → `collect_critical.result`
     * `Collect High` → `collect_high.result`
     * `Collect Medium` → `collect_medium.result`
     * `Collect Low` → `collect_low.result`
4. **Case: Empty Severity Groups (IF operators)**

   * If a severity collection is empty, replace it with an empty JSON array (`[]`) so downstream math and output shaping don’t break.
5. **Prepare Output**

   * Compute `total_results` by summing the lengths of each severity list.
   * Set `quiz_answer` to `"hyperautomation!"`.
6. **Exit**

   * Exit step returns:

     * `total_results: 50`
     * `quiz_answer: "hyperautomation!"`

---

### 3. Key Fixes I Made

The broken workflow had multiple subtle issues: bad paths into the JSON, incorrect conditions, and typos in variable names. Here are the concrete fixes.

#### 3.1 Loop source – iterate over the right JSON path

* **Problem:** The loop was not iterating over the actual vulnerability array from the CVE feed.
* **Fix:** Pointed the loop to the vulnerabilities array under the variable:

```torq
List: {{ $.cve_list.data.vulnerabilities }}
Value name: cve_value
Key name:   cve_key
```

This ensures each loop iteration receives a single vulnerability object in `cve_value`.

---

#### 3.2 Severity classification – correct JSON path to baseSeverity

Each CVE looks like this (simplified):

```json
{
  "cve": {
    "id": "CVE-2022-26486",
    "metrics": {
      "cvssMetricV31": [
        {
          "cvssData": {
            "baseSeverity": "CRITICAL",
            "version": "3.1"
          }
        }
      ]
    }
  }
}
```

* **Problem:** The switch conditions used an incorrect/unevaluable path into the metrics block, so **every CVE fell through to the default branch**.
* **Fix:** Set the `lvalue` for all four switch cases to the actual baseSeverity path from the loop value:

```torq
{{ $.cve_value.cve.metrics.cvssMetricV31.0.cvssData.baseSeverity }}
```

Then match on the literal severity string per branch:

* Critical branch:

  ```torq
  lvalue:  '{{ $.cve_value.cve.metrics.cvssMetricV31.0.cvssData.baseSeverity }}'
  operator: EQUALS
  rvalue:  CRITICAL
  ```

* High / Medium branches are identical except for `rvalue: HIGH` / `rvalue: MEDIUM`.

Each branch’s collector expression is just the CVE ID:

```torq
expression: '{{ $.cve_value.cve.id }}'
```

---

#### 3.3 Empty Medium group check – flip the logic

There is an IF block that normalizes the Medium group:

* **Original bug:** The condition was effectively **“if Medium is NOT empty”** instead of “if Medium is empty”, so the “empty group” handler never ran and the data shape was inconsistent.
* **Fix:** Corrected the condition to trigger when the Medium list is empty:

```torq
lvalue:  '{{ $.collect_medium.result }}'
operator: IS_EMPTY
rvalue:  ""
```

If the list is empty, I overwrite `collect_medium.result` with an empty JSON array (`[]`). Similar logic exists for other severities.

---

#### 3.4 Output math – fix typo in collector variable

The math step that computes the total count uses a small expression:

```torq
INPUT: '{{ len $.collect_critical.result }} + {{ len $.collect_medium.result }} + {{ len $.collect_high.result }} + {{ len $.collect_low.result }}'
```

* **Original bug:** One of the references used `collect_critical.results` (extra `s`), which made the expression unevaluable.
* **Fix:** Standardized all paths to `.result`, matching the actual collector outputs:

```torq
$.collect_critical.result
$.collect_high.result
$.collect_medium.result
$.collect_low.result
```

---

#### 3.5 Final validation

After the fixes:

* The Exit step shows `total_results: 50`.
* `quiz_answer` is set to `hyperautomation!`.
* The playground’s built-in validation script passes successfully.

---

### 4. How I’d Debug the Same Workflow in Other SOAR Platforms

The core skills here are **data path debugging, branching logic, and normalization of empty groups**. Here’s how I’d apply that in other tools.

#### 4.1 Shuffle SOAR

* Use a HTTP/JSON input or mock node to feed in NVD CVE JSON.
* Loop over `data.vulnerabilities` using a “For Each” node.
* Inside the loop:

  * Use JSONFilter / Jinja/templating to pull `metrics.cvssMetricV31[0].cvssData.baseSeverity`.
  * Route to separate branches (Critical/High/Medium/Low) using conditional nodes.
* Aggregate results in “List Append” nodes for each severity.
* At the end:

  * Normalize missing groups by explicitly initializing lists as `[]`.
  * Use a Python or Script node to compute `total_results` and `quiz_answer`.

Key debugging steps:

* Inspect node outputs for one sample vulnerability.
* Verify the template path returns `CRITICAL`, `HIGH`, or `MEDIUM`.
* Confirm list sizes match expectations before final output.

---

#### 4.2 Cortex XSOAR / Demisto

* Use a playbook that receives the CVE JSON in context (e.g., `CVE.Data`).
* Add a regular task to extract the vulnerabilities list to a key like `CVE.Vulnerabilities`.
* Use a **Loop** task over `CVE.Vulnerabilities`.
* Inside the loop:

  * A “Set” or “Script” task reads `item.cve.metrics.cvssMetricV31[0].cvssData.baseSeverity`.
  * Use a conditional task to branch into Critical / High / Medium / Low paths.
  * Append the `cve.id` to context keys like `CVEGroup.Critical`, etc.
* After the loop:

  * Add a script to ensure each group key exists and is at least an empty list.
  * Compute `total_results = len(CVEGroup.Critical) + ...`.
  * Set a final `quiz_answer` field.

Debug approach:

* Watch the **Work Plan** and context pane.
* Confirm each branch is hit for sample items.
* Check context keys for typos and list sizes.

---

#### 4.3 Splunk SOAR (Phantom / Splunk SOAR)

* Treat each vulnerability as an artifact or data row.
* Use a custom function or playbook block to loop through the vulnerabilities.
* Use simple decision blocks checking the CVSS severity field.
* Add to lists (container or custom function output) for each severity.
* Normalize: if a list is missing/None, set it to an empty list in a final custom function.
* Compute counts and attach a note or output JSON with `total_results` and `quiz_answer`.

Debug approach:

* Use playbook debug traces and custom function outputs.
* Inspect the JSON in the container’s “notes” or output to verify paths and counts.

---

### 5. Python Reference Implementation

Below is a Python script that mirrors the Torq workflow logic outside of any SOAR platform. It:

* Loads a CVE feed from `cve_list.json`.
* Groups CVEs by CVSS v3.1 base severity.
* Normalizes empty groups.
* Computes `total_results` and `quiz_answer`.

You can drop this into `torq/hyperautomation-practitioner/playground-fix-workflow.py` (or similar) in the repo.

```python
#!/usr/bin/env python3
"""
Hyperautomation Practitioner – Fix This Workflow (CVE Grouping)

Standalone Python version of the Torq workflow:
- Load NVD-style CVE JSON from cve_list.json
- Group CVEs by CVSS v3.1 base severity
- Normalize empty groups
- Compute total_results and quiz_answer
"""

import json
from collections import defaultdict
from pathlib import Path


def load_cves(path: str = "cve_list.json") -> list[dict]:
    data = json.loads(Path(path).read_text())
    # Input structure matches the Torq variable: cve_list.data.vulnerabilities
    return data["data"]["vulnerabilities"]


def get_base_severity(vuln: dict) -> str | None:
    """
    Extract CVSS v3.1 baseSeverity from a single vulnerability entry.
    Returns 'CRITICAL' / 'HIGH' / 'MEDIUM' / 'LOW' or None if missing.
    """
    try:
        metrics = vuln["cve"]["metrics"]["cvssMetricV31"]
        if not metrics:
            return None
        return metrics[0]["cvssData"]["baseSeverity"]
    except (KeyError, IndexError, TypeError):
        return None


def group_cves_by_severity(vulns: list[dict]) -> dict[str, list[str]]:
    groups: dict[str, list[str]] = defaultdict(list)

    for v in vulns:
        severity = get_base_severity(v)
        cve_id = v["cve"]["id"]
        if severity is None:
            # If severity is missing, you could either skip it or put it in a separate bucket.
            continue
        groups[severity.upper()].append(cve_id)

    # Normalize all four buckets so they always exist
    for sev in ("CRITICAL", "HIGH", "MEDIUM", "LOW"):
        groups.setdefault(sev, [])

    return dict(groups)


def main() -> None:
    vulns = load_cves()
    groups = group_cves_by_severity(vulns)

    total_results = sum(len(v) for v in groups.values())
    quiz_answer = "hyperautomation!"

    print("Grouped CVEs by severity:")
    for sev in ("CRITICAL", "HIGH", "MEDIUM", "LOW"):
        print(f"  {sev}: {len(groups[sev])} CVEs")

    print()
    print(f"total_results = {total_results}")
    print(f"quiz_answer   = {quiz_answer}")

    # Optional: enforce the same assertion the playground expects
    if total_results != 50:
        raise SystemExit(f"Expected 50 total CVEs, got {total_results}")


if __name__ == "__main__":
    main()
```

This script is intentionally generic so you can reuse the same pattern in Shuffle, XSOAR, Splunk SOAR, or a home-grown SOAR project: **loop → classify → normalize → aggregate → assert.**

---

### 6. Takeaways

* Broken workflows usually fail on:

  * Incorrect JSON paths.
  * Typos in variable names.
  * Inverted conditions (`IS_EMPTY` vs `IS_NOT_EMPTY`).
  * Missing normalization for empty lists.
* The “eyeglasses” / inspection tooling in Torq is critical: always validate **what the expression actually evaluates to** on a real test run.