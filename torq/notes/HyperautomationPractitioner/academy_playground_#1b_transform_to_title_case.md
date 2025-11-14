# Torq HyperAutomation Practitioner – Academy Playground #1 (Challenge B: Transform to Title Case & Manage Workflow Versions)

## 1. Summary

This challenge expands on the previous **“Convert text to uppercase”** workflow by modifying it to produce **Title Case** output instead.
It also introduces Torq’s **workflow management features**—including **export/import, versioning, annotations, and rollback**—to simulate real-world automation lifecycle management.

---

## 2. Objective

* Convert an existing workflow’s functionality from Uppercase → Title Case.
* Practice export/import of published workflows.
* Learn to use version control: publish, unpublish, modify, and restore.
* Validate output through test runs after each iteration.

---

## 3. Workflow Modification & Version Management Steps

### Step 1. Export the Original Workflow

* Opened **Convert text to uppercase** workflow.
* From the **Actions menu → Export published**, downloaded the YAML version.
* This creates a versioned copy of the published workflow for reuse or modification.

---

### Step 2. Import and Open for Editing

* Navigated to **Workflows → Import Workflow**.
* Selected the exported YAML file and confirmed import.
* The imported workflow automatically opened in **Designer** mode for editing.

---

### Step 3. Replace the String Utility

* From **Builderbox → Utilities → String**, dragged **Convert string to title case** under the On-Demand trigger.

* Set its **INPUT** to:

  ```text
  {{ $.event.text }}
  ```

* In the **Exit** operator:

  * Replaced the input for `newtext` with:

    ```text
    {{ $.convert_string_to_title_case.result }}
    ```

* Deleted the old **Convert string to uppercase** step.
  *(Steps can be disabled instead of deleted if future reuse is expected.)*

---

### Step 4. Update Metadata

* Edited annotation to:

  > “This workflow converts text to title case.”

* Opened **Workflow Settings → Rename →**
  **Convert text to title case** (case sensitive).

* Published the updated workflow.

---

### Step 5. Validate the New Behavior

* Ran the workflow with sample input:

  ```
  This is a sample text to test this workflow.
  ```
* Verified `Exit.newtext` =

  ```
  This Is A Sample Text To Test This Workflow.
  ```

---

### Step 6. Unpublish for Further Modification

* Unpublished the workflow to remove it from production, enabling safe testing of new logic.

---

### Step 7. Add Truncation Step

* From **Builderbox → String Utilities**, dragged **Truncate text string** under the Title Case step.

* Configured:

  * **INPUT:** `{{ $.convert_string_to_title_case.result }}`
  * **LIMIT:** `3`

* Updated the **Exit.newtext** parameter:

  ```text
  {{ $.truncate_text.result }}
  ```

* Ran the workflow with the same sample input and confirmed truncated result:

  ```
  Thi
  ```

---

### Step 8. Update Annotation and Publish

* Edited annotation to:

  > “This workflow truncates text.”
* Published new version with updated description.

---

### Step 9. Roll Back via Version History

* Opened the **workflow menu → Version History**.

* Selected the prior version (Title Case without truncation).

* Clicked **Restore version → Confirm**.

* Verified in Designer that truncation step was removed and only **Convert to Title Case** feeds the Exit operator.

* Published the restored version.

---

### Step 10. Final Validation

* Ran the restored workflow using input:

  ```
  this is text to convert to title case
  ```
* Confirmed output:

  ```
  This Is Text To Convert To Title Case
  ```
* Version control confirmed to be functioning as expected.

---

## 4. Workflow Output Examples

| Input                                        | Version             | Output                                       |
| -------------------------------------------- | ------------------- | -------------------------------------------- |
| This is a sample text to test this workflow. | Title Case          | This Is A Sample Text To Test This Workflow. |
| This is a sample text to test this workflow. | Truncate (Limit 3)  | Thi                                          |
| this is text to convert to title case        | Restored Title Case | This Is Text To Convert To Title Case        |

---

## 5. Key Learnings

1. **Export/Import Workflows** – enables sharing and versioning of production-ready YAMLs.
2. **Context Paths** – reused across modified utilities without breaking execution (`$.event.text`).
3. **Annotations and Naming Discipline** – essential for readability and future maintenance.
4. **Version History & Rollback** – mirrors real DevOps practices within a SOAR context.
5. **Iterative Testing** – small controlled changes help maintain workflow reliability while experimenting.

---

## 6. Optional Local Simulation (Python Equivalent)

Demonstrates both Title-Case conversion and truncation logic.

```python
def to_title_case(text: str) -> str:
    return text.title()

def truncate_text(text: str, limit: int) -> str:
    return text[:limit]

# Test data
sample_text = "this is a sample text to test this workflow."

# Simulate Title Case workflow
title_case_result = to_title_case(sample_text)
print("Title Case Output:", title_case_result)

# Simulate Truncation workflow
truncated_result = truncate_text(title_case_result, 3)
print("Truncated Output:", truncated_result)
```

**Output:**

```
Title Case Output: This Is A Sample Text To Test This Workflow.
Truncated Output: Thi
```

---

## 7. Folder Structure

```
/torq/automation-practitioner/academy-playground-1-transform-to-title-case/
├── README.md
└── scripts/
    └── transform_to_title_case.py
```

---

## 8. Reflection

This exercise emphasized **workflow lifecycle management** — not just functional automation.
It taught how to evolve, document, test, and restore changes efficiently while maintaining output integrity.
In a real SOC environment, these principles ensure **repeatable, traceable, and reversible** automation development processes.