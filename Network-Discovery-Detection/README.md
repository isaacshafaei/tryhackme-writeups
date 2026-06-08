# Network Discovery Detection

in the below image you can see the different phases of attacks:
![attack phases](attack-phases.png)

# Linux Log Analysis Cheat Sheet

## Common Workflow

```bash
Inspect → Extract → Count → Identify Anomalies
```

---

# cut

Extract specific columns from CSV files.

### Syntax

```bash
cut -d ',' -fCOLUMN file.csv
```

| Option | Meaning |
|----------|----------|
| `-d ','` | Comma delimiter |
| `-f3` | Extract field/column 3 |
| `-f1-5` | Extract columns 1 to 5 |
| `-f2,5` | Extract columns 2 and 5 |

### Examples

```bash
cut -d ',' -f3 log-session-2.csv
```

Show source IPs.

```bash
cut -d ',' -f1-5 log-session-2.csv
```

Show first 5 columns.

---

# sort

Sort output.

```bash
sort
```

### Example

```bash
cut -d ',' -f3 log-session-2.csv | sort
```

Sort IP addresses.

---

# uniq

Remove duplicates.

```bash
uniq
```

### Count duplicates

```bash
uniq -c
```

### Example

```bash
cut -d ',' -f3 log-session-2.csv | sort | uniq -c
```

Count occurrences of each IP.

---

# Top Talkers / Scanners

Find the most active IPs.

```bash
cut -d ',' -f3 log-session-*.csv | sort | uniq -c | sort -nr
```

### Breakdown

```text
cut       → Extract IPs
sort      → Group identical IPs
uniq -c   → Count occurrences
sort -nr  → Highest count first
```

---

# grep

Search for specific values.

### Find an IP

```bash
grep '192.168.230.127' log-session-*.csv
```

### Count occurrences

```bash
grep -c '192.168.230.127' log-session-*.csv
```

### Show filename

```bash
grep -H '192.168.' log-session-*.csv
```

---

# Security Investigation Tips

## Internal IP Ranges

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Internal IPs may indicate:
- Internal scanning
- Lateral movement
- Compromised hosts

---

## Port Scanning Indicators

```text
One Source IP
      +
Many Destination Ports
      +
Many Hosts
      =
Possible Scanner
```

---

## Common Investigation Questions

### Find Internal Scanner

```bash
cut -d ',' -f3 log-session-*.csv | sort | uniq -c | sort -nr
```

Look for private IPs.

### Find External Scanner

```bash
cut -d ',' -f3 log-session-*.csv | sort | uniq -c | sort -nr
```

Look for public IPs.

### Count Scanner Activity

```bash
grep -c 'IP_ADDRESS' log-session-*.csv
```

### Show Scanner Connections

```bash
grep 'IP_ADDRESS' log-session-*.csv
```

---

# Analyst Mindset

```text
Identify Source IP
        ↓
Check Internal/External
        ↓
Count Activity
        ↓
Inspect Ports & Hosts
        ↓
Determine Scanning Behavior
```
---

## Port Scanning

**Horizontal Scan:** One source IP → Same port → Multiple hosts  
*Goal:* Find hosts exposing a specific service.

**Vertical Scan:** One source IP → Multiple ports → One host  
*Goal:* Discover services/vulnerabilities on a target.

**Remember:**
- Horizontal = Find vulnerable hosts
- Vertical = Enumerate target services

with below command we can find the Horizontal and Vertical scanning:
`cut -d ',' -f3,5,6 log-session-*.csv | sort`
---

### Network Scanning Types (Summary)

* **Ping Sweep (ICMP)**: Sends ICMP requests to discover live hosts. Works by receiving ICMP replies, but often blocked by firewalls.

* **TCP SYN Scan**: Uses the TCP handshake (SYN → SYN-ACK) to detect open ports and live hosts. Stealthy and harder to detect.

* **UDP Scan**: Sends UDP packets; ICMP “port unreachable” indicates closed ports. No response may mean open or filtered, making it slow and unreliable.

* **SOC Context**: Organizations use internal scans for asset discovery and vulnerability checks. SOC analysts should recognize scan sources, types, and schedules to reduce alert noise.

--------------

