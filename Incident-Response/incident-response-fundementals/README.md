### Types of Security Incidents — Short Notes

A **security incident** is a confirmed harmful event that affects systems, networks, applications, or data.

* **Malware infection:** Malicious files or programs damage or compromise a system. Commonly delivered through email attachments, documents, or executables.
* **Security breach:** An unauthorized person gains access to confidential systems or data.
* **Data leak:** Confidential information is exposed, intentionally or accidentally through human error or misconfiguration.
* **Insider attack:** A trusted employee or internal user intentionally attacks the organization.
* **Denial of Service (DoS):** An attacker floods a service with requests, exhausting resources and making it unavailable.

**Key point:** Incident severity depends on the organization, its assets, and the business impact.

**Practice answers:**

* Compromise caused by an email attachment: **Malware infection**
* Attack targeting application availability: **Denial of Service**
----
### Incident Response Frameworks — Short Notes

Incident response frameworks provide a structured process for handling security incidents. The two common frameworks are **SANS** and **NIST**.

### SANS Framework — PICERL

* **Preparation:** Build the IR team, plans, tools, training, and security controls.
* **Identification:** Detect and confirm suspicious activity or incidents.
* **Containment:** Limit damage by isolating systems or disabling compromised accounts.
* **Eradication:** Remove malware, attacker access, and other threats.
* **Recovery:** Restore, rebuild, test, and return systems to operation.
* **Lessons Learned:** Review the incident, document gaps, and improve future response.

### NIST Framework

NIST uses four phases:

1. **Preparation**
2. **Detection and Analysis**
3. **Containment, Eradication and Recovery**
4. **Post-Incident Activity**

### SANS and NIST Mapping

* SANS **Preparation** → NIST **Preparation**
* SANS **Identification** → NIST **Detection and Analysis**
* SANS **Containment, Eradication, Recovery** → Same combined NIST phase
* SANS **Lessons Learned** → NIST **Post-Incident Activity**

### Incident Response Plan

A formally approved document explaining how an organization handles incidents before, during, and after they occur.

Important components:

* Roles and responsibilities
* Response methodology
* Stakeholder and law-enforcement communication
* Escalation procedures

**Practice answers:**

* Disabling a compromised machine’s internet connection: **Containment**
* NIST equivalent of Lessons Learned: **Post-Incident Activity**
---
### Incident Detection Tools and Playbooks — Short Notes

The **Identification** phase in SANS and **Detection and Analysis** phase in NIST use security tools to detect incidents.

### Main Security Solutions

* **SIEM:** Centralizes and correlates logs to detect suspicious activity.
* **Antivirus (AV):** Detects and removes known malware through regular scanning.
* **EDR:** Monitors endpoints for advanced threats and can support containment and eradication.

### Playbooks

A **playbook** provides comprehensive guidelines for responding to a specific incident.

Example phishing playbook:

1. Notify stakeholders
2. Analyse email headers and body
3. Analyse attachments
4. Check whether users opened them
5. Isolate infected systems
6. Block the sender

### Runbooks

A **runbook** contains detailed technical steps for executing a specific action, based on the tools and resources available.

**Key difference:**

* **Playbook:** What actions should be taken
* **Runbook:** Exactly how to perform those actions

**Practice answer:** Step-by-step comprehensive incident response guidelines are called **Playbooks**.
---
###example of an EDR attack chain
explorer.exe
└── chrome.exe
    └── AcroRd32.exe
        └── powershell.exe
            └── DNS request to malicious-domain.com
