Convert and Encode JSON Data (Torq-first, tool-agnostic)
Core Points

Convert various outputs to JSON so they can be filtered, enriched, or mapped inside a workflow.

Encode JSON into other formats (CSV, tables, files) for handling diverse data sources or sending to downstream systems.

Key Learning
1. Why Convert to JSON?

Many automation steps return text, streams, or encoded results.

SOAR platforms (Torq, Shuffle, Splunk SOAR, XSOAR) work best when input is normalized to JSON.

JSON provides predictable key-value pairs that can be parsed, filtered, enriched, and sent across steps.

2. Example Scenarios
A. PowerShell / Windows CLI Output

SSH or PowerShell steps may output JSON-like text that is actually an escaped string (e.g., backslashes for newlines, quotes escaped).

Step needed: JSON Parser.

Input: escaped string → Output: proper JSON object.

Once parsed, fields like VM Name, Status, and State can be filtered and re-structured.

Common next step: Create Markdown or table to send VM status to Slack or Teams.

Tool-agnostic parallel

Shuffle: Use “JSON Parse” step to normalize escaped strings.

jq (Linux): jq -r '.' will re-parse string into JSON.

PowerShell: Use ConvertFrom-Json after forcing raw CLI output to JSON (... | ConvertTo-Json).

B. Linux CLI Output

Example: Disk usage or system stats from df -h, free -m, etc.

Torq can fetch this via SSH. Output = raw text.

Next step: Convert/encode to JSON using “Convert to JSON” utility.

Output = JSON object with key:value pairs (filesystem, available, used).

Tool-agnostic parallel

Shuffle: Feed command output to “JSON Encode” step.

jq: Pipe with jq -R -s (read raw input → slurp to JSON string array).

Bash: Use awk + jq -n to shape structured JSON.

C. CSV → JSON Conversion

CSV files are common (user exports, threat feeds, firewall logs).

Convert step: From CSV to JSON.

Each column header → key.

Each row → JSON object.

Output = JSON array of objects.

Why useful: JSON array can be filtered (e.g., unique values of “username” or “ioc”).

Tool-agnostic parallel

jq: jq -R -s -f csv2json.jq input.csv (with mapping file).

Python: csv.DictReader then json.dumps.

Shuffle: Built-in CSV to JSON utility.

D. JSON → Other Encodings

Sometimes JSON is too verbose for presentation.

Encode into:

HTML table (for Slack/Teams messages).

ASCII table (for logs or email).

CSV snippet (for attachments).

Example: List of users with ID, Status, and Email → converted into ASCII table → posted to Slack.

Example: Firewall events JSON → converted to CSV → attached to Teams message.

Tool-agnostic parallel

jq: jq -r '.[] | [.id, .status, .email] | @csv'

Python: Pandas df.to_html() or df.to_csv().

Shuffle: Has “to table” / “to CSV” encoding apps.

3. Pitfalls to Avoid

Escaped strings masquerading as JSON (must parse before filtering).

Very large files (Torq has limits; use “File Reference” option for streaming big data).

Keys buried in nested arrays (may require jq filtering).

Practice Lab (Tool-Agnostic)

Lab A – Shuffle or Torq Substitute

Input: Take a CSV file with columns (username,email,role).

Step 1: Convert CSV to JSON.

Step 2: Extract only usernames.

Step 3: Convert usernames list to ASCII table.

Step 4: Send to Slack (or just print).

Lab B – jq / Linux

Input: Command df -h saved to text.

Step 1: Convert text → JSON using a parser.

Step 2: Filter .[] | select(.filesystem | test("^/dev/")).

Step 3: Export to CSV format with @csv.

Lab C – PowerShell

Input: Get-Process | ConvertTo-Json.

Step 1: Parse JSON inside SOAR.

Step 2: Filter by .ProcessName and .CPU.

Step 3: Re-encode into Markdown table → send to Teams.

Mini Cheat Sheet

Convert escaped string → JSON: Use JSON Parser (Torq), ConvertFrom-Json (PowerShell), or jq '.' (Linux).

Convert CSV → JSON: Use “CSV to JSON” step (Torq/Shuffle), jq -R -s -f csv2json.jq, or Python DictReader.

JSON → Table: ASCII or HTML table builders (Torq, Pandas, jq @csv).

JSON → Snippet (Slack/Teams): Better for large payloads (>3,000 chars).

Always use unique() or deduplication steps when parsing threat feeds (to avoid duplicates).