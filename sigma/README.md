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
## Sigma Rule Writing & Conversion — Short Note

Write Sigma rules based on **threat intelligence** and convert them to your SIEM query language. 

## Rule Writing Process

| Step                 | Purpose                                                   |
| -------------------- | --------------------------------------------------------- |
| **1. Analyze Intel** | Extract IOCs, commands, paths, registry keys, persistence |
| **2. Rule Info**     | Title, description, status                                |
| **3. Log Source**    | Select product & log category                             |
| **4. Detection**     | Define search fields + condition                          |
| **5. Metadata**      | Add severity, ATT&CK tags, references, false positives    |

## Detection Example

```yaml id="9vwqnt"
CommandLine|contains|all:
  - '--install'
  - '--start-with-win'
CurrentDirectory|contains:
  - 'C:\ProgramData\AnyDesk.exe'
condition: selection
```

## Sigma Conversion

Sigma rules must be converted before using them in a SIEM.

| Tool           | Purpose                                                      |
| -------------- | ------------------------------------------------------------ |
| **Sigmac**     | Convert Sigma → Splunk, Elastic, QRadar, etc. *(deprecated)* |
| **pySigma**    | Modern replacement for Sigmac                                |
| **Uncoder.io** | Online Sigma converter                                       |

### Sigmac Example

```bash id="0jgdq8"
python3.9 sigmac -t splunk -c splunk-windows rule.yml
```

## Workflow

```text id="t4wejp"
Threat Intel
      ↓
Write Sigma Rule
      ↓
Convert (Sigmac / pySigma / Uncoder)
      ↓
SIEM Query
      ↓
Search Logs & Validate
```

## Key Points

* Build rules from **real threat intelligence**
* Detect attacker **behavior**, not just IOCs
* Convert Sigma to your SIEM before deployment
* Test and tune queries to reduce false positives
---
## Sigma Investigation Notes — Short

Different SIEMs and converters may generate **different queries** from the same Sigma rule because each SIEM uses different field names and syntax. 

### Key Points

| Concept              | Meaning                                                                           |
| -------------------- | --------------------------------------------------------------------------------- |
| **Same Sigma Rule**  | Can produce different SIEM queries                                                |
| **Sigmac / pySigma** | Converts Sigma to SIEM-specific queries                                           |
| **Uncoder.io**       | Online Sigma converter                                                            |
| **Field Mapping**    | Different SIEMs use different field names (e.g., `Image` vs `process.executable`) |
| **Query Tuning**     | You may need to adjust regex, escaping, or field names for your environment       |

### Example

```text id="kz9w1l"
Sigma Rule
      ↓
Sigmac → Image + CommandLine
Uncoder → process.executable + process.command_line
```

Both detect the **same behavior**, but the generated queries differ.

### Key Idea

```text id="a34w8v"
Sigma is portable, but the converted query may need tuning to match your SIEM's field names and syntax.
```
---
