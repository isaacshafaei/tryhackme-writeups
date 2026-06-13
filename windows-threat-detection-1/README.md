**RDP Breach Detection – Summary**

### Risks of Exposed RDP

* Public RDP + weak password = high breach risk
* Common target for **ransomware deployment**
* Internet botnets constantly scan for exposed RDP

### Attack & Detection Flow

1. **RDP Brute Force**

   * Check **Event ID 4625** (Failed Logon)
   * Filter **Logon Type 3 & 10**
   * Look for **external Source IPs**
   * Many failures = brute-force attempt

2. **Initial Access**

   * Check **Event ID 4624** (Successful Logon)
   * Identify compromised account

3. **Post-Compromise Activity**

   * Filter **Logon Type 10** (interactive RDP)
   * Copy **Logon ID**
   * Search **Sysmon logs** with same Logon ID
   * Trace attacker processes/actions

**Key idea:** Hundreds of **4625** events often indicate exposed RDP; correlate with **4624 + Sysmon** to reconstruct the attack.
---
**SOC Investigation Mindset**

1. **Who logged in?** → Account (**4624**)
2. **From where?** → Source IP + Workstation Name
3. **How?** → Logon Type
4. **What happened next?** → Logon ID → Sysmon → Processes
5. **What was the objective?**

   * Explorer → Reconnaissance
   * CMD / PowerShell → Execution
   * ZIP + Network → Exfiltration
   * Encryption tools → Ransomware
------------
**Phishing – Summary**

### Why Phishing Works

* Harder to prevent than RDP attacks
* Users can download malware directly from the Internet
* Attackers disguise malicious files to look legitimate

### Common Phishing Techniques

**1. Malicious Binary Attachments**

* Executable files can use extensions like:

  * `.exe`, `.com`, `.scr`, `.cpl`
* Windows hides known extensions → files like:

  * `invoice.pdf.exe`
  * `cat.png.com`
* Users may mistake them for safe files

**2. LNK (Shortcut) Attachments**

* `.lnk` files can hide commands/scripts
* Often execute:

  * PowerShell
  * BAT
  * VB scripts
* May download and run malware (e.g. RATs)

### Investigation Tips

* Show **file extensions**
* Check **file properties → Shortcut → Target**
* Look for:

  * PowerShell commands
  * Downloads + execution chains
  * Suspicious file paths or URLs

**Typical chain:**
Phishing Email → Attachment → PowerShell → Malware Execution → System Compromise
-----------------

**Risks of Removable Media – Summary**

* USBs can bypass firewalls and spread malware offline
* Still widely used in attacks (USB worms, supply chain infections)

### Common Infection Scenarios

* Fake “gift” USBs (malicious files disguised as harmless content)
* Infected third-party systems (e.g., print shops spreading malware)
* User manually executes malicious files from USB

### Common Malware Tricks

* Hidden files + fake shortcuts (`RECOVERY.lnk`)
* Fake folders as `.exe` files (`Photos.exe`)
* Double extensions (`image.jpg.exe`)

### Detection Clues (Sysmon)

* Process execution from **external drive paths (E:, F:)**
* Explorer launching unknown executables
* Suspicious `.exe` originating from USB device

**Key idea:** USB attacks often mirror phishing → user-triggered execution via Explorer, making initial access hard to spot.
