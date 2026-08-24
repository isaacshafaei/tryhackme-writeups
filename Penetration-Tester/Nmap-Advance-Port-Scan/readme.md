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
### Short note — Maimon Scan

* **Option:** `-sM`
* Sets **2 flags:** `FIN + ACK`
* Some older BSD systems may behave differently and reveal open ports.
* Most modern systems respond with **RST** for both open and closed ports, so it's usually unreliable.
* Mainly useful for understanding TCP scanning techniques.

**Answer:** `2` flags.
---
## TCP ACK / Window / Custom Scans

* **ACK Scan:** `nmap -sA <target>`

  * Sends **ACK** → receives **RST**.
  * Cannot detect open ports.
  * Used mainly to **map firewall rules**.
  * `unfiltered` = firewall allows traffic.

* **Window Scan:** `nmap -sW <target>`

  * Similar to ACK scan.
  * Checks the **TCP Window field** in RST packets.
  * Can sometimes identify open/closed ports.

* **Custom Scan:** `nmap --scanflags <flags> <target>`

  * Example: `--scanflags RSTSYNFIN`
  * Lets you choose custom TCP flags.

**Key:** ACK/Window scans reveal **firewall behavior**, not necessarily running services.
**Questions**
* **TCP Window Scan flags:** 1 (ACK)
* **Reset flag:** `R`
* **ACK scan unfiltered ports:** 5

---
## Spoofing & Decoy Scans

* **IP Spoofing:** `-S SPOOFED_IP`

  * Makes packets appear to come from another IP.
  * Need `-e INTERFACE -Pn`.
  * Must be able to **capture the replies**.

* **MAC Spoofing:** `--spoof-mac SPOOFED_MAC`

  * Works when attacker and target are on the **same network**.

* **Decoy Scan:** `-D IP1,IP2,ME`

  * Makes the scan appear to come from multiple IPs.
  * `ME` = your real IP.

### Answers

1. Spoof source IP:
   `-S 10.10.10.11`

2. Add two decoys:
   `-D 10.10.20.21,10.10.20.28,ME`
---
## Firewall, IDS & Fragmentation

* **Firewall:** Allows or blocks network traffic based on rules.
* **IDS:** Inspects traffic for malicious patterns and raises alerts.
* **Fragmentation:** `-f` splits packets into **8-byte fragments**.

  * `-ff` = **16-byte fragments**.
  * `--mtu NUM` = custom fragment size (**must be a multiple of 8**).
* **Extra data:** `--data-length NUM` adds bytes to packets.

### Question

**64-byte TCP segment with `-ff` (16 bytes):**

**64 ÷ 16 = 4 IP fragments**
---
## Nmap Reasons & Verbosity

* **`--reason`** → Shows why Nmap considers a host/port open or closed.

  * Open port → usually `syn-ack`
  * Host up → e.g. `arp-response`
* **`-v`** → Verbose output.
* **`-vv`** → More verbose.
* **`-d` / `-dd`** → Debugging details.

**Command:**

```bash
nmap -sS -F --reason MACHINE_IP
```

**Answer:** `syn-ack` ✅
