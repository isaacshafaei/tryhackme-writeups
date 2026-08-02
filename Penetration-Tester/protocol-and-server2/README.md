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
