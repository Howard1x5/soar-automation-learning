# **Advanced Torq Actions

Sprint Functions & Golang Templates**

This module focuses on using **Sprint Functions** and **Go Templates** to manipulate text, JSON, numbers, Base64, and dates directly inside Torq expressions.
These functions reduce the need for separate steps and let you reshape data inline.

Covered topics:

* Text case transformations
* Prefix/suffix trimming
* Whitespace removal
* JSON prettifying
* Math helpers
* Base64 encoding and decoding
* Date/time formatting
* JSON escaping
* Go template conditional logic for dynamic output formatting

---

# 1. **Text Case Adjustments**

Sprint Functions allow you to normalize strings before evaluating them. Common functions:

### **Lowercase**

```gotemplate
{{ toLower .text }}
```

### **Uppercase**

```gotemplate
{{ toUpper .text }}
```

### **Title Case**

```gotemplate
{{ toTitle .text }}
```

### **Untitle Case**

Converts to lowercase except first characters of each word revert to lowercase:

```gotemplate
{{ unTitle .text }}
```

### **Use Case**

Helpful for:

* Normalizing incoming Slack commands
* Cleaning user input for comparison
* Ensuring consistent key matching in workflow logic

---

# 2. **Prefix & Suffix Removal**

### **Trim Prefix**

Remove leading characters:

```gotemplate
{{ trimPrefix "_" .value }}
```

### **Trim Suffix**

Remove trailing characters:

```gotemplate
{{ trimSuffix "=" .value }}
```

Used to sanitize:

* Filenames
* Identifiers
* API inputs
* Strings extracted via regex

---

# 3. **Whitespace Manipulation**

Given a string with inconsistent spacing:

```gotemplate
{{ trim .value }}           # Removes leading/trailing spaces
{{ nospace .value }}        # Removes ALL spaces
```

Outputs:

* Clean versions for exact matching
* Compact strings for API queries
* Readable logs or Slack alerts

---

# 4. **Trimming Specific Characters**

When a string has unwanted characters on both ends:

```gotemplate
{{ trimAll "=" .value }}
```

Example:
Input: `==hello==` → Output: `hello`

---

# 5. **JSON Pretty Printing**

Useful when sending large JSON structures to Slack or email.

### Collapsed JSON (default)

```gotemplate
{{ .json }}
```

### Pretty JSON

```gotemplate
{{ toPrettyJson .json }}
```

This inserts:

* Indentation
* Newlines
* Human-readable formatting

---

# 6. **Math Helpers**

Given:

```json
{
  "number": 6,
  "float": 3.14159
}
```

### Basic Math

```gotemplate
{{ add .number 1 }}
{{ sub .number 2 }}
{{ div .number 3 }}
{{ mul .number 4 }}
```

### Add 1 (shortcut)

```gotemplate
{{ add1 .number }}
```

### Round a Float

```gotemplate
{{ round .float }}
```

These eliminate the need for separate math steps.

---

# 7. **Base64 Encoding & Decoding**

Critical for:

* Sending binary data to Jira/ServiceNow
* Embedding payloads
* Preparing attachments
* Packaging JSON blobs

### Encode

```gotemplate
{{ base64Encode .json }}
```

### Decode

```gotemplate
{{ base64Decode .b64 }}
```

### Inline chaining (common pattern)

```gotemplate
{{ base64Encode (toPrettyJson .json) }}
```

---

# 8. **Date & Time Formatting**

Sprint Functions follow Go’s date format model, where:

```
Mon Jan 2 15:04:05 MST 2006
```

represents formatting tokens.

### Examples:

**Current date**

```gotemplate
{{ now | date "2006-01-02" }}
```

**Current time (24h format)**

```gotemplate
{{ now | date "2006-01-02 15:04:05" }}
```

**Unix timestamp**

```gotemplate
{{ unixEpoch }}
```

**Convert date formats**

```gotemplate
{{ dateConvert "2006-01-02" "02-01-2006" .original }}
```

---

# 9. **JSON Escaping & Encoding**

Used when:

* Passing JSON into CLI tools
* Embedding JSON inside Python or Bash steps
* Preparing requests for AWS CLI or Terraform-style commands

### Escape JSON

```gotemplate
{{ escapeJson .data }}
```

### Encode As JSON String

```gotemplate
{{ toJson .object }}
```

---

# 10. **Golang Templates for Conditional Output**

One of Torq’s most powerful features.

Given data:

```json
{
  "key1": "hello",
  "key2": "",
  "key3": null
}
```

### Conditional Template

```gotemplate
{{ if .key1 }}
key1 is set to {{ .key1 }}
{{ else }}
key1 is empty
{{ end }}

{{ if .key2 }}
key2 is set to {{ .key2 }}
{{ else }}
key2 was found to be empty
{{ end }}

{{ if .key3 }}
key3 is set to {{ .key3 }}
{{ else }}
key3 was found to be empty
{{ end }}
```

### Output

```
key1 is set to hello
key2 was found to be empty
key3 was found to be empty
```

---

## Common Use Cases

* Slack/Teams alerts where missing values shouldn’t produce blank lines
* Building clean email summaries
* Dynamically generating incident reports
* Constructing modular markdown blocks
* Conditional inclusion of artifacts

---

# **Summary**

Sprint Functions & Go Templates allow you to:

* Clean and manipulate strings inline
* Format JSON for readability
* Encode/decode data without extra steps
* Perform simple math directly in expressions
* Fix whitespace and unwanted characters
* Conditionally generate dynamic output