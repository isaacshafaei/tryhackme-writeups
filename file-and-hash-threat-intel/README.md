# File Analysis & Heuristics

## 📁 Suspicious Filepaths

Attackers drop files in specific directories to hide, persist, or execute without admin rights. Look for anomalies in these common staging areas:

* **`C:\Users\Public` (and `\Public Downloads`):** Allows cross-user access; hides in high-traffic noise.
* **`C:\Windows\Temp\`:** Used for ephemeral, drop-and-run payloads.
* **`C:\ProgramData\`:** A writable system path commonly abused for stealthy persistence.

## 🪪 Filename Heuristics

Malware authors manipulate filenames to trick users and bypass lazy detections.

* **Double Extensions:** `invoice.pdf.exe`. Exploits Windows' default "hide known file extensions" setting.
* **System Binary Impersonation:** `scvhost.exe` instead of `svchost.exe`. Relies on the analyst or user misreading core system processes. (Fix: Monitor *paths*, not just names).
* **High-Entropy Strings:** `jh8F21.exe`. Randomly generated alphanumeric garbage indicating automated packing or polymorphic malware.
* **Masquerading:** `backup-2300.exe`. Naming malware to blend in with legitimate administrative or routine system files.
---
# Malware File Hashing & Analysis

## 🧮 File Hashing

Hashes (SHA256, MD5) provide a unique, immutable fingerprint for files, bypassing filename changes. Any byte change alters the hash.

* **Best Practices:** Store in lowercase, hash both archives (ZIP) and extracted binaries, and always include context (where/when found).
* **Commands:**
* Windows CMD: `certutil -hashfile <file> SHA256`
* Windows PowerShell: `Get-FileHash -Algorithm SHA256 <file>`
* Linux: `sha256sum <file>`



## 🦠 VirusTotal Analysis

Aggregates AV vendor scans. Key pivot points:

* **Detection Score/Labels:** High vendor consensus equals high confidence. Note the malware family. Recheck low-detection files after 24-72 hours.
* **Upload Time:** Sudden detection spikes or historical growth indicate malware aging.
* **Signatures:** Look for missing, invalid, or stolen (unrelated entity) certificates.
* **Properties:** Check for odd compile timestamps or high entropy (>7.5) indicating packing.
* **Relations:** Identify malicious network infrastructure (IPs, DGA-like domains, CDNs).
* **Behavioral:** Look for process injection or critical registry modifications.

## 🗄️ MalwareBazaar

A crowdsourced malware intelligence database.

* **Search Syntax:** `sha256:<file_hash>`
* **Key Features:**
* **Malware Family Tagging:** Identifies specific malware (e.g., #IcedID).
* **YARA Rule Integration:** Provides detection rules for SIEM/EDR.
* **Campaign Attribution:** Links samples to Threat Actors (e.g., #TA551).
* **Sample Availability:** Download samples for isolated sandbox analysis.
---
# Dynamic Malware Analysis

## 🧪 Sandboxing Objectives

Sandboxes are disposable VMs that capture processes, registry writes, and network packets during malware execution.

* **Confirm Execution:** Validates if an alert is real or a decoy.
* **Extract IOCs:** Captures domains, mutexes, dropped payloads, and network connections.
* **Map to ATT&CK:** Automatically tags observed behaviors with MITRE technique IDs.

## 🛠️ Key Sandboxing Tools

* **Hybrid Analysis:** Provides behavior trees and a clean MITRE ATT&CK heatmap. Ideal for fast executive summaries.
* **Joe Sandbox:** Provides deep system calls, strings, and memory dumps. Ideal for deep-dive reverse engineering.

## ⚠️ Sandbox Limitations & Evasion

Do not blindly trust sandbox results. Malware is often designed to evade automated analysis.

* **Environment Awareness:** Malware checks for virtualized environments, debuggers, or specific hardware IDs and halts execution if detected.
* **Time Constraints:** Sandboxes typically terminate after 2-5 minutes. Malware uses long sleep calls (time-delayed attacks) to outlast the timer.
* **Encrypted Traffic:** SSL/TLS or DNS tunneling can blind the sandbox to the actual C2 payload.
* **Fileless/LotL Malware:** Memory-only payloads or abused built-in tools (PowerShell, WMI) often bypass traditional disk-focused sandbox triggers.
---
so in this room we learned that with virustotal and Hybrid Analysis we can find all important information about malwares
