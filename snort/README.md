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

