### Identification and Scoping — Short Notes

The **Identification phase** focuses on detecting, confirming, and reporting potential security incidents as quickly as possible.

### People, Process, and Technology

* **People:** Employees and analysts recognize, report, and investigate suspicious activity.
* **Process:** Clear reporting, communication, escalation, and investigation procedures.
* **Technology:** Tools such as SIEM, EDR, IDS/IPS, and antivirus generate alerts and detect threats.

### Security Alerts

Alerts or event notifications may indicate suspicious activity. Analysts must evaluate:

* Alert type and severity
* Affected users and systems
* Whether the alert is a real incident
* Who must be notified or involved

Tools support detection, but skilled analysts are required to interpret alerts correctly.

### Learning and Vigilance

Organizations should provide regular security training and encourage all employees to report anomalies. Management must support cybersecurity policies, awareness, and incident-reporting procedures.

### Transition to Scoping

After confirming an incident, analysts determine its **scope**, including:

* Which systems and accounts are affected
* What data may be exposed
* How far the attacker progressed
* The operational and business impact

**Key point:** Fast identification and accurate scoping reduce damage and improve containment and recovery.
---
### Incident Scoping — Short Notes

**Scoping** determines the full extent of a confirmed incident:

* Affected systems and users
* Data at risk
* Attacker activity
* Potential business impact

It helps responders prioritize containment and mitigation.

### Asset Inventory

A central list of organizational assets containing:

* Asset name and type
* IP address
* Operating system
* Owner

It helps identify affected systems, their importance, and responsible personnel.

### Spreadsheet of Doom — SoD

The **SoD** is a centralized and enriched list of Indicators of Compromise, such as:

* Malicious IP addresses
* Domains and URLs
* Email addresses
* File hashes

Each entry may include its threat type, source, and related incident ticket. It helps correlate evidence, identify recurring threats, and share information between responders.

**Key point:** The Asset Inventory identifies what is affected, while the SoD provides context about the threats involved.

### Practice Answers

* Asset owner needing updated endpoint protection: **Derick Marshall**
* Credential-phishing domain: `b24b-158-62-19-6.ngrok-free.app`
* Domain to add to the SoD: `kennaroads.buzz`
---
### Identification and Scoping Feedback Loop — Short Notes

Identification and scoping are a continuous **feedback loop**, not a one-time linear process. New evidence can expand or refine the incident scope.

### Feedback Loop Steps

1. **Event notification:** A suspicious issue is reported and triggers incident response.
2. **Documentation:** Record the incident, affected systems, threats, and current findings.
3. **Evidence collection:** Gather logs, emails, network traffic, files, and other evidence.
4. **Artefact identification:** Extract useful indicators such as IPs, domains, hashes, and email addresses.
5. **Pivot point discovery:** Use identified artefacts to search for related activity, systems, or victims.
6. **Repeat:** Document new findings and continue expanding or refining the scope.

### Intelligence-Driven Investigation

Investigations become stronger by combining:

* Current incident evidence
* Previous incident data
* Correlated logs
* Threat intelligence
* Analytics and machine learning

This improves detection, response speed, information sharing, and regulatory compliance.

### Practice Findings

* Email-spoofing domain: `emkei.cz`
* Other phishing recipient: `alexander.swift@swiftspend.finance`
* Additional pivot-point IoC: `sales.tal0nix@gmail.com`
* Compromised password: `Passw0rd!`

**Key point:** Every new artefact can become a pivot point that reveals additional affected users, systems, and threats.
---
### Identification and Scoping — Final Summary

The **Identification and Scoping phase** determines whether suspicious activity is a real incident and how far it has spread.

Its success depends on:

* **People:** Recognize, report, and investigate suspicious activity.
* **Processes:** Define reporting, escalation, documentation, and response procedures.
* **Technology:** Use SIEM, EDR, IDS/IPS, antivirus, and other monitoring tools.

Analysts must understand alerts, collect evidence, identify IoCs, determine affected systems and data, and notify the correct stakeholders.

**Key point:** Strong collaboration, continuous training, clear procedures, and effective security tools improve incident detection and reduce potential damage.

**Next phase:** Intelligence Creation and Containment.

