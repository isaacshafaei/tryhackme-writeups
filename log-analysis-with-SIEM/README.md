# SIEM Log Sources & Concepts
Collection → Parsing → Normalization → Enrichment → Storage → Correlation → Alerting

## 📡 Types of Log Sources

* **Host-Based:** Logs from individual workstations and servers. Used to monitor system-level behavior (e.g., malicious script execution, process creation).
* **Network-Based:** Logs from firewalls, routers, and IDS/IPS. Provides visibility into network traffic and inter-device communication.
* **Web-Based:** Logs from web applications. Essential for monitoring web-based attacks and vulnerabilities often used for initial access.
![host](host.svg)
![network](network.svg)
![web](web.svg)
## ⏱️ Time Pitfalls

Logs arrive from devices across different time zones. The timestamp in the SIEM may be standardized (e.g., UTC) and differ from your local timezone. Always verify timezone settings to build accurate incident timelines.

## 🔄 Log Normalisation

Different systems send logs in entirely different formats (JSON, XML, plain text) and naming conventions. **Normalisation** is the process of converting all these disparate logs into a single, consistent structure within the SIEM, allowing analysts to search and correlate data efficiently.

---

### Knowledge Check Answers

* **What is the process of converting logs from different formats into a single format for easier analysis in a SIEM?**
`Normalisation`
* **Which log source type can be used to detect the execution of a malicious script?**
`Host-Based`
------------
### SIEM Windows Log Analysis — Short Notes

Windows monitoring mainly uses two sources:

**1. Sysmon** — installed and configured separately. It provides detailed visibility into:

* Process execution — **Event ID 1**
* Network connections — **Event ID 3**
* Process injection
* Registry changes
* File creation

Example: Detect encoded PowerShell execution:

```spl
index=winenv EventCode=1 *powershell* AND *EncodedCommand*
```

Example: Investigate network connections:

```spl
index=winenv EventCode=3 ComputerName=WINHOST05
```

**2. Windows Event Logs** — built into Windows and contain over 200 log channels.

**Security logs** monitor authentication, account changes, process execution, file access, policy changes, and log clearing.

* **4720:** User account created
* **4722:** User account enabled

**System logs** monitor operating-system services and errors. They are useful for detecting persistence and privilege escalation.

* **7045:** New service created
* **7036:** Service started or stopped

**Key point:** Combining Sysmon and Windows Event Logs gives analysts better visibility into process activity, network connections, account persistence, and malicious services.
---


