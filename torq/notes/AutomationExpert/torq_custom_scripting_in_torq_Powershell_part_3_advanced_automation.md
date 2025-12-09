# **PowerShell in Torq — Part 3

Advanced Automation & API Handling**

This module extends the concepts from PowerShell Parts 1 and 2 and demonstrates how to:

* Pass dynamic workflow variables into a PowerShell script
* Construct an API request inside PowerShell
* Authenticate to VirusTotal using Torq-stored credentials
* Correctly output JSON from PowerShell (handling depth limits)
* Understand the difference between Torq-evaluated variables vs. PowerShell-native variables

This is essential for building **real-world automation** that enriches observables, pulls reputation data, or interacts with external APIs through native PowerShell scripting.

---

# **1. Passing Workflow Variables into PowerShell**

In the trigger, two variables are created:

* **`ip_address`** – supplied by the event
* **`vt_endpoint`** – VirusTotal base URL (e.g., `https://www.virustotal.com/api/v3/ip_addresses/`)

Inside Torq, these must be passed to the script using the correct syntax:

### Torq Variable → PowerShell Variable

```powershell
$IP = "${{ trigger.ip_address }}"
$VT = "${{ vars.vt_endpoint }}"
```

### Constructing the full VT URL

```powershell
$fullUrl = $VT + $IP
$fullUrl
```

PowerShell automatically prints the variable content (`Write-Output` not required).
Execution output confirms:

```
https://www.virustotal.com/api/v3/ip_addresses/8.8.8.8
```

---

# **2. Making the VirusTotal API Call**

A raw API call inside PowerShell uses:

```powershell
Invoke-RestMethod -Method GET -Uri $fullUrl -Headers $headers
```

Before invoking, we must attach the **Torq-stored VirusTotal API key** via an integration reference:

### Building Headers

```powershell
$token = "${{ integrations.virustotal.api_key }}"
$headers = @{ "x-apikey" = $token }
```

### Full API Call

```powershell
$response = Invoke-RestMethod -Method GET -Uri $fullUrl -Headers $headers
$response | ConvertTo-Json -Depth 9
```

---

# **3. Understanding the JSON Depth Problem**

PowerShell’s default:

```powershell
ConvertTo-Json
```

outputs only **2 levels deep**.

VirusTotal responses contain deeply nested objects (attributed engines, metadata, stats).
If you do not specify a depth, you'll get **truncated JSON**, missing most fields.

Example of broken output:

```json
{
  "data": {
    "attributes": "..."  <-- truncated
  }
}
```

### Fixing JSON Output

Specify a depth high enough:

```powershell
$response | ConvertTo-Json -Depth 9
```

This resolves truncation and yields a full JSON dictionary.

---

# **4. Output After Fixing the Depth**

Sample structure:

```json
{
  "data": {
    "id": "8.8.8.8",
    "type": "ip_address",
    "attributes": {
      "last_analysis_stats": {
        "harmless": 90,
        "malicious": 5,
        "suspicious": 2,
        "undetected": 20
      },
      "as_owner": "Google LLC"
    }
  }
}
```

This output is now ready for:

* JQ extraction
* Table creation
* Markdown formatting
* Further automation logic

---

# **5. Torq vs PowerShell Variables — DO NOT CONFUSE THEM**

Torq variables:

```powershell
"${{ vars.vt_endpoint }}"
"${{ trigger.ip_address }}"
"${{ integrations.virustotal.api_key }}"
```

PowerShell variables:

```powershell
$VT
$IP
$headers
$response
```

Rules:

* Torq variables use **`${{ }}`** and are substituted **before the script runs**
* PowerShell variables use **`$`** and exist only **inside the script**
* When combining them, Torq substitution always happens first

---

# **6. Summary of the Automation Pattern**

This module teaches:

### ✔ How to pass dynamic inputs to PowerShell

### ✔ How to build complete URLs inside PowerShell

### ✔ How to pull secrets (API keys) from Torq integrations

### ✔ How to authenticate with headers

### ✔ How to fix and output fully valid JSON

### ✔ How to structure data for downstream steps

This is the **standard pattern** for:

* VirusTotal enrichment
* HTTP API orchestration
* Indicator reputation checks
* PowerShell-based integrations
* Custom automation scripts

---

# **Cross-Platform Equivalents (SOAR Mapping)**

## **Splunk SOAR**

```python
import json
ip = container.get('artifact:*.cef.destinationAddress')
vt = f"https://www.virustotal.com/api/v3/ip_addresses/{ip}"
resp = requests.get(vt, headers={"x-apikey": vt_key})
parsed = resp.json()
```

## **Cortex XSOAR**

```python
!vt-ip-reputation ip=8.8.8.8
```

## **Shuffle**

Use Python block + HTTP block:

```python
requests.get(url, headers={"x-apikey": token}).json()
```

---

# **Python Equivalent (Reference)**

```python
import requests, json

VT = "https://www.virustotal.com/api/v3/ip_addresses/"
IP = "8.8.8.8"
token = VT_API_KEY

resp = requests.get(VT + IP, headers={"x-apikey": token})
print(json.dumps(resp.json(), indent=2))
```