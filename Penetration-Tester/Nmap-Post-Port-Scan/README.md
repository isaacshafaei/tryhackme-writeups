## Nmap Service & Version Detection

* **`-sV`** → Detects the **service and its version** on open ports.
* **`--version-intensity 0–9`** → Controls detection thoroughness.

  * `--version-light` = **2**
  * `--version-all` = **9**
* `-sV` connects to the service to collect version information, so it is **not a stealth SYN-only scan**.
* Use `sudo` when required.

### Answers

* **Port 143 version:** `Dovecot imapd`
* **No version detected:** `rpcbind` (port **111**)

### What is `rpcbind`?

`rpcbind` is a **service that maps RPC services to port numbers**. It acts like a directory: clients ask `rpcbind` where a specific RPC service is running. It commonly listens on **port 111**.
---

## OS Detection

* Use `-O` for OS detection: `nmap -sS -O MACHINE_IP`
* Nmap analyzes **TCP/IP fingerprints**, including TTL, TCP sequence behavior, and responses.
* `TTL 64` → commonly **Linux**
* `TTL 128` → commonly **Windows**
* OS detection may say **“No exact OS matches”** if the fingerprint doesn't match its database.
* Virtualization, firewalls, cloud networking, and customized kernels can reduce accuracy.
* In this example, the closest match is **Linux**.

## Traceroute

* Use `--traceroute` to discover the network hops between you and the target:
  `nmap -sS --traceroute MACHINE_IP`
* Nmap uses a **high TTL and decrements it** to discover hops.
* Standard `traceroute` does the opposite: starts with a low TTL and increases it.
* Some routers don't respond with ICMP TTL-exceeded messages, so some hops may be hidden.

**Answer:** `Linux`
---
