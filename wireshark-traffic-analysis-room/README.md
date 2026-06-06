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

