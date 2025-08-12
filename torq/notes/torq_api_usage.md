Torq API Usage
Module Summary

This module teaches how to use the Torq REST API inside workflow steps.
Key points covered:

    Authentication to the Torq API

    Available API resources and their purpose

    Building workflows that call the API

    Using API results to dynamically manage workflows, secrets, integrations, and logs

Key Concepts & Actions
1. Available Torq API Endpoints

The Torq API provides endpoints for:

    Activity logs

    Audit logs

    Secrets (create/read/update/delete)

    User management

    Integration management

    Workflow execution control

    Parameter adjustments

    Workspace operations

Each endpoint typically has:

    GET → Retrieve data

    POST → Create new resources

    PUT/PATCH → Update resources

    DELETE → Remove resources

2. Example Use Case — Exporting Logs to Splunk

    Template workflow: Gather Torq Audit Log + Activity Logs

    Parameters include:

        Integration Name (destination system, e.g., Splunk)

        Time checkpoint for log retrieval

    Workflow steps:

        Set checkpoint timestamp

        Retrieve logs since checkpoint

        Parse & package logs

        Send to Splunk (or another SIEM)

    Reusable for activity or audit logs by switching a parameter

3. Building a Workflow That Copies Another Workflow

Goal: Duplicate an existing workflow (e.g., for migration between workspaces)
Steps:

    Trigger – Ask for the workflow name to copy (input parameter).

    REST – List Workflows – Get all workflows in the workspace.

    Filter Step – Match the workflow name provided by the trigger.

        Exit with failure if no match or multiple matches found.

    REST – Get Workflow Details – Retrieve workflow YAML from the latest published revision.

    REST – Import Workflow – Create a new workflow from the retrieved YAML.

    Tag Workflow – Add identifying metadata (e.g., "Workflow Importer").

    Publish Workflow – Move workflow to production state.

4. Retrieving Executions and Logs

    After publishing, create a loop to:

        Wait until at least one execution appears in Activity Logs

        Retrieve the execution ID from the first available log

        Fetch execution details (outputs, status, errors)

    Outputs can be forwarded to:

        Slack

        Teams

        Email

        Incident tracking systems

Practical Applications

    Migration: Move workflows/secrets between Torq workspaces using API calls.

    Version Control: Export YAML and push to GitHub for change tracking.

    Automation: Chain API calls for CI/CD of workflows.

    SIEM Integration: Export logs periodically to Splunk, Elasticsearch, Sentinel.

Cross-Platform Mapping
Torq Term / Feature	Generic SOAR Concept	Shuffle / Open-Source / Other Platform Equivalent
Torq API	Native platform API for automation	REST API / Swagger UI in Shuffle, TheHive, etc.
Activity Logs API	Execution history endpoint	/api/v1/executions in Shuffle
Audit Logs API	Change/configuration log	Admin event logs in Sentinel or Splunk HEC
Workflow YAML Export	Workflow-as-code	JSON export in Shuffle, ARM templates in Sentinel
Import Workflow	Workflow restore/deploy	POST /workflows/import in Shuffle
Tag Workflow	Metadata labeling	Labels/tags in Splunk SOAR or XSOAR
Publish Workflow	Promote to production	"Enable workflow" toggle in Shuffle
Retrieve Execution	Get run outputs	Execution detail endpoint in any SOAR API
Integration Management API	Connector configuration	Integration management endpoints in open-source SOAR
Secrets API	Credential storage automation	Vault API / environment variable injection
Additional Learning Resources

Full API Docs:

    https://torq.io/api

    https://docs.shuffle.io/api

    https://learn.microsoft.com/en-us/rest/api/securityinsights/ (Microsoft Sentinel)

    https://docs.splunk.com/Documentation/SOAR/current/PlatformAPI/RESTAPI

Video Walkthroughs:

    https://www.youtube.com/watch?v=7wZ4Lk6Tr7I (Example API → Workflow copy logic in Shuffle)

    https://www.youtube.com/watch?v=_nv9tJ-x6rE (Export/import automation in SOAR tools)

Torq API Community Cheat Sheet
Authentication

curl -X POST "https://api.torq.io/v1/auth/token" \
  -H "Content-Type: application/json" \
  -d '{"apiKey": "YOUR_API_KEY"}'

    Response will contain an access token

    Use this token in all subsequent API calls:

-H "Authorization: Bearer YOUR_TOKEN"

List Workflows

curl -X GET "https://api.torq.io/v1/workflows" \
  -H "Authorization: Bearer YOUR_TOKEN"

Get Workflow Details (YAML Export)

curl -X GET "https://api.torq.io/v1/workflows/{workflowId}/revisions/{revisionId}" \
  -H "Authorization: Bearer YOUR_TOKEN"

Import Workflow

curl -X POST "https://api.torq.io/v1/workflows/import" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
        "yaml": "BASE64_ENCODED_YAML_CONTENT"
      }'

Publish Workflow

curl -X POST "https://api.torq.io/v1/workflows/{workflowId}/publish" \
  -H "Authorization: Bearer YOUR_TOKEN"

Get Activity Logs

curl -X GET "https://api.torq.io/v1/activitylogs?workflowId={workflowId}" \
  -H "Authorization: Bearer YOUR_TOKEN"

Get Executions

curl -X GET "https://api.torq.io/v1/executions?workflowId={workflowId}" \
  -H "Authorization: Bearer YOUR_TOKEN"

Send Workflow Output to Slack (Example)

curl -X POST "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX" \
  -H "Content-Type: application/json" \
  -d '{"text": "Workflow Execution Complete: {executionId}"}'

