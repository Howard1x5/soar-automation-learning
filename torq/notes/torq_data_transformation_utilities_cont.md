Utilities Continued

This module explains additional utilities that you can use when building workflows. Focus areas include:

    JSON manipulation with jqCommand

    Filtering, mapping, and deduplication of data

    Organizing nested JSON into usable tabular outputs

    Use cases: malware intelligence extraction, SentinelOne events, and log ingestion into Elasticsearch

🔑 Core Concepts

    jqCommand Utility

        jq is a lightweight JSON processor.

        Enables filtering, transforming, and restructuring JSON.

        Especially useful for:

            Extracting only needed fields (IDs, names, URLs).

            Filtering based on conditions (contains, select).

            Removing duplicates (unique).

            Converting nested structures into flat lists.

    Workflow Context

        Data often comes in nested JSON objects from APIs (Threat intel feeds, SentinelOne, Elastic).

        Instead of manually looping, jq allows precise one-line queries.

        jq can output lists, filtered subsets, or stringified JSON ready for ingestion.

    Unique & Map Functions

        map extracts values into a list.

        unique removes duplicate entries, leaving a distinct set.

        Example: Extracting unique malware types from a dataset.

    Nested Loop Parsing

        Events → Indicators → Tactics → Techniques

        jq simplifies nested iterations into direct queries, avoiding heavy workflow loops.

🧪 Example 1: Extract Malware Types

    Fetch MITRE ATT&CK dataset.

    Use jq to filter by type == "malware".

    Extract fields: id, name, url.

    Deduplicate with unique.

    Convert output into ASCII table or send to Slack.

.map(.type) | unique

🧪 Example 2: SentinelOne TTPs to Case Management

    Fetch events via API.

    Traverse nested JSON: Events → Indicators → Tactics → Techniques.

    Extract technique URLs.

    Deduplicate with unique.

    Fetch description text from MITRE URLs with HTTP + Python (BeautifulSoup).

    Collect all descriptions, join into HTML string, and attach to case notes.

Why jq here?

    jq handles nested traversal and avoids 4+ workflow loops.

    Cleaner, smaller, and faster.

🧪 Example 3: Export Logs to Elasticsearch

    Goal: Upload platform activity logs into an Elastic index.

    Elastic bulk upload format requires:

        Metadata line (index + action).

        JSON log entry as string.

    jq is used to:

        Transform raw logs into compliant format.

        Escape JSON into strings.

        Prepend with index metadata.

{ "index": { "_index": "search-logs" } }
{ "event": .event }

    Final output is a newline-separated string, ready for Elastic ingestion.

📘 Cross-Platform Notes
Feature / Action	Torque (jqCommand)	Shuffle	Splunk SOAR	Cortex XSOAR
Extract fields from JSON	jqCommand	Python jq lib or custom code step	Built-in JSON.parse + Filters	Transformer scripts
Deduplication	unique	Custom script step	dedup SPL	Custom transformer
Table creation	ASCII/HTML utilities	Slack / Email apps	SPL table cmd	Table transformers
Log ingestion	Elastic HTTP + jq	Elastic App or REST API step	Splunk forwarder	Integration w/ Elastic
🧑‍💻 Practice Lab – Utilities & jq

Goal: Rebuild Torque jqCommand examples in Shuffle or locally.

    Setup

        In Shuffle, create a workflow with HTTP step to fetch a public JSON API (example: MITRE ATT&CK or GitHub API).

    Exercise 1: Extract Malware Types

        Use Python app or jq wrapper to filter malware objects.

        Create ASCII table and send to Slack.

    Exercise 2: Event Drill-down

        Ingest nested JSON (multi-level).

        Traverse into techniques (simulate SentinelOne).

        Extract URLs and deduplicate.

        Send results to Slack.

    Exercise 3: Log Upload Simulation

        Take sample logs (JSON).

        Transform into Elastic bulk format.

        Output as raw string.

        (Optional) Push into Elastic or just review the format.

📚 Helpful References

    jq manual:
    https://stedolan.github.io/jq/manual/

    Elastic bulk API:
    https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html

    MITRE ATT&CK data sources:
    https://attack.mitre.org/resources/working-with-attack/

    Shuffle jq plugin (community reference):
    https://shuffler.io