## Summary Note: Linux Brute Force Investigation

SOC analyst receives an alert for possible brute force activity on a Linux host.

**Alert details:**

* **Alert:** Brute Force Activity Detection
* **Time:** 17/09/2025 09:00:21 AM
* **Target host:** `tryhackme-2404`
* **Source IP:** `10.10.242.248`
* **Splunk index:** `linux-alert`

---

### Initial Observations

* Source IP `10.10.242.248` is a **local/internal IP**.
* This may mean the attacker is already inside the organization network.
* Activity happened around **9 AM**, which is normal working time.
* Hostname alone does not give enough asset context.

---

### Splunk Investigation

Search for SSH login activity:

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248 
| search "Accepted password for" OR "Failed password for" OR "Invalid user"
| sort + _time
```

Findings:

* Many events from the same IP.
* Several attempts against **invalid/non-existent users**.
* This suggests account enumeration.

---

### Brute Force Confirmation

Search login attempts per user:

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| stats count values(src_ip) as src_ip by username
```

Key finding:

* User `john.smith` had **503 login attempts**.
* This clearly indicates brute force activity.

---

### Successful Login Check

Search for successful access:

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| stats count values(action) values(src_ip) as src_ip by username
```

Finding:

* Successful login detected for `john.smith`.
* The brute force attack was **successful**.
* Classification: **True Positive**.

---

### Escalation

As SOC Level 1, the case should be escalated to:

* **SOC Level 2**
* Possibly **Incident Response team**

Reason:

* Confirmed brute force attack.
* Successful access to Linux host.
* Possible internal compromise due to local source IP.

---

### Key Investigation Questions

* Why did the attacker use a local IP?
* Is the attacker already inside the network?
* How long has the attacker had internal access?
* How did they get valid usernames?
* What actions happened after login?
* Was privilege escalation performed?
* Was persistence created?

---

### Answers

* Failed login attempts for `john.smith`: **500**
`index="linux-alert" sourcetype="linux_secure" " john.smith" AND "failed"`
* Brute force duration: **5 minutes**
```
index="linux-alert" sourcetype="linux_secure" 10.10.242.248 "Failed password for"
| rex field=_raw "Failed password for(?: invalid user)? (?<username>\S+)"
| stats earliest(_time) as start latest(_time) as end count by username
| eval duration_minutes=round((end-start)/60,0)
| table username count duration_minutes
| sort - count
```
* Privilege escalation account: **root**
`index="linux-alert" sourcetype="linux_secure" ("sudo" OR "su:" OR "session opened for user") "john.smith"`
* Persistence account created: **system-utm**
`index="linux-alert" sourcetype="linux_secure" ("useradd" OR "adduser" OR "new user")`
---
## Summary Note: Windows Scheduled Task Persistence Investigation

### Scenario

SOC Level 1 receives an alert about a suspicious scheduled task created on a Windows host.

**Alert details:**

* **Alert:** Potential Task Scheduler Persistence Identified
* **Time:** 30/08/2025 10:06:07 AM
* **Host:** `WIN-H015`
* **User:** `oliver.thompson`
* **Task name:** `AssessmentTaskOne`
* **Splunk index:** `win-alert`

---

### Initial Context

Before searching in SIEM, check:

* **Host type:** `WIN-H015` likely indicates a workstation.
* **User role:** `oliver.thompson` is a **System Engineer**.
* This role may have more technical activity than a normal business user, but scheduled task creation still needs validation.

---

### SIEM Investigation

Search for the scheduled task creation event:

```spl
index="win-alert" EventCode=4698 AssessmentTaskOne
| table _time EventCode user_name host Task_Name Message
```

**Event ID 4698** means:

> A scheduled task was created.

Finding:

* Only one event was found for `AssessmentTaskOne`.
* Activity appears isolated to one host.

---

### Task Behavior Analysis

The task runs daily on a user workstation, which is suspicious.

From the task message:

* Uses `certutil` to download `rv.exe`
* Downloads from the `tryhackme` domain
* Saves it in the Temp folder as:

```text
DataCollector.exe
```

* Executes it using PowerShell:

```powershell
Start-Process
```

* Runs under the user:

```text
oliver.thompson
```

---

### Conclusion

This is suspicious because:

* Scheduled task provides persistence.
* `certutil` is used to download a payload.
* Payload is saved in Temp with a disguised name.
* PowerShell is used to execute it.
* Activity occurs on a workstation.

Classification:

```text
True Positive
```

Escalate to:

* **SOC Level 2**
* Possibly **Incident Response**

---

### Open Investigation Questions

SOC L2 should investigate:

* How was the task created?
* How did the attacker access `WIN-H015`?
* How was `oliver.thompson` compromised?
* What payload did `DataCollector.exe` execute?
* Was lateral movement performed?

---

### Answers

* Process ID that created the malicious task: **5816**
```
index="win-alert" EventCode=4698 AssessmentTaskOne
| table _time EventCode user_name host Task_Name Message
```
* Parent process name: **cmd.exe**
`index="win-alert" ProcessId=4128`
* Local group enumerated: **Administrators**
```
index="win-alert" oliver.thompson (EventCode=4799 OR "net group" OR "net localgroup" OR "Get-LocalGroup")
| table _time EventCode CommandLine TargetUserName Group_Name
```
* Attacker source workstation: **DEV-QA-SERVER**
```
index="win-alert" oliver.thompson EventCode=4624
| table _time Workstation_Name WorkstationName Source_Network_Address LogonType
```
---
## Summary Note: Web Shell Upload Investigation

### Scenario

SOC Level 1 receives an alert about possible web shell activity on a vulnerable web server.

**Alert details:**

* **Alert:** Potential Web Shell Upload Detected
* **Time:** 14/09/2025 09:31:51 AM
* **Resource:** `http://web.trywinme.thm`
* **Suspicious IP:** `171.251.232.40`
* **Splunk index/query:** `index=web-alert`

---

### Initial Context

The suspicious IP `171.251.232.40` was checked in AbuseIPDB.

Finding:

* The IP had been reported as malicious **3000+ times**.
* This gives strong threat intelligence support that the activity is suspicious.

---

### SIEM Investigation

Initial search:

```spl
index=web-alert 171.251.232.40
| table _time clientip useragent uri_path method status 
| sort + _time
```

Finding:

* Large number of requests from the same IP.
* User-Agent was:

```text
Mozilla/5.0 (Hydra)
```

* Hydra was used against:

```text
wp-login.php
```

This indicates brute force activity against WordPress login.

---

### Web Shell Investigation

Hydra traffic was excluded:

```spl
index=web-alert 171.251.232.40 useragent!="Mozilla/5.0 (Hydra)" 
| table _time clientip useragent uri_path referer referer_domain method status
```

Suspicious finding:

* POST request to:

```text
admin-ajax.php
```

* Referer pointed to:

```text
theme-editor.php?file=b374k.php
```

This is highly suspicious because `b374k.php` is a known web shell name.

---

### Web Shell Activity

Search for `b374k.php`:

```spl
index=web-alert 171.251.232.40 b374k.php
| table _time clientip useragent uri_path referer referer_domain method status
| sort + _time
```

Findings:

* Attacker accessed the suspected web shell file.
* There were **4 successful POST requests** through the web shell.
* Logs did not show how the file was originally uploaded.

---

### Conclusion

This activity is malicious because:

* Source IP has strong malicious reputation.
* Hydra was used for brute force against WordPress.
* Suspicious interaction with `b374k.php`.
* `b374k.php` is a known web shell.
* Successful POST requests were made through the shell.

Classification:

```text
True Positive
```

Escalate to:

* **SOC Level 2**
* **Incident Response team**

---

### Open Investigation Questions

SOC L2 should investigate:

* Was the Hydra brute force successful?
* How was `b374k.php` uploaded?
* What commands did the attacker execute?
* Was data accessed, modified, or exfiltrated?
* Are there other web shells or backdoors?

---

### Answers

* Hydra brute force start time: **2025-09-14 21:20:27**
```
index=web-alert 171.251.232.40 "wp-login.php"
| search useragent="Mozilla/5.0 (Hydra)"
| sort + _time 
| table _time clientip uri_path method status useragent
```
* Web shell User-Agent: **Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36**
```
index=web-alert 171.251.232.40 b374k.php
| table _time uri_path method useragent referer status
| sort + _time
```
* Number of web shell requests: **4**
```
index=web-alert 171.251.232.40 b374k.php
| table _time method uri_path status
| sort + _time
```
