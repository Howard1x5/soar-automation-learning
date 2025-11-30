# Torq Automation Expert — Convert and Encode JSON Data

## 1. Summary

This module covers how to:

* Convert various data sources into **JSON**
* Parse JSON that is embedded as a **string**
* Convert JSON to other formats (e.g. **Java** representation)
* Use **encoding utilities** and **jq** to handle CSV, CLI output, and large JSON documents

Core focus: make data machine-readable so you can filter, transform, and automate decisions in workflows and command-line environments (PowerShell, Linux, etc.).

---

## 2. Why JSON Conversion Matters in Automation

In real SOAR workflows, data comes from:

* PowerShell commands (often emitting JSON as escaped string)
* Linux commands (`ip addr`, `df`, `lsblk`, custom scripts)
* CSV exports (from TIPs, SIEM, inventories, etc.)
* Large JSON documents (MITRE ATT&CK, threat feeds, configs)

To automate with confidence, you need to:

* Normalize everything into **JSON** where possible
* Extract only the relevant fields
* Convert JSON into other formats (e.g. markdown tables, Java code/objects) when needed for downstream systems

This module demonstrates four patterns:

1. PowerShell JSON string → parsed JSON → Markdown table → Slack
2. Linux CLI output → JSON
3. CSV → JSON → jq over file (for large datasets)
4. Big JSON (MITRE ATT&CK) → filtered subset → JSON → Java representation

---

## 3. Example 1 – PowerShell Output: JSON String → JSON → Markdown Table

### 3.1 Scenario

* SSH into a Windows host
* Run a PowerShell command against Hyper-V to list virtual machines
* PowerShell is instructed to output JSON, but the result comes back as a **JSON string** (escaped) rather than a real JSON object

Result characteristics:

* Newlines and quotation marks are escaped
* Torq sees it as a **string** containing a JSON document, not a JSON object

### 3.2 Steps in Torq

1. **SSH Step**

   * Connect to Windows via SSH
   * Execute PowerShell to get VM list as JSON (e.g. `Get-VM | ConvertTo-Json`)
   * Output: JSON-as-string

2. **JSON Parser Step**

   * Input: raw string from SSH step
   * Purpose: remove escape characters and convert into a **proper JSON document**
   * After parsing, you get a JSON array of objects (VM records).

3. **Filter / Map JSON**

   * Build a reduced JSON structure with only fields you care about, for example:

     * VM name
     * State
     * Status

4. **Convert JSON to Markdown Table**

   * Use a template to render a Markdown table, e.g.:

     ```markdown
     | Name      | State  | Status   |
     |-----------|--------|----------|
     | VM1       | Running| Healthy  |
     | VM2       | Off    | Stopped  |
     ```

5. **Send to Slack**

   * Use Slack step with Markdown enabled
   * Result: nicely formatted VM status table in a channel

### 3.3 Lessons

* PowerShell often returns JSON **as text**, not as structured JSON.
* Always run suspicious “JSON” through a **JSON parser** operator when you see escaped characters.
* Once in JSON, you can:

  * Filter fields
  * Build tables
  * Feed downstream decision logic

---

## 4. Example 2 – Linux CLI Output → JSON

### 4.1 Scenario

* Use SSH to run Linux commands like:

  * Network info (e.g. `ip addr`, `ifconfig`)
  * Disk usage (e.g. `df -h`)
* Output is plain text, not structured JSON.

### 4.2 Convert to JSON

Torq offers a **“Convert to JSON”** / **conversion** utility step that can:

* Take CLI output
* Parse it into a JSON representation
* Support other formats like **XML**, **YAML**, etc.

Steps:

1. **SSH Step**

   * Connect to Linux
   * Run commands:

     * Network info
     * Disk space

2. **Convert to JSON Step**

   * Input: output from the SSH command
   * Input type: select relevant mode (e.g. “CLI output” or equivalent pattern)
   * Output: JSON with structured keys/values that represent interfaces, IPs, mountpoints, and sizes.

3. **Downstream Usage**

   * Now you can:

     * Filter by specific interfaces (e.g. `eth0`)
     * Extract IP addresses
     * Check for free disk space thresholds
     * Trigger alerts if conditions are met

### 4.3 Lessons

* Converting CLI output → JSON turns ad-hoc admin commands into **predictable, automatable data sources**.
* Once data is JSON, it can be consumed in loops, conditions, and enrichment steps.

---

## 5. Example 3 – CSV → JSON → jq Over File (Big Data)

### 5.1 Scenario

* Fetch a CSV file over HTTP (e.g. threat feed, vulnerability list, inventory).
* First line: column headers.
* Subsequent lines: records.

Goal:

* Convert CSV → JSON
* Filter for a specific field value (e.g. a certain column)
* Possibly deduplicate values or build unique lists
* Use **jq** on a file for large datasets.

### 5.2 Steps in Torq

1. **Fetch CSV**

   * Use HTTP Request or similar step to download CSV
   * Output: raw CSV text (possibly as file or string)

2. **Convert CSV to JSON**

   * Use encoding utility: **CSV → JSON**
   * Input: CSV content
   * Output: JSON array

     * Each object key = column name
     * Each value = row value

   Example structure:

   ```json
   [
     { "field": "value1", "severity": "high" },
     { "field": "value2", "severity": "medium" }
   ]
   ```

3. **Treat JSON as File (for Large Data)**

   * Convert JSON output to a file-type variable
   * This is useful for large data sets (many rows).

4. **Use jq with File Input**

   * Call jq using the “file” function so it reads from the JSON file instead of inline text.
   * Example: filter on a specific field and build a **unique list**.

5. **Result**

   * A filtered JSON list with only the values you care about (e.g. all distinct values in a certain column).

### 5.3 Lessons

* CSV → JSON is a critical step for integrating with legacy systems.
* Treating large JSON as a **file** is more efficient and avoids payload-size issues.
* jq + JSON file is powerful for:

  * De-duplication
  * Field-level filtering
  * Complex transformations without writing code.

---

## 6. Example 4 – Large JSON → Filter → JSON → Java

### 6.1 Scenario

* You retrieve a large JSON document (e.g. MITRE ATT&CK data, ~30 MB).
* Only a subset is relevant (for example, specific technique entries or items matching a type or symbol like “T9xx” for 90s techniques).
* After filtering, you want to convert the JSON into a **Java representation** for embedding in code or integration with Java-based systems.

### 6.2 Steps in Torq

1. **Fetch Large JSON**

   * HTTP Request to MITRE or other source
   * Output: JSON file (large dataset)

2. **First jq Filter**

   * Use jq to:

     * Perform an exact match on a specific `type` or `category` field
     * Reduce huge dataset to a smaller list of relevant objects

3. **Second jq Filter**

   * Further narrow down results (e.g. items whose ID matches a pattern related to the 90s symbol or specific technique ID pattern).

4. **Result**

   * Smaller JSON document containing only relevant subset.

5. **Convert JSON to Java**

   * Use encoding utility: **JSON → Java**
   * Input: filtered JSON subset
   * Output: Java-style representation (e.g. Java objects, class/entity structure, or static definitions depending on the tool’s implementation).

6. **Use Cases**

   * Static Java code for:

     * Embedded threat intel mapping
     * Prebuilt enums/objects for ATT&CK techniques
   * Auto-generated Java configuration objects.

### 6.3 Lessons

* Always filter **before** converting to secondary formats.
* jq is excellent for carving out only the part of a large JSON you actually care about.
* JSON → Java conversion allows workflow output to directly feed developer artifacts.

---

## 7. Cross-SOAR & CLI Use Cases

### General Patterns Across SOAR Tools

All major SOAR platforms rely heavily on JSON as the working format:

* **Torq**

  * JSON parser, Convert to JSON, CSV → JSON, JSON → Java, jq utilities.
* **Cortex XSOAR**

  * `json`, `Set`, `Filter`, and transformers to parse command output into context (JSON-like).
  * Built-in scripts to parse CSV, XML, HTML, etc.
* **Shuffle SOAR**

  * Many apps return JSON; non-JSON outputs are typically transformed via Python or regex steps before further use.
* **Splunk SOAR (Phantom)**

  * Custom functions and actions frequently parse raw CLI or script output into JSON using Python inside playbooks.

### CLI & Scripting Angle

On the command line:

* `jq` for JSON operations
* `csvkit` or Python’s `csv` library for CSV → JSON
* PowerShell `ConvertTo-Json` / `ConvertFrom-Json` for Windows
* Python `json` module for structured manipulation

All of these patterns map naturally into SOAR workflows.

---

## 8. Python Equivalents (For Portfolio Use)

### 8.1 Parse JSON String from PowerShell

```python
import json

def parse_powershell_json(raw_string: str):
    # raw_string looks like: "{\"Name\":\"VM1\",\"State\":\"Running\"...}"
    return json.loads(raw_string)

raw = r"{\"Name\":\"VM1\",\"State\":\"Running\",\"Status\":\"Healthy\"}"
parsed = parse_powershell_json(raw)
print(parsed["Name"], parsed["State"])
```

### 8.2 Convert CSV → JSON

```python
import csv
import json
from io import StringIO

def csv_to_json(csv_text: str):
    f = StringIO(csv_text)
    reader = csv.DictReader(f)
    return list(reader)

csv_data = """name,severity
ioc1,high
ioc2,medium
"""
json_rows = csv_to_json(csv_data)
print(json.dumps(json_rows, indent=2))
```

### 8.3 Filter Large JSON (MITRE-style) with Python

```python
import json

def filter_attck(data, type_filter, id_prefix=None):
    results = []
    for item in data:
        if item.get("type") != type_filter:
            continue
        if id_prefix and not item.get("id", "").startswith(id_prefix):
            continue
        results.append(item)
    return results

# Example usage:
# big_data = json.load(open("enterprise-attack.json"))
# subset = filter_attck(big_data["objects"], "attack-pattern", "attack-pattern--90")
# print(len(subset))
```

### 8.4 JSON → Java (Conceptual Shape)

While actual Java code generation depends on desired format, here is a simplistic example:

```python
def json_to_java_constants(json_list, class_name="MitreTechniques"):
    lines = [f"public class {class_name} {{"]
    for item in json_list:
        tech_id = item.get("id", "UNKNOWN").replace("-", "_").upper()
        name = item.get("name", "").replace('"', '\\"')
        lines.append(f'    public static final String {tech_id} = "{name}";')
    lines.append("}")
    return "\n".join(lines)

# subset would be the filtered JSON list from above
# java_code = json_to_java_constants(subset)
# print(java_code)
```

This mirrors Torq’s “JSON → Java” concept at a high level.

---

## 9. Key Patterns and Best Practices

1. **Suspect escaped JSON?**

   * Run it through a JSON parser to convert from “string-of-JSON” to real JSON.

2. **CLI output (Linux/Windows) → JSON**

   * Convert to JSON for reliable field access. Treat CLI as a data source, not just text.

3. **CSV feeds**

   * Convert CSV → JSON early.
   * For large files, treat the JSON as a **file**, and use jq/file-based operations.

4. **Large JSON documents**

   * Always filter to smaller subsets with jq (or equivalent) before secondary conversions or integrations.

5. **Encoding out of JSON**

   * JSON → Markdown (tables), JSON → Java, etc., are powerful when you need to hand results off to humans or developers.

6. **Reusability**

   * Wrap conversion patterns into nested workflows / functions so you don’t rewrite parsing logic in every new automation.