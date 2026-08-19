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

### Short note

**Null Scan `-sN`**

* Sets **0 flags**.
* No response → `open|filtered`
* RST → `closed`

**FIN Scan `-sF`**

* Sets **1 flag: FIN**
* No response → `open|filtered`
* RST → `closed`

**Xmas Scan `-sX`**

* Sets **3 flags: FIN + PSH + URG**
* No response → `open|filtered`
* RST → `closed`

### Key idea

```text
Null → 0 flags
FIN  → 1 flag
Xmas → 3 flags
Launch a FIN scan against the target VM. How many ports appear as open|filtered? 9
Repeat your scan launching a null scan against the target VM. How many ports appear as open|filtered? 9
```

These scans are mainly useful against **stateless firewalls**; stateful firewalls can usually detect/block them.
---
