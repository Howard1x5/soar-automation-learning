# Torq Playground A – Challenge 3 - Capture and Present IOCs

## 1. Summary

This workflow ingests an unstructured threat intelligence report, extracts Indicators of Compromise (IOCs), and displays them to the analyst in a structured table for selection. A nested workflow performs IOC extraction, and the parent workflow stores, formats, and presents the results.

## 2. Objectives

1. Accept an unstructured threat report via Torq Interact.
2. Invoke a nested workflow to extract IOCs (domains, IP addresses, URLs, hashes, emails).
3. Handle success and failure paths using a conditional step.
4. Store extracted IOC data in workflow context.
5. Convert IOC data into a markdown table.
6. Display the table to the analyst and capture a selected IOC.
7. Identify the IOC type for downstream enrichment.

---

## 3. Torq Implementation Steps (Condensed)

### A. Input and Setup

1. Open: `Hyperautomation Practitioner → Enrich IOCs`.
2. Add an annotation and store the sample threat report.

### B. Nested Workflow: Extract IOCs

1. Add a `Workflow` operator named `Extract IOCs`.
2. Workflow: `Nested IOC Intake`.
3. Parameter:
   `Raw intelligence report = {{ jsonEscape $.event.raw_intelligence_report }}`
4. Execution options: `Ignore failure = Yes`.
5. Test and confirm IOC extraction in the Execution Log.

### C. Condition: IOCs Found

1. Condition operator:
   `{{ $.extract_iocs.error_code }} Equals 0`
2. False branch = “No IOCs Found” interaction.
   True branch = table-building process.

### D. Storing IOC Data

1. On the TRUE branch → add `Set Variable`:

   * Name: `List of Discovered IOCs`
   * Variable: `iocs`
   * Type: `JSON`
   * Value: `{{ $.extract_iocs.iocs.iocs }}`

### E. Formatting the IOC Table

1. Add `Create Markdown table` utility.
2. Input: `{{ $.list_of_discovered_iocs.iocs }}`

### F. Displaying the Table and Selecting an IOC

1. Open `Display and Select IOC Interaction`.
2. Replace Markdown block with:

```
Here are the IOCs identified in the report.
Select one of them to enrich.
{{ jsonEscape $.create_ioc_markdown_table.tabulate_output }}
```

3. Submit the form and verify `answers.selected_ioc`.

### G. Identify IOC Type

1. Add `Extract IOCs` utility → rename to `Identify IOC Type`.
2. Input:
   `{{ $.display_and_select_ioc_interaction.answers.selected_ioc }}`
3. Confirm the type is correctly identified.

### H. Publish and Test End-to-End

* Upload the final input, generate the table, select an IOC, and verify the identified IOC type.

---

## 4. Python Equivalent (Standalone Script)

This script mirrors the lab’s logic and can be used in other SOAR platforms or for local development.

Save as:

```
ioc_capture_cli.py
```

```python
import re
from textwrap import dedent

IP_REGEX = re.compile(r"\b(?:\d{1,3}\.){3}\d{1,3}\b")
URL_REGEX = re.compile(r"https?://[^\s]+")
EMAIL_REGEX = re.compile(r"\b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b")
SHA256_REGEX = re.compile(r"\b[a-fA-F0-9]{64}\b")
DOMAIN_REGEX = re.compile(r"\b(?:(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,})\b")

def extract_iocs(text: str):
    iocs = []

    def add(matches, type_):
        for m in matches:
            iocs.append({"value": m.group(0), "type": type_})

    add(IP_REGEX.finditer(text), "ip_address")
    add(URL_REGEX.finditer(text), "url")
    add(EMAIL_REGEX.finditer(text), "email")
    add(SHA256_REGEX.finditer(text), "sha256")

    domain_candidates = {m.group(0) for m in DOMAIN_REGEX.finditer(text)}
    existing_values = {ioc["value"] for ioc in iocs}

    for d in domain_candidates:
        if d in existing_values:
            continue
        if d.startswith("http"):
            continue
        if "@" in d:
            continue
        iocs.append({"value": d, "type": "domain"})

    return iocs

def make_markdown_table(iocs):
    if not iocs:
        return "No IOCs found."

    headers = ["#", "IOC", "Type"]
    rows = []

    for idx, ioc in enumerate(iocs, start=1):
        rows.append([str(idx), ioc["value"], ioc["type"]])

    lines = []
    lines.append("| " + " | ".join(headers) + " |")
    lines.append("| " + " | ".join(["---"] * len(headers)) + " |")

    for r in rows:
        lines.append("| " + " | ".join(r) + " |")

    return "\n".join(lines)

def detect_type(ioc_value: str):
    if IP_REGEX.fullmatch(ioc_value):
        return "ip_address"
    if URL_REGEX.fullmatch(ioc_value):
        return "url"
    if EMAIL_REGEX.fullmatch(ioc_value):
        return "email"
    if SHA256_REGEX.fullmatch(ioc_value):
        return "sha256"
    if DOMAIN_REGEX.fullmatch(ioc_value):
        return "domain"
    return "unknown"

def main():
    report = dedent("""
    Subject: IOC intake from recent intelligence report

    Observed domains:
    statsrvv.com
    1312services.ru
    kaspersky-secure.ru

    IPs:
    94.152.193.247
    211.101.234.142

    URLs:
    https://jsonlint.com/

    Hashes:
    226a723ffb4a91d9950a8b266167c5b354ab0db1dc225578494917fe53867ef2

    Email:
    jefferson@airplane.com
    """).strip()

    iocs = extract_iocs(report)

    if not iocs:
        print("No IOCs found.")
        return

    print("Here are the IOCs identified:")
    print(make_markdown_table(iocs))
    print()

    choice = int(input("Select IOC #: ").strip())
    selected = iocs[choice - 1]

    detected = detect_type(selected["value"])

    print("\nSelected IOC:")
    print(f"Value: {selected['value']}")
    print(f"Type (from extraction): {selected['type']}")
    print(f"Type (re-detected): {detected}")

if __name__ == "__main__":
    main()
```

---

## 5. Cross-Platform SOAR Comparison

Below is what this challenge would look like in other SOAR tools.

### Shuffle SOAR

Equivalent components:

* **HTTP trigger** or **Email trigger** → Accept the threat report.
* **Python app step** → Run the IOC extraction code directly.
* **JSON storage** in global or workflow context.
* **Custom UI step** → Use Shuffle Forms (HTML/Markdown block).
* **User Selection** handled via form → `/answer` webhooks.
* **Classification** performed in another Python step.

Key insight:
Shuffle is highly script-driven. The Python module above can run with almost zero modification inside a `Python App` node. Markdown table rendering and selection UI would rely on Shuffle’s form system or a small HTML template.

---

### Splunk SOAR (Phantom)

Equivalent components:

* **Prompt** block → capture threat report.
* **Custom Function** → Python-based IOC extraction.
* **vault_add / container artifacts** → store discovered IOCs.
* **Prompt** block → present options and receive selected IOC.
* **Indicator tagging** → identify IOC type and mark it.

Key insight:
Phantom uses action blocks and prompts. The extraction would be implemented in a **Custom Function**, and the IOC list would be turned into artifacts. Selection is done via a user prompt.

---

### Cortex XSOAR (Demisto)

Equivalent components:

* **Integration command** or **User “Ask” task** → collect threat text.
* **Automation script (Python)** → IOC extraction.
* **Context storage**: `setContext` stores the IOC array.
* **Ask task (multi-select)** → present IOC table to analyst.
* **Use built-in `extractIndicators`** for classification.

Key insight:
XSOAR has native `extractIndicators` support, so IOC handling is simpler. Table formatting uses Markdown directly in the “Ask User” task.

---

### Microsoft Sentinel Automation (Logic Apps)

Equivalent components:

* **HTTP trigger** → receive threat report.
* **Azure Function (Python)** → extract IOCs.
* **Compose** → store IOC JSON.
* **Adaptive Card** → present markdown table to analyst via Teams.
* **Teams Card Submit** → capture selected IOC.
* **Azure Function** → detect IOC type.

Key insight:
Sentinel automation doesn’t natively support IOC extraction; you offload to an Azure Function. Analyst interaction is done through Adaptive Cards, which handle tables and dropdowns well.

---

## 6. Notes and Recommendations

* This challenge is an ideal “cross-SOAR baseline”: user input → IOC extraction → presentation → selection → classification.
* The Python script included here lets you recreate the logic inside any SOAR tool that supports Python apps.
* For your GitHub portfolio, document Torq, Shuffle, Splunk SOAR, and XSOAR equivalents in separate subfolders to demonstrate multi-platform fluency.

---