# Cyber Threat Intelligence (CTI) Fundamentals

## The Intelligence Lifecycle
*   **Data:** Raw, unprocessed observables (e.g., an IP address).
*   **Information:** Data enriched with factual context (e.g., an IP registered to a specific hosting provider).
*   **Intelligence:** Actionable analysis that answers "so what?" (e.g., that IP is an active C2 server; block it).

## Key Terminology
*   **IOC (Indicator of Compromise):** Evidence that a breach *has occurred* (e.g., known malware hash, C2 IP).
*   **IOA (Indicator of Attack):** Evidence that an attack is *currently underway* (e.g., anomalous PowerShell execution).
*   **TTP (Tactics, Techniques, and Procedures):** The adversary's methodology and behavioral patterns (mapped via MITRE ATT&CK).

## Feeds vs. Platforms
*   **Feeds:** Streams of indicators (CSV, STIX, TAXII). High volume, requires strict curation to avoid false positive fatigue.
*   **Platforms:** Centralized repositories (e.g., MISP, OpenCTI) that store, map, and contextualize indicators to act as a single source of truth.

## CTI Classifications
*   **Strategic:** High-level trends and long-term risks (for business decisions).
*   **Tactical:** Adversary behaviors and methodologies (TTPs).
*   **Operational:** Campaign-specific details, motives, and target identification.
*   **Technical:** Atomic artifacts and indicators (IPs, domains, hashes) used for immediate, front-line triage.
----
```markdown
# Cyber Threat Intelligence (CTI) Basics

## 🔑 Core Concepts
* **IOC (Indicator of Compromise):** Evidence of a past breach (e.g., malicious IP, file hash).
* **IOA (Indicator of Attack):** Evidence of an active attack (e.g., abnormal PowerShell usage).
* **TTP (Tactics, Techniques, & Procedures):** Adversary methodologies (mapped via MITRE ATT&CK).
* **STIX:** Standardized JSON format for machine-readable threat sharing.

## 🚦 Traffic Light Protocol (TLP)
* 🔴 **RED:** Named recipients only. Highly restricted.
* 🟡 **AMBER:** Internal organization and limited need-to-know external partners.
* 🟢 **GREEN:** Peer community sharing. Not for public release.
* ⚪ **CLEAR:** Public distribution. No restrictions.

## 🔄 The 6-Phase Intelligence Lifecycle
1. **Direction:** Define the mission and specific intelligence requirements.
2. **Collection:** Gather raw data from feeds, OSINT, and internal telemetry.
3. **Processing:** Normalize formats, deduplicate data, and assign TLP labels.
4. **Analysis:** Cross-check data against local logs, validate threats, and filter false positives.
5. **Dissemination:** Deliver actionable intelligence (YARA rules, blocklists, reports) to specific teams.
6. **Feedback:** Measure impact (e.g., lower dwell time) and refine the next cycle.

```
---

# CTI Standards & Frameworks

## ⚔️ MITRE Frameworks

* **ATT&CK:** The offense dictionary. Maps adversary behaviors to specific Tactic/Technique IDs (e.g., T1059.001). Essential for tagging alerts uniformly.
* **D3FEND:** The defense dictionary. Maps defensive countermeasures to ATT&CK techniques. Tells you exactly *how* to block the attack.

## ⛓️ Cyber Kill Chain (Lockheed Martin)

The 7 chronological stages of an attack. Break the chain at any step to stop the adversary.

1. **Reconnaissance:** Target selection and intel gathering.
2. **Weaponization:** Crafting the exploit/malware.
3. **Delivery:** Transmitting the weapon (e.g., phishing email).
4. **Exploitation:** Triggering the malicious code.
5. **Installation:** Establishing persistence (backdoors).
6. **Command & Control (C2):** Establishing remote communication.
7. **Actions on Objectives:** Data exfiltration, encryption, or destruction.

## 🛡️ Vulnerabilities

* **CVE (Common Vulnerabilities and Exposures):** The unique ID for a vulnerability (e.g., CVE-2023-4863).
* **CVSS (Common Vulnerability Scoring System):** The severity score from 0 (low) to 10 (critical).
* **NVD (National Vulnerability Database):** The master database linking CVEs, CVSS scores, and patch data.

## 🤝 Intel Sharing

* **STIX:** The structured JSON *language* used to describe threat intel.
* **TAXII:** The *transport protocol* (APIs) used to securely send and receive STIX data.
* **Warning:** Filter before sharing. Blindly uploading internal telemetry violates NDAs and alerts attackers that you are onto them.
---
