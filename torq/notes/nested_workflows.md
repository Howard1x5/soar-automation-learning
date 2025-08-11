Torq — Nested Workflows (Automation Analyst Certification)
1) Overview

    Definition:
    Nested workflows are workflows designed to be called by other workflows, functioning like reusable subroutines in programming.

    Purpose:
    Break down complex automation tasks into smaller, reusable units that can be applied in multiple parent workflows.

    Analogy:
    Think of a nested workflow as a “function” or “module” in coding — you build it once, then call it wherever needed.

    Benefits:

        Reusability: Write once, reuse across multiple workflows.

        Modularity: Easier to troubleshoot and maintain.

        Consistency: Ensures standardization for common processes.

        Scalability: Makes large automations manageable.

2) Example Use Case — Threat Intelligence Management

    Scenario:
    An analyst finds IOCs (Indicators of Compromise) during threat hunting and needs to upload them into a Threat Intelligence Platform (TIP).

    Without automation:
    Manual copy-paste of IOCs, formatting, and uploading — time-consuming and error-prone.

    With nested workflow:

        Extract IOCs from raw text.

        Transform them into JSON format.

        Upload them to the TIP via API.

    Why nesting helps:
    These steps (extract, enrich, upload) are common across many SOC workflows — phishing response, malware analysis, SIEM correlation, etc.

3) Structure of a Nested Workflow

    Trigger:
    Often a manual or API-triggered event; set to “nested only” to prevent direct execution.

    Inputs:

        Main input text (e.g., raw IOCs from an investigation).

        Optional integrations (API keys, platform credentials).

    Processing Steps:

        Parsing/normalizing data.

        Removing duplicates.

        Consolidating into JSON.

        Filtering out empty values (e.g., using jqfilter to keep only indicators with a count > 0).

    Outputs:

        JSON object with:

            Indicator type

            Count of items

            Extracted data

4) Output Handling & Error Control

    Exit Operator:

        Assigns the final structured output to a named variable accessible by the parent workflow.

    Conditional Logic for Errors:

        If no IOCs are found:

            Return { "error_message": "IOC not found", "error_code": 1 }

        Else:

            Return full JSON results.

    Execution Flow:

        Successful runs return status + execution log.

        Parent workflow can branch logic based on:

            IOC count > 0 → process further.

            IOC count = 0 → handle as informational/no action.

5) Calling a Nested Workflow from a Parent Workflow

    Call Method:

        Use workflow ID to call the nested workflow.

        Pass parent workflow variables as inputs.

    Best Practices:

        Escape JSON strings if passing as text.

        Clearly map variables in the “Run Workflow” step.

    Example Parent Workflow Flow:

        Receive IOC data from webhook/email parsing.

        Pass raw text to nested workflow.

        Get parsed JSON results back.

        Loop over IOCs to insert into TIP or another security platform.

6) Publishing Rules

    Important:

        Both parent and nested workflows must be published for production use.

        Nested workflows can be set to “nested only” so they aren’t triggered directly by events.

    Order:

        Publish nested workflow first.

        Then publish the parent workflow.

7) Practical Example from Module

    Analyst pastes raw text containing potential IOCs into an automation form.

    Nested workflow:

        Extracts IP addresses.

        Filters empty/duplicate entries.

        Creates JSON output with counts and details.

    Parent workflow:

        Loops over IOCs.

        Sends each IOC into OpenCTI or another TIP.

    Result:

        All IOCs auto-ingested into the platform with zero manual formatting.

8) Additional Implementation Notes

    Security:

        If nested workflow calls external APIs, secure credentials in environment variables or Torq vault.

    Testing:

        Always run nested workflow standalone in dev mode to verify parsing logic before embedding in parent workflow.

    Version Control:

        When updating a nested workflow, ensure all parent workflows depending on it are tested.

    Naming Conventions:

        Prefix nested workflow names with NW_ or similar to identify them quickly.

9) Related Resources

    Torq Documentation — Nested Workflows:
    https://docs.torq.io/docs/nested-workflows

    Torq GitHub Example Workflows (Unofficial):
    https://github.com/torq-io/workflows

    YouTube Walkthrough:
    https://www.youtube.com/watch?v=9Y9J_0B7i_A