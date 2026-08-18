### Short note

This room covers **advanced Nmap scans and evasion techniques**.

**Advanced scans:**

* `-sN` → Null scan (no TCP flags)
* `-sF` → FIN scan
* `-sX` → Xmas scan (FIN + PSH + URG)
* `-sM` → Maimon scan (FIN + ACK)
* `-sA` → ACK scan → mainly maps **firewall rules**
* `-sW` → Window scan
* `--scanflags` → custom TCP flags

**Evasion/spoofing:**

* `-S` → spoof source IP
* `--spoof-mac` → spoof MAC address
* `-D` → decoy scan
* `-f` / `-ff` → fragment packets
* `-sI` → Idle/Zombie scan

**More information:**

* `--reason` → explains why Nmap determined a state
* `-v` / `-vv` → more verbose output
* `-d` → debugging information

**Main idea:** These scans manipulate TCP flags to learn about **open/closed ports or firewalls** without performing a normal TCP connection.
---

