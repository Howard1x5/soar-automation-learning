# Triggering Workflows in Torq Using Webhooks

## 1. Summary

This module explains how to use **webhooks** to trigger Torq workflows.
Webhooks are the primary mechanism for receiving **external events** from:

* XDR platforms
* SIEM systems
* IAM tools
* Any third-party service capable of sending HTTP requests

This is the real-world method used for alert ingestion, automation triggers, and system-to-system communication.

Core concepts covered:

* Creating a webhook integration
* Sending URL parameters and JSON bodies
* Authentication headers
* Receiving binary files
* Enabling raw HTTP mode
* Using webhooks as workflow triggers
* Applying trigger conditions (event filtering)

---

## 2. What a Webhook Is in Torq

A webhook is a **unique URL endpoint** that Torq exposes for receiving data.
Any system can POST or GET to this endpoint to start a workflow.

Properties:

* Accepts URL parameters, JSON bodies, XML, plaintext
* Can accept binary files if configured
* Supports authentication headers
* Can be reused across workflows
* Must be **published** before it can trigger workflows

Use cases:

* XDR alert ingestion
* IAM events
* Cloud logs
* Custom monitoring scripts
* CI/CD notifications
* Anything that can send an HTTP request

---

## 3. Creating a Webhook in Torq

1. Go to **Integrations → Add Integration**
2. Search for **Webhook**
3. Click **Add**
4. Name your webhook (e.g., *First Webhook*)
5. Save

Torq immediately provides a usable webhook URL.

Example:

```
https://api.torq.io/webhook/12345-abcd-9999
```

You can open it in a browser to test connectivity (Torq will log the request, but the event will be empty).

---

## 4. Using a Webhook as a Workflow Trigger

To attach the webhook to a workflow:

1. Open your workflow
2. Click the trigger block
3. Select your webhook from the list
4. If the webhook does not appear:

   * Refresh the browser
   * Reopen the workflow designer

Webhooks must be **published** before they can automatically invoke workflows.

---

## 5. Event Formats – URL Parameters vs JSON Bodies

### 5.1 URL Parameter Example

Calling:

```
https://<webhook-url>?color=red&severity=high&source=gateway1
```

Results in an event object:

```json
{
  "color": "red",
  "severity": "high",
  "source": "gateway1"
}
```

In Torq expressions, reference these as:

```
{{ $.event.color }}
{{ $.event.severity }}
{{ $.event.source }}
```

---

### 5.2 JSON Body (POST)

Using Postman or curl:

* Method: **POST**
* Body: **raw JSON**
* Content-Type: `application/json`

Example JSON:

```json
{
  "color": "red",
  "severity": "high",
  "source": "gateway1"
}
```

This produces a differently structured event—Torq unwraps JSON bodies cleanly, unlike query parameters.

---

## 6. Capturing the Event into a Variable

Often you want to store the entire event as a JSON object for processing.
Create a new variable:

* Name: `webhook_event_json`
* Type: JSON
* Value:

```
{{ $.event }}
```

This makes the entire payload accessible to downstream steps.

---

## 7. Publishing the Webhook (Required)

A webhook must be **published** before it will run workflows automatically.

* Unpublished = receives events but does NOT trigger workflows
* Published = evaluates conditions and triggers workflow on match

---

## 8. Securing Webhooks with Authentication Headers

Before publishing, add security:

1. Go to **Integrations → Your Webhook**
2. Add a custom header (example: `X-Torq-Auth`)
3. Add a secret token string
4. Save

Client systems must then send that header:

**Header:**

```
X-Torq-Auth: <your-secret-token>
```

If this header is missing or incorrect, the event is rejected.

This is the correct way to prevent unauthorized systems from triggering your workflows.

---

## 9. Handling Binary File Uploads

By default, Torq only accepts recognizable payloads:

* JSON
* XML
* Text

To accept arbitrary files:

1. Open the webhook configuration
2. Enable:

```
Accept raw HTTP request
```

Now Torq will store binary uploads and expose them as files represented as Base64 strings with metadata.

### 9.1 Converting the Received Binary to a Torq File

Create a new variable:

* Name: `file`
* Value: `{{ $.event.file }}`
* Enable **Output as File**
* Mark as **Shareable** if other tools need direct access

Torq stores this file internally and exposes metadata:

* Size
* Hashes
* Internal public URL

Useful for:

* Malware samples
* Log archives
* PCAPs
* Attachments
* Vendor-specific exports

---

## 10. Using Vendor-Specific Webhooks (Example: Okta)

Torq supports vendor-specific webhook integrations.
Example: Okta → Torq workflow trigger.

Steps:

1. Create a new webhook (name it `OktaWebhooks`)
2. Attach it as the workflow trigger
3. View incoming Okta events in Event Log
4. Build logic based on event fields (e.g., actor.id, eventType, IP, app)

This pattern is repeatable with:

* Okta
* Azure AD
* CrowdStrike
* SentinelOne
* AWS
* Custom alerting systems

---

## 11. Trigger Conditions (Filtering)

Without trigger conditions, **every** webhook event triggers the workflow.

To filter:

1. Open the Webhook Trigger
2. Add a condition (e.g., event.actor.id == "abc123")
3. Workflow only executes when the condition is met

### Example Condition

```
{{ $.event.actor.id }} EQUALS "00u99abcd123"
```

If the incoming event does not match, the workflow simply ignores it.

Trigger conditions let you:

* Only process certain alert types
* Only run when a field exists
* Avoid executing workflows unnecessarily
* Separate “event ingestion” from “workflow logic”

---

## 12. Cross-SOAR Comparison

### Shuffle SOAR

* Uses webhooks extensively—major mechanism for alert ingestion.
* Accepts JSON, text, or arbitrary payloads depending on HTTP handler.
* Trigger conditions typically implemented within workflow logic.

### Cortex XSOAR

* Integrations usually push alerts into XSOAR via REST API or webhook-like collectors.
* Conditional execution handled in playbook tasks.

### Splunk SOAR (Phantom)

* Uses JSON-based REST ingestion.
* Each incoming event automatically becomes a “container”.
* Filtering done via playbook logic or ingestion rules.

Across all platforms, webhooks or API ingestion endpoints are the backbone of automated event handling.

---

## 13. Quick Reference Summary

* Webhooks are Torq’s mechanism for receiving external events.
* They support URL parameters, JSON bodies, XML, text, and binary (if enabled).
* Always add authentication headers before publishing.
* Binary files require enabling “Accept raw HTTP request”.
* Trigger conditions control when workflows fire.
* Webhooks must be published to trigger workflows automatically.
* Capturing `$.event` into a variable makes downstream logic easier.