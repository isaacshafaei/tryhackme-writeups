## Cyber Kill Chain

The **Cyber Kill Chain**, introduced by Lockheed Martin in 2011, explains how cyberattacks progress through seven stages:

1. **Reconnaissance:** Gather information about the target.
2. **Weaponisation:** Create or modify a malicious payload.
3. **Delivery:** Send the payload to the target.
4. **Exploitation:** Exploit a vulnerability.
5. **Installation:** Install malware or a backdoor for persistence.
6. **Command and Control (C2):** Remotely control the compromised system.
7. **Actions on Objectives:** Steal data, move laterally, or attack other systems.

Understanding these stages helps organisations detect and stop attacks before the attacker reaches their objective.
---
## Reconnaissance

Reconnaissance is the process of gathering information about a target to identify weaknesses and possible entry points.

### Types

* **Passive Reconnaissance:** Collects information without directly interacting with the target.

  * WHOIS and DNS lookups
  * Website crawling and scraping
  * Social media research
  * Google Dorking

* **Active Reconnaissance:** Directly interacts with the target and may be detected.

  * Port scanning
  * Vulnerability scanning
  * Social engineering
  * Physical reconnaissance

### Countermeasures

* Limit public information on websites and social media.
* Use WHOIS privacy protection.
* Reduce unnecessary DNS exposure.
* Monitor network traffic and service logs.
* Detect port and vulnerability scans.

**Search engines used to find sensitive data:** Google Dorking
**Checking social media pages:** Passive reconnaissance
---
## Weaponisation

Weaponisation is the stage where an attacker creates a malicious payload based on information gathered during reconnaissance.

The attacker may:

* Use a ready-made exploit.
* Modify an existing exploit.
* Create a new exploit.
* Use exploit kits to package malicious code.
* Hide payloads inside Office documents, PDFs, executables, emails, websites, or USB drives.
* Use encryption or obfuscation to avoid detection.

A common method is using malicious Microsoft Office macros that execute code when the document is opened.

### Countermeasures

* Train users to recognise suspicious emails and attachments.
* Disable unnecessary software and browser plugins.
* Disable Office macros or allow only signed macros.
* Restrict risky features through Group Policy.
* Reduce the overall attack surface.

**Technique used to make malicious code harder to analyse:** Obfuscation
**What built-in feature makes creating a malicious MS Office document possible? ** Macro
---
## Delivery

Delivery is the stage where the attacker sends the prepared malicious payload to the target.

### Common Delivery Methods

* **Phishing:** Malicious email attachments or links.
* **Spear phishing:** Targeted emails impersonating a trusted person.
* **Malicious links:** Spoofed domains or shortened URLs.
* **File-sharing platforms:** Hosting malicious files on trusted services.
* **Malvertising:** Malicious ads redirect users to harmful websites.
* **Smishing:** SMS messages containing malicious links.
* **Social engineering:** Convincing users to run malware.
* **Physical delivery:** USB drives, DVDs, or other removable media.

### Countermeasures

* Security awareness training
* Email and web filtering
* Web Application Firewalls
* Network monitoring
* Patch management

**Malicious advertisements:** Malvertising
**Phishing through SMS:** Smishing

---
## Exploitation

The attacker uses a weakness to gain access, such as:

* Weak/default passwords
* Software vulnerabilities
* Misconfigurations
* SQL injection or buffer overflow
* Zero-day exploits

### Defences

* Strong passwords and MFA
* Patch management
* Vulnerability scanning
* IPS and WAF

**Unknown-to-vendor exploit:** Zero-day
**Protection with stolen credentials:** MFA
---

