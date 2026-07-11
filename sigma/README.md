## Sigma — Short Note

**Sigma** is an **open-source, vendor-independent detection rule language** for log analysis.

Think of it as:

```text
Sigma → Log files
Snort → Network traffic
YARA → Files/Malware
```

It lets analysts **write one generic detection rule** and convert it into queries for different SIEMs (e.g., Splunk, Elastic, Sentinel).

## Why Sigma?

| Use                 | Purpose                             |
| ------------------- | ----------------------------------- |
| Share detections    | Share rules with other analysts     |
| Vendor-independent  | Avoid SIEM vendor lock-in           |
| Threat intelligence | Share detections with the community |
| Custom detections   | Detect specific attacker behaviors  |

## Sigma Workflow

```text
Logs
   ↓
Sigma Rule (YAML)
   ↓
Sigma Converter
   ↓
SIEM Query (Splunk / Elastic / Sentinel / QRadar ...)
   ↓
Alerts
```

## Sigma Components

| Component           | Purpose                                 |
| ------------------- | --------------------------------------- |
| **Sigma Rule**      | Detection rule written in YAML          |
| **Sigma Converter** | Converts Sigma to SIEM-specific queries |
| **Machine Query**   | Final query executed by the SIEM        |

## Key Idea

```text
Write once → Convert → Use on multiple SIEMs
```

Sigma makes detection rules **portable, shareable, and easy to maintain** across different security platforms.
---

