# **1. Returning a Plain String from a Python Script**

Torq captures anything printed via `print()` and treats it as the step output.

### **Basic Example — String Output**

```python
value = "{{ .metadata.account_name }}"
print(value, end="")
```

### Why `end=""`?

Without it, Python adds a newline (`\n`) at the end of the output, which pollutes formatting for:

* Switch operators
* String comparison
* Slack messages
* Templates

**Always use `end=""` when returning simple string output.**

---

## **SOAR Platform Equivalents**

### **Splunk SOAR**

```python
phantom.comment(container, "{{ account_name }}")
```

### **XSOAR**

```python
return_results("{}".format(demisto.args()['account_name']))
```

### **Shuffle**

```python
print(data["account_name"], end="")
```

---

# **2. Returning JSON from a Python Script**

This is where most new users break scripts.

**Printing raw Python dictionaries does NOT produce JSON.**
You must explicitly serialize with `json.dumps()`.

---

## **Step 1 — Import workflow JSON as a string**

```python
raw = '{{ .metadata }}'
```

**Single quotes** prevent double-quote collisions.

---

## **Step 2 — Convert string → Python dictionary**

```python
import json
obj = json.loads(raw)
```

---

## **Step 3 — Build your new JSON**

```python
output = {
    "account": obj["account_name"],
}
```

---

## **Step 4 — Serialize to JSON**

```python
print(json.dumps(output))
```

### Result

Torq recognizes this as a **real JSON object**, not a string.

---

## **SOAR Equivalents**

### **Splunk SOAR**

```python
return {"account": container["account_name"]}
```

### **XSOAR**

```python
return_results({"account": demisto.args()['account_name']})
```

### **Shuffle**

```python
print(json.dumps({"account": data["account_name"]}))
```

---

# **3. Outputting Files from a Python Script**

Files are extremely powerful in SOAR engineering for:

* Exporting reports
* Generating CSVs
* Producing PCAPs
* Sending raw logs
* Passing artifacts to downstream tools

Torq supports file output directly inside script settings.

---

## **Example — Return Metadata as a File**

### Python Script

```python
import json
raw = '{{ .metadata }}'
print(raw)
```

### Step Settings

* **File Name:** `metadata.json`
* **Return Response as File:** enabled

### Torq Output

Torq creates an internal artifact:

```
file_id: abc123
size: 1.8 KB
sha256: …
sha1: …
md5: …
```

---

## **Making the File Public**

Enable:

* **Shareable link**

Torq returns:

```
https://files.torq.io/public/.../metadata.json
```

This URL may be used in:

* Slack messages
* Email notifications
* API responses
* Integrations

---

## **Cross-SOAR File Output Equivalents**

### **Splunk SOAR**

```python
phantom.add_artifact(container=container_id,
                     artifact={"name": "metadata.json", "data": obj},
                     overwrite=False)
```

### **XSOAR**

```python
return_results(file_result('metadata.json', json.dumps(obj)))
```

### **Shuffle**

```python
with open('/tmp/metadata.json', 'w') as f:
    f.write(json.dumps(obj))
print("metadata.json")
```

---

# **4. Python Cookbook Patterns (Reusable)**

## **A. Clean String Output (No Trailing Newlines)**

```python
print(result_string, end="")
```

---

## **B. Convert Torq JSON → Python dict**

```python
import json
data = json.loads('{{ .json_var }}')
```

---

## **C. Convert Python dict → JSON Output**

```python
print(json.dumps(data))
```

---

## **D. Create File Output**

```python
content = json.dumps(data)
print(content)   # Torq will write file based on config
```

---

## **E. Pretty JSON Output (Human Readable Logs)**

```python
print(json.dumps(data, indent=2), end="")
```

---

# **5. Real SOC Automation Use Cases**

### **1. EDR Timeline Export**

Generate a CSV of process ancestry and return it as a downloadable file.

### **2. Alert Evidence Packaging**

Bundle logs, screenshots, and enrichment into a file sent to Slack/Jira.

### **3. JSON Normalization Layer**

Standardize vendor alerts into a single XDR-style structure.

### **4. Machine Learning Feeds**

Export cleaned JSON/CSV data for ingestion by ML pipelines.

### **5. Audit Trails**

Return JSON-formatted script logs for compliance workflows.

---

# **6. Summary**

This module showed how to output:

| Output Type | How Torq Receives It               | Common Use                  |
| ----------- | ---------------------------------- | --------------------------- |
| **String**  | Raw text                           | Slack, switch conditions    |
| **JSON**    | Structured object                  | Downstream steps, playbooks |
| **File**    | Artifact with optional public link | Reports, evidence bundles   |

Mastering this unlocks full control of Python automation inside Torq workflows.