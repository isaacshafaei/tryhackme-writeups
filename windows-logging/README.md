**Logging Overview**

* OS records events (program start, file creation, login) as **logs** → helps SOC with:

  * **Incident Response** – identify when/how attacks happened
  * **Threat Hunting** – find malicious activity
  * **Alerting & Triage** – create detections

**Windows Logs (EVTX)**

* Stored in: `C:\Windows\System32\winevt\Logs`
* Binary format → viewed with **Event Viewer** (`eventvwr`)
* Each EVTX = a log category:

  * **Application Logs** → app activity (IIS, SQL)
  * **Security Logs** → logins, processes, user actions

**Event Viewer Key Fields**

* **Keywords** → success/failure
* **Date & Time** → event timestamp
* **Event ID** → unique event type (e.g., failed login = 4625)
* **Details** → full event data (text/XML)
* **Filters** → search/filter logs

**Note:** Thousands of Event IDs exist; only some are logged by default.
---
**Windows Security Logs – Quick Notes**

### Key Security Event IDs

* **4624 – Successful Logon**

  * Detect suspicious RDP/network logins
  * Logged on target machine
  * Limitation: very noisy (many events)

* **4625 – Failed Logon**

  * Detect brute force, password spraying, scanning
  * Logged on target machine
  * Limitation: can be misleading/inconsistent

### Important Fields (4624 / 4625)

* **Logon ID** → unique session ID (track activity)
* **Logon Type** → login method
* **Username**
* **Source IP**
* **Workstation Name**

### Detect RDP Brute Force (4625)

* Filter **Event ID = 4625**
* Focus on **Logon Type 3 (Network)** and **10 (RDP)**
* Red flags:

  * Many usernames → password spraying
  * Many failures on one account → brute force
  * Unusual workstation name
  * Unexpected source IP

### Analyze Successful RDP Logins (4624)

* Filter **Event ID = 4624**
* Focus on **Logon Type 10**
* If **NLA enabled**, check preceding **Type 3** event
* Red flags:

  * Previous brute-force activity
  * Suspicious IP/hostname
* Save **Logon ID** → use it to trace attacker actions after login.

---
**User Management Events – Summary**

Attackers often manipulate accounts for **persistence, privilege escalation, or disruption**.

### Key Event IDs

* **4720 / 4722 / 4738** → User **created / enabled / modified** → possible backdoor account
* **4725 / 4726** → User **disabled / deleted** → may target admin/SOC accounts
* **4723 / 4724** → **Password changed / reset** → attacker gains account access
* **4732 / 4733** → User **added / removed from group** → privilege escalation (e.g., Administrators)

### Event Structure

* **Subject** → who performed action
* **Object** → target user/account
* **Details** → exact changes made
* **Logon ID** → link event to previous **4624 login**

### Red Flags

* Unknown account changes
* Actions outside working hours
* Unusual usernames
* Users added to privileged groups

**Investigation:** Check suspicious events → copy **Logon ID** → find matching **4624 login** → trace attacker activity.
---
**Process Monitoring – Summary**

Process logs help identify **how a system was compromised**, not just who logged in.

### Process Creation Logs

* **4688 (Security Log)** → Process creation + command line + parent process

  * Disabled by default
* **Sysmon Event ID 1** → Enhanced process monitoring (hash, signature, metadata)

  * Requires Sysmon installation (**preferred**)

### Key Sysmon Event Fields

* **Process Info** → PID, path, command line
* **Parent Info** → parent process (build attack chain)
* **Binary Info** → hash, signature, metadata
* **User Context** → user + **Logon ID**

### Red Flags

* Process runs from unusual paths (`C:\Temp`, `C:\Users\Public`)
* Random/suspicious names (`aa.exe`)
* File hash matches malware
* Unexpected parent process (e.g., Notepad → CMD)

### Investigation Flow

1. Filter **Sysmon Event ID 1**
2. Check process + parent details
3. Trace parent processes upward (process tree)
4. Use **Logon ID** to correlate with Security logs and reconstruct the attack chain.
----
salam
