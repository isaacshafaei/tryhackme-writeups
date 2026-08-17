### Ports & Nmap States — Short Note
![ps](ps.png)
* **Port:** Identifies a network service on a host.
* Common ports:

  * **TCP 80** → HTTP
  * **TCP 443** → HTTPS
  * **TCP 22** → SSH
  * **UDP 53** → DNS
* **Open:** Service is listening.
* **Closed:** No service is listening, but reachable.
* **Filtered:** Firewall/security device blocks access.
* **Unfiltered:** Reachable, but Nmap can't determine open/closed.
* **Open|Filtered:** Nmap can't distinguish.
* **Closed|Filtered:** Nmap can't distinguish.

**Nmap considers 6 port states.**

🎯 **Most interesting for a pentester:** **Open** — it means a service is accessible and can be investigated.
---
### TCP Flags — Short Note
![tcp](tcp.png)
TCP has **6 important flags**:

* **URG** → urgent data
* **ACK** → acknowledges data
* **PSH** → push data to application
* **RST** → reset/terminate connection
* **SYN** → start TCP connection
* **FIN** → finish connection

**Answers:**

* Reset flag → **RST**
* First TCP handshake flag → **SYN**
---
### TCP Connect Scan — Short Note

* `-sT` → performs a **TCP 3-way handshake** to check open ports.
* **Open:** SYN → SYN/ACK → ACK → then connection is closed with RST/ACK.
* **Closed:** Target responds with RST/ACK.
* Unprivileged users → **`-sT` is the available TCP scan**.
* Default → scans **1000 common ports**.
* `-F` → fast scan, **100 ports**.
* `-r` → scan ports in sequential order.

### Answers

* **FTP (port 21):** **open** *(the `open` is masking the answer in your copied text)*
* **Port 53:** **domain/DNS** Domain

---
### TCP SYN Scan — Short Note

* `-sS` → **TCP SYN scan**.
* Requires **root/sudo (privileged user)**.
* Sends **SYN** and waits for a response.
* **SYN/ACK → port is open**, then Nmap sends **RST** instead of completing the handshake.
* Faster/stealthier than `-sT` because no full TCP connection is established.
* `-sS` is Nmap's **default scan for privileged users**.

### From the example:

```text
21/tcp  open  ftp
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
```

* **Open ports:** **4**
* **SYN-ACK packets received:** **4** (one for each open port).
---
### Short note

**UDP scan (`-sU`)** checks UDP ports. UDP has **no handshake**, so open ports may not reply.

* **Closed UDP port** → usually sends ICMP **Port Unreachable** → Nmap marks `closed`.
* **No response** → Nmap considers it **open|filtered** because it can't confirm whether it's open or filtered.
* `-sU` → UDP scan
* `--top-ports 10` → scan the 10 most common UDP ports
* Can combine with TCP scans, e.g. `-sS -sU`.

Example:

```bash
nmap -sU --top-ports 10 TARGET
```

In the example, **UDP 53 = open DNS**.
What is the state of port number 161 over UDP in the target machine? **closed**

Correct Answer
What is the service name according to Nmap on port 161? **snmp**
---
