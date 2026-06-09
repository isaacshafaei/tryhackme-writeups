## Data Exfiltration

**Data Exfiltration:** Unauthorized transfer of sensitive data outside an organization.

### Motives
- Financial gain
- Espionage
- Ransomware/extortion
- Sabotage
- Reconnaissance

### Exfiltration Phases
1. Discovery & Collection
2. Staging (compress/encrypt)
3. Data Transfer
4. C2 Coordination

### Common Techniques
- HTTP/HTTPS uploads
- FTP/SFTP/SCP
- DNS tunneling
- Cloud storage uploads
- USB/removable media
- PowerShell, curl, wget, rclone

### Indicators of Compromise (IoCs)
- Large outbound data transfers
- Unusual DNS activity
- Suspicious processes/commands
- Archive creation (ZIP/RAR)
- Cloud storage uploads
- External file sharing

### Detection Sources
- Proxy & Firewall Logs
- DNS Logs
- NetFlow
- Sysmon / EDR
- Cloud Audit Logs
---
## DNS Exfiltration (DNS Tunneling)

**DNS Exfiltration:** Abuse of DNS to secretly transfer data by encoding it in DNS queries/responses.

### Why DNS?
- DNS traffic is common and usually allowed.
- Hard to detect without inspection.
- Data can be hidden in subdomains or TXT records.

### Common Indicators
- High volume of DNS queries to one domain.
- Very long query names/subdomains (>30–100 chars).
- High-entropy/Base64-like strings.
- Excessive TXT record usage.
- Frequent NXDOMAIN responses.
- Regular interval queries (beaconing).

### Detection (Wireshark)
```text
dns
dns.flags.response == 0
dns && frame.len > 70
dns && dns.qry.name contains <domain>
```

### Detection (Splunk)
```spl
index=data_exfil sourcetype=DNS_logs
index=data_exfil sourcetype=DNS_logs | stats count by src_ip
index=data_exfil sourcetype=DNS_logs | stats count by query | sort -count
index=data_exfil sourcetype=DNS_logs | where len(query) > 30
```

### Key Findings
- Multiple compromised hosts.
- Data sent in DNS query chunks.
- Single external domain receiving exfiltrated data.
- Large number of DNS requests with no responses.
---

### FTP Data Exfiltration (Key Notes)

* FTP is a legacy TCP protocol used for file transfer; often abused for data exfiltration.
* Attackers exploit:

  * Compromised credentials (USER/PASS)
  * Misconfigured/public FTP servers
  * Non-standard ports or tunneling

### Key Indicators

* Cleartext credentials: `USER`, `PASS`
* File transfer commands: `STOR` (upload), `RETR` (download)
* Large or repeated transfers
* Unusual external IP connections
* Data on ephemeral/PASV ports
* Suspicious file types (e.g., `.csv`, `.pdf`, `.txt`)
* Large packets (`ftp && frame.len > 90`)

### Wireshark Analysis Steps

* Filter FTP traffic: `ftp || ftp-data`
* Extract credentials: `ftp.request.command == "USER" || ftp.request.command == "PASS"`
* Find uploads: `ftp contains "STOR"`
* Search sensitive files: `ftp contains "csv"`
* Inspect sessions: Follow → TCP Stream
* Detect large transfers: `ftp && frame.len > 90`

### Goal Insight

* Identify abnormal login activity + large file transfers to external IPs indicating possible data exfiltration.

---
### HTTP Data Exfiltration (Key Notes)

* HTTP is commonly abused to exfiltrate sensitive data because it blends with normal web traffic and bypasses firewalls.

### Common Attack Methods

* `POST` requests carrying bulk data in request body
* `GET` requests with encoded data in URL parameters
* Data hidden in HTTP headers (e.g., `X-Data`)
* Chunked/multipart uploads to split large files
* Use of HTTPS/TLS to encrypt and hide payloads
* Abuse of trusted services (cloud/CDN/GitHub/Dropbox)
* Low-and-slow beaconing + staged exfiltration

### Indicators of Attack (IoCs)

* Large or unusual `POST` requests to external domains
* Rare/untrusted domains or unusual destinations
* High `bytes_sent` values
* Repeated small requests followed by large uploads
* Chunked or multipart transfer patterns

---

## Splunk Analysis (Key Queries)

* Load HTTP logs:

```spl
index="data_exfil" sourcetype="http_logs"
```

* Focus on POST requests:

```spl
index="data_exfil" sourcetype="http_logs" method=POST
```

* Analyze data volume:

```spl
index="data_exfil" sourcetype="http_logs" method=POST 
| stats count avg(bytes_sent) max(bytes_sent) by domain
```

* Isolate large exfiltration attempts:

```spl
index="data_exfil" sourcetype="http_logs" method=POST bytes_sent > 600
| table _time src_ip uri domain dst_ip bytes_sent
```

---

## Wireshark Analysis (Key Steps)

* Filter HTTP traffic:

```wireshark
http
```

* Focus on POST requests:

```wireshark
http.request.method == "POST"
```

* Identify large payloads:

```wireshark
http.request.method == "POST" && frame.len > 750
```

* Follow HTTP stream to extract content:
  → Right-click packet → Follow TCP/HTTP Stream

---

### Goal

* Correlate Splunk logs + PCAP analysis
* Identify suspicious POST upload
* Extract exfiltrated sensitive data from HTTP stream
---

### ICMP Data Exfiltration (Key Points)

* ICMP (ping) can be abused to tunnel and exfiltrate data because it is often allowed through firewalls.
* Attackers hide encoded/encrypted data inside ICMP payloads.

### Attack Techniques

* ICMP Echo Request (`Type 8`) / Echo Reply (`Type 0`) tunneling
* Base64/Hex encoded payloads
* Packet fragmentation and reassembly
* Encrypted or obfuscated payloads
* Unusual ICMP types/codes

### Indicators

* Frequent ICMP traffic to external hosts
* Large ICMP payloads (`frame.len > 100`)
* High-entropy (random-looking) payloads
* Regular, periodic ICMP packets
* Multiple ICMP fragments

### Wireshark Analysis

* Show all ICMP:

```wireshark
icmp
```

* Echo Requests only:

```wireshark
icmp.type == 8
```

* Suspicious large pings:

```wireshark
icmp.type == 8 && frame.len > 100
```

### Goal

* Identify unusually large or frequent ICMP packets carrying hidden data.
* Investigate payloads significantly larger than normal ping traffic (~74 bytes).
---
finish
