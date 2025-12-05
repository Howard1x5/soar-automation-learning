# Torq Automation Expert

## Advanced Data Transformation — JQ Deep Dive

This module teaches how to use **JQ** for high-performance JSON manipulation inside Torq workflows, and how to achieve the equivalent transformations in Python, Splunk SOAR, Cortex XSOAR, and Shuffle.

---

# 1. Overview

### What JQ Enables

* Query JSON
* Transform JSON
* Group data
* Rename fields
* Filter on conditions (including dates)
* Deduplicate arrays
* Reshape complex nested structures
* Generate new JSON objects from existing ones

### Why use JQ instead of loops?

✔ Runs in **one step**
✔ Much faster than nested loops
✔ Reduces memory use
✔ Reduces workflow complexity
✔ Perfect for working with large datasets (hundreds/thousands of items)

---

# 2. Dataset Used in Examples

Same 300-record user dataset used in earlier modules:

```json
[
  {
    "uid": "x123",
    "name": "John",
    "department": "IT",
    "active": true,
    "last_login": "2024-07-15"
  },
  ...
]
```

---

# 3. Torq Implementation (JQ Step)

All examples below are performed using:

**Utilities → Run JQ Command**

---

## 3.1. Basic Select / Transform

### **Extract name + active status**

```jq
{ username: .name, status: .active }
```

### Torq Output

```json
[
  { "username": "John", "status": true },
  ...
]
```

---

## 3.2. Group by field

### Group by department

```jq
group_by(.department)
```

### Group by department **and** active status

```jq
group_by({department: .department, active: .active})
```

---

## 3.3. Reshape Grouped Output

Create structured output:

```jq
group_by({department: .department, active: .active})
| map({
    dept: .[0].department,
    status: .[0].active,
    users: map({ uid: .uid, name: .name, last_login: .last_login })
})
```

---

## 3.4. Add counts to groups

```jq
group_by({department: .department, active: .active})
| map({
    dept: .[0].department,
    status: .[0].active,
    count: length,
    users: map({ uid: .uid, name: .name, last_login: .last_login })
})
```

---

## 3.5. Filter by date (convert to Unix time)

Torq/JQ query:

```jq
map(select(
  (.last_login | fromdate) >
  ("2024-08-01T00:00:00Z" | fromdate)
))
```

---

## 3.6. Full JQ Example (Grouping + Transforming + Count + Filtering)

```jq
map(select(
  (.last_login | fromdate) >
  ("2024-08-01T00:00:00Z" | fromdate)
))
| group_by({department: .department, active: .active})
| map({
    dept: .[0].department,
    active: .[0].active,
    count: length,
    users: map({
      uid: .uid,
      name: .name,
      last_login: .last_login
    })
})
```

---

# 4. Python Equivalent

Python is the closest match to JQ in terms of capability but requires more manual logic.

---

## 4.1. Group by department + active

```python
from itertools import groupby
import operator

data = users  # JSON list

# First sort by grouping keys
sorted_data = sorted(data, key=lambda x: (x["department"], x["active"]))

grouped = []
for key, group in groupby(sorted_data, key=lambda x: (x["department"], x["active"])):
    dept, active = key
    group_list = list(group)
    grouped.append({
        "dept": dept,
        "active": active,
        "count": len(group_list),
        "users": [
            {
                "uid": u["uid"],
                "name": u["name"],
                "last_login": u["last_login"],
            }
            for u in group_list
        ],
    })
```

---

## 4.2. Filter by date

```python
from datetime import datetime

cutoff = datetime.fromisoformat("2024-08-01T00:00:00")

filtered = [
    u for u in users
    if datetime.fromisoformat(u["last_login"]) > cutoff
]
```

---

## 4.3. Transform user fields

```python
transformed = [
    {"username": u["name"], "status": u["active"]}
    for u in users
]
```

---

# 5. Splunk SOAR (Phantom) Equivalent

Phantom cannot run JQ, so the approach is:

* Use **Filter List** blocks
* Use **Custom Python** to emulate JQ transformations
* Use **Format Block** for output objects

---

## 5.1. Grouping (Python Automation Block)

```python
data = phantom.collect2(container=container, datapath=["users"])[0][0]

groups = {}
for u in data:
    key = (u["department"], u["active"])
    groups.setdefault(key, []).append(u)

result = []
for (dept, active), items in groups.items():
    result.append({
        "dept": dept,
        "active": active,
        "count": len(items),
        "users": [
            {"uid": i["uid"], "name": i["name"], "last_login": i["last_login"]}
            for i in items
        ]
    })

phantom.save_run_data(key="jq_style_result", value=result)
```

---

# 6. Cortex XSOAR Equivalent

XSOAR supports:

* Transformers
* Built-in automation scripts
* Python automations that run very similarly to Torq Python steps

### 6.1. Filter by date

Transformer: `DateGreaterThan`

### 6.2. Group by + Count

Requires **Python automation**:

```python
results = []

groups = {}

for u in users:
    key = (u['department'], u['active'])
    groups.setdefault(key, []).append(u)

for (dept, active), items in groups.items():
    results.append({
        'dept': dept,
        'active': active,
        'count': len(items),
        'users': [{'uid': i['uid'], 'name': i['name'], 'last_login': i['last_login']} for i in items]
    })

return_results(results)
```

---

# 7. Shuffle SOAR Equivalent

Shuffle supports:

* JSONPath filters
* Python scripts
* For heavy transformation, **Python step is the correct approach**

---

## 7.1. JSONPath filtering example

Active users:

```
$[?(@.active==true)]
```

Department="IT":

```
$[?(@.department=='IT')]
```

---

## 7.2. Python step for JQ-like transformation

```python
import json
from itertools import groupby

users = data  # from previous Shuffle step

# Sort and group
sorted_data = sorted(users, key=lambda u: (u["department"], u["active"]))

result = []
for key, group in groupby(sorted_data, key=lambda u: (u["department"], u["active"])):
    dept, active = key
    group_list = list(group)
    result.append({
        "dept": dept,
        "active": active,
        "count": len(group_list),
        "users": [
            {"uid": u["uid"], "name": u["name"], "last_login": u["last_login"]}
            for u in group_list
        ]
    })

return result
```

---

# 8. When to Use JQ Instead of Python or Loops

### Use **JQ** when:

* You’re transforming JSON
* You need grouping / filtering / selection
* You want to avoid nested loops
* You want a compact one-liner
* Data set could be thousands of records

### Use **Python** when:

* The transformation requires complex logic
* External libraries are needed
* You’re scraping data
* You need stateful iteration

### Use **Loops** only when:

* You must call an API per item
* You must enrich each item individually

---

# 9. Summary

JQ in Torq allows:

* High-speed data transformations
* Eliminating loops
* Grouping, mapping, counting
* Filtering by field and date
* Reshaping JSON structures
* Preparing data for downstream workflows

Replicating JQ behavior in other SOAR platforms typically requires:

* **Python automations**
* **Transform blocks**
* **JSONPath filters**