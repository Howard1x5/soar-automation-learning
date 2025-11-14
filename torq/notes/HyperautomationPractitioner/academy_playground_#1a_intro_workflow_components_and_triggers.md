# Torq HyperSOC – My First Workflow (Convert Text to Uppercase)

## 1. Summary

This introductory **Torq Playground lab** demonstrates how to build a workflow **from scratch**, use **On-Demand triggers**, reference context paths, and return data using the **Exit operator**.

The exercise walks through creating a simple string manipulation automation: taking user-provided input text and converting it to uppercase through a Utility function.

---

## 2. Objective

* Build and publish a workflow from a blank canvas.
* Learn to use On-Demand triggers and runtime form parameters.
* Reference context paths correctly in workflow steps.
* Output workflow data through an Exit operator.

---

## 3. Workflow Design Steps

### Step 1. Create New Workflow

* Navigated to **Workflows → New blank workflow**.
* Named workflow: **My first workflow**.
* Created new section: **Playground Workflows** for organization.
* Selected **On-Demand Trigger** → **Create Workflow**.

---

### Step 2. Configure On-Demand Trigger

* Clicked the trigger block → added a parameter:

  * **Type:** Short text
  * **Name:** `text` *(must match exactly)*
  * **Required:** Enabled

`$.event.text` will later reference this input parameter at runtime.

Saved the configuration.

---

### Step 3. Add String Utility

* Opened **Builderbox → Utilities → String**.
* Dragged **Convert string to uppercase** onto the canvas under the trigger.
* Opened the step properties → set **INPUT** to:

  ```
  {{ $.event.text }}
  ```
* Confirmed proper context path (`event.text`, not `event.test`).
* Saved the step.

---

### Step 4. Add Exit Operator

* From **Builderbox → Operators**, dragged **Exit** operator below the string utility.
* Opened properties:

  * Added new parameter:

    * **Type:** Short text
    * **Name:** `newtext`
  * Set input to:

    ```
    {{ $.convert_string_to_uppercase.result }}
    ```

This passes the uppercase output into the final exit payload.

Saved the configuration.

---

### Step 5. Add Metadata and Publish

* Renamed workflow: **Convert text to uppercase** (exact capitalization).

* Right-clicked canvas → **Add Annotation:**

  > “This workflow converts text to uppercase.”

* Clicked **Publish** → added Version description:

  > “This workflow converts text to uppercase.”

* Added **tag:** `strings`.

* Clicked **Publish Workflow**.

---

### Step 6. Run and Validate

* Clicked **Run Workflow** → entered test input:

  > “This is a sample text to test this workflow.”
* Clicked **Execute**.

**Validation:**

* Opened **Run Log** → selected last run.
* Reviewed both steps:

  * `convert_string_to_uppercase` log: shows input and uppercase output.
  * `Exit` operator log: confirms `newtext` = uppercase result.

---

## 4. Workflow Output Example

**Input:**

```
This is a sample text to test this workflow.
```

**Output (Exit.newtext):**

```
THIS IS A SAMPLE TEXT TO TEST THIS WORKFLOW.
```

---

## 5. Key Learnings

1. **On-Demand Triggers** – enable user-initiated workflows with runtime parameters.
2. **Context Paths (`$.event.*`)** – reference form inputs dynamically.
3. **Builderbox Utilities** – provide modular string and data manipulation tools.
4. **Exit Operator** – standard method to return data from workflow execution.
5. **Annotations and Tags** – improve documentation and searchability.

---

## 6. Optional Local Simulation (Python Equivalent)

To replicate the same logic locally:

```python
def convert_to_uppercase(text: str) -> dict:
    """Simulates the Torq workflow behavior."""
    newtext = text.upper()
    return {"newtext": newtext}

# Example input
event = {"text": "This is a sample text to test this workflow."}
result = convert_to_uppercase(event["text"])

print(result)
```

**Output:**

```json
{
  "newtext": "THIS IS A SAMPLE TEXT TO TEST THIS WORKFLOW."
}
```

---

## 7. Folder Structure

```
/torq/automation-practitioner/my-first-workflow/
├── README.md
└── scripts/
    └── convert_to_uppercase.py
```

---

## 8. Reflection

This lab established foundational Torq workflow design skills:

* Creating workflows from scratch.
* Understanding triggers and runtime context.
* Returning data through standardized outputs.