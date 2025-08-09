Torq — Workflow Building Fundamentals (Hands‑On Notes)
1) What You’ll Build First (Mental Model)

A minimal, professional “hello‑real‑world” workflow:

    Trigger: On‑demand or Webhook

    Steps:

        Optional: Validate / normalize inputs

        HTTP request to an external API

        Branch on status_code (200 vs non‑200)

        Extract fields from JSON and format a result

        Optional: Post to Slack/Jira/email, or return data to caller

    Data handling: Context variables, parameters, and safe credential use

    Ops hygiene: Version notes and annotations

This pattern generalizes to 80% of SOAR work: ingest → enrich → decide → act.
2) Create a New Workflow (Step‑By‑Step)

    New Workflow

        Click Create Workflow (or equivalent).

        Name: Enrich Indicator via HTTP (be descriptive; avoid “test”).

        Description: One sentence on what/why (helps later reviewers).

        Version Note (optional): “Initial scaffold.”

    Choose a Trigger

        On‑Demand (fastest to test from the UI)

            Use this for manual runs while developing.

        Scheduled (only if you have a periodic task)

            Example: daily hygiene checks.

        Webhook (useful to simulate alert ingestion)

            Torq provides a unique URL; copy it for later curl tests.

        IMAP (only if you truly need mailbox‑driven flows)

    Recommendation while learning: Start with On‑Demand. Add Webhook later so you can curl inputs during testing.

    Define Parameters (Inputs)

        Add parameters to keep your workflow reusable.

        Example parameters:

            indicator (string) — “The IP/domain/email to enrich”

            source (string, optional) — “Caller or source system”

        Set sensible defaults for quick testing.

        Why: Parameters prevent hard‑coding and make the workflow callable from other workflows or external systems.

    Credentials & Integrations (If Needed)

        Add any required integrations (e.g., Slack/Jira) and store credentials in Torq’s secure store (never hard‑code).

        For pure HTTP tests, prefer a public endpoint first (e.g., https://httpbin.org/status/200 or https://jsonplaceholder.typicode.com/users/1) to avoid auth while you learn.

3) Add Core Steps (with Reasoning)
Step A: (Optional) Validate Inputs

    Utility / Data step: Check that indicator exists and matches basic format.

    If invalid: Set a failure message and end early.

    Why: Input validation reduces noisy downstream errors.

Step B: HTTP Request (Enrichment)

    Action: “Send HTTP Request” (or similar)

    Method: GET (or POST with JSON body if the API requires)

    URL examples (no auth needed for practice):

        https://httpbin.org/status/200 (return a specific status)

        https://jsonplaceholder.typicode.com/users/1 (returns JSON to parse)

    Headers: Accept: application/json

    Body: usually empty for GET

    Save Output: Store full response object into context, e.g., http_response

Pro tip: Name outputs predictably: http.status_code, http.body, http.headers.* if Torq exposes them separately; otherwise keep a single JSON and reference its fields.
Step C: Branch on Status Code

    Condition Step: if http_response.status_code == 200

        TRUE path: proceed to parse JSON

        FALSE path: log error info and end (or take remediation)

    Why: Branching on status is a universal SOAR pattern. Make success/failure flows explicit.

Step D: Parse JSON (Success Path)

    Data Extraction Step: Pick fields from http_response.body

        Example (jsonplaceholder):

            name = body.name

            email = body.email

        Store each extracted field into named context variables.

    Why: Downstream steps should consume clean, named variables, not raw blobs.

Step E: Format Result / Take Action

    Utility Step: Construct a message (string template) summarizing outcome:

        Example: Enrichment succeeded: name={{name}}, email={{email}}

    Optional Action:

        Post to Slack/Jira/email, or

        Return a JSON object as the workflow output (for Webhook/API calls)

    Why: Every run should produce an auditable, human‑readable outcome or a well‑structured machine response.

4) Variables, Context, and Data Passing

    Local Context: Lives within a single run. Use it for step outputs and temporary values.

    Global Variables: For constants shared across workflows (e.g., base URLs).
    Avoid storing secrets here—use the credential store.

    Selectors / Mappings: Use Torq’s UI to map step_output.field → next_step.input.
    Keep names short and consistent (status_code, json, name, email).

    Null Safety: Always guard against missing keys. If a field may be absent, branch or default it.

Naming Convention (suggested):

    Parameters: param_* (e.g., param_indicator)

    HTTP step outputs: http_* (e.g., http_status, http_json)

    Derived variables: enrich_* (e.g., enrich_name)

5) Conditional Logic & Loops
IF / ELSE

    Branch on values like status_code, json.length, or indicator type.

    Keep branches narrow; favor small, readable decision points over massive trees.

LOOP (For Each)

    Use when API returns arrays (e.g., multiple matches or artifacts).

    Inside the loop:

        Extract per‑item fields.

        Aggregate results into a list (Torq usually offers “append to array” utilities).

    Exit loop with a summarized result (count, joined strings, array of objects).

6) Nested Workflows (Modularity)

    Create small, focused workflows:

        wf_enrich_indicator

        wf_notify_slack

        wf_create_ticket

    Call Workflow Step: The parent workflow passes parameters to the child.
    Child returns a structured result (status + fields).

    Why: Modularity improves reuse and testing. If Slack changes, you only update wf_notify_slack.

7) Annotations, Versioning, and Auditability

    Annotations: Add short notes on non‑obvious logic (“We branch here because API X returns 204 on success”).

    Version Notes: Summarize changes—“v1.1: added non‑200 retry.”

    Outputs: Ensure every run leaves evidence: status, key fields, and error messages logged in execution details.

8) Testing & Debugging (Tight Loop)

    Dry Run (On‑Demand): Use parameter defaults and run.

        Check outputs at each step (expand “last run” details).

    Webhook Test (if added):

        Copy webhook URL.

        Send a test payload:

        curl -X POST '<WEBHOOK_URL>' \
          -H 'Content-Type: application/json' \
          -d '{"indicator":"example.com","source":"local-test"}'

        Confirm the payload appears under trigger input; trace through steps.

    Negative Tests: Force a 404 or 500 with httpbin to validate your error branch.

    Edge Cases: Missing indicator, unexpected JSON shape, empty arrays.

If something fails:

    Inspect each prior step’s raw output.

    Confirm your selector paths (exact key names, case sensitivity).

    Re‑run with logging enabled at key points.

9) Example Config (Concrete Values)

Trigger: On‑Demand (later add Webhook)

Parameters:

    indicator (string) — default: example.com

    source (string, optional) — default: manual

Step: HTTP GET

    Method: GET

    URL: https://jsonplaceholder.typicode.com/users/1

    Headers: Accept: application/json

    Save as: http_response

Step: Condition

    Expression: http_response.status_code == 200

On TRUE (Parse)

    Extract:

        user_name = http_response.body.name

        user_email = http_response.body.email

    Build message:
    result_msg = "Enrichment OK: name=" + user_name + ", email=" + user_email

On FALSE (Error)

    Build message:
    result_msg = "Enrichment FAILED: status=" + http_response.status_code

Final Step

    Return/Log result_msg (and the parsed fields if desired).

    Optional: Send to Slack/Jira.

10) Operational Hygiene & Guardrails

    Secrets: Use Torq’s credential store. Never hard‑code tokens or keys.

    Timeouts & Retries: For flaky APIs, set reasonable timeouts and limited retries with backoff.

    Rate Limits: If an API returns 429, honor Retry‑After.

    Idempotency: For POST actions that create tickets/cases, include idempotency logic (e.g., dedupe key) when the API supports it.

    Observability: Capture status_code, endpoint, elapsed time, and any request_id for post‑mortems.

    Error Messages: Prefer concise, actionable errors (“401 from API—check credentials in integration X”).

11) “Why This Choice?” Cheat Sheet

    On‑Demand first: Quickest feedback while designing.

    Webhook second: Makes it callable by tools and easy to curl.

    Parameters over constants: Reuse and testing without editing the canvas.

    HTTP to a public endpoint: Learn mechanics without auth friction.

    Branch on status code: Separates happy path from failure logic early.

    Small, named variables: Clear data flow and easier debugging.

    Nested workflows for actions: Reusable building blocks (notify, ticket).

12) What to Commit (Later, Portfolio‑Safe)

    Sanitized workflow export (no secrets).

    README.md describing: problem, inputs, outputs, status handling.

    Screenshot(s) of canvas and a successful/failed run.

    Sample curl for the webhook trigger with placeholder values.

13) Fast Expansion Paths (When Ready)

    Replace the public API with a real enrichment source (requires credentials).

    Add a For‑Each loop when the API returns arrays (e.g., multiple results).

    Add Slack/Jira branching: only notify on high‑value outcomes.

    Wrap the whole thing as a child workflow—callable from anywhere.