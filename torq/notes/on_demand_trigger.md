Torq — On-Demand Trigger (Automation Analyst Certification)
1) Overview

    Purpose: The trigger is the first step in any Torq workflow.

    Function: It defines the source event that will execute the workflow.

    Event Format: Events are structured data, typically in JSON, ingested by the trigger and evaluated against conditions to decide whether to run the workflow.

    Why It Matters: Triggers determine when automation runs and with what input. Getting this right ensures the workflow is efficient, reusable, and secure.

2) Types of Triggers in Torq

Triggers can be swapped at any time via the Replace icon in the trigger step, which opens the Selection Window.
1. On-Demand (Manual)

    Start workflow manually from the UI.

    Useful for:

        Development/testing.

        Ad-hoc executions.

        Manually reprocessing events.

    Allows selection of input values before execution.

    Tip: This is the safest trigger for new workflows since nothing will run until you explicitly start it.

2. Form-Based

    Starts a workflow when a form is submitted.

    Can be used for:

        Interactive, human-driven workflows.

        Collecting structured data from users.

    Tip: Ideal for SOC analysts requesting enrichments or checks without writing queries themselves.

3. Scheduled

    Runs at specific times (e.g., daily at noon, every 30 minutes).

    Ideal for:

        Periodic data pulls.

        Automated maintenance jobs.

    Security Consideration: Be careful with scheduled tasks that call external APIs—avoid hitting rate limits.

4. Case Activity

    Trigger workflow from within a case.

    Can attach files, add notes, or perform changes in context of a case.

    Great for post-incident actions (e.g., IOC enrichment, artifact extraction).

5. Email (IMAP)

    Watches an inbox for specific incoming messages.

    Can process attachments or email content directly.

    Tip: Use filters to prevent triggering on every message.

6. Integration Event

    Executes on events from third-party systems (via HTTP/webhooks).

    Common for:

        Receiving alerts from security tools.

        Running automation in response to external events.

    Best Practice: Secure webhook endpoints with tokens and validate payloads before processing.

3) Setting Parameters in Triggers

Parameters make workflows reusable and flexible.

    Add parameters in the Trigger Step using the + icon in the parameters section.

    Types include:

        Short text (e.g., IP address)

        JSON

        Select lists (single or multi)

    Reference parameters in workflow steps:

        Example: event.json.ip_address

    Can predefine initial values for testing.

    Execution can be triggered with:

        Default values.

        Modified/custom values.

    Best Practice: Use descriptive parameter names (e.g., indicator_ip, ticket_id) to make mapping easier in larger workflows.

4) Reusing Historical Events

    Any past event in the trigger’s history can be re-run.

    Click the Re-run icon at the top of the trigger step.

    Option to:

        Execute event as-is.

        Modify inputs before execution.

    Pro Tip: This is useful for regression testing—rerun an old event after making workflow changes to confirm nothing broke.

5) Using Webhooks with On-Demand Workflows

    Webhooks can be linked to workflows to allow external triggering.

    Steps:

        Move trigger to Webhook type.

        Copy webhook URL.

        Paste into browser or API client to trigger workflow.

    Useful for:

        Internal testing.

        Allowing other systems to run your workflow.

    Security Tip: Always validate webhook payloads and authenticate where possible to avoid abuse.

6) Web Form Customization

    Workflows can present a web form to collect inputs.

    Create a new draft of a published workflow to edit forms.

    Form Step (Utility):

        Name the form.

        Configure inputs (text, select lists, etc.).

        Can chain multiple forms together (“Next Form”).

    Example:

        First form: collect IP address.

        Second form: choose “engine” for analysis.

        Third form: review and confirm.

    Pro Tip: Use forms to control what users can input—this reduces errors and enforces consistent formatting.

7) Conditional Logic with Form Inputs

    Use If/Else conditions based on form responses.

    Example:

        If “engine = VirusTotal” → run VirusTotal enrichment.

        Else → run alternative enrichment.

    Pass form outputs to steps as parameters.

    Pro Tip: Keep branching simple; complex decision logic is easier to manage in smaller, modular workflows.

8) Output Handling

    Final step can present:

        Summary of results.

        Raw JSON (with json_escape to handle formatting).

    Example:

        Extract fields from an API call and present as text.

        Show only relevant summary info to the user.

    Best Practice: Outputs should be human-readable when needed but also machine-usable for chaining into other workflows.

9) Example Use Case Walkthrough

    Trigger: On-Demand (IP address parameter).

    First step: VirusTotal reputation check.

    Parameters passed from trigger to API request.

    Conditional: If high-risk → take action; else → log only.

    Output:

        Risk score.

        Relevant metadata.

    Optional: Make accessible via webhook for team use.

    Extra Tip: Document each step in the workflow to help other engineers understand logic flow.

10) Best Practices

    During development: Use On-Demand for quick testing.

    For production: Switch to event-based triggers once validated.

    Parameterize everything that may change between runs.

    Annotate workflows for maintainers.

    Use form steps for human-in-the-loop automation.

    Store secrets in Torq’s credential vault, not in parameters.

    Audit logs regularly—triggers can be an attack surface if not monitored.

11) Additional Learning Resources

These videos and guides walk through creating On-Demand and event-based triggers in Torq and other SOAR tools:

    https://www.youtube.com/watch?v=sXQxhojSdZM — Intro to Torq Workflows & Triggers (Torq Official)

    https://www.youtube.com/watch?v=q0o34XAh2Qk — Using On-Demand Triggers in SOAR Automation (Community Example)

    https://www.youtube.com/watch?v=mv2qgGgJ0ac — Webhook and Manual Trigger Best Practices (SOAR General)