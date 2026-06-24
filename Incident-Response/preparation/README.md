### Incident Response Preparation — Short Notes

The **Preparation phase** ensures the organization is ready to detect, handle, and recover from incidents.

### Key Areas

* **People:** Train employees to recognize phishing, social engineering, and suspicious activity.
* **CSIRT:** Create a **Cyber Security Incident Response Team** with technical, business, legal, and public-relations experts.
* **Access control:** Give CSIRT members controlled permissions required during investigations.
* **Training:** Conduct phishing simulations, awareness sessions, forensic exercises, and log-analysis training.
* **Documentation:** Record investigation details for evidence, mitigation, and lessons learned.
* **Policies:** Define monitoring authority, acceptable use, incident-handling rules, and privacy requirements.
* **Communication plan:** Define contacts, escalation procedures, and when to notify management, law enforcement, media, or third parties.
* **Chain of custody:** Documents who collected, accessed, transferred, or handled evidence.
* **Response procedures:** Management-approved steps for responding quickly and restoring normal operations.

**Practice answers:**

* Team handling cyber incidents: **Cyber Security Incident Response Team — CSIRT**
* Document tracking evidence handling: **Chain of custody document**
---
### Technical Preparation for Incident Response — Short Notes

Incident response requires tools, systems, and infrastructure to prevent, detect, investigate, and contain attacks.

### Asset Inventory

Maintain updated **hardware and software inventories**, including:

* Asset name and type
* Operating system
* IP address
* Business importance

Asset classification helps prioritize protection of critical infrastructure, data, intellectual property, and services.

### Technical Monitoring

Map all systems, networks, cloud platforms, and applications. Deploy:

* Antivirus and EDR
* DLP
* IDS/IPS
* Centralized log collection
* Firewalls and network segmentation
* DMZs and subnets

Tools such as **TheHive** can track incidents and investigation data.

### Investigation Capabilities

The CSIRT should have:

* Disk and memory imaging tools
* Secure evidence storage
* Sandboxes for malware analysis
* Remote script and software execution capabilities
* Network monitoring tools

### Incident-Handling Jump Bag

A **jump bag** contains essential response tools, such as:

* Evidence storage drives
* FTK Imager, EnCase, or Sleuth Kit
* Network taps
* USB, SATA cables, and adapters
* Computer repair tools
* Incident forms and communication playbooks

**Practice answer:** A kit containing incident-handling tools is called a **Jump bag**.
---

## Visibility and Windows Logging — Study Note

### What is visibility?

Visibility means collecting and monitoring activity across systems so the SOC can detect, investigate, and respond to suspicious behaviour.

### Why is it important?

It provides evidence about:

* Who performed an action
* What happened
* When and where it happened
* Emerging threats and vulnerabilities
* Compliance and incident-response investigations

### Main log types

* **Event logs:** Logins, processes, network activity
* **Audit logs:** Successful and failed security actions
* **Error logs:** Service or system failures
* **Debug logs:** Troubleshooting information

### Common log sources

* Network devices: routers, switches, packet captures
* Perimeter devices: firewalls, proxies, VPNs
* Operating systems: Windows and Linux logs
* Applications: web servers, databases, cloud services

A **SIEM** centralises and analyses these logs.

---

## What we did in the lab

### 1. Enabled Windows Event Logging

Registry location:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog
```

The `Start` DWORD value controls startup:

```text
2 = Automatic
4 = Disabled
```

Important: `Start` is a value inside the `EventLog` key, not a subfolder.

Check it:

```powershell
reg query "HKLM\SYSTEM\CurrentControlSet\Services\EventLog" /v Start
```

Check service status:

```powershell
Get-Service EventLog
```

After changing the value, reboot if the service does not start.

### 2. Simulated ransomware activity

```powershell
Invoke-AtomicTest T1486 -ShowDetailsBrief
Invoke-AtomicTest T1486-5
```

This Atomic Red Team test creates a simulated ransomware note.

### 3. Verified the activity in Sysmon

Event Viewer path:

```text
Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

Use **Find** to search for the test file or activity.

### Important answers

| Question                                    | Answer           |
| ------------------------------------------- | ---------------- |
| Sysmon File Create Event ID                 | **11**           |
| Default Software Restriction security level | **Unrestricted** |
| Audit logon events setting                  | **Failure**      |

### Key conclusion

Without logging, the SOC has little evidence for investigation. Enabling Windows Event Logs, Sysmon, audit policies, and central SIEM collection provides the visibility required to detect and investigate incidents.
---
### Preparation Phase — Final Summary

The **Preparation phase** is the foundation of incident response. It focuses on:

* Training employees and the CSIRT
* Defining policies and response procedures
* Maintaining asset inventories
* Deploying monitoring and security tools
* Collecting logs and maintaining visibility
* Preparing forensic tools and documentation

Frameworks such as the **NIST Computer Security Incident Handling Guide** provide best practices for building an effective incident response process.

After preparation, the next stages are **Identification and Scoping**, where analysts detect incidents and determine their impact.

**Key point:** Prepare people, policies, systems, and networks before an incident occurs.

