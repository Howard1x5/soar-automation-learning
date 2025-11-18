# Torq Academy – Playground 3C - Summarize and Save the Report

## 1. Objective

Build an end-to-end enrichment workflow that:

1. Uses an AI task to summarize enrichment results into clean Markdown.
2. Shows the summary to the analyst in an interaction form.
3. Lets the analyst decide whether to save the report.
4. If saved, writes it into a Torq workspace variable using the Torq API integration, with a unique report ID.

This turns raw enrichment data into a reusable SOC knowledge artifact.

---

## 2. High-Level Workflow Logic

True branch from “If IOCs Found”:

1. Sort / process IOCs (built in previous challenges).
2. AI Task -> generate Markdown report from `enrichment_results`.
3. Interaction form -> show Markdown report, ask “Save or close?”
4. IF -> check whether the user chose to save.

   * FALSE -> Exit.
   * TRUE -> Generate random report ID.
5. Torq integration -> create a workspace variable:

   * Name: includes report ID + IOC.
   * Value: full AI Markdown report.
6. Final interaction -> tell user “Report saved as workspace variable” and Exit.

---

## 3. Torq Implementation Details

### 3.1 AI Task – Create Enrichment Report

On the TRUE branch of `If IOCs Found`, under the IOC sort switch:

* Add an **AI Task** operator.

* Step name: `Create Enrichment Report`.

* Prompt (example):

  ```text
  Summarize this enrichment report in markdown. Start with a short high level summary. 
  Return only the markdown text.

  {{ $.enrichment_results.enrichment_results }}
  ```

* Run the step once and verify in the Execution Log that:

  * Output is Markdown.
  * It starts with a high-level summary.
  * No extra system text comes back (only the report).

This step is essentially: “LLM, turn noisy enrichment JSON into a readable report.”

---

### 3.2 Display the Report to the Analyst

Use the existing interaction step:

* Enable `Display Enrichment Report Interaction`.
* Edit the **Markdown** element:

  ```text
  {{ jsonEscape $.create_enrichment_report.response }}
  ```

This takes the AI response and safely renders it as Markdown in the interaction form.

Test run:

1. Submit the original threat report text.
2. Pick an IOC from the table.
3. Confirm the summary report shows up.
4. Click “Save the report as a workspace variable and exit the form.”

---

### 3.3 IF – Should We Save the Report?

Below `Display Enrichment Report Interaction`:

* Add an **IF** operator: `If Save Report`.
* Condition: driven by the interaction button selection (Save vs Close).
* FALSE branch:

  * Add an **Exit** operator and leave defaults.

This gives you the analyst decision point: “Is this report worth persisting?”

---

### 3.4 Generate a Unique Report ID

TRUE branch of `If Save Report`:

* Add **Generate Random Number** utility.
* Step name: `Generate Random Report ID`.
* Min: `10000`
* Max: `99999`

This acts as a simple unique report key (human-readable, easy to reference).

---

### 3.5 Torq API Integration – Create Workspace Variable

First, create a Torq API key and integration:

1. Profile icon → **API Keys**.
2. Create API Key → copy Client ID and Client Secret.
3. Back in the workflow, add a **Torq Integration** step:

   * Action: `Create a workspace variable`.
   * Create a new integration instance `torq_playground` with the Client ID / Secret.

Step config:

* Step name: `Save the report as a workspace variable`.

* Integration: `torq_playground`.

* Variable pretty name:

  ```text
  Enrichment Report #{{ $.generate_random_report_id.result }} for {{ $.identify_ioc_type.results.0.value }}
  ```

* Value string:

  ```text
  {{ $.create_enrichment_report.response }}
  ```

Result: each saved report becomes a workspace variable like:

* Name: `Enrichment Report #54321 for 1.2.3.4`
* Value: full Markdown summary from the AI task.

These workspace variables can later be referenced by other workflows (e.g., follow-up investigations, reports, or ticketing automations).

---

### 3.6 Final Interaction – Confirm Save

Under `Save the report as a workspace variable`:

* Add a new Interaction step (`Report Saved`).

* Title:

  ```text
  Torq Automated IOC Enrichment
  ```

* Markdown:

  ```markdown
  **Enrichment Report #{{ jsonEscape $.generate_random_report_id.result }} for {{ jsonEscape $.identify_ioc_type.results.0.value }}** has been saved as a workspace variable.
  ```

* Remove the short text field.

* Change the button label from `Submit` to `Close`.

Then add an **Exit** operator under `Report Saved`.

---

### 3.7 End-to-End Test

1. Publish the workflow.
2. Submit the original annotation text.
3. Copy one IOC, submit to enrichment.
4. Review the AI-generated summary.
5. Click “Save the report as a workspace variable.”
6. Go to Workflow / Workspace Variables and confirm a new variable exists with:

   * Pretty name matching the report ID and IOC.
   * Value containing the Markdown report.

This is your proof that the workflow now creates durable, shareable AI enrichment reports.

---

## 4. Real-World SOC Use Cases

Patterns this implements:

* Turn verbose enrichment output into a one-page, human-readable report.
* Let the analyst decide which cases deserve persistence (vs. throwaway routine alerts).
* Store reports in a central “workspace” so:

  * Other workflows can fetch them by name or ID.
  * Analysts can re-use prior enrichment for recurring IOCs or recurring customers.
* Provide a simple reference handle (`Report #NNNNN`) that can be:

  * Added to tickets.
  * Shared in chat.
  * Referenced in post-incident reviews.

This is exactly the kind of “polished analyst experience” hiring managers want to see in a SOAR portfolio.

---

## 5. How to Recreate This in Other SOAR Platforms

### 5.1 Shuffle SOAR

Conceptual node layout:

1. **Enrichment nodes**

   * Existing Shuffle apps for VirusTotal, URL/IP/file hash lookups, etc.
   * Aggregate all enrichment JSON into a single payload.

2. **LLM / HTTP Node**

   * Use an HTTP app or OpenAI app to send:

     * Prompt: “Summarize these enrichment results in Markdown…”
     * Context: JSON of enrichment results.
   * Save response as `summary_markdown`.

3. **Form / UI Node**

   * Shuffle interaction node (WebForm / Manual Task).
   * Show `summary_markdown` rendered as Markdown.
   * Buttons: `Save` / `Close`.
   * Output variable: `save_report` = true/false.

4. **Condition Node**

   * If `save_report` == false → end workflow.
   * If true → continue.

5. **Random ID Node (Python app or built-in)**

   * Small Python function:

     ```python
     import random
     report_id = random.randint(10000, 99999)
     ```

6. **Persistence Layer**
   Options:

   * Use Shuffle “Storage” / “Data” app to write to a key-value store (e.g., key = `EnrichmentReport-<id>`).
   * Write to an external DB (PostgreSQL, MongoDB, etc.).
   * Create a ticket in Jira/ServiceNow with the ID in the title and summary Markdown as the description.

7. **Final Manual Task**

   * Simple confirmation “Report saved as EnrichmentReport-<id>”.

This mirrors Torq’s pattern: AI summarize → human review → optional persistence.

---

### 5.2 Splunk SOAR (Phantom)

High-level playbook design:

1. **Enrichment phase**

   * Existing apps: URL reputation, IP reputation, file detonation, etc.
   * Use a custom function to bundle enrichment artifacts into a single JSON.

2. **LLM call**

   * HTTP app to call OpenAI / internal LLM API:

     * Prompt + enrichment JSON.
   * Store output in a container note field or custom field `enrichment_summary_markdown`.

3. **Prompt the analyst**

   * Use `prompt` block to show the Markdown summary (Phantom UI supports formatted prompts).
   * Prompt question: “Save this report?”

     * Responses: “Yes, save” / “No”.

4. **Decision block**

   * Branch based on the prompt answer.

5. **Generate ID**

   * Custom function (Python):

     ```python
     import random
     def generate_report_id():
         return random.randint(10000, 99999)
     ```

6. **Persist report**
   Options in Phantom:

   * Create a new container (type: “Enrichment Report”) with:

     * Name: `Enrichment Report #<id> for <IOC>`.
     * Description: Markdown summary.
   * Attach a file to the current container containing the Markdown.
   * Write to an external database via the HTTP app.

7. **Final note / prompt**

   * Add a note or prompt to confirm the report ID and storage location.

---

### 5.3 Cortex XSOAR (Demisto)

Playbook outline:

1. **Enrichment tasks**

   * Standard IOC enrichment: `!url`, `!ip`, `!file`, etc.
   * Collect results into context key `Enrichment.Results`.

2. **LLM summarization task**

   * Use OpenAI / Generic LLM integration:

     * Prompt: summarize enrichment results to Markdown.
     * Input: `Enrichment.Results`.
   * Save response to `Enrichment.ReportMarkdown`.

3. **User decision task**

   * “Ask” task (manual): show `Enrichment.ReportMarkdown`.
   * Question: “Save this report as a reusable asset?”
   * Answers: Yes/No.

4. **Conditional task**

   * If No → playbook end.
   * If Yes → continue.

5. **Generate ID**

   * Automation or script:

     ```python
     import random

     def main():
         report_id = random.randint(10000, 99999)
         demisto.setContext('Enrichment.ReportID', report_id)

     if __name__ in ('__builtin__', 'builtins'):
         main()
     ```

6. **Persist as a List or Incident/Indicator**
   Options:

   * Create a new Incident type `Enrichment-Report` with fields:

     * `report_id`, `ioc_value`, `markdown_report`.
   * Or append to an XSOAR List, e.g. `EnrichmentReports`, storing JSON rows:

     * `{ "id": 12345, "ioc": "...", "markdown": "..." }`.

7. **Final task**

   * Note / automation that posts: “Enrichment Report #<id> for <IOC> saved to <destination>”.

---

### 5.4 Generic Pattern / Takeaway

Across Torq, Shuffle, Splunk SOAR, XSOAR, the pattern is identical:

1. Aggregate enrichment results.
2. Use an LLM to compress them into a readable Markdown summary.
3. Present that to an analyst.
4. Let the analyst choose whether the report is worth persisting.
5. If yes:

   * Generate a unique, human-friendly ID.
   * Save the report to a durable store (workspace variable, list, DB, ticket).

That’s the pattern you’re demonstrating on GitHub.

---

## 6. Python Reference Implementation (Tool-Agnostic)

Below is a standalone Python example that implements the same idea you just built in Torq. You can drop this into a `python` subfolder as `summarize_and_save_report.py` and reference it from the README.

```python
import json
import os
import random
from datetime import datetime
from typing import Any, Dict

WORKSPACE_FILE = "workspace_enrichment_reports.json"


def call_llm_api(prompt: str) -> str:
    """
    Placeholder for an LLM call.
    In a real environment, wire this to OpenAI, a local LLM, or another API.
    """
    # TODO: Replace this with actual LLM logic.
    # For GitHub demo purposes we just echo a fake summary.
    return f"# Enrichment Summary\n\n(Example LLM summary)\n\nPrompt length: {len(prompt)} characters."


def load_workspace() -> Dict[str, Any]:
    if not os.path.exists(WORKSPACE_FILE):
        return {}
    with open(WORKSPACE_FILE, "r") as f:
        return json.load(f)


def save_workspace(data: Dict[str, Any]) -> None:
    with open(WORKSPACE_FILE, "w") as f:
        json.dump(data, f, indent=2)


def summarize_enrichment(enrichment_results: Dict[str, Any]) -> str:
    prompt = (
        "Summarize this enrichment report in markdown. "
        "Start with a high level summary and return only the markdown text.\n\n"
        f"{json.dumps(enrichment_results, indent=2)}"
    )
    return call_llm_api(prompt)


def generate_report_id() -> int:
    return random.randint(10000, 99999)


def save_report(ioc_value: str, report_markdown: str) -> str:
    workspace = load_workspace()

    report_id = generate_report_id()
    key = f"Enrichment Report #{report_id} for {ioc_value}"

    workspace[key] = {
        "id": report_id,
        "ioc": ioc_value,
        "markdown": report_markdown,
        "created_at": datetime.utcnow().isoformat() + "Z",
    }

    save_workspace(workspace)
    return key


def main():
    # Example fake enrichment results:
    enrichment_results = {
        "ioc": "1.2.3.4",
        "sources": ["VirusTotal", "InternalThreatIntel"],
        "findings": {
            "reputation": "malicious",
            "first_seen": "2025-10-01",
            "last_seen": "2025-11-17",
            "related_domains": ["malicious-example.com"],
        },
    }

    ioc_value = enrichment_results.get("ioc", "unknown-ioc")

    # 1) Summarize via LLM
    report_markdown = summarize_enrichment(enrichment_results)
    print("=== Enrichment Report Preview ===")
    print(report_markdown)
    print("=================================")

    # 2) Ask analyst if they want to save it
    choice = input("Save this report as a workspace variable? [y/N]: ").strip().lower()
    if choice != "y":
        print("Report not saved.")
        return

    # 3) Save to workspace storage
    key = save_report(ioc_value, report_markdown)
    print(f"Report saved under key: {key}")
    print(f"Workspace file: {os.path.abspath(WORKSPACE_FILE)}")


if __name__ == "__main__":
    main()
```

### How this Python script maps to the Torq workflow

* `summarize_enrichment()`
  = Torq **AI Task** step (`Create Enrichment Report`).

* `input("Save this report...?")`
  = Torq **Display Enrichment Report Interaction** + `If Save Report`.

* `generate_report_id()`
  = **Generate Random Report ID** utility.

* `save_report()` writing into `workspace_enrichment_reports.json`
  = Torq **Create a workspace variable** step.