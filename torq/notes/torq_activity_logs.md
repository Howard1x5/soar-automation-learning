Activity Logs Module
Overview

This module explains how to use Activity Logs in Torq, including:

    Workflow runs and associated data

    Step executions and associated data

    Filtering and searching logs

    Exporting logs to SIEM or other tools

Core Concepts from Video
1. ActivityVault

    Torq’s central UI section for reviewing workflow activity.

    Includes multiple filter options:

        Timeframe: last 15 minutes, 1 hour, 30 days, or custom via calendar.

        Workflow filter: view logs for specific workflows only.

        Status filter: running, stopped, on hold, queued.

        Source filter: by user or webhook.

    Payload Search:

        Search within the raw event payload.

        Useful for finding logs tied to a specific action (e.g., “make user super admin”).

2. Viewing Raw Events

    Raw Event format:

        Shows newline characters explicitly (\n).

        Displays the entire HTTP body and headers received.

        Best for deep inspection of HTTP-level data (e.g., troubleshooting authentication headers).

    JSON Event format:

        Easier to read.

        Standard JSON formatting — ideal for parsing and automation.

3. Step-Level Activity Logs

    Possible to query activity logs from within a workflow.

    Parameters when querying via workflow step:

        Start time / End time.

        Workflow ID.

        Page size (limit results returned).

        Next page token (pagination).

        Filter by any data track visible in UI.

    Example:

        pageSize = 2 returns 2 most recent logs.

        Each log includes:

            Log ID.

            Workflow name.

            User.

            Web uploader (source).

            Additional event metadata.

4. Exporting Logs

    Torq can send logs to:

        Elasticsearch

        Singularity XDR

        Splunk

    Export schedule can be:

        Every 15 minutes

        Hourly

        Custom intervals

    Templates available for quick setup.

    Use cases:

        Central SIEM correlation.

        Compliance archiving.

        Offline forensic review.

Extra Notes & Cross-Platform Mapping
Torq Term / Feature	Generic SOAR Concept	Shuffle Equivalent / Open-Source Method
ActivityVault	Activity log / workflow run history	Execution logs (View → Logs tab)
Workflow Filter	Search by workflow ID or name	Filter logs by workflow UUID
Status Filter	Run state filter	Filter by status=success/failed/running
Payload Search	Search event payload contents	Full-text search on raw event JSON
Raw Event View	Unprocessed HTTP body/headers	Request/Response inspection
JSON Event View	Parsed structured data	Pretty-printed JSON payload
Export to Splunk/Elasticsearch	SIEM integration	Send via REST API connector
Page Size & Pagination	Log query control	API call with limit & offset params
Practical Usage & Best Practices

    Incident Response:
    Quickly search payloads for unique IOC values (IP, domain, hash) to find related workflow executions.

    Troubleshooting:
    Inspect raw event headers to identify missing/misconfigured auth parameters.

    Forensics:
    Export workflow execution history to SIEM for retention beyond Torq’s storage limits.

    Performance Tuning:
    Use status filtering to quickly spot failed or queued workflows.

External Resources

Since Torq’s training is video-only, the following external resources can help reinforce concepts:

    Shuffle SOAR Logs Documentation:
    https://shuffler.io/docs/logging

    Splunk SOAR Activity Monitoring:
    https://docs.splunk.com/Documentation/SOAR/current/DevelopApps/Logactivity

    Elastic Ingest Pipelines (for logs):
    https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html