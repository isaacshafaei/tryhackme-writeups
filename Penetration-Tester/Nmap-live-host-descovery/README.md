### Nmap — Short Notes

* **Nmap (Network Mapper):** Open-source network scanning tool by **Gordon Lyon (Fyodor)**.
* Used for:

  * Finding live hosts
  * Scanning ports
  * Detecting services/versions
  * Vulnerability scanning (NSE scripts)

### Host Discovery Methods:

* **ARP Scan:** Finds hosts in the same LAN using broadcast requests.
* **ICMP Scan:** Uses ping, timestamp, and address mask requests.
* **TCP Ping:** Uses SYN/ACK packets to detect online hosts.
* **UDP Ping:** Uses ICMP "port unreachable" responses.

**Flow:** Host Discovery → Port Scanning → Service Detection → Vulnerability Analysis.
---
### Network Segments & Subnets — Short Notes

* **Network segment:** Physical network connection (switch/Wi-Fi).
* **Subnet:** Logical network with its own IP range, connected through a router.
* **Router:** Connects different subnets; **firewall** may control traffic between them.

### Subnet Examples:

* **/16 → 255.255.0.0** ≈ 65,000 hosts
* **/24 → 255.255.255.0** ≈ 250 hosts

### ARP Discovery:

* **ARP** finds live hosts by requesting their **MAC addresses**.
* Works only inside the **same subnet** (Layer 2).
* ARP packets **cannot cross routers**.
* For different subnets, traffic goes through the **default gateway**.

**Key Point:**
ARP scan → Same subnet only.
Different subnet → Use IP-based discovery (ICMP/TCP/UDP).
---

### TCP/IP Host Discovery — Short Notes

* Host discovery can use:

  * **ARP** → Link Layer
  * **ICMP** → Network Layer
  * **TCP** → Transport Layer
  * **UDP** → Transport Layer

### Protocols:

* **ARP:** Broadcasts a request to find a device’s **MAC address** from its IP.
* **ICMP Ping:** Uses:

  * Type 8 = Echo Request
  * Type 0 = Echo Reply
* **TCP/UDP Scans:** Send packets to specific ports to check if a host responds, useful when ICMP is blocked.

**Important:**

* Same subnet ping → ARP happens first to find MAC address.
* After ARP cache is stored → No need for new ARP requests for the same device.
---
### Nmap Target Specification — Short Note

* **List:** `IP1 IP2 IP3` → scans specific targets.
* **Range:** `10.11.12.15-20` → scans `.15` to `.20` = **6 IPs**.
* **Subnet:** `10.10.12.13/29` → scans **8 IPs** (`.8–.15`).
* **File:** `nmap -iL list.txt` → scans targets from a file.
* **List targets only:** `nmap -sL TARGETS` → shows what Nmap would scan, **without scanning**.
* **`-n`** → disables DNS resolution.

### Your Commands

`nmap -sL -n 10.10.12.13/29`

* `-sL` = list targets only
* `-n` = no DNS lookup
* Result: **8 IP addresses**, `.8–.15`.

`nmap -sL -n 10.10.0-255.101-125`

* `10.10.0-255` = **256** values
* `101-125` = **25** values
* **256 × 25 = 6,400 IP addresses**

✅ **Answer: 6,400 IP addresses**.
----
### Nmap Host Discovery — Short Note

* **Goal:** Find live hosts before port scanning.
* `nmap -sn TARGETS` → Host discovery only, **no port scan**.
* **Privileged + same subnet:** Nmap uses **ARP**.
* **Privileged + remote:** ICMP + TCP ACK/SYN + ICMP timestamp.
* **Unprivileged + remote:** TCP SYN to ports **80/443**.
* `-PR` → Force **ARP scan**.
* `-PR -sn` → ARP host discovery only.
* ARP works only on the **same subnet**.
* **ARP Reply → host is alive.**

### Command

`nmap -PR -sn CONNECTION_IP/24`

Scans **256 IPs** and reports which are alive.

**Answer:** **1 host alive**.
---
### ICMP Host Discovery — Short Note

* **ICMP Echo:** `-PE` → Type 8 → Echo Reply (Type 0)
* **ICMP Timestamp:** `-PP` → Type 13 → Reply Type 14
* **ICMP Address Mask:** `-PM` → Type 17 → Reply Type 18
* Add **`-sn`** → discover hosts without port scanning.
* ICMP can be **blocked by firewalls**, so use multiple discovery methods.

### Answers

1. Timestamp → **`-PP`**
2. Address Mask → **`-PM`**
3. Echo → **`-PE`**
---
### TCP/UDP Host Discovery — Short Note

* **TCP SYN Ping:** `-PS` → sends SYN. Response = host is alive.

  * **Does not require privileged account:** unprivileged users complete 3-way handshake.
* **TCP ACK Ping:** `-PA` → sends ACK, normally requires **privileged/root** access.

  * Response is usually **RST**.
* **UDP Ping:** `-PU` → sends UDP packet. **ICMP Port Unreachable** = host is alive.
* **`-sn`** → host discovery only, no port scanning.

### Port Examples

* `-PS80` → TCP SYN to port 80
* `-PS21-25` → ports 21–25
* `-PS80,443` → ports 80 and 443

### Answers

1. **Unprivileged:** TCP **SYN Ping (`-PS`)**
2. **Privileged:** TCP **ACK Ping (`-PA`)**
3. **Telnet port (23):** `-PS23`
---
### Reverse DNS (rDNS) — Short Note

* **rDNS:** IP address → hostname (opposite of normal DNS).
* `-R` → force reverse DNS lookup for **all hosts**, including offline ones.
* `-n` → disable DNS lookups.
* `--dns-servers IP` → use a specific DNS server.
* Useful for discovering **hostnames and network roles**.

✅ **Answer:** `-R`
---

