# Torq Automation Expert — Using the Torq API

## 1. Summary

This module introduces how to use the **Torq REST API** and Torq’s built-in **Torq API integration steps** to interact programmatically with:

* Workflows
* Secrets
* Integrations
* Activity logs
* Audit logs
* Step runners
* Executions

You can use API calls either:

* Directly (via external scripts like Python/Postman/curl), or
* Internally using **Torq API Steps** inside workflows.

This module demonstrates automating:

* Listing workflows
* Filtering/searching
* Retrieving workflow YAML
* Importing workflows
* Tagging workflows
* Publishing workflows
* Waiting for executions
* Using activity logs as execution triggers

This pattern becomes extremely useful for:

* CI/CD for workflows
* Workspace replication
* Dev → Test → Prod migrations
* Automated workflow validation
* Automated backup/export/versioning

---

## 2. Torq API Overview

Torq publishes a full REST API documentation at:

**learn.torq.io → Documentation → Torq API**

API collections include:

* **Activity Logs API**
* **Audit Logs API**
* **Secrets API**
* **Users API**
* **Workflows API**
* **Integrations API**
* **Execution API**
* **Step Runner API**
* **Role / Claim Mapping API**

Each API group includes:

* GET list
* GET by ID
* POST create
* PUT update
* DELETE remove

Torq also supplies **Torq API Steps** inside workflows so you can automate management tasks internally without external scripts.

---

## 3. Workflow Example From Module

Torq provides template workflows such as the Splunk forwarding template:

* `generator-gather-torq-audit-or-activity-logs`

This template:

* Accepts start/end timestamps
* Uses pagination to pull activity or audit logs
* Parses and consolidates data
* Outputs final logs back to parent workflow

It demonstrates how the Torq API is used internally to retrieve logs across paginated endpoints.

---

## 4. Building an API-Driven Automation Workflow (Step-by-Step)

The module walks through building a workflow that:

* Takes a workflow name
* Finds its workflow ID via API
* Validates that only one workflow matches
* Retrieves workflow YAML
* Imports a copy of the workflow
* Tags the new copy
* Publishes it
* Waits for the new workflow to execute
* Retrieves execution output

This is effectively **“self-modifying automation”**—a very high-value SOAR capability.

Below is the full process in structured notes.

---

## 5. Steps in Detail

### 5.1 Input – Ask for Workflow Name

Create a manual trigger → add an Interaction Form asking for:

* Workflow name (text input)

This becomes `$.event.workflow_name`.

---

### 5.2 List All Workflows (Torq API Step)

Use **List Workflows** step.

This returns all workflows in the workspace.

---

### 5.3 Filter Workflows by Name

Use **Filter Array** to find workflow(s) where:

```
$.list_workflows.response.workflows[*].name == $.event.workflow_name
```

Rename step: **Filter Workflow**

Expected result:

* Exactly one workflow
* If not → Exit operator (workflow not found / ambiguous match)

---

### 5.4 Retrieve Workflow YAML

Use **Retrieve Workflow** Torq API Step.

Inputs:

* Workflow ID → from filtered result
* Revision ID → choose the latest published revision

Output will include:

* YAML
* Metadata
* Revision information

---

### 5.5 Import the Workflow (Create a Copy)

Use **Import Workflow** step.

Inputs:

* YAML from Retrieve Workflow step
* This creates a new workflow inside the workspace

The response contains:

* `import_workflow.workflow_id`
* `import_workflow.revision_id`

This is effectively cloning the workflow programmatically.

---

### 5.6 Tag the New Workflow

Use **Add Workflow Tag** step.

Target:

* New workflow ID from import step

Example tag:

```
Workflow-Importer
```

---

### 5.7 Publish the New Workflow

Use **Publish Workflow** step.

Inputs:

* Imported Workflow ID
* Current Revision ID

After publishing:

* Workflow banner turns green → ready for execution

---

### 5.8 Wait for the Workflow to Execute

The imported workflow (Demo Workflow 1) runs every minute.

To detect its execution:

1. Create a **loop** (1..20 iterations)
2. Inside loop → call **List Activity Logs**
3. Filter logs for the imported workflow ID
4. If execution is found → break
5. Else → Wait (e.g., 5 seconds) and continue

This simulates an event-driven “wait for workflow execution” pattern.

---

### 5.9 Retrieve Execution Details

Once an execution ID is detected:

Use **Retrieve Execution** step.

Outputs include:

* Start time
* End time
* All step results
* Errors if present
* Any variables returned

This is ideal for:

* Automated testing
* CI/CD verification
* Workflow promotion pipelines

---

## 6. Practical Uses of Torq API Inside Workflows

You can automate:

* Migration of workflows between tenants
* Automated tagging/versioning
* Daily exporting of YAML to GitHub
* Automated deployment pipelines
* Auto-publishing workflows after lint/validation
* Workspace replication (Secrets, Integrations, Workflows)

This gives Torq true **infrastructure-as-code** capabilities.

---

## 7. Cross-SOAR Equivalent Concepts

### Cortex XSOAR

* Use “`/playbook`” and “`/automation`” API endpoints
* Export & import playbooks via API
* Common use cases:

  * CICD pipelines
  * Git integration
  * Automated deployment validation

### Shuffle SOAR

* Shuffle supports:

  * Workflow import/export
  * API-based migration
  * Updating workflows via REST
* JSON-based workflow structure makes programmatic cloning simple.

### Splunk SOAR (Phantom)

* Playbooks can be pulled via REST (`/rest/playbook`)
* Automations:

  * Versioning
  * Promotion between dev/test/prod
  * Exporting artifacts

Across all platforms:

> Torq’s internal Torq API steps are more advanced than most SOARs because they let you manipulate platform-state from inside a workflow without external scripts.

---

## 8. Python Equivalent Script (Workflow Cloning Example)

This Python example mirrors the Torq process programmatically and is portfolio-safe:

```python
import requests
import time

BASE_URL = "https://api.torq.io"
API_KEY = "<your-api-key>"

def torq_get(path):
    return requests.get(
        BASE_URL + path,
        headers={"Authorization": f"Bearer {API_KEY}"}
    ).json()

def torq_post(path, payload):
    return requests.post(
        BASE_URL + path,
        json=payload,
        headers={"Authorization": f"Bearer {API_KEY}"}
    ).json()

# 1) List workflows
workflows = torq_get("/api/v1/workflows")
demo = next(w for w in workflows["workflows"] if w["name"] == "Demo Workflow 1")

# 2) Retrieve workflow YAML
workflow_yaml = torq_get(f"/api/v1/workflows/{demo['id']}/latest")

# 3) Import (clone) workflow
clone = torq_post("/api/v1/workflows/import", {
    "yaml": workflow_yaml["yaml"]
})

# 4) Publish workflow
publish = torq_post(f"/api/v1/workflows/{clone['id']}/publish", {
    "revision_id": clone["revision_id"]
})

# 5) Wait for execution
exec_id = None
for _ in range(30):
    logs = torq_get("/api/v1/activitylogs")
    matches = [x for x in logs["logs"] if x["workflow_id"] == clone["id"]]
    if matches:
        exec_id = matches[0]["execution_id"]
        break
    time.sleep(5)

# 6) Get execution details
if exec_id:
    details = torq_get(f"/api/v1/executions/{exec_id}")
    print("Execution Output:", details)
```

This is a highly practical automation pattern for GitHub portfolio usage.

---

## 9. Key Takeaways

* Torq exposes a full REST API for platform automation.
* Torq API steps allow you to manage the platform **from inside workflows**.
* Capabilities include:

  * Workflow import/export
  * Publishing/republishing workflows
  * Tagging workflows
  * Querying activity/audit logs
  * Retrieving execution results
* Enables full CI/CD automation for SOAR workflows.
* Mirrored patterns exist in XSOAR, Shuffle, and Splunk SOAR.
* Python scripts can replicate Torq workflows externally.