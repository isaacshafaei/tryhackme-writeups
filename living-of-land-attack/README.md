# Living Off the Land (LotL)

## The Core Concept
Attackers weaponize built-in, trusted OS tools to execute code, establish persistence, and move laterally. This eliminates the need to drop obvious malware binaries, reduces noise, and blends seamlessly with normal admin activity.

## Commonly Abused Windows Binaries
* **PowerShell:** In-memory scripting and remote payload downloads.
* **WMIC / WMI:** Local/remote command execution and system state queries.
* **Certutil:** Fetching external files and encoding/decoding payloads.
* **Mshta:** Executing HTA content or inline scripts.
* **Rundll32:** Invoking DLL exports.
* **Schtasks:** Scheduling execution for persistence.
* **Sysinternals Suite:** Trusted admin tools like `PsExec` (remote execution) and `Autoruns` (persistence manipulation).

## Public Threat Intelligence Repositories
* **Windows:** LOLBAS (Living Off The Land Binaries and Scripts)
* **Unix/Linux:** GTFOBins

## Defense Strategy
* **Application Control:** Implement AppLocker or Windows Defender Application Control (WDAC) to strictly define permitted executables/scripts.
* **Least Privilege:** Restrict access to management utilities to administrators only.
* **Enhanced Telemetry:** Tune logging to capture full command lines and process trees to identify anomalous behavior in benign binaries.

---

### Which public site lists Unix/Linux native binaries and how they can be abused?
`GTFOBins`

### Which Microsoft toolset includes PsExec and Autoruns, used for admin tasks and often misused by attackers?
`Sysinternals`
---
# Threat Actor LotL Operations

## Real-World Case Studies
* **APT29 (Nobelium):** Leverages **WMI event subscriptions** and **PowerShell** for fileless persistence. Payloads are read, decrypted, and executed directly from WMI properties to avoid dropping artifacts on disk.
* **BlackCat (ALPHV):** Weaponizes **PowerShell** (scripting/defense evasion), **PsExec** (lateral movement/remote execution), and **certutil** (fetching/decoding payloads).
* **Cobalt Strike Loaders (QakBot & IcedID):** Abuses trusted Windows binaries like **rundll32.exe** and **mshta.exe** to bootstrap and execute Cobalt Strike beacons directly in memory, blending in with legitimate processes.

---

### What MITRE technique ID covers WMI event subscriptions?
`T1546.003`

### Which abbreviated name refers to one of the services that C2s, like Cobalt Strike, use to start or listen for remote services?
`SMB`
---
-----------
# Windows Living Off the Land Techniques & Detections

## 1. PowerShell
*   **Purpose:** In-memory code execution, automation, and bypassing execution policy restrictions.
*   **Key Patterns:** `IEX (New-Object System.Net.WebClient).DownloadString()` downloads and runs scripts entirely in memory without hitting disk. `-EncodedCommand` obfuscates scripts using Base64 to bypass basic log filters.

## 2. WMIC (Windows Management Instrumentation Command-line)
*   **Purpose:** Querying local/remote configurations and launching processes.
*   **Key Patterns:** `process call create` spawns binaries remotely or locally. The `/hidden` flag hides the application interface from the user interface.

## 3. Certutil
*   **Purpose:** Microsoft certificate tool hijacked for file manipulation.
*   **Key Patterns:** `-urlcache -split -f` downloads raw files to a target folder. `-decode` and `-encode` allow binary files to be transported past network filters as plaintext Base64 blobs and reconstructed locally.

## 4. MSHTA
*   **Purpose:** Executes HTML Application (`.hta`) files containing VBScript or JavaScript.
*   **Key Patterns:** Passing inline string URIs (`javascript:...`) allows mshta to drop directly into `WScript.Shell` active objects and execute commands without saving an intermediate script file.

## 5. Rundll32
*   **Purpose:** Runs exported functions straight from dynamic link libraries (`.dll`).
*   **Key Patterns:** Pointing to temporary folder paths (`\Windows\Temp\`) or leveraging built-in logic like `url.dll,FileProtocolHandler` to trigger external system URLs.

## 6. Scheduled Tasks (`schtasks`)
*   **Purpose:** Maintaining persistence across reboots or escalating on strict schedules.
*   **Key Patterns:** Employs triggers like `/SC ONLOGON` or masquerades under administrative titles (`WindowsUpdate`) to blend with routine operations.

---

### Which PowerShell switch is used to download text/strings and execute them?
`IEX`

### Which WMIC keyword triggers the creation of a new process on a remote host?
`create`
