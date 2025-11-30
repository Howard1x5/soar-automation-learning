# Torq Automation Expert

## Data Transform Utilities – Advanced Techniques

### Module Overview

This module covers advanced data transformation techniques in Torq, focusing on:

* Using **JQ** (`jqfilter`) to manipulate large JSON objects
* Removing nested loops by replacing them with efficient JQ queries
* De-duplication of arrays
* Extracting deeply nested data
* Using Python (with BeautifulSoup) for HTML scraping
* Building HTML-formatted runbook entries
* Converting JSON to **raw text** suitable for SIEM ingestion (Elasticsearch example)
* Using array/string utilities to prepare outputs for case management or ingestion APIs

This module is critical for scaling complex workflows involving large JSON datasets, multi-level nested objects, and TTP extraction from security platforms like SentinelOne.

---

# 1. JQ for Filtering Large JSON Documents

Torq provides a `jqfilter` utility that lets you:

* Traverse deeply nested JSON
* Remove loops entirely by extracting all target values in a single query
* De-duplicate arrays
* Convert JSON into text or CSV structures
* Reformat JSON for ingestion pipelines

This is a major upgrade compared to building multi-level nested loops.

---

# 2. Use Case: Extracting Tactics, Techniques, and Procedures (TTPs) from SentinelOne Events

### Objective

Collect **all unique MITRE ATT&CK technique URLs** associated with SentinelOne threat events in the last 8 days and build a runbook section in a Torq Case.

### Legacy Approach

Would require **four nested loops**, because the structure is:

```
events[]
   → Indicators[]
      → Tactics[]
         → Techniques[]
            → { ttp_link }
```

This results in:

* Slow execution
* Hard-to-maintain nesting
* Lots of duplicated technique URLs

### JQ Approach

Replace all loops with:

1. One `jqfilter`
2. A Python scraper
3. A final array join to produce case-ready HTML

---

# 3. Removing Nested Loops with JQ

### Steps Performed

1. Add a JQ step beneath the “Get Threats” API call.
2. Copy the raw SentinelOne JSON output.
3. Paste into **jqplay.org** to build the query.
4. Build the filter to traverse the nested structure:

### Example JQ Filter Used

```jq
.[]
  .Indicators[]
  .tacticList[]
  .techniqueList[]
  .tacticTechLink
| unique
```

But because `unique` requires a list, wrap the entire expression:

```jq
[
  .[]
    .indicatorList[]
    .tacticList[]
    .techniqueList[]
    .ttpLink
] | unique
```

### Results

* Output is now a **flat array of unique technique URLs**
* All nested loops eliminated
* Execution becomes instantaneous

---

# 4. Python Scraping to Extract Technique Descriptions

Each technique URL returns an HTML page.
The goal is to pull out only the descriptive text from each page.

### Python Step

Uses **BeautifulSoup** inside Torq’s Python sandbox.

#### Script Behavior

* Loop through each URL (Torq handles isolation per URL)
* Fetch the HTML
* Parse using BeautifulSoup
* Extract description content between known tags
* Clean up unnecessary HTML tags
* Add custom `<br>` tags to format the final runbook content
* Add a title/header containing the MITRE page URL

### Output

Each Python step produces:

* One HTML-formatted `<p>` body string per technique

A collector aggregates these into a list.

---

# 5. Converting the List to a Single HTML Runbook Block

A Torq Case runbook **cannot consume a list** directly.

Solution:
Use **Array → Join** with an HTML delimiter.

### Join Configuration

* Input = List of strings (from Python collector)
* Delimiter = `<br><br>`
* Output = Single large HTML string

This produces:

* Clean, readable runbook entry
* Proper spacing between technique descriptions
* Analyst-friendly format

---

# 6. Final Case Output

The final HTML string is stored in a variable and inserted directly into the Case runbook.

Runbook contains:

* Technique names
* MITRE URLs
* Descriptions
* HTML formatting for readability

This provides immediate context for analysts triaging the SentinelOne threats.

---

# 7. Using JQ to Generate Raw Text for SIEM Ingestion (Elasticsearch Example)

### Objective

Send Torq Activity Logs directly to Elasticsearch using the **Bulk API**, which requires:

* **Raw text**, not JSON
* Specific formatting per line:

  1. Metadata line (specifies `create` and index name)
  2. Event data line (JSON converted to raw text — no commas between entries)

### Required Format

Example bulk ingestion body:

```
{ "create": { "_index": "search-torq-logs" } }
{ event_json_here }
{ "create": { "_index": "search-torq-logs" } }
{ event_json_here }
...
```

### JQ Filter Used

```jq
[
  .output[]
    | "{ \"create\": { \"_index\": \"search-torq-logs\" } }"
    , (.) | tostring
] | .[]
```

Key notes:

* `tostring` converts JSON to one-line string
* Output must end with a **newline**
* Quotes inside strings must be escaped

### Why this matters

Most SIEMs ingest:

* Logstash pipes
* Bulk API uploads
* Raw text streams

And many do **not** accept nested JSON arrays.

---

# 8. Equivalent Implementations in Other SOAR Platforms

### Splunk SOAR (Phantom)

* Use **Code Blocks** (Python) for BeautifulSoup and array processing.
* Use **custom functions** to build reusable TTP extractors.
* Use **Jinja templates** to generate HTML runbook content.
* For raw text → create a block that iterates events and builds the bulk body manually.

### Cortex XSOAR (Demisto)

* Use `jq` inside the **`jq` automation** or a Python automation.
* Use **Transformations** in lists to map/flatten data.
* Use **Templates** or **Markdown** for runbook formatting.
* Bulk ingest formatting can be created in setAndAppend commands.

### Shuffle SOAR

* Workflows must implement loops manually unless using Python.
* BeautifulSoup scraping is performed in a Python app.
* Unique + flattening done either with Python or Shuffle’s minimal array utilities.
* Bulk ingest text output done via Python script.

### Python CLI / Open Source SOAR

* Use `jq` or Python `json` module + `set` for de-duplicating.
* Use BeautifulSoup for scraping.
* Use Python to generate raw text files for ingestion.
* Highly flexible but requires more code.

---

# 9. Summary

This module demonstrates how to:

* Replace deeply nested loops with a single **JQ filter**
* Use **unique** to de-duplicate arrays efficiently
* Use **Python + BeautifulSoup** for HTML scraping
* Collect and join array results into a single HTML string
* Insert formatted data into a Case runbook
* Convert JSON to **raw text** for SIEM ingestion using JQ
* Build massive data automation at scale with performance and legibility