# NetworkMiner Cheat Sheet

## What It Does
- Parse PCAP files
- Identify protocols
- OS fingerprinting
- Extract files (images, HTML, emails)
- Extract credentials
- Find cleartext keywords
- Quick traffic overview

---

## Operating Modes

### Packet Parsing (Recommended)
- Analyze PCAPs
- Quick investigation
- Find "low-hanging fruit"

### Sniffer Mode
- Capture live traffic
- Windows only
- Less reliable than Wireshark

---

## Pros
- OS fingerprinting
- Easy file extraction
- Credential discovery
- Keyword discovery
- Fast overview of traffic

## Cons
- Weak live sniffing
- Limited filtering
- Not ideal for large PCAPs
- Limited deep packet analysis

---

## NetworkMiner vs Wireshark

| Feature | NetworkMiner | Wireshark |
|----------|------------|-----------|
| Purpose | Quick Overview | Deep Analysis |
| OS Fingerprinting | ✅ | ❌ |
| File Extraction | ✅ | ✅ |
| Credential Discovery | ✅ | ✅ |
| Filtering | Limited | ✅ |
| Protocol Analysis | ❌ | ✅ |
| Payload Analysis | ❌ | ✅ |
| Statistics | ❌ | ✅ |

---

## Best Practice

```text
Capture Traffic
      ↓
NetworkMiner (Quick Overview)
      ↓
Wireshark (Deep Investigation)
```
