# Transform Data Using Python

## Cross-Platform SOAR Implementations (Torq, Python, XSOAR, Phantom, Shuffle)

This module teaches how to transform workflow data using Python scripts across multiple SOAR platforms.
Primary use cases:

* Convert **HTML tables → JSON**
* Compute statistics (median, average, 90th percentile)
* Perform arbitrary JSON restructuring
* Handle transformations not possible through built-in utilities

Python gives every SOAR platform "superpowers" where built-in steps stop being sufficient.

---

# 1. Torq Implementation

## Example 1 — Convert HTML table → JSON using Pandas

### Use Case

Torq includes a built-in step to convert JSON → HTML table, but **not** the reverse.
Python fills that gap.

### Torq Python Step Code

```python
import pandas as pd
from bs4 import BeautifulSoup

# HTML table from a previous Torq step
html_table = data["html_table"]

# Parse HTML table
soup = BeautifulSoup(html_table, "html.parser")
table = soup.find("table")

# Convert to DataFrame
df = pd.read_html(str(table))[0]

# Output as JSON records
print(df.to_json(orient="records"))
```

### Output

```json
[
  {
    "uid": "x123",
    "department": "IT",
    "last_login": "2024-07-15"
  },
  ...
]
```

---

## Example 2 — Compute Stats Using NumPy (90th Percentile, Median, Average)

### Torq Python Step Code

```python
import numpy as np
import json

users = data["users"]  # list of { user, duration }

durations = [u["duration"] for u in users]

p90 = float(np.percentile(durations, 90))
avg = float(np.mean(durations))
median = float(np.median(durations))

result = []

for u in users:
    entry = dict(u)
    entry["p90"] = round(p90, 2)
    entry["average"] = round(avg, 2)
    entry["median"] = round(median, 2)
    result.append(entry)

print(json.dumps(result))
```

### Output Example

```json
[
  {
    "user": "alpha",
    "duration": 15,
    "p90": 29.3,
    "average": 18.4,
    "median": 17.0
  },
  ...
]
```

---

# 2. Pure Python Reference Version

(Useful for GitHub and local testing before deploying into a SOAR step)

### Convert HTML Table → JSON

```python
import pandas as pd
from bs4 import BeautifulSoup

def html_table_to_json(html_str):
    soup = BeautifulSoup(html_str, "html.parser")
    table = soup.find("table")
    df = pd.read_html(str(table))[0]
    return df.to_dict(orient="records")
```

---

### Compute p90, median, mean on an array of objects

```python
import numpy as np

def compute_stats(items):
    durations = [i["duration"] for i in items]
    
    p90 = float(np.percentile(durations, 90))
    avg = float(np.mean(durations))
    median = float(np.median(durations))

    for obj in items:
        obj["p90"] = round(p90, 2)
        obj["average"] = round(avg, 2)
        obj["median"] = round(median, 2)

    return items
```

---

# 3. Splunk SOAR (Phantom) Equivalent

Splunk SOAR uses **custom functions** or **on-success scripts** to replicate this.

## Example 1 — HTML Table → JSON

(Requires BeautifulSoup + Pandas inside Phantom's Python 3 runtime)

```python
import pandas as pd
from bs4 import BeautifulSoup

html = phantom.collect2(container=container, datapath=["artifact:*.cef.html_table"])[0][0]

soup = BeautifulSoup(html, "html.parser")
table = soup.find("table")
df = pd.read_html(str(table))[0]

phantom.save_run_data("html_json", df.to_dict(orient="records"))
```

---

## Example 2 — Compute Stats

```python
import numpy as np

records = phantom.collect2(container=container, datapath=["artifact:*.cef.duration"])[0]

durations = [int(x) for x in records]

stats = {
    "p90": float(np.percentile(durations, 90)),
    "avg": float(np.mean(durations)),
    "median": float(np.median(durations))
}

phantom.save_run_data("computed_stats", stats)
```

---

# 4. Cortex XSOAR Equivalent

XSOAR supports Python automations natively — easiest platform for this.

## Example 1 — Convert HTML Table → JSON

```python
import pandas as pd
from bs4 import BeautifulSoup

html = demisto.args().get("html")

soup = BeautifulSoup(html, "html.parser")
table = soup.find("table")
df = pd.read_html(str(table))[0]

return_results(df.to_dict(orient="records"))
```

---

## Example 2 — Compute Statistics

```python
import numpy as np

items = demisto.args().get("items")

durations = [i["duration"] for i in items]

p90 = float(np.percentile(durations, 90))
avg = float(np.mean(durations))
median = float(np.median(durations))

for i in items:
    i["p90"] = p90
    i["average"] = avg
    i["median"] = median

return_results(items)
```

---

# 5. Shuffle SOAR Equivalent

Shuffle supports Python directly but lacks Pandas/NumPy by default.
You typically bundle pure-Python percentile logic.

For example, 90th percentile without NumPy:

```python
import math

items = data["items"]
durations = sorted([i["duration"] for i in items])

index = math.ceil(0.9 * len(durations)) - 1
p90 = durations[index]
avg = sum(durations) / len(durations)

# median
mid = len(durations) // 2
median = durations[mid] if len(durations) % 2 else (durations[mid] + durations[mid-1]) / 2

for obj in items:
    obj["p90"] = p90
    obj["average"] = avg
    obj["median"] = median

return items
```

### HTML Table → JSON (Shuffle)

```python
from bs4 import BeautifulSoup
import pandas as pd

html = data["html"]

soup = BeautifulSoup(html, "html.parser")
table = soup.find("table")
df = pd.read_html(str(table))[0]

return df.to_dict(orient="records")
```

---

# 6. When to Use Python Instead of JQ or Native Steps

Use Python when:

* You need to **convert formats** (HTML → JSON, CSV → JSON, binary → base64)
* You need **advanced statistical operations**
* You require external libraries like NumPy, Pandas, BeautifulSoup
* You are generating **graphs**, PDFs, or binary output
* You are doing **multi-step logic** with branching and state

Python is your escape hatch for all transformations that exceed native SOAR tool capabilities.

---

# 7. Summary

Python enables:

* Arbitrary data reshaping
* Statistical computations
* Parsing of non-JSON formats
* Backfilling missing SOAR utilities
* Custom outputs (graphs, tables, enriched JSON)