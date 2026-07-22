## Penetration Testing Frameworks

A penetration testing framework provides a structured process for planning, scoping, testing, exploitation, reporting, and remediation.

### Benefits

* **Thoroughness:** Prevents important areas from being missed.
* **Consistency:** Produces repeatable results across testers.
* **Compliance:** Aligns testing with regulations and standards.
* **Communication:** Makes the process understandable to clients and auditors.

### Main Frameworks

* **OSSTMM:** Scientific and metrics-based testing
* **OWASP WSTG:** Web application testing
* **NIST SP 800-115:** Government security-testing guidance
* **PTES:** Practical, phase-based penetration testing
* **ISSAF:** Detailed nine-step assessment methodology
* **MITRE ATT&CK:** Maps attacker tactics and techniques

**Skipping network mapping and scope:** Thoroughness
**Important benefit for HIPAA:** Compliance
---

## OSSTMM

**OSSTMM** is a scientific, metrics-based penetration testing framework developed by ISECOM. It aims to produce measurable, repeatable, and verifiable results.

### Five Security Channels

* **HUMSEC:** Human and social engineering
* **PHYSSEC:** Physical security
* **SPECSEC:** Wireless communications
* **COMSEC:** Telecommunications
* **DATASEC:** Networks and applications

### Key Metric

**Risk Assessment Value (RAV):** Measures the balance between attack surface and security controls.

### Four Phases

1. **Induction:** Discover and verify assets.
2. **Interaction:** Probe assets and measure exposure.
3. **Inquiry:** Attempt exploitation and privilege escalation.
4. **Intervention:** Contain issues, audit controls, and test detection.

Reports follow the **STAR** format.

**Attack surface vs controls metric:** Risk Assessment Value
**Phase after Interaction:** Inquiry — test whether exposure can lead to unauthorised access.
---

## OWASP Web Security Testing Guide (WSTG)

The **WSTG** is a detailed framework for testing web applications. It contains more than 90 test cases across 12 categories, including:

* Authentication and authorisation
* Session management
* Input validation
* Cryptography
* Business logic
* Client-side and API testing

It uses a **risk-based approach**, prioritising vulnerabilities by exploitability and impact.

### SDLC Phases

1. Define security requirements before development.
2. Review architecture and threat models during design.
3. Review and test code during development.
4. Verify security during deployment.
5. Perform periodic testing during maintenance.

**Input validation identifier:** `WSTG-INPV`
**Testing finished code before deployment:** Phase 3
---
## NIST SP 800-115

NIST SP 800-115 is a structured security-testing guide widely used by government and regulated organisations.

### Objectives

* Identify vulnerabilities
* Validate security controls
* Assess exploitability

### Phases

1. **Planning:** Define scope, objectives, rules, and communication.
2. **Execution:** Perform:

   * Reviews
   * Target identification
   * Vulnerability validation
   * Penetration testing
3. **Post-Testing:** Analyse findings, prioritise risks, and recommend remediation.

**Scanner findings should first undergo:** Target Vulnerability Validation
---
## PTES

The **Penetration Testing Execution Standard (PTES)** provides a practical, end-to-end workflow for penetration tests.

### Seven Phases

1. **Pre-Engagement:** Define scope, rules, testing times, contacts, and legal authorization.
2. **Intelligence Gathering:** Perform passive and active reconnaissance.
3. **Threat Modeling:** Identify valuable assets and likely attack paths.
4. **Vulnerability Analysis:** Find and verify vulnerabilities.
5. **Exploitation:** Exploit confirmed weaknesses to demonstrate impact.
6. **Post-Exploitation:** Pivot, escalate privileges, and assess business risk.
7. **Reporting:** Provide executive and technical findings with remediation.

**Scope, rules, and authorization:** Phase 1 — Pre-Engagement Interactions
---
## ISSAF

**ISSAF** is an older, unmaintained penetration testing framework. Its tools are outdated, but its attack methodology remains useful.

### Three Phases

1. **Planning and Preparation:** Define scope, limits, contacts, and tools.
2. **Assessment:** Follow the nine-step attack model.
3. **Reporting and Cleanup:** Report findings and remove testing artifacts.

### Nine Assessment Steps

1. Information gathering
2. Network mapping
3. Vulnerability identification
4. Penetration
5. Privilege escalation
6. Further enumeration
7. Lateral movement
8. Maintaining access
9. Covering tracks

**Final step:** Covering tracks
---
## MITRE ATT&CK

**MITRE ATT&CK** is a knowledge base of real-world attacker behaviour. It complements penetration testing frameworks by providing standard names and IDs for adversary actions.

### Structure

* **Tactics:** High-level attacker goals — the **why**.
* **Techniques:** Methods used to achieve goals — the **how**.
* **Sub-techniques:** More specific versions of techniques.

ATT&CK helps map pentest findings to real attack behaviour, detection guidance, and mitigations.

Examples:

* Phishing attachment: `T1566.001`
* Public-facing application exploit: `T1190`
* Credential dumping: `T1003`
* Lateral movement with stolen credentials: `T1550`

**Matrix columns:** Tactics
**Rows:** Techniques
**Public-facing application exploit:** `T1190`
---

## Specialized Security Frameworks

* **WASC Threat Classification:** Older taxonomy for web application threats; mostly replaced by OWASP.
* **CSA Cloud Controls Matrix (CCM):** Cloud governance and compliance controls.
* **OWASP MASTG:** Security testing guide for Android and iOS applications.
* **PCI DSS Penetration Testing Guidelines:** Mandatory testing requirements for organisations handling cardholder data.
* **CBEST:** Threat-intelligence-led penetration testing for UK financial institutions.

Framework selection depends on the client’s industry, technology, regulations, and location.

**Online retailer processing cards:** PCI DSS Penetration Testing Guidelines
**iOS banking application:** OWASP Mobile Application Security Testing Guide
**AWS controls assessment:** CSA Cloud Controls Matrix
---
## Selecting a Penetration Testing Framework

Framework choice depends on:

* **Target type:** Web, mobile, network, cloud, or physical systems
* **Regulations:** PCI DSS, CBEST, NIST, or HIPAA requirements
* **Measurement needs:** OSSTMM supports repeatable metrics
* **Team resources:** PTES is practical for standard engagements

Multiple frameworks may be combined:

* **Web applications:** OWASP WSTG
* **Mobile applications:** OWASP MASTG
* **Corporate networks:** PTES
* **Federal environments:** NIST SP 800-115
* **Payment systems:** PCI DSS
* **UK financial institutions:** CBEST

**E-commerce web, mobile, and payment systems:**
`OWASP WSTG, OWASP MASTG, PCI DSS`
---
finito

