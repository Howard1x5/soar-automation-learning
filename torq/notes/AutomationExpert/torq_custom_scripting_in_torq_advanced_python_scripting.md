# **Advanced Python Scripting in Torq

(Downloading Files, TempFS, Pandas, Output Files)**

This module demonstrates how to use **advanced scripting techniques** inside Torq Python steps, including:

* Installing and pinning Python package versions
* Downloading external files directly into the **temporary filesystem (tmpfs)**
* Performing data processing using **Pandas**
* Outputting processed data as a **file**, including public shareable links
* Cross-SOAR equivalents for Splunk SOAR, XSOAR, and Shuffle
* Python cookbook patterns for reuse

---

# **1. Installing Python Package Requirements in Torq**

Torq Python steps run in an isolated container. Only a few libraries are preinstalled.

To add missing packages (e.g., Pandas):

**Properties → Requirements (pip)**

```
pandas==2.0.2
```

You can pin to **any version** required to match your code dependencies.

### Python Script (example: checking installed version)

```python
import pandas as pd
print(pd.__version__)
```

Torq will download the dependency on first execution.

---

## **SOAR Equivalents**

### **Splunk SOAR**

Custom Docker image required → install dependencies inside container.

### **Cortex XSOAR**

Use Docker image selector:

```
demisto/pandas:2.0.2
```

### **Shuffle**

Specify pip libraries in the Python app config.

---

# **2. Downloading Files into tmpfs (Faster I/O)**

Torq allows writing files into the **temporary filesystem**, which lives in RAM:

* Faster than disk
* Auto-deleted when step completes
* Ideal for Pandas, NumPy, ML models, CSV processing, large JSON

### **Download & Store CSV in Temporary File**

```python
import requests
import tempfile

url = "https://example.com/data.csv"
response = requests.get(url)

# Create temporary CSV file in memory
tmp = tempfile.NamedTemporaryFile(suffix=".csv", delete=False)
tmp.write(response.content)
tmp.flush()

print(tmp.name)  # path to file
```

Example output:

```
/tmp/tmpab3k19x7.csv
```

This confirms the file is stored **in memory**, not on persistent disk.

---

# **3. Reading and Processing Data with Pandas**

Once the file is downloaded, pass the filepath to Pandas:

```python
import pandas as pd

df = pd.read_csv(tmp.name)

# Example: keep first 3 columns only
df = df.iloc[:, :3]

print(df.to_string(index=False))
```

Output will include:

* Column headers
* Rows
* Pandas default formatting

---

## **SOAR Equivalents**

### **Splunk SOAR**

```python
df = pd.read_csv('/opt/phantom/tmp/file.csv')
```

### **XSOAR**

```python
df = pd.read_csv(demisto.args()['file'])
```

### **Shuffle**

```python
df = pd.read_csv(tmp_file_path)
```

---

# **4. Outputting Processed Data as a File**

Torq supports returning script output as:

* Internal file
* Public external file via shareable link

### **Step Settings**

**Optional → Filename:**

```
results.csv
```

**Execution Options → Return response as file:**
Checked.

**Execution Options → Shareable link:**
Checked (if you want a public URL).

---

### **Python Script Example**

```python
import requests
import tempfile
import pandas as pd

# Download file
url = "https://example.com/data.csv"
response = requests.get(url)

# Write to tmpfs
tmp = tempfile.NamedTemporaryFile(suffix=".csv", delete=False)
tmp.write(response.content)
tmp.flush()

# Read with Pandas
df = pd.read_csv(tmp.name)
df = df.iloc[:, :3]

# Output CSV (string)
print(df.to_csv(index=False))
```

---

### **Torq Output Example**

```
https://files.torq.io/public/.../results.csv
```

Opening the link displays a file containing:

```
industry,code,industry_name
11,AG,AGRICULTURE
21,MIN,MINING
...
```

---

# **5. Python Cookbook Patterns (Reusable)**

These patterns apply across Torq, XSOAR, Splunk SOAR, and Shuffle.

---

## **A. Download File to tmpfs**

```python
import requests, tempfile

f = tempfile.NamedTemporaryFile(suffix=".csv", delete=False)
f.write(requests.get(url).content)
f.flush()
path = f.name
```

---

## **B. Read CSV with Pandas**

```python
import pandas as pd
df = pd.read_csv(path)
```

---

## **C. Keep First N Columns**

```python
df = df.iloc[:, :3]
```

---

## **D. Convert DataFrame → CSV String**

```python
print(df.to_csv(index=False))
```

---

## **E. Convert DataFrame → JSON**

```python
print(df.to_json(orient="records"))
```

---

## **F. Force Pandas to Show All Rows**

```python
pd.set_option('display.max_rows', None)
```

---

## **G. Make Shareable File Output in Torq**

*(Done via UI — not Python)*

* Filename → `"results.csv"`
* Enable **Return response as file**
* Enable **Shareable link**

---

# **6. Real SOC / Automation Use Cases**

### **1. Malware summary CSV generation**

Download malware IOC dataset → filter → return CSV file.

### **2. EDR timeline processing**

Normalize EDR events into Pandas → export file to Slack.

### **3. Threat intel ingestion pipeline**

Download STIX/CSV → Pandas → normalize → output JSON for ingestion.

### **4. Cost-optimized enrichment**

Store intermediate data in tmpfs for fast multi-step processing.

### **5. Automated SOX evidence generation**

Pull logs + normalize in Pandas → export CSV evidence file.

---

# **7. Summary**

Advanced Python scripting in Torq allows you to:

* Install libraries dynamically
* Use RAM-backed storage for performance
* Process large files with Pandas
* Output files cleanly into workflows
* Integrate with cross-SOAR ecosystems