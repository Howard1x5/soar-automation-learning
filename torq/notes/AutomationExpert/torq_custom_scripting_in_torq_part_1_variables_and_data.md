# **Bring Your Own Code: Custom Scripting in Torq — Part 1

Working with Variables and Data**

This module explains how to safely and correctly pass different data types from Torq workflows into Python scripts.

Torq scripts can accept:

* **Numbers (int/float)**
* **Strings**
* **Raw text (with special characters)**
* **Entire JSON objects**

The primary challenge is **quoting**, **escaping**, and **ensuring Python receives valid syntax**.

This module shows how to do that in Torq — then provides equivalents for other SOAR platforms and direct Python usage.

---

# **1. Passing Single Numbers to Python Scripts**

Torq lets you reference workflow metadata using:

```
.metadata
```

Example field:

```
.metadata.status_code
```

### **Torq Python Script**

```python
number = {{ .metadata.status_code }}
print(number)
print(type(number))
print(number + 5)
```

### **Output**

```
1
<class 'int'>
6
```

### **Key Point**

Numbers in Torq do **not** require quoting — they import cleanly into Python as integers.

---

## **SOAR Equivalents**

### **Splunk SOAR**

```python
number = int(container.get('status_code'))
```

### **XSOAR**

```python
number = int(demisto.args()['status_code'])
```

### **Shuffle**

```python
number = int(data["status_code"])
```

### **Plain Python Pattern**

```python
number = int(input_value)
```

---

# **2. Passing Strings to Python**

Strings in Torq **must be wrapped in quotes**, otherwise Python thinks they are variables.

### **Incorrect (breaks script)**

```python
value = {{ .metadata.account_name }}
```

### **Correct**

```python
value = "{{ .metadata.account_name }}"
print(value)
print(type(value))
print(value + "_extended")
```

### Output

```
torq-acme-demo
<class 'str'>
torq-acme-demo_extended
```

---

## **Cross-SOAR Equivalents**

### **Splunk SOAR**

```python
value = container.get('account_name')
```

### **XSOAR**

```python
value = demisto.args()['account_name']
```

### **Shuffle**

```python
value = str(data["account_name"])
```

### **Python Pattern**

```python
value = str(input_string)
```

---

# **3. Passing Raw Text with Special Characters**

Text with:

* Quotes `" "`
* Apostrophes `'`
* Brackets
* Escaped JSON

…will break Python if not properly escaped.

### **Torq Example**

Workflow variable:

```
.text contains: "He said, \"Hello.\""
```

### Broken Script

```python
text = "{{ .text }}"
```

Fails due to unescaped quotes inside the string.

---

## **Fix Option A — Use Torq's Built-In Escape Function**

```python
text = "{{ .text | jsonEscape }}"
print(text)
```

### Output

Valid Python string with escaped characters.

---

## **Fix Option B — Escape via Convert Utility (External Step)**

Use the **Escape JSON String** Torq step before the Python script.

---

## **Python Equivalent Cookbook Pattern**

```python
import json

safe = json.dumps(raw_text)
text = json.loads(safe)
```

---

## **SOAR Equivalents**

### **Splunk SOAR**

```python
import json
text = json.loads(json.dumps(container.get('text')))
```

### **XSOAR**

```python
text = demisto.args()['text']
```

### **Shuffle**

```python
text = data["text"]
```

---

# **4. Passing Entire JSON Objects Into Python**

This is one of the most powerful Torq scripting features.

### **Key Rule:**

Use **single quotes** around the inserted JSON.

### **Torq Script**

```python
import json

metadata = '{{ .metadata }}'   # single quotes prevent JSON breakage
obj = json.loads(metadata)

print(obj)
print(type(obj))
print(obj["account_name"])
```

### Why single quotes?

Because JSON contains **double quotes** which would terminate Python strings.

---

## **Output**

Python receives the entire JSON object cleanly.

---

## **Cross-SOAR Patterns**

### **Splunk SOAR**

```python
obj = phantom.get_container(container_id)
```

### **XSOAR**

```python
obj = demisto.args()['context']
```

### **Shuffle**

```python
obj = json.loads(data["json_blob"])
```

---

## **Python Cookbook Pattern**

```python
import json

raw = input_json_string
parsed = json.loads(raw)

# Use it
```

---

# **5. Real-World SOC Use Cases**

### **1. Passing Raw EDR Events for Lightweight Python Enrichment**

* Extracting process ancestry
* Calculating risk score
* Performing custom time deltas (first seen vs last seen)

### **2. Converting JSON Alerts to Python Objects**

Useful when writing:

* Deduplication logic
* Alert scoring
* Similarity hashing
* Correlation rules

### **3. Wrapping Vendor API Responses in Python**

Sometimes you must do:

* Pagination
* Retry logic
* Data normalization
* Recursive extraction

### **4. Handling User Input From Slack / Email**

Regex → cleaned text → JSON → Python

### **5. Preparing Data for a SOAR Output**

Python → JSON → Table → Slack / Email summary

---

# **6. Summary**

You now have the foundational skills for integrating scripts into Torq:

| Data Type               | Torq Requirement | Python Result |
| ----------------------- | ---------------- | ------------- |
| Number                  | No quotes        | int           |
| String                  | Quotes required  | str           |
| Text with special chars | Escape required  | valid str     |
| Entire JSON             | Single quotes    | dict          |

This module sets the stage for:

* Part 2 (API requests, handling responses)
* Part 3 (Multi-script workflows and shared variables)
* Part 4 (Building reusable Python utility blocks inside Torq)