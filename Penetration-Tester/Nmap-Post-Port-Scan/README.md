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
## Nmap Scripting Engine (NSE) — Short Note

* **NSE** allows Nmap to run **Lua scripts** for advanced scanning and enumeration.
* Scripts are stored in:

  ```bash
  /usr/share/nmap/scripts
  ```
* Run default scripts:

  ```bash
  nmap -sC MACHINE_IP
  ```

  `-sC` = `--script=default`
* Run a specific script:

  ```bash
  nmap --script "http-date" MACHINE_IP
  ```
* Run scripts by pattern:

  ```bash
  nmap --script "ftp*" MACHINE_IP
  ```
* Useful categories:

  * `auth` → authentication
  * `brute` → brute force
  * `discovery` → information gathering
  * `exploit` → exploitation
  * `safe` → generally safe checks
  * `vuln` → vulnerability detection
  * `version` → service versions
* **Be careful:** some scripts are intrusive, perform brute force, DoS checks, or exploitation. Only use them on authorized targets.

### Answers

1. `http-robots.txt` → checks for **robots.txt** and extracts/displays its entries.
2. MS15-034 / CVE-2015-1635 → **`http-vuln-cve2015-1635`**
3. Port 80 `http-title` → **Welcome to nginx on Debian!**
4. SSH host key algorithm using SHA2-512 → **`rsa-sha2-512`**
---
