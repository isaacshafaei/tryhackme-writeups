# Wireshark Traffic Analysis: Detecting Nmap Scans

> A quick reference guide for identifying common Nmap network mapping signatures using Wireshark display filters.

---

## Core TCP Flag Filters

| Flag(s) | Decimal Filter | Boolean Filter |
| :--- | :--- | :--- |
| **SYN only** | `tcp.flags == 2` | `tcp.flags.syn == 1` |
| **ACK only** | `tcp.flags == 16` | `tcp.flags.ack == 1` |
| **SYN, ACK** | `tcp.flags == 18` | `(tcp.flags.syn == 1) and (tcp.flags.ack == 1)` |
| **RST only** | `tcp.flags == 4` | `tcp.flags.reset == 1` |
| **RST, ACK** | `tcp.flags == 20` | `(tcp.flags.reset == 1) and (tcp.flags.ack == 1)` |
| **FIN only** | `tcp.flags == 1` | `tcp.flags.fin == 1` |

---

## Scan Signatures & Detection

### 1. TCP Connect Scan (`-sT`)
* **Behavior:** Completes the full 3-way handshake. Executed by non-privileged users.
* **Signature:** Window size is typically `> 1024` bytes (expects data).
* **Wireshark Filter:** 
```wireshark
  tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024
