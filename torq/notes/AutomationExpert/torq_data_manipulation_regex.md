# **1. Overview**

Regex (Regular Expressions) provide flexible pattern matching for text, allowing workflows to:

* Trigger only on specific message patterns
* Extract meaningful substrings
* Validate identifiers (IDs, hashes, URLs, usernames)
* Parse log headers
* Branch logic based on matched patterns
* Enhance JQ filters with regex operators (`match`, `test`, `scan`)

Regex is applied across:

| Where                     | How                                                        |
| ------------------------- | ---------------------------------------------------------- |
| **Triggers / Webhooks**   | Run workflow only when incoming data matches pattern       |
| **IF / Switch Operators** | Regex-based branching logic                                |
| **Extraction Utilities**  | Regex pattern extraction from raw text or downloaded files |
| **JQ Filters**            | `match`, `scan`, `test` for deep JSON pattern queries      |
| **Python steps**          | Standard Python `re` operations                            |

---

# **2. Regex in Torq Triggers**

### **Use Case:**

Triggering a workflow only when a Slack message contains specific commands (e.g., jq test, CSV regex, etc.)

### **Torq Configuration**

Trigger source: **Slack Message Trigger**

Conditions:

1. **Channel Regex Match**

   * Ensures commands only run in specific Slack channel(s)

2. **Text Regex Match**

   * Expression allows flexible input such as:

     * `jq test`
     * `JQ test`
     * `csv regex`
     * `CSV regex`

Example pattern:

```
^(?:jq|JQ)\s*test$|^(?:csv|CSV)\s*regex$
```

### **Behavior**

* `jq test` → triggers
* `JQTest` → triggers
* `jQ test` → triggers
* `csv regex` → triggers
* `CSV REGEX` → *does not trigger* unless explicitly included in the pattern

---

# **3. Regex in Switch Operators**

Torq switch branches can evaluate patterns using **Regex Match**.

### Example: Extract Only Organization IDs Matching Pattern

Dataset: CSV of organization IDs
Pattern requirement:

* First **3 characters** must be **letters (A–Z, case-insensitive)**
* Remaining **12 characters** may be **letters or digits**
* Total length = 15

Regex:

```
^[A-Za-z]{3}[A-Za-z0-9]{12}$
```

Switch branch:

| Branch | Condition                      |
| ------ | ------------------------------ |
| `A`    | Regex Match on organization ID |
| `B`    | Default / no match             |

---

# **4. Regex Extraction Utilities in Torq**

Utility: **Extract All Using Regex**

### Example pattern:

```
[A-Za-z]{3}[A-Za-z0-9]{12}
```

Torq autodownloads the file from a URL and extracts only matching IDs.

Output (JSON array of all matched values):

```json
[
  "ABC123456789AB",
  "DEF987654321CD",
  ...
]
```

---

# **5. Regex Inside JQ (match, test, scan)**

JQ provides 3 regex functions:

| Function | Purpose                        | Output                        |
| -------- | ------------------------------ | ----------------------------- |
| `match`  | Extracts matching substring(s) | Matching JSON object(s)       |
| `scan`   | Returns array of all matches   | Strings                       |
| `test`   | Boolean (true/false)           | Quickly check if match exists |

---

## **Example Dataset:** Email Headers

JSON object containing SMTP Received headers and message metadata.

Two regexes:

```jq
# Servers under Outlook domain starting with "email"
"email[0-9]+\.outlook\.com"

# Headers starting with "Report"
"^Report.*domain.*"
```

### **JQ Using match**

```jq
.headers[] | match("email[0-9]+\\.outlook\\.com")
```

Output:
Matched objects, including index and substring metadata.

### **JQ Using test**

```jq
.headers[] | test("^Report.*domain.*")
```

Output:
`true` or `false` for each header.

### **Difference**

| Function | Returns                                  | Use Case          |
| -------- | ---------------------------------------- | ----------------- |
| `match`  | Extracted match with structured metadata | Data extraction   |
| `test`   | Boolean only                             | Conditional logic |

---

# **6. Python Equivalent Regex Logic**

```python
import re

pattern = r"^[A-Za-z]{3}[A-Za-z0-9]{12}$"

matches = [item for item in items if re.match(pattern, item)]

print(matches)
```

### Regex in Python for email header parsing:

```python
emails = re.findall(r"email[0-9]+\.outlook\.com", header_text)
reports = re.findall(r"^Report.*domain.*", header_text, flags=re.M)
```

---

# **7. Splunk SOAR Equivalent**

### **Regex Condition (filters / playbooks)**

```python
import re

if re.search(r"^[A-Za-z]{3}[A-Za-z0-9]{12}$", org_id):
    # proceed
```

### **Regex block in Phantom**

Use the “Regex” action in playbooks.

---

# **8. Cortex XSOAR Equivalent**

Automation script:

```python
import re

matches = re.findall(r"email[0-9]+\.outlook\.com", text)
return_results(matches)
```

Conditional:

```python
if re.match(pattern, value):
    return_results(True)
```

---

# **9. Shuffle SOAR Equivalent**

Python app:

```python
import re

output = re.findall(pattern, input_string)
```

---

# **10. SOC Real-World Applications of Regex**

### **1. Extracting IoCs from Alerts**

* IPv4, IPv6
* URLs
* Domains
* MD5/SHA hashes

Regex is often faster than Python parsing.

---

### **2. Normalizing alert text**

Parse log lines like:

```
AUTHFAIL user=admin src=10.0.0.5 port=443
```

Regex:

```
user=(\w+)
src=([0-9\.]+)
port=(\d+)
```

---

### **3. Triaging phishing emails**

Extract:

* Mail server hops
* Return-Path
* DKIM results
* SPF alignment failures

---

### **4. Triggering ChatOps commands**

Slack commands such as:

```
!lookup 8.8.8.8
!scan xyz.com
!ioc add hash=aaaa
```

All powered by regex.

---

### **5. JQ + Regex for EVTX or Sysmon data**

Example:

```jq
.select(.Image | test("powershell.exe$"))
| .CommandLine
```

---

# **11. Summary**

Regex provides powerful, flexible pattern-matching capabilities inside Torq and across all major SOAR platforms.

### You can now perform:

| Operation                    | Torq | JQ               | Python | SOAR Platforms |
| ---------------------------- | ---- | ---------------- | ------ | -------------- |
| Trigger workflows with regex | ✓    | —                | ✓      | ✓              |
| Branch logic via regex       | ✓    | —                | ✓      | ✓              |
| Extract text using regex     | ✓    | `match` / `scan` | ✓      | ✓              |
| Boolean pattern detection    | ✓    | `test`           | ✓      | ✓              |
| Validate IDs, URLs, hashes   | ✓    | ✓                | ✓      | ✓              |

Regex works best when paired with **JQ**, **Python**, and **Torq extraction utilities** to solve nearly any text-based parsing problem.