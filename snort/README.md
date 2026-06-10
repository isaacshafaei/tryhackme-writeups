### IDS, IPS & Snort Overview

* **IDS (Intrusion Detection System):** Monitors traffic and generates alerts for suspicious activity.

  * **NIDS:** Monitors network traffic.
  * **HIDS:** Monitors a single host.

* **IPS (Intrusion Prevention System):** Detects and actively blocks malicious activity.

  * **NIPS:** Protects network traffic.
  * **HIPS:** Protects a single host.
  * **NBA:** Detects anomalies based on learned behavior.
  * **WIPS:** Protects wireless networks.

**Detection Methods:**

* **Signature-Based:** Detects known threats.
* **Behavior-Based:** Detects unknown/anomalous threats.
* **Policy-Based:** Detects policy violations.

### Snort

**Snort** is an open-source, rule-based **NIDS/NIPS** developed by Martin Roesch and maintained by Cisco Talos.

**Features:**

* Traffic analysis
* Attack detection
* Packet logging
* Protocol analysis
* Real-time alerts
* Cross-platform support

**Modes:**

1. **Sniffer** – View packets.
2. **Packet Logger** – Save packet data.
3. **NIDS/NIPS** – Detect or block malicious traffic using rules.
---
#Operation 1: Sniffer Mode
### Snort Sniffer Mode

Snort can capture and display network traffic using different flags:

| Flag         | Description                            |
| ------------ | -------------------------------------- |
| `-v`         | Verbose output (packet headers)        |
| `-d`         | Display packet payload data            |
| `-e`         | Display link-layer (Ethernet) headers  |
| `-X`         | Full packet dump in HEX and ASCII      |
| `-i <iface>` | Listen on a specific network interface |

**Examples**

```bash
snort -v              # Verbose mode
snort -d              # Show packet payloads
snort -de             # Show payloads + Ethernet headers
snort -X              # Full packet dump
snort -v -i eth0      # Sniff on interface eth0
```

Flags can be combined (`-vd`, `-de`, `-vde`) to provide more packet details.
---
#2-packet Logger Mode
### Snort Logger Mode

Snort can capture and save network traffic for later analysis.

| Flag        | Description                                          |
| ----------- | ---------------------------------------------------- |
| `-l <dir>`  | Save logs to a directory (default: `/var/log/snort`) |
| `-K ASCII`  | Save logs in human-readable ASCII format             |
| `-r <file>` | Read and analyze a saved log/PCAP file               |
| `-n <num>`  | Process only a specified number of packets           |

**Examples**

```bash
snort -dev -l .                 # Log packets in binary format
snort -dev -K ASCII -l .        # Log packets in readable ASCII format
snort -r snort.log             # Read a saved log file
snort -r snort.log icmp        # Show only ICMP packets
snort -r snort.log tcp         # Show only TCP packets
snort -r snort.log 'udp and port 53'
snort -dvr snort.log -n 10     # Read first 10 packets
```

**Notes**

* Binary logs (`snort.log.*`) can be opened with Snort, tcpdump, or Wireshark.
* ASCII logs are human-readable and organized by IP/protocol.
* Running Snort with `sudo` creates logs owned by `root`.

For the exercise question, inspect the generated directory:

```bash
cd 145.254.160.237
ls
```

Look for a file named similar to:

```bash
UDP:<source-port>-53
```

The number before `-53` is the source port used to connect to DNS port 53.

---
#Operation Mode 3: IDS/IPS
## Snort IDS/IPS Mode (Quick Guide)

Snort can run in IDS/IPS mode to inspect traffic and apply rules-based actions (alert, log, drop).

### Basic usage

```bash
sudo snort -c /etc/snort/snort.conf
```

### Key options

* `-c <file>` → config file
* `-T` → test config
* `-N` → disable logging
* `-D` → run in background (daemon)
* `-A <mode>` → alert mode
* `-q` → quiet output
* `-Q --daq afpacket` → IPS mode (inline packet handling)

### Alert modes (`-A`)

* `console` → live alerts in terminal
* `cmg` → full packet + payload (hex/text)
* `fast` → short alert + IP/ports + timestamp
* `full` → detailed alert info
* `none` → no alerts (logs only)

### Rule example

```snort
alert icmp any any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)
```

### Common run examples

Test config:

```bash
sudo snort -c /etc/snort/snort.conf -T
```

Console alerts:

```bash
sudo snort -c /etc/snort/snort.conf -A console
```

Fast alerts:

```bash
sudo snort -c /etc/snort/snort.conf -A fast
```

Full alerts:

```bash
sudo snort -c /etc/snort/snort.conf -A full
```

No alerts:

```bash
sudo snort -c /etc/snort/snort.conf -A none
```

Background mode:

```bash
sudo snort -c /etc/snort/snort.conf -D
```

Rules-only mode:

```bash
sudo snort -c /etc/snort/rules/local.rules -A console
```

IPS mode (inline):

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1
```

### Summary

* IDS = detect + alert/log
* IPS = detect + block/drop traffic
* Behavior depends on rules + alert mode

### Note

Use traffic generators to trigger rules (ICMP/HTTP) and verify alerts/logs.
---
#pcap Investigation 4
## PCAP Analysis with Snort

Snort can analyze `.pcap` files using rules to detect suspicious traffic and generate alerts.

### PCAP Options

| Option                  | Description                              |
| ----------------------- | ---------------------------------------- |
| `-r <file>`             | Analyze a single PCAP                    |
| `--pcap-list="<files>"` | Analyze multiple PCAPs                   |
| `--pcap-show`           | Display the current PCAP being processed |

### Examples

Analyze a single PCAP:

```bash
sudo snort -c /etc/snort/snort.conf -q -r icmp-test.pcap -A console
```

Analyze multiple PCAPs:

```bash
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console
```

Show PCAP names during analysis:

```bash
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show
```

### Summary

* Use `-r` for a single capture file.
* Use `--pcap-list` for multiple capture files.
* Use `--pcap-show` to identify which PCAP generated each alert.
* Alerts are generated according to the loaded Snort rules.
---
```markdown
# Snort IDS/IPS Cheat Sheet

## 📝 Rule Structure
`action protocol src_ip src_port direction dst_ip dst_port (options)`

**Example:**
`alert icmp any any <> any any (msg:"ICMP Found"; sid:1000001; rev:1;)`

---

## ⚙️ Core Components

| Category | Options |
| :--- | :--- |
| **Actions** | `alert`, `log`, `drop`, `reject` |
| **Protocols** | `IP`, `TCP`, `UDP`, `ICMP` |
| **Direction** | `->` (Unidirectional), `<>` (Bidirectional) |

---

## 🔍 Detection Options

### Payload & Matching
* `content:"string"`: Match specific payload content.
* `nocase;`: Case-insensitive search.
* `fast_pattern;`: Optimize matching performance.

### Non-Payload Attributes
* `flags:S;`: Detect SYN packets.
* `flags:PA;`: Detect Push-Ack packets.
* `id:123456;`: Match specific IP ID.
* `dsize:100<>300;`: Filter by payload size.
* `sameip;`: Detect source and destination IP matching.

---

## 🚀 Common Analysis Commands

**1. Run Snort with Rules:**
`sudo snort -c local.rules -A fast -l . -r <file.pcap>`

**2. BPF Filter (CLI Level):**
`snort -vd -r <file.pcap> '<filter>'`
*(e.g., `tcp port 80` or `ip[4:2] == 35369`)*

**3. Analyze Detections:**
`grep "<msg_text>" alert | wc -l`

---

## 🛠️ Configuration
* **Local Rules File:** `/etc/snort/rules/local.rules`
* **Workflow:** Clear alert file (`> alert`) → Modify rules → Run Snort → Count detections.

```
---

```markdown
# Snort IDS/IPS Setup & Documentation

## 🏗️ Architecture Overview
* **Packet Decoder:** Collects and prepares packets.
* **Pre-processors:** Modifies packets for the detection engine.
* **Detection Engine:** The heart of Snort; applies rules to analyze traffic.
* **Logging & Alerting:** Generates security events.
* **Outputs & Plugins:** Integrates with external systems (Syslog, SQL).

---

## ⚙️ Configuration (`snort.conf`)
* **Network Variables:**
    * `HOME_NET`: Your internal network (protected).
    * `EXTERNAL_NET`: External traffic (usually `any` or `!$HOME_NET`).
* **DAQ (Data Acquisition):**
    * `PCAP`: Default Sniffer mode.
    * `Afpacket`: Inline IPS mode (best for blocking/inline deployment).
* **Rule Paths:** Store custom rules in `/etc/snort/rules/local.rules`. 
    * *Tip: Never replace the base `snort.conf`; edit manually or use update tools.*

---

## 📜 Snort Rules Cheat Sheet

### 1. Rule Structure
`action protocol src_ip src_port direction dst_ip dst_port (options)`

* **Example:** `alert icmp any any <> any any (msg:"ICMP Found"; sid:1000001; rev:1;)`

### 2. Components
* **Actions:** `alert`, `log`, `drop`, `reject`
* **Direction:** `->` (Unidirectional), `<>` (Bidirectional)

### 3. Detection Options
| Type | Option | Purpose |
| :--- | :--- | :--- |
| **Payload** | `content:"string"` | Match specific payload |
| **Non-Payload**| `flags:S;` | Detect SYN packets |
| | `flags:PA;` | Detect Push-Ack packets |
| | `id:123456;` | Match IP ID |
| | `dsize:100<>300;` | Filter by payload size |
| | `sameip;` | Detect source/dest IP match |

---

## 🚀 Analysis Workflow

**1. Run Snort with Rules:**
`sudo snort -c local.rules -A fast -l . -r <file.pcap>`

**2. BPF Filtering (Command Line):**
`snort -vd -r <file.pcap> '<filter>'`
*(e.g., `'tcp port 80'` or `'ip[4:2] == 35369'`)*

**3. Analyze/Count Detections:**
`grep "<msg_text>" alert | wc -l`

---

## 💡 Rule Management
* **`sid`**: Unique ID (Never change this).
* **`rev`**: Revision number (Increment this when modifying a rule).
* **Rule Sets:**
    * **Community:** Free (GPLv2).
    * **Registered:** Free (requires registration).
    * **Subscriber:** Paid (updated twice weekly).



```
in this folder there is a guide of Snort in pdf file
