# **1. Overview**

This module covers the most common data format conversions required in automation workflows. Being able to convert formats cleanly is essential for interoperability between APIs, enrichment tools, SIEM/SOAR platforms, case management systems, and data processing pipelines.

### **You will learn to convert:**

* JSON → YAML
* YAML → JSON
* JSON → XML
* XML → JSON
* JSON → CSV
* CSV → JSON
* JSON → ASCII Table
* JSON → Markdown Table
* JSON → HTML Table
* HTML Table → JSON (via Python)

These transformations allow your workflow to adapt data for whatever downstream system needs it — Slack, email, TIP systems, ticketing, case platforms, SOAR tasks, databases, enrichment engines, etc.

---

# **2. JSON → YAML → JSON**

## **Torq Implementation**

### **Step: Convert JSON to YAML**

Use **Convert JSON to YAML** (Encoding utilities).

Input:

* Any JSON object.

Output:

* YAML-formatted string with indentation and line breaks.

### **Step: Convert YAML back to JSON**

Use **Convert to JSON** utility.

Options:

* `unescape_input: true/false`
  (Set to *true* when YAML contains escaped characters.)

Result:

* Returns original JSON structure.

---

## **Splunk SOAR Equivalent**

Splunk SOAR uses Python for most transformations.

### JSON → YAML:

```python
import yaml
data = results['input_json']
yaml_output = yaml.dump(data)
```

### YAML → JSON:

```python
import yaml, json
converted = yaml.safe_load(yaml_string)
json_output = json.dumps(converted)
```

---

## **Cortex XSOAR Equivalent**

### JSON → YAML:

```python
import yaml
return_results(yaml.dump(json.loads(args['input'])))
```

### YAML → JSON:

```python
return_results(json.dumps(yaml.safe_load(args['yaml_input'])))
```

---

## **Shuffle Equivalent**

Shuffle uses Python apps or scripts:

```python
import yaml
output = yaml.dump(input_json)
```

Reverse:

```python
json_data = yaml.safe_load(input_yaml)
```

---

## **Python General Equivalent**

```python
import yaml, json

json_data = {...}

yaml_converted = yaml.dump(json_data)
json_back = json.loads(yaml.safe_load(yaml_converted))
```

---

## **SOC Usage Example**

* Vendor config files (YAML) → JSON for parsing.
* Converting YAML rules (Sigma, Falco, K8s manifests) into JSON for manipulation.

---

# **3. JSON → XML and XML → JSON**

## **Torq Implementation**

### JSON → XML

Use Python step with `xmltodict`.

```python
import json
import xmltodict

data = json.loads(input)
xml_string = xmltodict.unparse({"root": data})
print(xml_string)
```

### XML → JSON

Use **Convert to JSON** utility.

* Input Type: XML
* Output: JSON object identical to original.

---

## **Splunk SOAR Equivalent**

```python
import xmltodict, json

xml_output = xmltodict.unparse({"root": json_input})
json_output = xmltodict.parse(xml_input)
```

---

## **XSOAR Equivalent**

```python
xml = xmltodict.unparse({...})
json_back = xmltodict.parse(xml_input)
```

---

## **Shuffle Equivalent**

Same pythonic approach:

```python
xml = xmltodict.unparse({"root": json_input})
json_data = xmltodict.parse(xml)
```

---

## **Python General Equivalent**

```python
xml_str = xmltodict.unparse({"data": json_data})
json_back = xmltodict.parse(xml_str)
```

---

## **SOC Usage Example**

* Converting legacy XML alerts from older firewalls or IDS into JSON for SOAR processing.
* Parsing XML outputs from Nessus, Qualys, Rapid7 exports.

---

# **4. JSON → CSV and CSV → JSON**

## **Torq Implementation**

### Step: Create CSV

Configure:

* Select JSON array
* Choose header order
* Remove columns you don’t want.

Output example:

```
name,email,age
Bob,bob@org.com,32
Alice,alice@org.com,28
```

### Step: CSV → JSON

Use **Convert CSV to JSON**.

Automatically maps headers → fields.

---

## **Splunk SOAR Equivalent**

```python
import csv, json
from io import StringIO

# JSON → CSV
output = StringIO()
writer = csv.DictWriter(output, fieldnames=data[0].keys())
writer.writeheader()
writer.writerows(data)
csv_str = output.getvalue()

# CSV → JSON
reader = csv.DictReader(StringIO(csv_str))
json_list = list(reader)
```

---

## **XSOAR Equivalent**

```
!ConvertCSVToJSON csv="name,age\nbob,32"
```

JSON → CSV via automation script.

---

## **Shuffle Equivalent**

Use built-in CSV parsers or python scripts.

---

## **Python General Equivalent**

```python
import csv, json
from io import StringIO

# JSON → CSV  
csv_buffer = StringIO()
writer = csv.DictWriter(csv_buffer, fieldnames=data[0].keys())
writer.writeheader()
writer.writerows(data)
csv_output = csv_buffer.getvalue()

# CSV → JSON  
reader = csv.DictReader(StringIO(csv_output))
json_data = list(reader)
```

---

## **SOC Usage Example**

* Exporting SOAR case metadata to CSV for management.
* Converting CSV threat feeds into JSON for enrichment engines.
* Processing user lists from HR systems.

---

# **5. ASCII Tables + Markdown Tables**

## **Torq Implementation**

### Create ASCII Table

Configure:

* Headers
* Column widths
* Option: Markdown output

ASCII version is large; Markdown is compact.

### Markdown Example

```
| name  | email        | age |
|-------|--------------|-----|
| Bob   | bob@org.com  | 32  |
| Alice | alice@org.com| 28  |
```

Markdown is preferred for Slack, Jira, Torq Forms.

---

## **Splunk SOAR Equivalent**

Use Python table generator:

```python
from tabulate import tabulate
table = tabulate(data, headers="keys", tablefmt="github")
```

---

## **XSOAR Equivalent**

```
return_results(tableToMarkdown("User List", data))
```

---

## **Shuffle Equivalent**

Plugins or python tabulate.

---

## **Python Equivalent**

```python
from tabulate import tabulate
md = tabulate(json_data, headers="keys", tablefmt="github")
```

---

## **SOC Usage Example**

* Summary tables for Slack alerts
* Case notes in Markdown
* Analyst handoff reports

---

# **6. HTML Tables + HTML → JSON (Python)**

## **Torq Implementation**

### Create HTML Table

Configure:

* Headers
* Border color
* Border width
* Border style
* Output HTML

### Example snippet produced:

```html
<table border="1" style="border-color:#0070FF;">
<tr><th>name</th><th>email</th></tr>
<tr><td>Bob</td><td>bob@org.com</td></tr>
</table>
```

---

## **Convert HTML Table → JSON (Python Script)**

#### Torq Python step (Pandas-based)

```python
import pandas as pd

html = variables["html_table"]
df = pd.read_html(html)[0]
print(df.to_json(orient="records"))
```

Pandas automatically parses the HTML table.

---

## **Splunk SOAR Equivalent**

Same approach with pandas or BeautifulSoup.

---

## **XSOAR Equivalent**

Would use Python automation:

```python
import pandas as pd
df = pd.read_html(args["html"])[0]
return_results(df.to_dict(orient="records"))
```

---

## **Shuffle Equivalent**

Also Python/pandas.

---

## **SOC Usage Example**

* Scraping internal dashboards that expose data only via HTML output
* Pulling IoCs or alerts from web portals
* Converting vendor HTML report exports into structured JSON for enrichment

---

# **7. Summary**

### **Formats you can now convert:**

| Conversion            | Torq Built-in | Python Needed? | SOAR Compatibility |
| --------------------- | ------------- | -------------- | ------------------ |
| JSON → YAML           | Yes           | No             | Universal          |
| YAML → JSON           | Yes           | No             | Universal          |
| JSON → XML            | Python        | Yes            | Universal          |
| XML → JSON            | Yes           | No             | Universal          |
| JSON → CSV            | Yes           | No             | Universal          |
| CSV → JSON            | Yes           | No             | Universal          |
| JSON → ASCII Table    | Yes           | No             | Universal          |
| JSON → Markdown Table | Yes           | No             | Universal          |
| JSON → HTML Table     | Yes           | No             | Universal          |
| HTML → JSON           | Python        | Yes            | Universal          |

### **Practical SOC Workflows Enabled**

* Data normalization across platforms
* Pull/transform/enrich IoCs
* Convert reports into structured data
* Publish formatted tables to Slack, Jira, Teams
* Generate readable case notes
* Ingest non-JSON vendor output into your automation pipeline