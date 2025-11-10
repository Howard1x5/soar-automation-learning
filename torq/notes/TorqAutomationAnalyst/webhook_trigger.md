Torq — Webhook Trigger (Automation Analyst Certification)
1) Overview

    Purpose: Webhook triggers allow Torq workflows to run in response to events sent from third-party systems.

    Common Sources:

        XDR (Extended Detection and Response)

        SIEM

        IAM (Identity and Access Management)

        Ticketing or alerting systems

    Why It Matters: Webhooks are the bridge between external platforms and your automation. They allow near real-time reaction to events from other tools.

2) Creating a Webhook in Torq

    Navigate to the Integrations section and find Webhook.

    Click Add → name the webhook (e.g., first_webhook).

    Save — Torq will generate a unique webhook URL.

    You can test the webhook directly in your browser by pasting the URL.

        Without parameters, the event will be empty.

        Logs will show incoming connections in the Webhook Event Log.

Pro Tip: Store this webhook URL securely. Treat it like a credential.
3) Sending Data to the Webhook
A. URL Parameters (GET Requests)

    Append parameters directly to the webhook URL:

    https://<torq-instance>/webhook/abc123?color=red&severity=high&source=pgw1

    Parameters appear in the event payload under key/value format.

    Useful for quick manual testing.

    Limitations: URL length restrictions and less secure for sensitive data.

B. JSON Payload (POST Requests)

    Send JSON in the body of a POST request (e.g., from Postman).

    Set Content-Type: application/json.

    Example body:

    {
      "color": "red",
      "severity": "high",
      "source": "pgw1"
    }

    Benefit: Cleaner structure, can send nested objects, avoids URL length limits.

4) Authentication Headers

    Add security to your webhook by requiring a header with a secret token.

    Steps:

        Open your webhook configuration in Torq.

        Add a header, e.g.:

            Key: XTorqAuth

            Value: Random secret string.

        In your HTTP client (Postman, curl, etc.), add this header to all requests.

        Torq will only process events where the authentication header matches.

    Best Practice: Rotate secrets periodically and avoid reusing across workflows.

5) Sending Files via Webhook

    Webhooks can accept binary files (e.g., logs, samples).

    Steps:

        In Postman (or curl), change request type to binary.

        Select the file to upload.

        Send to webhook URL with authentication header (if enabled).

    Torq Configuration:

        Enable the trigger step to accept files.

        Reference uploaded files in the workflow via event metadata:

            File size.

            Hash values (MD5/SHA1/SHA256).

            Download URL.

    Security Tip: Treat inbound files as potentially malicious; use sandbox analysis before further processing.

6) Publishing the Workflow

    Creating a webhook does not start automation until:

        The workflow is published.

        The webhook trigger is linked as the start step.

    Always confirm:

        Correct trigger is selected.

        All parameters and authentication are configured.

        Testing works in both dev and prod environments.

7) Trigger Conditions

    By default, a webhook will trigger the workflow on any incoming event.

    You can filter execution based on conditions in the event payload.

    Example:

        Condition: Run only when event.actor.id == "12345".

        In Torq:

            Open the trigger step.

            Add a condition referencing the JSON path: event.actor.id.

            Compare to expected value.

    Best Practice:

        Keep conditions as early as possible to avoid unnecessary workflow execution.

        Combine with authentication for defense-in-depth.

8) Example Use Case Flow

    Webhook Source: SIEM sends alert to Torq webhook.

    Authentication: Request includes XTorqAuth header with secret.

    Payload: JSON with alert metadata (e.g., source IP, detection rule).

    Trigger Condition: Run only if severity = "high".

    Workflow Actions:

        Enrich IP from threat intel feeds.

        Notify SOC channel in Slack.

        Open incident ticket in case management system.

    Output: Store results in a case file and log to SIEM for recordkeeping.

9) Best Practices

    Use HTTPS only — Torq webhook URLs are TLS-protected by default.

    Authenticate every webhook — even if only used internally.

    Use POST + JSON for structured, secure data.

    Keep payloads small where possible; large file uploads can slow processing.

    Log and monitor webhook usage — unusual spikes may indicate abuse attempts.

    Document the expected payload structure for each webhook.

10) Additional Learning Resources (Full FQDNs)

    https://www.youtube.com/watch?v=sdCIfJEuA_0 — Torq Webhook Trigger Official Walkthrough

    https://www.youtube.com/watch?v=dBxlbQVDQf8 — General Webhook Security Best Practices

    https://www.youtube.com/watch?v=7t2rRCRgnWc — Sending Files via HTTP Webhooks

    https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST — HTTP POST Method Overview