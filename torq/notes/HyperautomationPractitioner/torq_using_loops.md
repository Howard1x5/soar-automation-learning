# Using Loops in Torq Workflows

## 1. Summary

This module explains how to use **loops** in Torq workflows to process collections of data, generate new data, accumulate results, and control execution flow.

Covered concepts:

* Loop types: *range* and *collections*
* Loop modes: *sequential* vs *parallel*
* Naming loop keys and values
* Using **collect** to accumulate data from loops
* Using **break** to stop loop execution
* Nested loops
* Flattening results (pagination scenarios)

Loops are essential for building scalable, dynamic automation workflows—especially when interacting with APIs, arrays, or paginated data.

---

## 2. Creating a Simple Loop (Range-Based)

Start with a new workflow:

1. Add a **Loop** operator.
2. Set to **Sequential** mode.
3. Set loop type to **Range**:

   * Start: `1`
   * End: `5`
   * Step: `1`
4. Rename the loop key/value:

   * `GenerateKey`
   * `GenerateValue`

This loop will run 5 iterations.

You can verify iterations in the execution log, which displays `iteration: 0`, `iteration: 1`, etc.

---

## 3. Generating Random Values Inside a Loop

Add a **Generate Random Number** operator inside the loop:

* Min: `1`
* Max: `6`

Running the loop generates 5 random numbers—one per iteration.

---

## 4. Collecting Loop Output (Building an Array)

To capture each random number:

1. Add a **Collect** operator inside the loop.
2. Name the collection `random_numbers`.
3. Build the collected object:

```json
{
  "number": "{{ $.GenerateRandomNumber.result }}"
}
```

Execution now yields:

```json
[
  {"number": 4},
  {"number": 5},
  {"number": 3},
  {"number": 3},
  {"number": 2}
]
```

The collect operator builds arrays automatically.

---

## 5. Accumulating a Total (Using Variables Inside Loops)

Add a **Set Variable** operator:

* Name: `total`
* Type: number

Expression:

```
{{ add($.GenerateRandomNumber.result, $.num) }}
```

This accumulates a running total as the loop executes.

Example:

* Loop iteration values: 4, 5, 3, 3, 2
* Final total: **17**

---

## 6. Looping Over Collected Results

Nest another loop under the first loop.

### Outer Loop

* Name: `loopOverNumbers`
* Mode: **Parallel** (allowed because these operations are independent)
* Loop input:
  `{{ $.collect_random_numbers.result }}`

Rename keys:

* `numKey`
* `numValue`

### Inner Loop

* Name: `generateStrings`
* Type: **Range**
* Start: `1`
* End: `{{ $.numValue.number }}`
* Mode: **Parallel**

Inside the inner loop, generate random strings:

* **Generate Random String**
* Length: `8`

Then collect them:

```
{{ $.GenerateRandomString.result }}
```

---

## 7. Structuring Output from Nested Loops

Once inner loop collects random strings, build an object combining the number and its strings:

```json
{
  "num": "{{ $.numValue.number }}",
  "strings": {{ $.collectStrings.result }}
}
```

Collect these objects into `collectResults`.

This produces structured output:

```json
[
  {
    "num": 4,
    "strings": [ "abc123ef", "91ks0sd0", "pqo391xm", "...." ]
  },
  {
    "num": 5,
    "strings": [ "83jdke8s", "02ksk3md", "...." ]
  }
]
```

---

## 8. Breaking Out of a Loop

Break is only supported in **sequential** loops.

Example:

* Insert an **If** operator inside the outer loop.
* Condition:

```
{{ $.numValue.number }} EQUALS 3
```

If true → add a **Break** operator.

If false → continue to generate strings.

This stops further iterations when the number `3` appears.

---

## 9. Important Note About Collect in Nested Loops

**Collect resets every time its parent loop iterates.**

Meaning:

* If `collectStrings` is inside an inner loop, it resets for each number.
* To capture all results across the entire nested structure, you must:

  * Collect inside the inner loop
  * Collect again at the outer loop

If you only view a `collectStrings` placed after the parent loop, it will only reflect the **last iteration**, not the full dataset.

---

## 10. Flattened Results (Pagination Scenario)

Torq provides a **Flatten Results** option underneath loop parameters.

Use cases:

* Pagination against an API returning pages of 100 results
* Want a single flat array instead of an array of arrays

When enabled:

* Instead of: `[ [100], [100], [100] ]`
* You get: `[300 elements]`

This simplifies working with paginated IAM, HR, SIEM, or ticketing system data.

---

## 11. Cross-SOAR Comparison

### Shuffle SOAR

* Supports loop-like operations by chaining workflow steps.
* Looping often achieved through:

  * “For each” actions
  * Re-triggering workflow with pagination offsets
* No native “break,” requires conditional branching.

### Cortex XSOAR

* Built-in loops via “For Each,” “While,” and task-level script loops.
* Collecting results handled via context keys.
* Break logic implemented through conditional exits.

### Splunk SOAR (Phantom)

* Looping usually handled through:

  * Custom functions
  * Playbook loops
  * Container iteration
* Break requires explicit logic.

Across all systems, looping is used heavily for:

* Pagination
* Dataset traversal
* Batch processing
* API enrichment tasks

Torq’s loops are more visual, flexible, and ergonomic.

---

## 12. Quick Reference Summary

* Use **range** loops for generating values.
* Use **collection** loops to iterate over arrays.
* Use **sequential mode** when using **break**.
* Use **parallel mode** to speed up independent operations.
* **Collect** stores loop output but resets each parent iteration.
* **Flattened results** help manage pagination.
* Nested loops require thoughtful collection placement.
* Combine loops, collect, and variables to build complex data structures.