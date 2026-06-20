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

