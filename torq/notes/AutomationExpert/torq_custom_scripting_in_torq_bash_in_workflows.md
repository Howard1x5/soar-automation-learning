# **Using Bash in Torq Workflows

(HTTP calls, VT lookups, jq filtering, output formatting)**

Bash scripting inside Torq allows you to:

* Run **curl**, **jq**, **sed**, **awk**, and standard Linux utilities
* Build API calls (e.g., VirusTotal, OTX, AbuseIPDB)
* Manipulate output into JSON
* Pass workflow variables into Bash
* Return structured output to the rest of the workflow

This module demonstrates passing variables, building API calls, formatting output with `jq`, and producing readable results.

---

# **1. Bash Basics in Torq**

When you drop a **Bash Script** step, Torq inserts a starter template:

```bash
#!/bin/bash
echo "Hello from bash" | jq .
```

Key notes:

* `echo` outputs text
* `jq` ensures the output is valid JSON for downstream steps
* Torq variables must use **${$.path.to.value}**
* Internal bash variables use **$varname**

---

# **2. Passing Variables Into Bash**

Suppose your trigger defines two parameters:

* `ip_address`
* `vt_base_url` → `https://www.virustotal.com/api/v3/ip_addresses/`

You reference them as:

```bash
url="${vt_base_url}${ip_address}"
echo "$url"
```

Example output:

```
https://www.virustotal.com/api/v3/ip_addresses/8.8.8.8
```

---

# **3. Making a VirusTotal API Request (Bash)**

### **Retrieve API Key From Torq Integration Vault**

Inside the script:

```bash
token="${integrations.virustotal.api_key}"
```

### **Execute cURL Request**

```bash
response=$(curl -s \
    -H "x-apikey: $token" \
    "$url")
```

Notes:

* `-s` silences progress meter → JSON only
* Store output in a variable to pipe into jq

### **Output Clean JSON**

```bash
echo "$response" | jq .
```

---

# **4. Extracting Specific Data Using jq**

VirusTotal response structure:

```
data
  attributes
    last_analysis_stats
```

To pull ONLY these stats:

```bash
echo "$response" | \
jq '{result: .data.attributes.last_analysis_stats}'
```

Output:

```json
{
  "result": {
    "harmless": 85,
    "malicious": 19,
    "suspicious": 3,
    "timeout": 0,
    "undetected": 28
  }
}
```

---

# **5. Build a Table for Slack**

Torq’s *Create ASCII/Markdown Table* step requires a **list**, so wrap output in brackets:

```
[ { "harmless": 85, "malicious": 19, "suspicious": 3 } ]
```

Then use the table creator → send snippet to Slack.

Example Slack output:

```
Reputation results for 8.8.8.8

| harmless | malicious | suspicious |
|----------|-----------|------------|
|   85     |    19     |     3      |
```

---

# **6. SOAR Platform Equivalents**

---

## **Splunk SOAR**

Use **BASH App** or custom app:

```bash
curl -s -H "x-apikey: {{ vt_api_key }}" \
     "https://www.virustotal.com/api/v3/ip_addresses/{{ ip }}"
```

Post-process via `jq` or run a *custom function* to transform data.

---

## **Cortex XSOAR (XSOAR)**

Use **Bash** (automation with Docker image) or use the built-in **VirusTotal v3** integration.

Custom Bash automation:

```bash
#!/bin/bash
response=$(curl -s -H "x-apikey:$VT_API_KEY" \
"https://www.virustotal.com/api/v3/ip_addresses/$1")

echo "$response" | jq '.data.attributes.last_analysis_stats'
```

Run via:

```
!RunCustomScript ip=8.8.8.8
```

You must define:

* Docker image with jq
* Environment variable `VT_API_KEY`

---

## **Shuffle**

Shuffle’s Bash app can run:

```bash
curl -s -H "x-apikey: $VT_API_KEY" "$VT_URL" | jq '.data.attributes.last_analysis_stats'
```

Outputs feed directly into downstream JSON steps.

---

# **7. Python Equivalent (Reusable Cookbook Snippet)**

*This replicates the VT lookup but uses Python instead of Bash.*

```python
import requests
import json

url = f"{vt_base_url}{ip_address}"
headers = {"x-apikey": vt_api_key}

resp = requests.get(url, headers=headers)
data = resp.json()

stats = data["data"]["attributes"]["last_analysis_stats"]

print(json.dumps({"result": stats}))
```

This produces the exact same JSON that Torq expects.

---

# **8. Common Bash Patterns (Cookbook)**

### **A. Combine Torq vars into a URL**

```bash
url="${base_url}${observable}"
```

### **B. Silent curl with JSON response**

```bash
resp=$(curl -s "$url")
```

### **C. Add headers**

```bash
-H "Authorization: Bearer $token"
```

### **D. Extract nested fields**

```bash
echo "$resp" | jq '.a.b.c'
```

### **E. Create a wrapped list for table utilities**

```bash
echo "[$resp]"
```

---

# **9. Common Mistakes to Avoid**

| Mistake                         | Why it fails                        |
| ------------------------------- | ----------------------------------- |
| Using `$var` for Torq variables | Must use `${$.path.to.var}`         |
| Forgetting quotes               | Spaces break the script             |
| Missing `-s` on curl            | Progress meter destroys JSON output |
| Forgetting jq                   | Downstream steps crash without JSON |

---

# **10. Summary**

This module showed how to:

* Pass Torq variables into Bash
* Build VirusTotal requests
* Use curl properly with headers
* Filter JSON using jq
* Produce clean JSON for downstream steps
* Output tables for Slack

Bash steps are ideal when:

* You want lightweight dependency-free automation
* You need quick HTTP calls
* jq filtering is simpler than Python
* You are working with traditional CLI tools