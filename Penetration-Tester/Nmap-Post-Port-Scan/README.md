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

