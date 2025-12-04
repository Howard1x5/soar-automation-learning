# Torq Automation Expert

## Advanced Data Transformation — Loop or Transform?

This module explains when to use loops vs. transformations in Torq — AND how to perform the same logic in Python, Splunk SOAR, Phantom, Cortex XSOAR, and Shuffle.

---

# 1. Concept Overview

### Loops

Used when **each item must trigger an action** (API call, enrichment, Python script, ticket creation).

### Transformations

Used when you just need to reorganize, filter, or compare JSON datasets.

### Why this matters

Choosing the wrong one can make workflows:

* Slow
* Hard to maintain
* More expensive (API rate limits)
* Overcomplicated

---

# 2. Example Dataset

Used throughout the module:

```json
[
  {
    "uid": "x123",
    "name": "John",
    "department": "IT",
    "active": true,
    "last_login": "2024-10-10"
  },
  ...
]
```

Also includes a “day 2” version for comparisons.

---

# 3. Torq Version

## A) Filtering Active Users — Loop Method (Inefficient)

### Torq Steps:

1. Loop → over JSON array
2. IF → active == true
3. Collect → into new array

### Result

Correct, but slow for large datasets.

---

## B) Filtering Active Users — Transformation Method (Correct & Efficient)

### Step: Filter Array

```
Path: active
Condition: equals true (Boolean)
```

Output: Only active users.

### Cascading example

Filter again:

```
department == "IT"
```

Now you have only active IT users.

---

## C) Extract Only Specific Fields

Use `Extract Fields`:

```
Fields:
- uid
```

Then `Join String` → comma-separated string:

```
u123,u985,u772
```

---

## D) Compare Two Arrays Without Loops

Use `Subtract Array`.

### Example:

Left Array = Day1Users
Right Array = Day2Users

**Left – Right:** Users removed or changed
**Right – Left:** New or changed users

---

# 4. Python Equivalent

Python can replace both loops *and* transformations.

### A) Filter active users

```python
active_users = [u for u in users if u["active"]]
```

### B) Filter to active + IT

```python
active_it = [u for u in users if u["active"] and u["department"] == "IT"]
```

### C) Extract UIDs

```python
uids = [u["uid"] for u in active_it]
```

### D) Compare arrays

```python
import json

day1_ids = {u["uid"] for u in day1}
day2_ids = {u["uid"] for u in day2}

added = day2_ids - day1_ids
removed = day1_ids - day2_ids
```

### E) Get updated items (field-level comparison)

```python
updated = [
    u2 for u2 in day2
    if u2["uid"] in day1_ids
    and u2 != next(u1 for u1 in day1 if u1["uid"] == u2["uid"])
]
```

---

# 5. Splunk SOAR (Phantom) Equivalent

### A) Filter active users

Use **Filter List** playbook block:

```
condition1: JSONPath users[*].active == true
```

### B) Filter additional fields

```
condition2: JSONPath users[*].department == "IT"
```

### C) Extract UIDs

Use **Transform List** block:

```
value: $.uid
```

### D) Compare arrays

In Phantom, use the **custom code block (Python)**:

```python
results = phantom.collect2(container=container, datapath=["users"])
day1 = results[0][0]["day1"]
day2 = results[0][0]["day2"]

day1_set = {u["uid"] for u in day1}
day2_set = {u["uid"] for u in day2}

phantom.debug("Added: {}".format(day2_set - day1_set))
phantom.debug("Removed: {}".format(day1_set - day2_set))
```

---

# 6. Cortex XSOAR Equivalent

XSOAR uses automation scripts + transformers.

### A) Filter active users:

**Transformer:** `Where Field Equals`

```
field: active
value: true
```

### B) Filter by IT

Chain transformer:

```
Where Field Equals:
  field: department
  value: IT
```

### C) Extract UIDs

Transformer:

```
Get Field:
  field: uid
```

### D) Compare Arrays

Using built-in transformer **List Difference**

```
left: day1_uids
right: day2_uids
```

---

# 7. Shuffle SOAR Equivalent

Shuffle supports JSONPath + Python.

### A) Filter users (JSONPath):

```
$[?(@.active==true)]
```

### B) Filter users to IT:

```
$[?(@.active==true && @.department=='IT')]
```

### C) Extract UIDs:

```
$[*].uid
```

### D) Compare UsIng Python Step:

```python
day1_ids = {u["uid"] for u in day1}
day2_ids = {u["uid"] for u in day2}

return { 
  "added": list(day2_ids - day1_ids),
  "removed": list(day1_ids - day2_ids)
}
```

---

# 8. When To Loop vs Transform

### Use TRANSFORM when:

* Filtering JSON
* Extracting fields
* Comparing datasets
* Reshaping output
* Reducing dataset *before* looping
* Joining or flattening arrays
* Deduplication

### Use LOOPS when:

* Each item needs an API call
* Each item needs Python execution
* Each item must create a ticket or object
* Enrichment per item
* Pagination through API lists
* Actions that cannot accept arrays

---

# 9. Optimal Pattern (Best Practice)

The best workflows combine both:

1. **Transform first** (Filter → Extract → Reduce)
2. **Loop only over remaining items**
3. **Perform enrichment / API calls inside loop**

This gives massive performance and cost savings.