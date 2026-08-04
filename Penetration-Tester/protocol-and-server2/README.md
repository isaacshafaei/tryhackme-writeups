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
### TLS — Short Summary Note

| Topic                | Key Point                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------- |
| **SSL/TLS**          | TLS encrypts network communication to protect **confidentiality and integrity** and help prevent MITM attacks. |
| **Current standard** | **TLS 1.3** is the latest standard; TLS 1.2 is still widely used. SSL 2.0/3.0 and TLS 1.0/1.1 are deprecated.  |
| **HTTPS**            | HTTP + TLS → **Port 443**                                                                                      |
| **FTPS**             | FTP + TLS → **Port 990** (implicit TLS)                                                                        |
| **SMTPS**            | SMTP + TLS → **Port 465**                                                                                      |
| **POP3S**            | POP3 + TLS → **Port 995**                                                                                      |
| **IMAPS**            | IMAP + TLS → **Port 993**                                                                                      |
| **DoT**              | DNS over TLS → **Port 853**                                                                                    |
| **DoH**              | DNS over HTTPS → **Port 443**                                                                                  |
| **Implicit TLS**     | Encryption starts immediately when connecting.                                                                 |
| **STARTTLS**         | Starts with a cleartext connection, then upgrades to TLS.                                                      |
| **TLS Handshake**    | Client and server negotiate parameters, authenticate using certificates, and establish shared secret keys.     |
| **TLS 1.3**          | Faster handshake, forward secrecy by default, simplified cipher suites, and more encrypted handshake data.     |
| **Certificates**     | Trusted CAs verify the server's identity. Key details: **issued to, issuer, validity period**.                 |
| **Testing tools**    | `testssl.sh`, `sslyze`, SSL Labs, `nmap ssl-enum-ciphers`                                                      |

**Important answer:**
**DNS over TLS = DoT** → **Port 853**.
---

### SSH — Short Summary Note

| Topic                          | Key Point                                                                                                      |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **SSH**                        | Secure protocol for remote administration; provides **confidentiality, integrity, and server authentication**. |
| **Default port**               | **22/TCP**                                                                                                     |
| **Password authentication**    | Simple but vulnerable to brute-force attacks if passwords are weak.                                            |
| **Public key authentication**  | Recommended; uses a **private key** (secret) and **public key** (stored on server).                            |
| **Certificate authentication** | Uses an SSH Certificate Authority for scalable authentication.                                                 |
| **MFA**                        | Combines SSH authentication with another factor, such as an OTP.                                               |
| **Host key verification**      | Verifies the server's identity and helps prevent MITM attacks. Keys are stored in `~/.ssh/known_hosts`.        |
| **Generate key**               | `ssh-keygen -t ed25519`                                                                                        |
| **Public key location**        | `~/.ssh/id_ed25519.pub`                                                                                        |
| **Private key**                | `~/.ssh/id_ed25519` — keep secret and protect with a passphrase.                                               |
| **Authorized keys**            | Server stores allowed public keys in `~/.ssh/authorized_keys`.                                                 |
| **SFTP**                       | Secure file transfer over SSH; recommended for interactive transfers.                                          |
| **SCP**                        | Secure file copying over SSH; traditionally used but being phased out in favour of SFTP.                       |
| **rsync over SSH**             | Efficient for large transfers and directory synchronization.                                                   |
| **SSH hardening**              | Disable password/root login, restrict users, use modern algorithms, and use tools like Fail2ban.               |

### Useful Commands

```bash
# Connect
ssh username@IP

# Non-standard port
ssh -p 2222 username@IP

# Use private key
ssh -i ~/.ssh/id_ed25519 username@IP

# Execute a command remotely
ssh username@IP "uname -r"

# Copy remote file to local
scp username@IP:/path/file.txt ./

# Copy local file to remote
scp file.txt username@IP:/path/

# SFTP
sftp username@IP
```

### Key Answers from the Task

* **Kernel release:** `5.15.0-119-generic`
* **SCP download size:** **415 KB**
---
### Password Attacks — Short Summary Note

| Topic                   | Key Point                                                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Authentication**      | Proves who you are using **something you know, have, or are**.                                                            |
| **Password attacks**    | Target the **"something you know"** factor.                                                                               |
| **Password Guessing**   | Uses personal information about the target.                                                                               |
| **Dictionary Attack**   | Tries common words from a wordlist.                                                                                       |
| **Brute Force**         | Tries all possible character combinations; longer passwords are harder to crack.                                          |
| **Credential Stuffing** | Uses leaked username/password pairs from previous breaches.                                                               |
| **Password Spraying**   | Tries a few common passwords against many accounts to avoid lockouts.                                                     |
| **Hybrid Attack**       | Combines dictionary words with patterns, numbers, or substitutions.                                                       |
| **Wordlists**           | Common examples: `rockyou.txt`, SecLists, CrackStation.                                                                   |
| **THC Hydra**           | Automates password attacks against services such as **FTP, SSH, POP3, IMAP, SMTP, and HTTP**.                             |
| **Hydra syntax**        | `hydra -l username -P wordlist.txt server service`                                                                        |
| **Other tools**         | Medusa, Ncrack, NetExec, Burp Suite Intruder, Hashcat, John the Ripper.                                                   |
| **Defences**            | Strong passwords, MFA, rate limiting, account lockout, CAPTCHA, passwordless authentication, breached-password detection. |

### Useful Hydra Options

* `-l` → single username
* `-L` → username list
* `-p` → single password
* `-P` → password wordlist
* `-s` → custom port
* `-V` / `-vV` → verbose
* `-t` → number of parallel tasks
* `-f` → stop after finding a valid password

### Task Answer

* **IMAP username:** `lazie`
* **Password:** `butterfly`
---

