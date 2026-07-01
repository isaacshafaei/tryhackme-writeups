## Splunk Search & Reporting App

* Default Splunk interface for searching and analyzing logs.
* Main features:

  * **Search Head:** where analysts write queries.
  * **Time Picker:** selects search time range.
  * **Search History:** stores previous searches.
  * **Data Summary:** shows available hosts, sources, and sourcetypes.

## First Search

* Use the `windowslogs` index.
* Example query:

```spl
index=windowslogs
```

* Set time range to **All time**.
* Both formats are valid:

```spl
index=windowslogs
index = windowslogs
```

## Fields Sidebar

* Located on the left side of Splunk search results.
* Helps analysts explore extracted fields.

Key parts:

* **Selected Fields:** default fields shown in results.
* **Interesting Fields:** useful fields Splunk detects.
* **Numeric Fields `#`:** fields with numbers.
* **Alpha-numeric Fields `α`:** fields with text/string values.
* **Count:** number of events containing that field.
* **More Fields:** shows additional available fields.
---
## Splunk SPL Short Notes

### What is SPL?

* **SPL = Search Processing Language**
* Used to search, filter, transform, and analyze Splunk logs.

---

### Free Text Search

Search for any event containing a keyword:

```spl
index=windowslogs alice
```

* Finds events containing `alice`
* Case-insensitive
* Useful when field names are unknown

---

### Relational Operators

| Operator | Meaning       | Example                |
| -------- | ------------- | ---------------------- |
| `=`      | equals        | `UserName=Mark`        |
| `!=`     | not equal     | `UserName!=Mark`       |
| `<`      | less than     | `Age<10`               |
| `<=`     | less/equal    | `Age<=10`              |
| `>`      | greater than  | `Outbound_Traffic>50`  |
| `>=`     | greater/equal | `Outbound_Traffic>=50` |

Example:

```spl
index=windowslogs AccountName!=SYSTEM
```

Shows events where `AccountName` is not `SYSTEM`.

---

### Logical Operators

| Operator | Meaning                             | Example                                    |
| -------- | ----------------------------------- | ------------------------------------------ |
| `NOT`    | field/value does not exist or match | `NOT UserName=*`                           |
| `AND`    | both conditions must match          | `UserName=David AND IPAddress=10.10.10.10` |
| `OR`     | either condition can match          | `UserName=David OR UserName=John`          |
| `IN`     | shorter OR alternative              | `UserName IN(David, John)`                 |

`AND` can be implied:

```spl
index=windowslogs AccountName!=SYSTEM AccountName=James
```

Same as:

```spl
index=windowslogs AccountName!=SYSTEM AND AccountName=James
```

---

### Wildcards and CIDR

| Search                        | Meaning                                      |
| ----------------------------- | -------------------------------------------- |
| `status=*fail*`               | matches `failed`, `failure`, `appfail`, etc. |
| `DestinationIp=172.*`         | matches IPs starting with `172.`             |
| `DestinationIp=172.18.0.0/16` | matches IPs inside that subnet               |

---

### Quotes

Use quotes for exact phrases:

```spl
index=windowslogs failed login
```

Finds `failed` and `login` anywhere.

```spl
index=windowslogs "failed login"
```

Finds exact phrase `failed login`.

Quotes can also escape operators:

```spl
index=windowslogs "TO BE OR NOT TO BE"
```

---

### Parentheses

Use parentheses to control logic order.

Wrong/unclear:

```spl
index=windowslogs alice AND bob OR charlie
```

Splunk may treat it like:

```spl
index=windowslogs alice AND (bob OR charlie)
```

Correct:

```spl
index=windowslogs (alice AND bob) OR charlie
```
---
## Splunk Filtering Commands Short Notes

### Pipes `|`

* Used to connect SPL commands.
* Output of one command becomes input for the next.

---

### `fields`

Used to include or exclude fields.

```spl
index=windowslogs 
| fields host User SourceIp
```

Shows only selected fields.

Exclude a field:

```spl
| fields - SourceIp
```

---

### `dedup`

Removes duplicate results based on a field.

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| dedup SourceIp
```

Returns one event per unique `SourceIp`.

Useful for:

* Removing repeated events
* Cleaning noisy logs
* Finding unique values

---

### `rename`

Renames fields for better readability.

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| rename User as Employee
```

Useful for reports and cleaner output.

Also useful for JSON/XML fields:

```spl
index=jsondata
| rename request.* as *
```

Example:

* `request.path` → `path`
* `request.ip` → `ip`

---

### `regex`

Filters results using regular expressions.

```spl
index=windowslogs 
| regex Image="\.exe$"
```

Shows events where `Image` ends with `.exe`.

Useful for:

* Pattern matching
* Poorly parsed logs
* Complex searches
* Finding specific file types or formats
----
## SPL Structuring Commands Short Notes

### Purpose

* SPL structuring commands help organize raw logs.
* Useful for filtering, ordering, formatting, and building timelines.

---

### `table`

Displays selected fields in a clean table.

```spl
index=windowslogs 
| table _time EventID Hostname SourceName
```

Useful for:

* Timelines
* Host/user investigations
* Comparing key fields

---

### Useful Structuring Commands

| Command   | Example                          | Purpose                      |
| --------- | -------------------------------- | ---------------------------- |
| `head`    | `index=windowslogs \| head 20`   | Shows first/newest 20 events |
| `tail`    | `index=windowslogs \| tail 20`   | Shows last/oldest 20 events  |
| `sort`    | `index=windowslogs \| sort User` | Sorts results by a field     |
| `reverse` | `index=windowslogs \| reverse`   | Reverses event order         |

---

### Timeline Example

```spl
index=windowslogs Hostname=Salena.Adam
| table _time Hostname EventID Category
| reverse
```

* Shows events for one host in chronological order.
* Helps reconstruct what happened step by step.

---

### Subsearches

Used to correlate data from different event types.

Example: connect Sysmon process creation with Windows logon context.

```spl
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

---

### How It Works

* Subsearch runs first.
* It finds `EventID=4624` logon events.
* Renames `TargetLogonId` to `LogonId`.
* Main search finds `EventID=1` Sysmon process events.
* `join` matches both datasets using `LogonId`.
* Final table shows process + logon details together.

---

### Key Note

* Subsearches are powerful but slow on large datasets.
* Better alternatives for large data: `stats` and `eval`.
* Use `join` when enriching one data source with fields from another.
---
## Splunk Transforming Commands Short Notes

### Purpose

* Transforming commands convert raw logs into summaries, statistics, and visualizations.
* Useful for finding trends, anomalies, and patterns.

---

### `top`

Shows most frequent values.

```spl
index=windowslogs | top User limit=5
```

---

### `rare`

Shows least frequent values.

```spl
index=windowslogs | rare User limit=5
```

---

### `highlight`

Highlights keywords/fields in raw logs.

```spl
index=windowslogs | highlight User EventID Image "Process accessed"
```

---

### `stats`

Calculates statistics.

Common functions:

* `avg()` = average
* `max()` = maximum
* `min()` = minimum
* `sum()` = total
* `count` = number of events

Example:

```spl
index=windowslogs | stats count by EventID | sort EventID
```

---

### `chart`

Creates table output for visualization.

```spl
index=windowslogs | chart count by User
```

---

### `timechart`

Shows changes over time.

```spl
index=windowslogs Image!="" 
| timechart span=30m count by Image limit=5
```

Useful for spotting:

* spikes
* trends
* anomalies over time

---

### `iplocation`

Adds geographic info to IP addresses.

```spl
index=windowslogs 
| iplocation SourceIp 
| stats count by Country
```

Adds fields like:

* City
* Region
* Country

---

### `lookup`

Enriches logs using external CSV/lookup tables.

```spl
index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole
| stats count by Hostname UserRole
```

---

### `eval`

Creates or modifies fields.

```spl
index=windowslogs
| eval LogonTypeDesc = case(LogonType == 3, "Network Logon", LogonType == 5, "Service")
| stats count by LogonType LogonTypeDesc
```

Meaning:

* `LogonType 3` = Network Logon
* `LogonType 5` = Service

---

### Answers

* Most common `Image`: **C:\windows\system32\svchost.exe**
* Source IP region: **California**
* Highest `RiskScore` image: **C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe**
---
## Anomaly Detection Short Notes

### Purpose

* Used to find **outliers** in large log datasets.
* Example: suspicious VPN logins based on country or login time.

---

## Detecting Outliers by Country

Goal: find users logging in from unusual countries.

```spl
index=vpnlogs
| eventstats count as logins_by_user by user 
| eventstats count as logins_by_user_country by user src_country 
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```

### Meaning

* Count total logins per user.
* Count logins per user and country.
* Calculate how often each country appears for that user.
* Show rare country logins below threshold `0.1`.

### Key Point

* Rare login countries may indicate:

  * VPN usage
  * compromised account
  * suspicious access

---

## Detecting Outliers by Hour

Goal: find users logging in at unusual times.

```spl
index=vpnlogs
| eval hour=tonumber(strftime(_time, "%H")) + tonumber(strftime(_time, "%M"))/60
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user
| eval zscore=abs(hour - typical_hour) / stdev_hour
| where zscore > 3
| eval hour=round(hour, 2), typical_hour=round(typical_hour, 2)
| eval stdev_hour=round(stdev_hour, 2), zscore=round(zscore, 2)
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
```

### Meaning

* `typical_hour`: average login time for the user.
* `stdev_hour`: how predictable the user’s login time is.
* `zscore`: how unusual the login time is.
* `zscore > 3` means highly anomalous.

### Example

* User usually logs in around `13:30`.
* Login at `18:30` may be suspicious.

---

## Impossible Travel / ML

* More advanced anomaly detection can use:

  * `iplocation`
  * threat intel `lookup`
  * `fit` and `apply` ML commands
* Useful for detecting impossible travel and future outliers.

