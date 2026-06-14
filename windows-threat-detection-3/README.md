**Command & Control (C2) – Summary**

C2 lets attackers **keep remote control of a compromised system** after initial access.

### Without C2

* Attacker works directly through **RDP**
* Access ends if RDP is blocked

### With C2

* Malware creates a **persistent connection** to attacker’s server
* Attacker can send commands anytime

### Common C2 Setup

```text
Phishing / USB
↓
Initial Access
↓
Run Attachment
↓
Download C2 Malware
↓
Connect to C2 Server
↓
Remote Control
```

### Detection

* Check **Sysmon logs**
* Look for:

  * Suspicious process → network connection
  * Download → execution chain
  * Unknown processes in Temp folders
  * Persistent outbound connections (beaconing)

**Key idea:** C2 = attacker’s remote control channel to the victim machine.
-----------
**Persistence – Summary**

Persistence allows attackers to **keep long-term access** after initial compromise.

### Common Persistence Methods

* Reuse exposed services (RDP, vulnerable apps)
* Create **backdoor/web shell**
* Create a **new user account**
* Add user to **privileged groups**
* Reset passwords of old accounts

### Useful Event IDs

* **4720** → User account created
* **4732** → User added to privileged group
* **4724** → Password reset

### Detection

Investigate:

* **Who created/modified the account**
* **Source IP + login time**
* **Related events in same session**
* Unexpected admin or RDP privileges

### Typical Persistence Flow

```text
Initial Access
↓
Create User
↓
Add to Administrators / RDP Users
↓
Maintain Access
```
---------
**Malware Persistence – Summary**

Attackers make malware **survive reboot** and maintain **C2 access**.

### Common Persistence Methods

**1. Windows Service**

* Starts automatically after boot
* Created with `sc.exe`
* Detection:

  * **Sysmon ID 1** → `sc.exe`
  * **Security ID 4697** → service created
  * Suspicious child of `services.exe`

**2. Scheduled Task**

* Runs automatically on trigger/startup
* Created with `schtasks.exe`
* Detection:

  * **Sysmon ID 1** → `schtasks.exe /create`
  * **Security ID 4698** → task created
  * Suspicious child of `svchost.exe -s Schedule`

### SOC Investigation

Check:

* Service/task name
* Executed file path
* Parent process
* Startup behavior
* Unexpected binaries

### Typical Flow

```text
Initial Access
↓
Create Service / Task
↓
Reboot
↓
Malware Starts
↓
Reconnect to C2
```
---------
**Run Keys & Startup Persistence – Summary**

Attackers use **user-level persistence** that runs when a user logs in.

### Methods

**1. Startup Folder**

* Malware placed in Startup directory
* Runs on user login
* Detection:

  * **Sysmon Event ID 11** → file created in Startup folder
* Path:

  * `AppData\...\Startup\`

**2. Run Registry Keys**

* Malware added to registry Run key
* Executes on login
* Detection:

  * **Sysmon Event ID 13** → registry change
* Keys:

  * `HKCU\...\Run`
  * `HKLM\...\Run`

### SOC Notes

* Startup items usually launch via **explorer.exe**
* Run keys are harder to notice than Startup folder
* Both are commonly abused for stealth persistence

### Flow

```text id="n8p4kd"
Login
↓
Startup Folder OR Run Key
↓
Malware executes
↓
C2 / persistence maintained
```
------
**Need for Persistence – Summary**

Attackers keep access after breach instead of leaving immediately to:

* Build **botnets** (mining, C2, data theft)
* Conduct **long-term espionage**
* Use victim as a **network entry point**

### Why it matters

* Corporate Windows networks (Active Directory) are high-value targets
* Main risk: **ransomware attacks** causing major disruption and data loss

### Key idea

All major attacks start small, so the best point to stop them is:

* **Initial Access** 🚨
