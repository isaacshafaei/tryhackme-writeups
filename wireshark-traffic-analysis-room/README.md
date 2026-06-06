# Wireshark Traffic Analysis: Detecting Nmap Scans

> A quick reference guide for identifying common Nmap network mapping signatures using Wireshark display filters.

---

## Core TCP Flag Filters

| Flag(s) | Decimal Filter | Boolean Filter |
| :--- | :--- | :--- |
| **SYN only** | `tcp.flags == 2` | `tcp.flags.syn == 1` |
| **ACK only** | `tcp.flags == 16` | `tcp.flags.ack == 1` |
| **SYN, ACK** | `tcp.flags == 18` | `(tcp.flags.syn == 1) and (tcp.flags.ack == 1)` |
| **RST only** | `tcp.flags == 4` | `tcp.flags.reset == 1` |
| **RST, ACK** | `tcp.flags == 20` | `(tcp.flags.reset == 1) and (tcp.flags.ack == 1)` |
| **FIN only** | `tcp.flags == 1` | `tcp.flags.fin == 1` |

---

## Scan Signatures & Detection

### 1. TCP Connect Scan (`-sT`)
* **Behavior:** Completes the full 3-way handshake. Executed by non-privileged users.
* **Signature:** Window size is typically `> 1024` bytes (expects data).
* **Wireshark Filter:** `tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024`

### 2. TCP SYN Scan (`-sS`)
* **Behavior:** Incomplete handshake (tears down the connection with an RST before final ACK). Requires privileged access.
* **Signature:** Window size is typically `<= 1024` bytes (does not expect data).
* **Wireshark Filter:** `tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024`

### 3. UDP Scan (`-sU`)
* **Behavior:** Stateless protocol mapping. Open ports typically provide no response.
* **Signature:** Closed ports return an ICMP Type 3, Code 3 message (Destination/Port Unreachable), which encapsulates the original UDP request data.
* **Wireshark Filter:** `icmp.type==3 and icmp.code==3`

---

### ARP Poisoning/Spoofing (Man In The Middle Attack)

## ARP analysis in a nutshell:
* Works on the local network
* Enables the communication between MAC addresses
* Not a secure protocol
* Not a routable protocol
* It doesn't have an authentication function
* Common patterns are request & response, announcement and gratuitous packets.

**ARP Flooding:** is when an attacker blasts thousands of fake ARP packets to fill up a network switch's memory, forcing it to broadcast all private traffic to everyone so the attacker can sniff it.
---

## ARP Investigation Summary

ARP normally works as:

| Packet Type | Description |
|------------|-------------|
| ARP Request (Opcode 1) | Broadcast: "Who has IP X.X.X.X?" |
| ARP Reply (Opcode 2) | Host responds: "I am IP X.X.X.X" |

### Indicators of ARP Spoofing
- Two different MAC addresses claim the same IP address.
- A host suddenly claims ownership of a gateway IP.
- Duplicate ARP replies appear in the network.
- Traffic is redirected through an unexpected MAC address.

### Indicators of ARP Flooding
- One host sends a large number of ARP requests.
- ARP requests target many IP addresses in the same subnet.
- Excessive ARP traffic causes network noise.

### MITM Detection
- Victim traffic is forwarded to the attacker's MAC address.
- The attacker impersonates the gateway using ARP spoofing.
- HTTP or other traffic unexpectedly flows through the attacker.

---

## Useful Wireshark Filters

| Purpose | Filter |
|----------|----------|
| Show all ARP traffic | `arp` |
| ARP requests | `arp.opcode == 1` |
| ARP replies | `arp.opcode == 2` |
| Detect duplicate IP addresses | `arp.duplicate-address-detected` |
| Detect duplicate ARP frames | `arp.duplicate-address-frame` |
| Empty destination MAC | `arp.dst.hw_mac==00:00:00:00:00:00` |
| ARP requests from a specific MAC | `((arp) && (arp.opcode == 1)) && (arp.src.hw_mac == <MAC>)` |

### Wireshark Hunt Menu

- **Hunt → ARP Scanning**
- **Hunt → Possible ARP Poisoning Detection**
- **Hunt → Possible ARP Flooding Detection**
---

### Protocols that can be used in Host and User identification:

* Dynamic Host Configuration Protocol (DHCP) traffic
* NetBIOS (NBNS) traffic 
* Kerberos traffic

### DHCP Commands:
```markdown
## DHCP Investigation Cheat Sheet

| Purpose | Wireshark Filter |
|----------|----------|
| All DHCP traffic | `dhcp` or `bootp` |
| DHCP Request | `dhcp.option.dhcp == 3` |
| DHCP ACK (accepted) | `dhcp.option.dhcp == 5` |
| DHCP NAK (denied) | `dhcp.option.dhcp == 6` |
| Search hostname | `dhcp.option.hostname contains "keyword"` |
| Search domain name | `dhcp.option.domain_name contains "keyword"` |

### Important DHCP Options

| Option | Description |
|----------|----------|
| 12 | Hostname |
| 15 | Domain Name |
| 50 | Requested IP Address |
| 51 | Lease Time |
| 56 | NAK Message / Rejection Reason |
| 61 | Client MAC Address |

### Quick Notes

| Packet Type | Use Case |
|------------|----------|
| Request (3) | Contains hostname, requested IP, lease time, MAC |
| ACK (5) | Accepted request, assigned IP information |
| NAK (6) | Denied request, check Option 56 for reason |
```
---

#### NetBIOS or Network Basic Input/Output System is the technology responsible for allowing applications on different hosts to communicate with each other.
```mardown
## NBNS Investigation Cheat Sheet

| Purpose | Wireshark Filter |
|----------|----------|
| All NBNS traffic | `nbns` |
| Search hostname | `nbns.name contains "keyword"` |

### Important NBNS Fields

| Field | Description |
|----------|----------|
| Query | Hostname lookup request |
| Name | Queried hostname |
| TTL | Time to Live |
| Address | Associated IP address |

```
#### Kerberos is the default authentication service for Microsoft Windows domains. It is responsible for authenticating service requests between two or more computers over the untrusted network. The ultimate aim is to prove identity securely.
```markdown
## Kerberos Investigation Cheat Sheet

| Purpose | Wireshark Filter |
|----------|----------|
| All Kerberos traffic | `kerberos` |
| Search user/host | `kerberos.CNameString contains "keyword"` |
| Show users only (exclude hosts) | `kerberos.CNameString and !(kerberos.CNameString contains "$")` |
| Kerberos v5 traffic | `kerberos.pvno == 5` |
| Search domain | `kerberos.realm contains ".org"` |
| Kerberos TGT requests | `kerberos.SNameString == "krbtgt"` |

### Important Kerberos Fields

| Field | Description |
|----------|----------|
| CNameString | Username or hostname (`$` = hostname) |
| pvno | Kerberos protocol version |
| realm | Domain name |
| SNameString | Requested service |
| addresses | Client IP and NetBIOS name (requests only) |

### Quick Notes

| Indicator | Meaning |
|------------|----------|
| Username ends with `$` | Computer account |
| Username without `$` | User account |
| `krbtgt` service | Ticket Granting Ticket (TGT) request |
| `pvno == 5` | Kerberos Version 5 |
```
---

```mardown
## ICMP Tunneling Cheat Sheet

| Indicator | Wireshark Filter |
|------------|------------------|
| All ICMP traffic | `icmp` |
| Suspicious large ICMP packets | `data.len > 64 and icmp` |

### Red Flags
- High volume of ICMP traffic
- ICMP packets larger than normal (64 bytes)
- Encapsulated TCP/HTTP/SSH data in ICMP payload
- Possible C2 communication or data exfiltration

---

## DNS Tunneling Cheat Sheet

| Indicator | Wireshark Filter |
|------------|------------------|
| All DNS traffic | `dns` |
| Detect dnscat | `dns contains "dnscat"` |
| Long DNS queries | `dns.qry.name.len > 15 and !mdns` |

### Red Flags
- Long or encoded subdomains
- Unusual DNS query lengths
- High volume of DNS requests to one domain
- Possible C2 communication or data exfiltration
- Patterns such as `dnscat` or `dns2tcp`
```
### Cleartext Protocol Analysis (Wireshark Cheat Sheet)

#### Global

| Task        | Wireshark Filter |
| ----------- | ---------------- |
| FTP traffic | `ftp`            |

---

### FTP Response Codes

#### 1xx (Info)

| Code | Meaning          | Filter                     |
| ---- | ---------------- | -------------------------- |
| 211  | System status    | `ftp.response.code == 211` |
| 212  | Directory status | `ftp.response.code == 212` |
| 213  | File status      | `ftp.response.code == 213` |

#### 2xx (Connection)

| Code | Meaning               | Filter                     |
| ---- | --------------------- | -------------------------- |
| 220  | Service ready         | `ftp.response.code == 220` |
| 227  | Passive mode          | `ftp.response.code == 227` |
| 228  | Long passive mode     | `ftp.response.code == 228` |
| 229  | Extended passive mode | `ftp.response.code == 229` |

#### 3xx (Auth)

| Code | Meaning       | Filter                     |
| ---- | ------------- | -------------------------- |
| 230  | Login success | `ftp.response.code == 230` |
| 231  | Logout        | `ftp.response.code == 231` |
| 331  | Username OK   | `ftp.response.code == 331` |
| 430  | Auth failed   | `ftp.response.code == 430` |
| 530  | Login failed  | `ftp.response.code == 530` |

---

### FTP Commands

| Command | Purpose          | Filter                          |
| ------- | ---------------- | ------------------------------- |
| USER    | Username         | `ftp.request.command == "USER"` |
| PASS    | Password         | `ftp.request.command == "PASS"` |
| CWD     | Change directory | `ftp.request.command == "CWD"`  |
| LIST    | List files       | `ftp.request.command == "LIST"` |

---

### Credential Hunting

| Case               | Filter                                                                  |
| ------------------ | ----------------------------------------------------------------------- |
| Password field     | `ftp.request.arg == "password"`                                         |
| Failed logins      | `ftp.response.code == 530`                                              |
| Username + failure | `(ftp.response.code == 530) and (ftp.response.arg contains "username")` |
| Password spray     | `(ftp.request.command == "PASS") and (ftp.request.arg == "password")`   |

### Extra Useful Commands:
```
### find the downaloaded files in server:
ftp.request.command == "RETR"

### find the uploaded files in server:
ftp.request.command == "STOR"

### find the commands that attacker ran in FTP server:
ftp.request.command == "SITE"
```
---

# HTTP Analysis Cheat Sheet (Wireshark)

## Why Analyze HTTP?
HTTP is a cleartext, client-server, request/response protocol commonly used for web traffic.

### Detectable Threats
- Phishing pages
- Web attacks
- Data exfiltration
- Command & Control (C2) traffic

---

## Global Filters

| Purpose | Filter |
|----------|----------|
| HTTP traffic | `http` |
| HTTP/2 traffic | `http2` |

---

## HTTP Request Methods

| Method | Purpose | Filter |
|----------|----------|----------|
| GET | Retrieve data | `http.request.method == "GET"` |
| POST | Send data | `http.request.method == "POST"` |
| All requests | List requests | `http.request` |

---

## HTTP Response Codes

| Code | Meaning | Filter |
|------|----------|----------|
| 200 | OK | `http.response.code == 200` |
| 301 | Moved Permanently | `http.response.code == 301` |
| 302 | Moved Temporarily | `http.response.code == 302` |
| 401 | Unauthorized | `http.response.code == 401` |
| 403 | Forbidden | `http.response.code == 403` |
| 404 | Not Found | `http.response.code == 404` |
| 405 | Method Not Allowed | `http.response.code == 405` |
| 408 | Request Timeout | `http.response.code == 408` |
| 500 | Internal Server Error | `http.response.code == 500` |
| 503 | Service Unavailable | `http.response.code == 503` |

---

## HTTP Parameters

### Client-Side

| Purpose | Filter |
|----------|----------|
| User-Agent contains keyword | `http.user_agent contains "keyword"` |
| Detect Nmap | `http.user_agent contains "nmap"` |
| URI contains admin | `http.request.uri contains "admin"` |
| Full URI contains admin | `http.request.full_uri contains "admin"` |

### Server-Side

| Purpose | Filter |
|----------|----------|
| Apache server | `http.server contains "apache"` |
| Host contains keyword | `http.host contains "keyword"` |
| Exact host match | `http.host == "keyword"` |
| Persistent connection | `http.connection == "Keep-Alive"` |
| Search response text | `data-text-lines contains "keyword"` |

---

# User-Agent Analysis

## Indicators of Suspicious Activity
- Multiple User-Agents from same host
- Custom/non-standard User-Agents
- Misspellings (`Mozlila`, `Mozlilla`)
- Security tools in User-Agent
- Payloads embedded in User-Agent

### Global Filter

```wireshark
http.user_agent
```

### Security Tool Detection

```wireshark
(http.user_agent contains "sqlmap") or
(http.user_agent contains "Nmap") or
(http.user_agent contains "Wfuzz") or
(http.user_agent contains "Nikto")
```

---

# Log4Shell (Log4j) Analysis

## Common Indicators
- Attack often starts with POST requests
- JNDI LDAP strings
- Exploit.class references
- Obfuscated payloads in User-Agent

### Detect POST Requests

```wireshark
http.request.method == "POST"
```

### Detect JNDI / Exploit Strings

```wireshark
(ip contains "jndi") or (ip contains "Exploit")
```

```wireshark
(frame contains "jndi") or (frame contains "Exploit")
```

### Suspicious User-Agent

```wireshark
(http.user_agent contains "$") or
(http.user_agent contains "==")
```

---

## Quick Threat-Hunting Filters

| Threat | Filter |
|----------|----------|
| POST activity | `http.request.method == "POST"` |
| Admin page access | `http.request.uri contains "admin"` |
| Nmap scan | `http.user_agent contains "Nmap"` |
| SQLMap activity | `http.user_agent contains "sqlmap"` |
| Nikto scan | `http.user_agent contains "Nikto"` |
| Wfuzz scan | `http.user_agent contains "Wfuzz"` |
| Log4Shell attempt | `frame contains "jndi"` |
| Exploit payload | `frame contains "Exploit"` |
| Suspicious User-Agent | `http.user_agent contains "$"` |

---

# HTTPS / TLS Analysis Cheat Sheet

## Why Analyze HTTPS?
HTTPS encrypts HTTP traffic using TLS. Traffic cannot be decrypted without the TLS key log file.

### Detectable Threats
- Malicious HTTPS websites
- C2 traffic
- Data exfiltration
- Phishing

---

## Common Filters

| Purpose | Filter |
|----------|----------|
| HTTP requests | `http.request` |
| TLS traffic | `tls` |
| Client Hello | `tls.handshake.type == 1` |
| Server Hello | `tls.handshake.type == 2` |
| SSDP traffic | `ssdp` |

---

## TLS Handshake

| Event | Filter |
|---------|---------|
| Client Hello | `(http.request or tls.handshake.type == 1) and !(ssdp)` |
| Server Hello | `(http.request or tls.handshake.type == 2) and !(ssdp)` |

Use these filters to identify communicating IPs.

---

## HTTPS Decryption

### Requirements
- SSL/TLS Key Log File (`SSLKEYLOGFILE`)
- Browser support (Chrome, Firefox)

### Add Key Log File
```
Edit → Preferences → Protocols → TLS
```

### Result
After loading the key log file, Wireshark can decrypt HTTPS traffic and reveal:
- URLs
- HTTP/HTTP2 data
- Headers
- Reassembled content

---

## Useful Views After Decryption
- Decrypted TLS
- Decompressed Header
- Reassembled TCP
- Reassembled SSL

---
### Question:
What is the frame number of the "Client Hello" message sent to "accounts.google.com"?
`tls.handshake.type == 1 && tls.handshake.extensions_server_name contains "accounts.google.com"`
---

````markdown
# Key Lessons Learned (Wireshark HTTP/HTTPS Analysis)

## HTTP Analysis
- HTTP traffic is **cleartext** and easy to inspect.
- Learn to identify:
  - GET / POST requests
  - Response codes (200, 404, 500, etc.)
  - User-Agents
  - URLs and parameters
- Detect:
  - Phishing
  - Web attacks
  - Data exfiltration
  - C2 traffic

---

## User-Agent Analysis
- Never trust User-Agent strings.
- Look for:
  - Security tools (`Nmap`, `sqlmap`, `Nikto`, `Wfuzz`)
  - Misspellings
  - Custom or unusual values
- User-Agent is an indicator, not proof of compromise.

---

## HTTPS / TLS Analysis
- HTTPS encrypts web traffic.
- Without keys, you only see metadata:
  - IPs
  - TLS handshakes
  - SNI (requested hostname)
- Learn to identify:
  - Client Hello (`tls.handshake.type == 1`)
  - Server Hello (`tls.handshake.type == 2`)

---

## TLS Decryption
- Decryption requires a valid **SSLKEYLOGFILE**.
- After loading keys, Wireshark can reveal:
  - HTTP/2 traffic
  - URLs
  - Headers
  - Downloaded files

---

## Object Extraction
- Export transferred files:
  ```
  File → Export Objects → HTTP
  ```
- Useful for:
  - Malware analysis
  - Data exfiltration investigations
  - Recovering documents/images
  - Finding flags in CTFs

---

# What a Security Analyst Must Learn

### Core Skills
- TCP/IP fundamentals
- HTTP/HTTPS protocols
- TLS handshake process
- Wireshark filtering

### Investigation Skills
- Follow streams
- Analyze requests/responses
- Identify anomalies
- Validate suspicious User-Agents
- Extract files and artifacts

### Threat Hunting Mindset
- Don't trust appearances.
- Look for abnormal behavior.
- Correlate multiple indicators before concluding.
- Always inspect encrypted traffic when decryption keys are available.

## Golden Rule
**Packets tell the truth. Learn how to extract, filter, and correlate them efficiently.**
````
---
# Bonus: Cleartext Credential Hunting

## Why It Matters
- Credentials sent over cleartext protocols can be captured.
- Useful for detecting:
  - Weak security practices
  - Brute-force attempts
  - Credential exposure

---

## Supported Protocols
- FTP
- HTTP
- IMAP
- POP
- SMTP

---

## Find Credentials

```text
Tools → Credentials
```

Wireshark extracts:
- Username
- Password
- Protocol
- Packet number

---

## Analyst Notes
- Works in Wireshark v3.1+
- Not all protocols are supported
- Always verify manually
- Do not rely solely on automated detection

---

## Security Takeaway
- Never transmit credentials in cleartext.
- Check for exposed usernames/passwords during investigations.
- Use credential findings as indicators, then validate with packet analysis.
---

# Bonus: Actionable Results

## Purpose
Turn investigation findings into firewall rules to block malicious traffic.

---

## Generate Firewall Rules

```text
Tools → Firewall ACL Rules
```

Wireshark can create rules based on:
- IP Address
- Port
- MAC Address

---

## Supported Firewalls
- iptables (Linux)
- Cisco IOS
- IPFilter
- IPFW
- PF (Packet Filter)
- Windows Firewall (netsh)

---

## Security Takeaway
- Detection is only the first step.
- Identify the malicious source.
- Generate blocking rules.
- Implement controls to prevent further communication.

## Incident Response Flow

```text
Detect → Investigate → Identify → Contain → Block → Monitor
```
---

