# **PowerShell in Torq — Part 2

Managing VMs, JSON Outputs, and Remote Execution via SSH**

This module demonstrates how to run **PowerShell commands on a Windows host** from Torq by using an **SSH connection**.
This avoids PowerShell Core limitations inside Torq’s Linux container and lets you access:

* Hyper-V cmdlets
* Windows-specific modules
* Local system state
* Any PowerShell command normally available on a Windows Server

---

# **1. Overview: Why SSH for PowerShell?**

Torq executes PowerShell **inside a Linux container (PowerShell Core 7.2)**.
This environment **cannot run Windows-only modules**, such as:

* Hyper-V (`Get-VM`, `New-VM`)
* Active Directory cmdlets
* System-level management modules
* Exchange Management Shell

By using **SSH to a Windows Server**, you bypass these limitations:

### Advantages of SSH → Windows:

* You run **full Windows PowerShell** or **PowerShell 7** on the host
* You gain access to **all modules installed on that server**
* Hyper-V and VM commands run natively
* Torq only needs SSH credentials, not modules

---

# **2. Running a PowerShell VM Command via SSH**

Torq’s **SSH step** allows:

* Hostname / IP
* Username
* Password or Key
* Script content (PowerShell)

Example script executed remotely:

```powershell
Get-VM | Select-Object Name, State, Status | ConvertTo-Json -Depth 10
```

### Torq SSH Step Fields:

* **Hostname:** `windows-hyperv.internal`
* **Username:** `Administrator`
* **Password:** Stored in Vault
* **Command:** PowerShell command above

### Execution Output

Torq receives:

* A **JSON object**, but wrapped as a **string**
* It will appear messy and not immediately structured

This is normal.
PowerShell sent JSON, but SSH output is “stringified” inside Torq.

---

# **3. Decoding the JSON using Torq Utilities**

Use the **“JSON Parse”** step to fix the structure.

Input:
`$.steps.ssh_vm_details.output`

This converts the raw JSON string into a real JSON object Torq can handle.

Example parsed structure:

```json
[
  {
    "Name": "Win10-Test",
    "State": "Running",
    "Status": "Operating normally"
  }
]
```

Now you can:

* Filter
* Loop
* Create tables
* Send alerts
* Build enrichment data

---

# **4. Filtering VM Details**

Extract only relevant fields:

* Name
* State
* Status

Using JQ:

```jq
map({name: .Name, state: .State, status: .Status})
```

Sample output:

```json
[
  {
    "name": "Win10-Test",
    "state": "Running",
    "status": "OK"
  }
]
```

---

# **5. Creating an ASCII / Markdown Table (Torq Output)**

Use **Create ASCII Table** with Markdown enabled.

Headers:

```
name
state
status
```

The table appears like:

```
| name        | state    | status            |
|-------------|----------|-------------------|
| Win10-Test  | Running  | Operating normally|
```

---

# **6. Sending Results to Slack**

Use **Slack Snippet** or regular Slack message.

Example snippet content:

```
Hyper-V VM Status Summary
```

Then attach the ASCII/Markdown table.

This allows:

* SOC notifications
* Scheduled VM health checks
* Daily infrastructure reports

---

# **7. Equivalent Concepts in Other SOAR Platforms**

---

## **Splunk SOAR**

Use the **SSH App**:

* Run PowerShell commands on a Windows target
* Capture stdout
* Use `json.loads()` to parse the PS output

Equivalent playbook block:

```python
import json
vm_data = json.loads(action_result.get_output("stdout"))
```

---

## **Cortex XSOAR**

Use the **Remote PowerShell / SSH integration**:

* Runs Windows PowerShell directly on the host
* Parses output if JSON format

Example:

```powershell
Get-VM | ConvertTo-Json
```

Return this as the script output.

---

## **Shuffle**

Shuffle does not natively support SSH + PowerShell, but you can:

* Use the **SSH app**
* Point it at a Windows host running PowerShell 7
* Output JSON and parse in Python

---

# **8. Python Equivalent (Cross-Reference)**

If doing hypervisor automation via Python:

```python
import subprocess, json

cmd = [
    "ssh", "administrator@hyperv.internal",
    "powershell.exe Get-VM | ConvertTo-Json"
]

raw = subprocess.check_output(cmd)
parsed = json.loads(raw.decode())

print(json.dumps(parsed, indent=2))
```

This mirrors the Torq SSH → PowerShell method.

---

# **9. Summary**

This module taught how to:

* Use SSH to run true Windows PowerShell instead of Linux PowerShell Core
* Execute Hyper-V commands (`Get-VM`, `New-VM`)
* Convert SSH output to JSON using Torq’s parser
* Filter VM details using JQ
* Build an ASCII or markdown table
* Send formatted results to Slack