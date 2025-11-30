# Torq Automation Expert — Data Transform Utilities (Core Tools)

## 1. Summary

This module covers **core data transformation utilities** in Torq and how to use them to:

* Extract fields from plain text and JSON
* Reformat lists and arrays into human-friendly strings
* Build **HTML / ASCII / Markdown tables**
* Send those tables to Slack (including Slack’s own CSV→table rendering)
* Leverage Linux CLI skills (grep, sed, wc, etc.) inside workflows via the data transformer

These tools are the backbone for turning messy inputs (HTML, logs, raw JSON) into structured, readable data for both machines and analysts.

---

## 2. Core Utilities and Concepts

Torq’s **Utilities** tab includes:

* Text extraction:

  * Extract data from plain text (patterns like emails, IPs, CDs, etc.)
  * Extract fields from JSON lists/objects
* String utilities:

  * Join strings
  * Replace / manipulate substrings
* Array utilities:

  * Work with lists (dedupe, map, etc.)
* Table utilities:

  * Create HTML tables
  * Create ASCII / Markdown tables
* Data transformer:

  * Run shell-like commands (grep, sed, wc, etc.) against input text

The overall pattern:

1. Get data (HTTP/API/SSH/log file/etc.)
2. Extract/reshape it
3. Reformat as a table or string
4. Push to Slack/email/TIP/etc.

---

## 3. Example 1 — Extracting Values from a Website and Formatting for Slack

### 3.1 Scenario

* You visit a website listing **CD titles**.
* You want a list of those CDs posted regularly in a Slack channel.
* You want each item wrapped in backticks for readability in Slack, e.g.:

  ```text
  `CD_1` `CD_2` `CD_3`
  ```

### 3.2 Torq Workflow Steps

1. **HTTP Request – Fetch Website**

   * Step: `HTTP Request` (GET)
   * URL: target page with CD list
   * Output: raw HTML as text

2. **Extract CDs with Utility**

   * Step: Torq "extract" utility (pattern-based CD extractor)
   * Input: HTTP response body
   * Output: JSON array of strings, e.g.:

     ```json
     ["CD_1", "CD_2", "CD_3", ...]
     ```

3. **Join List into Single String**

   * Step: `Join strings`
   * Input: `$.extract_cds.result` (the list)
   * Initial delimiter: `,`
   * Result example:

     ```text
     CD_1,CD_2,CD_3
     ```

4. **Customize Delimiter with Backticks + Space**

   * Still in the `join` utility, change delimiter from `,` to:

     ```text
     ` `
     ```

     (backtick, space, backtick)

   * Result:

     ```text
     CD_1` `CD_2` `CD_3
     ```

   * This formats the “middle” items correctly but the string still needs wrapping at both ends.

5. **Add Leading and Trailing Backticks**

   * Use a **set variable / string manipulation** step:

     * New value:

       ```text
       `{{ joined_string }}`
       ```
   * Final output:

     ```text
     `CD_1` `CD_2` `CD_3`
     ```

6. **Send to Slack**

   * Step: Slack message (snippet or message body)
   * Message body: formatted CD string from previous step

### 3.3 Key Takeaways

* Extraction utility saves you from hand-writing complex regex.
* `join` + delimiter tricks are extremely useful for formatting lists.
* A small post-processing step (add leading/trailing delimiters) finishes the formatting.

---

## 4. Example 2 — Firewall Log Parsing with the Data Transformer

### 4.1 Scenario

* You have a **network firewall log sample**:

  * Lines include `access`, `deny`, `TCP`, `UDP`, etc.
* You want to:

  * Count total lines
  * Filter lines with `deny`
  * Highlight patterns like `deny udp` by converting them to uppercase

### 4.2 Torq Workflow Steps

1. **Fetch Log Sample**

   * Step: `HTTP Request` (GET)
   * Output: multiline log text
   * Optionally send to Slack once for a visual sanity check

2. **Data Transformer Utility**

   * Step: `Data transformer`
   * Input: log text
   * You can use familiar Linux CLI commands:

   Examples:

   * Count lines:

     ```bash
     wc -l
     ```

   * Filter for `deny` and count:

     ```bash
     grep deny | wc -l
     ```

   * Filter and highlight `deny udp`:

     ```bash
     grep access \
       | grep deny \
       | sed 's/deny udp/DENY UDP/g'
     ```

3. **Send Transformed Output to Slack**

   * Change Slack step input to the data transformer output
   * Slack shows:

     * The filtered lines (e.g., only `deny` events)
     * The highlighted occurrences (`DENY UDP` in caps)
     * Any line counts you printed

### 4.3 Key Takeaways

* You can reuse **existing sysadmin / Linux skills** directly in Torq.
* Data transformer is ideal for quick:

  * Filtering
  * Highlighting
  * Summarizing logs
* You don’t have to rebuild every transformation with GUI steps; CLI is fair game.

---

## 5. Example 3 — Okta Users to HTML/ASCII/Markdown/CSV Tables

### 5.1 Scenario

* You pull a list of **enabled Okta users** via API.
* Response: JSON list of user objects with many fields.
* Goal:

  * Extract a few key fields: `id`, `status`, `email`
  * Convert this into:

    * HTML table
    * ASCII/Markdown table
    * CSV that Slack renders as a table

### 5.2 Torq Workflow Steps

1. **List Okta Users**

   * Step: Okta integration / HTTP Request to Okta API
   * Output: JSON list of users, e.g.:

     ```json
     [
       {
         "id": "...",
         "status": "ACTIVE",
         "profile": {
           "email": "user@example.com",
           ...
         },
         ...
       },
       ...
     ]
     ```

2. **Extract Fields**

   * Step: `Extract fields`

   * Input: the list of Okta users

   * Path: the array element containing users (e.g. root list)

   * Fields to extract:

     * `id`
     * `profile.email`
     * `status`

   * Initially, you’ll see nested `profile` in the output.

3. **Flatten the JSON**

   * Still in `Extract fields`:

     * Enable `flat response = true`

   * Output is now a flat list where keys are:

     * `id`
     * `profile.email`
     * `status`

   * This flat format is ideal for table conversion.

4. **Create HTML Table**

   * Step: `Create HTML table`
   * Input: the flat list from `Extract fields`
   * Output: HTML table
   * Good for:

     * Emails
     * Portals / wiki pages
     * Any destination that renders HTML tables

5. **Create ASCII / Markdown Table**

   * Step: `Create ASCII table`

   * Input: same flat list

   * Options:

     * Standard ASCII table
     * ASCII as Markdown

   * Example Markdown-style output:

     ```markdown
     | id | profile.email | status |
     | --- | --- | --- |
     | 123 | user@example.com | ACTIVE |
     ```

   * Send this directly to Slack as a code block or snippet.

6. **Convert List to CSV for Slack-rendered Table**

   * Step: `Convert list to CSV`

   * Input: same flat list

   * Output: CSV string

   * Send CSV to Slack:

     * In the Slack step, mark the snippet/file type as CSV (or let Slack detect it).
     * Slack automatically renders the CSV as a native table.

7. **Compare Outputs**

   * HTML table: best for emails/web rendering.
   * ASCII/Markdown table: best for quick text/snippet views.
   * Slack CSV table: Slack itself draws a clean table using the CSV content.

### 5.3 Key Takeaways

* `Extract fields` + `flat response` gives you a table-friendly structure.
* Once you have a flat list, you can render:

  * HTML
  * ASCII
  * Markdown
  * CSV (Slack auto-table)
* Multiple formats from the same source list let you adapt to different consumers (analysts, managers, systems).

---

## 6. Cross-SOAR Parallels

The patterns in this module exist in other SOAR tools too:

* **Cortex XSOAR**

  * Transformers: `Extract`, `ToTable`, `Join`, `Replace`, `Uniq`, etc.
  * Automation scripts (Python) for custom text/JSON transformations.
  * Tables in Slack/email via Markdown or HTML.

* **Splunk SOAR (Phantom)**

  * Custom functions / Python blocks to:

    * Select and reshape artifacts
    * Generate HTML or CSV tables
    * Post results to Slack, ServiceNow, email

* **Shuffle SOAR**

  * JQ / Python apps for flattening JSON and extracting fields.
  * Slack and email apps for sending text, Markdown, or CSV outputs.

Overall pattern is universal:

1. Extract
2. Flatten
3. Transform / format
4. Deliver to humans / downstream systems

---

## 7. Python Equivalents (For Local Automation & Portfolio)

### 7.1 Format a List of Items with Backticks

```python
def format_items_backticks(items):
    # ["CD1", "CD2"] -> "`CD1` `CD2`"
    if not items:
        return ""
    joined = "` `".join(items)
    return f"`{joined}`"

cds = ["CD_1", "CD_2", "CD_3"]
print(format_items_backticks(cds))
# -> `CD_1` `CD_2` `CD_3`
```

### 7.2 Filter Log Lines and Highlight a Pattern

```python
def filter_and_highlight(log_text, include="deny", pattern="deny udp"):
    lines = log_text.splitlines()
    filtered = [line for line in lines if include in line]
    highlighted = [line.replace(pattern, pattern.upper()) for line in filtered]
    return "\n".join(highlighted)

sample_log = """
access allow tcp 10.0.0.1:80 ...
access deny udp 10.0.0.2:53 ...
access deny udp 10.0.0.3:53 ...
"""

print(filter_and_highlight(sample_log))
```

### 7.3 Convert a List of Users to Markdown Table

```python
def users_to_markdown_table(users, columns):
    header_line = "| " + " | ".join(columns) + " |"
    sep_line = "| " + " | ".join(["---"] * len(columns)) + " |"
    body_lines = []

    for u in users:
        row = [str(u.get(col, "")) for col in columns]
        body_lines.append("| " + " | ".join(row) + " |")

    return "\n".join([header_line, sep_line] + body_lines)

users = [
    {"id": "00u1", "email": "alice@example.com", "status": "ACTIVE"},
    {"id": "00u2", "email": "bob@example.com", "status": "SUSPENDED"},
]

print(users_to_markdown_table(users, ["id", "email", "status"]))
```

### 7.4 Convert List to CSV (Slack-friendly)

```python
import csv
import io

def list_to_csv(rows, columns):
    buf = io.StringIO()
    writer = csv.DictWriter(buf, fieldnames=columns)
    writer.writeheader()
    for r in rows:
        writer.writerow({c: r.get(c, "") for c in columns})
    return buf.getvalue()

csv_text = list_to_csv(users, ["id", "email", "status"])
print(csv_text)
```

You could mirror Torq’s tables by sending this CSV to Slack via a simple Python/Slack bot.

---

## 8. Patterns and Best Practices

* Use **extract utilities** first; let Torq do the heavy lifting for pattern matching.
* Flatten JSON early if you know you’re going to build tables.
* Keep formatting steps separate from extraction:

  * Step 1: Extract data
  * Step 2: Reshape (flatten, select fields)
  * Step 3: Format (HTML/Markdown/CSV)
  * Step 4: Deliver (Slack/email/etc.)
* Where text is messy or logs are complex, lean on the **data transformer** and your existing CLI muscle memory.
* Build **reusable nested workflows** for:

  * “Extract + table + Slack” patterns
  * “Log filter + highlight + summary count” patterns