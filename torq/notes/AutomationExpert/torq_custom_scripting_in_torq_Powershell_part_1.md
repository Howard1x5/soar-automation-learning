# **PowerShell in Torq Workflows

(Environment, Modules, JSON Output, and Dependency Handling)**

Torq supports **PowerShell Core**, not Windows PowerShell.
This means:

* PowerShell executes inside a **Linux container**
* Only modules compatible with **PowerShell Core 7+** can run
* You must format output using **ConvertTo-Json** for downstream steps

---

# **1. PowerShell Core Environment in Torq**

Torq uses **PowerShell 7.2 LTS**.

To check version:

```powershell
$PSVersionTable | ConvertTo-Json
```

Example output:

```json
{
  "PSVersion": "7.2.x",
  "PSEdition": "Core",
  "OS": "Linux"
}
```

Key implications:

* No Windows-only cmdlets (e.g., Get-EventLog, Get-WmiObject)
* Cross-platform cmdlets work
* You must convert all output to JSON

---

# **2. Listing Available Modules in Torq**

Base container includes a limited module set.

Get list of installed modules:

```powershell
Get-Module -ListAvailable | Select-Object Name, Version | ConvertTo-Json
```

Downstream jq filtering (inside Torq) to simplify:

```bash
jq '.[].Name'
```

Example result:

```
Microsoft.PowerShell.Management
Microsoft.PowerShell.Utility
ThreadJob
```

---

# **3. Adding Modules to Torq PowerShell Steps**

PowerShell modules must be:

* Cross-platform
* Installable via PowerShell Gallery
* Usable inside a Linux container

Inside the **script step properties**, add modules under:

**Properties → Optional Fields → Modules**

Example:

```
Carbon.Core
Powershell.Ja
```

Then import them inside the script:

```powershell
Import-Module Carbon.Core
Import-Module PowerShell.Ja
```

Verify installation:

```powershell
Get-Module -ListAvailable Carbon.Core | ConvertTo-Json
```

---

# **4. Why ConvertTo-Json Is Mandatory in Torq PowerShell**

PowerShell normally outputs objects in table format:

```
Name   Version   Path
----   -------   ----
...
```

That breaks Torq processing.

Always end your scripts with:

```powershell
| ConvertTo-Json -Depth 10
```

Without this, Torq cannot convert PowerShell output to structured JSON for downstream steps.

---

# **5. SOAR Platform Equivalents**

---

## **Splunk SOAR**

Use **PowerShell App** or create a custom app container.

Example:

```powershell
$modules = Get-Module -ListAvailable
return $modules
```

Splunk SOAR handles PS output as JSON automatically if using `phantom.debug()` + dictionary return.

---

## **XSOAR (Cortex XSOAR)**

PowerShell scripts run inside Docker:

* Must be PowerShell Core compatible
* Must return dictionaries → XSOAR auto-converts to JSON

Example:

```powershell
$version = $PSVersionTable
Write-Output ($version | ConvertTo-Json)
```

XSOAR requires a Docker image with PowerShell installed (`mcr.microsoft.com/powershell`).

---

## **Shuffle**

Shuffle does not natively run PowerShell.
You must:

* Use a custom app container
* Or wrap PowerShell inside a Python step calling `subprocess`
  (rare, but possible)

Example workaround:

```python
import subprocess, json
output = subprocess.check_output(["pwsh", "-c", "Get-Module | ConvertTo-Json"])
print(output.decode())
```

---

# **6. Python Equivalent (For Cross-Language Parity)**

To replicate module enumeration:

```python
import json
import subprocess

result = subprocess.check_output(["pwsh", "-c", "Get-Module -ListAvailable | ConvertTo-Json"])
modules = json.loads(result)
print(json.dumps({"modules": modules}))
```

---

# **7. Common PowerShell Patterns in Torq**

### **A. Dynamic variable injection**

```powershell
$ip = "${$.event.ip_address}"
```

### **B. Always convert output**

```powershell
$object | ConvertTo-Json -Depth 10
```

### **C. Test module availability**

```powershell
Get-Module Carbon.Core -ListAvailable
```

### **D. Fail gracefully**

```powershell
try {
    # code
} catch {
    Write-Error $_ | ConvertTo-Json
}
```

---

# **8. Summary**

This module taught:

* Torq uses **PowerShell Core 7.2 in Linux**, not Windows
* How to verify version & environment
* How to enumerate and install modules
* How to format output into valid JSON
* How to use PowerShell in SOAR environments (Torq, SOAR, XSOAR, Shuffle)