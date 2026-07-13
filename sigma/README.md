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
## Sigma Rule Syntax — Short Note

Sigma rules are written in **YAML** and describe what log events should trigger an alert. 

## YAML Basics

* Case-sensitive
* File extension: `.yml`
* Use **spaces**, not tabs
* `#` → comment
* `:` → key-value pair
* `-` → list item

## Main Sigma Rule Fields

| Field              | Purpose                                 |
| ------------------ | --------------------------------------- |
| **title**          | Rule name                               |
| **id**             | Unique UUID                             |
| **status**         | Rule maturity                           |
| **description**    | What the rule detects                   |
| **logsource**      | Log source (product, category, service) |
| **detection**      | Detection logic (required)              |
| **falsepositives** | Known benign matches                    |
| **level**          | Severity (Informational → Critical)     |
| **tags**           | MITRE ATT&CK, CVEs, categories          |

## Status Values

| Status           | Meaning                        |
| ---------------- | ------------------------------ |
| **Stable**       | Production ready               |
| **Test**         | Under testing                  |
| **Experimental** | May be noisy / false positives |
| **Deprecated**   | Replaced by another rule       |
| **Unsupported**  | Not usable                     |

## Detection Structure

Every Sigma rule has:

```text id="6xtwz6"
Search Identifiers
        +
Condition Expression
```

Example:

```yaml
selection:
  EventID:
    - 19
    - 20
condition: selection
```

## Search Identifiers

| Type                    | Logic   |
| ----------------------- | ------- |
| **Lists** (`-`)         | **OR**  |
| **Maps** (`key: value`) | **AND** |

## Common Value Modifiers

| Modifier     | Purpose                            |
| ------------ | ---------------------------------- |
| `contains`   | Value appears anywhere             |
| `startswith` | Begins with value                  |
| `endswith`   | Ends with value                    |
| `all`        | All listed values must match (AND) |
| `base64`     | Match Base64-encoded values        |
| `re`         | Regular expression                 |

Example:

```yaml
Image|endswith: cmd.exe
CommandLine|contains: powershell
```

## Condition Examples

```text id="gs4d4q"
selection
selection and filter
selection and not filter
1 of selection*
all of selection*
```

## Key Points

```text id="74d3k4"
Lists = OR
Maps = AND
Detection = Search Identifiers + Condition
Modifiers refine matching (contains, startswith, endswith, regex)
```

## Questions

| Question                              | Answer                                         |
| ------------------------------------- | ---------------------------------------------- |
| Which status may be noisy but useful? | **Experimental**                               |
| Detection consists of?                | **Search Identifiers + Condition Expressions** |
| Search identifier data structures?    | **Lists and Maps**                             |
---
