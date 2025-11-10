Torq — HTTP Step Options (Reference Notes)
1) Purpose of the HTTP Step

    Acts as a universal connector to call external APIs (REST, GraphQL, SOAP).

    Can be used to:

        Pull data into the workflow.

        Push updates to another system.

        Test or prototype integration steps before building full connectors.

    Useful for quick enrichment, alert escalation, and sending commands to other platforms.

2) Required Fields
URL

    Full API endpoint (include https://).

    Can be static (hardcoded) or dynamic (use parameters/variables).

    Example:

https://api.example.com/v1/users

or

    https://api.example.com/v1/users/{{user_id}}

Method

    GET: Retrieve data (default).

    POST: Create resource/send data.

    PUT/PATCH: Update resource.

    DELETE: Remove resource.

    HEAD/OPTIONS: Metadata or preflight requests.

Authorization

    None: For public APIs or internal open endpoints.

    Basic Auth: Username & password (Torq stores creds securely).

    Bearer Token: For token-based APIs — often obtained from a prior auth step.

    Custom Header: Add Authorization: Bearer {{token}} manually if needed.

3) Optional Parameters (Gear Icon)

    These enhance stability, performance, and compatibility — important for production automations.

    Timeout: How long to wait for a response before failing.

        Default is reasonable, but reduce for fast-failing services.

    Skip SSL Verification:

        Only use for trusted internal systems with self-signed certs.

        Avoid in production unless absolutely necessary.

    Response Encoding:

        Usually UTF-8.

        Change only if API returns other encodings (legacy systems).

    Retries:

        Number of retry attempts for transient failures (e.g., 502, 503).

        Combine with Delay Between Retries (seconds).

    Custom Certificates:

        Upload/use custom CA bundle for special TLS validation cases.

4) Output Handling

    Output Format:

        JSON (default for most APIs).

        Text, XML, or raw data if the API doesn’t return JSON.

    File Storage:

        Can store the response body as a file for later steps (e.g., evidence attachment).

    Save Output:

        Always name output variables clearly (http_status, http_body, http_headers).

5) Best Practices for Using the HTTP Step in Automations

    Parameterize whenever possible:

        Use workflow parameters for endpoint fragments, IDs, or tokens.

    Separate Auth from Business Logic:

        If token generation is needed, do it in a prior step and pass the token here.

    Branch Early:

        Check status_code before trying to parse the body.

    Log Key Details:

        Capture endpoint, params, and status for troubleshooting.

    Fail Fast on Bad Config:

        Test your HTTP step in isolation before chaining it with other steps.

6) Time-Saving Features

    Copy/Paste Commands:

        Paste a cURL command and Torq will parse it into step settings automatically.

    Save Step for Reuse:

        Turn common API calls into saved templates for future workflows.

7) Common Use Cases in SOAR

    Query threat intel API (IP/domain/file hash enrichment).

    Create or update tickets in ITSM tools (ServiceNow, Jira).

    Pull logs from SIEM or EDR APIs.

    Trigger scans or response actions in security tools (e.g., isolate endpoint).

    Post messages or alerts to collaboration platforms (Slack, Teams).

8) Troubleshooting Tips

    401 Unauthorized:

        Check token/creds, ensure they’re current and in correct header format.

    403 Forbidden:

        Verify account permissions for that endpoint.

    404 Not Found:

        Check path variables (IDs), confirm the resource exists.

    429 Too Many Requests:

        Reduce call frequency or implement backoff.

    5xx Errors:

        Retry after short delay; check API provider status.

9) GitHub Commit Guidelines (Future)

When saving HTTP step examples in your repo:

    Sanitize all URLs, tokens, usernames/passwords.

    Include:

        Example configuration screenshot.

        Exported step JSON/YAML with placeholders.

        Short README describing:

            API purpose

            Key inputs/outputs

            Example use case in automation