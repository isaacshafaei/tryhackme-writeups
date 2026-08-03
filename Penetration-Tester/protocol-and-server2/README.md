### Protocol Security — Short Note

| Topic                   | Key Point                                                                      |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Insecure protocols**  | Telnet, HTTP, FTP, SMTP, POP3, IMAP can transmit data in cleartext             |
| **Secure alternatives** | SSH, HTTPS, SFTP/FTPS, SMTPS, POP3S, IMAPS                                     |
| **Sniffing**            | Captures network traffic → **Disclosure** → affects **Confidentiality**        |
| **MITM**                | Intercepts/modifies communication → **Alteration** → affects **Integrity**     |
| **Password attacks**    | Brute force, credential stuffing, password spraying → can cause **Disclosure** |
| **Vulnerabilities**     | Exploitation can cause DoS (Availability) or RCE (severe impact)               |
| **CIA Triad**           | **Confidentiality, Integrity, Availability**                                   |
| **DAD**                 | **Disclosure, Alteration, Destruction**                                        |
| **Modern protections**  | TLS, HSTS, certificate pinning, Certificate Transparency                       |
| **Password defences**   | Strong passwords, account lockout, MFA                                         |
| **Hydra**               | Tool for testing password strength using wordlists                             |

**Key relationship:**
**CIA (what to protect) → DAD (what attacks aim to cause)**

* Confidentiality → Disclosure
* Integrity → Alteration
* Availability → Destruction
---
### Sniffing Attacks — Short Note

| Topic                    | Key Point                                                                                 |
| ------------------------ | ----------------------------------------------------------------------------------------- |
| **Sniffing**             | Capturing network packets to read unencrypted data                                        |
| **What can be exposed?** | Credentials, private messages, and other sensitive data                                   |
| **Tools**                | `tcpdump` (CLI), Wireshark (GUI), `tshark` (CLI)                                          |
| **Requirement**          | Access to network traffic + sufficient privileges                                         |
| **Common targets**       | Cleartext protocols like Telnet, HTTP, FTP, POP3                                          |
| **POP3 Port**            | **110**                                                                                   |
| **Telnet Port**          | **23**                                                                                    |
| **Main mitigation**      | Use encryption/TLS; replace Telnet with SSH                                               |
| **Other defences**       | Network segmentation, encrypted tunnels/VLANs, 802.1X, Zero Trust, ARP spoofing detection |

### Useful Commands

```bash
# Capture POP3 traffic
sudo tcpdump port 110 -A

# Capture Telnet traffic
sudo tcpdump port 23 -A

# Capture HTTP traffic
sudo tcpdump port 80 -A

# Capture FTP traffic
sudo tcpdump port 21 -A

# Save capture
sudo tcpdump -w capture.pcap

# Read capture
tcpdump -r capture.pcap -A
```

### Wireshark Filters

* **IMAP:** `imap`
* **POP3:** `pop`
* **HTTP:** `http`
* **FTP:** `ftp`

**Key idea:** Cleartext protocols allow anyone with access to the traffic path to potentially capture sensitive information.
---
### MITM Attack — Short Summary Note

| Topic                | Key Point                                                                                                               |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **MITM**             | Attacker secretly positions themselves between two communicating parties and can **intercept or modify** communication. |
| **Main requirement** | Weak/missing **authentication and integrity protection**.                                                               |
| **ARP Spoofing**     | Redirects local network traffic to the attacker by forging ARP messages.                                                |
| **DNS Spoofing**     | Provides fake DNS responses to redirect victims to attacker-controlled systems.                                         |
| **Rogue AP**         | Fake Wi-Fi access point that routes victim traffic through the attacker.                                                |
| **BGP Hijacking**    | Redirects traffic by announcing false BGP routes at the Internet routing level.                                         |
| **Common tools**     | Bettercap, Ettercap, mitmproxy, Responder.                                                                              |
| **Against HTTPS**    | SSL stripping, fake certificates, or compromised/rogue CAs.                                                             |
| **Defences**         | HTTPS/TLS, HSTS, Certificate Transparency, certificate pinning, DANE, proper certificate validation.                    |
| **Core protection**  | Use **strong authentication + encryption/signing** to ensure confidentiality and integrity.                             |

**Answers:**

* Ettercap interfaces: **3**
* Ways to invoke Bettercap: **3**
---
