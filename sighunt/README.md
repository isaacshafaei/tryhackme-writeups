# Sigma Rule Template (Complete)

Use this as your base template for most Windows process creation detections.

```yaml
title: <Rule Name>
id: <UUID>
status: experimental
description: <What does this rule detect?>
author: <Your Name>
date: YYYY-MM-DD
modified: YYYY-MM-DD

logsource:
  product: windows
  service: process_creation    # THM lab
  # category: process_creation # Standard Sigma

detection:
  selection:
    EventID: 1

    ParentImage|endswith: '\parent.exe'     # Optional
    Image|endswith: '\program.exe'

    CommandLine|contains:
      - 'argument1'
      - 'argument2'

    CommandLine|contains|all:
      - 'argument1'
      - 'argument2'

    Hashes|contains:
      - 'MD5=<hash>'
      - 'SHA256=<hash>'

    TargetFilename|endswith: '.txt'

  filter:
    User: SYSTEM

  condition: selection and not filter

fields:
  - Image
  - ParentImage
  - CommandLine
  - User
  - Hashes
  - TargetFilename

falsepositives:
  - Legitimate administrative activity

level: medium

tags:
  - attack.execution
  - attack.t1059
```

---

# Where each field is used

| Field            | Purpose                      | Example                                |
| ---------------- | ---------------------------- | -------------------------------------- |
| `title`          | Rule name                    | Netcat Execution                       |
| `id`             | UUID                         | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `status`         | Rule maturity                | experimental, stable                   |
| `description`    | Detection purpose            | Detect Netcat reverse shell            |
| `author`         | Rule writer                  | Isaac                                  |
| `date`           | Creation date                | 2026-07-15                             |
| `modified`       | Last update                  | 2026-07-15                             |
| `logsource`      | Which logs to search         | Windows Process Creation               |
| `detection`      | Detection logic              | selection/filter                       |
| `fields`         | Important fields after alert | CommandLine, User                      |
| `falsepositives` | Possible legitimate matches  | Admin usage                            |
| `level`          | Severity                     | low, medium, high, critical            |
| `tags`           | MITRE ATT&CK mapping         | attack.t1105                           |

---

# Detection Section

This is the **most important part**.

Everything inside **one selection** is **AND**.

Example:

```yaml
selection:
    Image|endswith: '\certutil.exe'
    CommandLine|contains:
        - '-urlcache'
```

means

```text
Image = certutil.exe
AND
CommandLine contains -urlcache
```

---

## Multiple selections

Example

```yaml
selection1:
    Image|endswith: '\nc.exe'

selection2:
    Hashes|contains:
        - 'MD5=xxxx'

condition: selection1 or selection2
```

means

```text
(Netcat executed)

OR

(Known malicious Netcat hash)
```

---

# Most useful Sigma modifiers

| Modifier     | Meaning       | Example           |                           |
| ------------ | ------------- | ----------------- | ------------------------- |
| `contains`   | String exists | `CommandLine      | contains: 'powershell'`   |
| `contains    | all`          | All strings exist | `- '-e'` + `- 'cmd.exe'`  |
| `endswith`   | Ends with     | `Image            | endswith: '\cmd.exe'`     |
| `startswith` | Starts with   | `Image            | startswith: 'C:\Windows'` |
| `re`         | Regex         | `CommandLine      | re:`                      |
| `exists`     | Field exists  | `Hashes           | exists: true`             |

---

# Common Sysmon Event IDs

|  Event ID | Meaning            | Common Sigma logsource |
| --------: | ------------------ | ---------------------- |
|     **1** | Process Creation   | process_creation       |
|     **3** | Network Connection | network_connection     |
|     **6** | Driver Loaded      | driver_load            |
|     **7** | Image/DLL Loaded   | image_load             |
|    **10** | Process Access     | process_access         |
|    **11** | File Creation      | file_event             |
| **12-14** | Registry Events    | registry_*             |
|    **22** | DNS Query          | dns_query              |

---

# THM Sighunt Detection Summary

| Challenge         | Detect                | Important Fields                          |
| ----------------- | --------------------- | ----------------------------------------- |
| HTA Payload       | Browser → mshta       | ParentImage, Image                        |
| Certutil Download | File download         | Image, CommandLine                        |
| Netcat            | Reverse shell         | Image, CommandLine, Hashes                |
| PowerUp           | Privilege enumeration | Image, CommandLine                        |
| Service Binary    | Service hijacking     | Image, CommandLine (`config`, `binPath=`) |
| RunOnce           | Registry persistence  | Image, CommandLine                        |
| 7z                | Password archive      | Image, CommandLine                        |
| Curl              | Data exfiltration     | Image, CommandLine                        |
| Ransomware        | File creation         | EventID 11, TargetFilename                |

---

# How to think like a Detection Engineer

Before writing any Sigma rule, answer these questions:

### 1. What behavior am I detecting?

Example:

```text
Netcat reverse shell
```

---

### 2. Which log contains it?

Example:

```text
Sysmon Event ID 1
```

because a new process starts.

---

### 3. Which process?

```yaml
Image|endswith: '\nc.exe'
```

---

### 4. Which arguments make it malicious?

```yaml
CommandLine|contains|all:
    - '-e'
    - 'cmd.exe'
```

---

### 5. Any parent process?

```yaml
ParentImage|endswith: '\mshta.exe'
```

---

### 6. Any IOC?

```yaml
Hashes|contains:
    - 'MD5=...'
```

---

### 7. Any whitelist?

```yaml
filter:
    User: SYSTEM
```

---

### 8. Final condition

```yaml
condition: selection and not filter
```

---

# Detection Engineering Workflow

```text
Threat Intelligence
        │
        ▼
Understand the attacker behavior
        │
        ▼
Choose the correct log source
        │
        ▼
Choose the correct Event ID
        │
        ▼
Choose the important fields
(Image, ParentImage, CommandLine, Hashes, TargetFilename...)
        │
        ▼
Write the Sigma rule
        │
        ▼
Convert Sigma (pySigma / Uncoder)
        │
        ▼
Run the query in SIEM (Elastic, Splunk, Sentinel...)
        │
        ▼
Validate hits
        │
        ▼
Tune false positives
        │
        ▼
Deploy to production
```

This workflow is what you'll repeatedly use in real SOC and detection engineering work, not just in the THM Sighunt lab.
---------
this is and example of which we can use(the most important part is selection which is dynamic based on examples and needs):
```
title: Sighunt
id: 232c5562-f775-4ad4-a162-816c99b013a6
status: rule testing 
description: Curl Data
author: lordofficial
date: 06/02/2023
modified: 06/02/23
logsource:
  product: windows
  service: process_creation
detection:
  selection:
    EventID: 1 
 
    Image|endswith: '\curl.exe'
    CommandLine|contains: 
     - ' -d '
 
  
   
  condition: selection
```
