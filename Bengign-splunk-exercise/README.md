#Bengin Splunk 

for this first question:
index="win_eventlogs"
| stats count by UserName


for Crhis:
index="win_eventlogs" ("schtasks.exe" OR "schtasks") AND HostName="HR_01" OR HostName="HR_02" OR HostName="HR_03"
| table UserName Severity ProcessID HostName CommandLine _time

for all rest questions
index="win_eventlogs" HostName="HR_*"
("certutil.exe" OR "bitsadmin.exe" OR "powershell.exe" OR "mshta.exe" OR "curl.exe" OR "wget.exe")
| table _time UserName HostName ProcessName CommandLine
---
below are more note about how can we reach to the answers:
Use **March 2022** as the time range first, then run these Splunk commands.

### 1. Count March logs

```spl
index=win_eventlogs
```

Check total events shown.

**Answer:** `13959`

---

### 2. Find imposter account

```spl
index=win_eventlogs
| top limit=20 UserName
```

Look for a username similar to a real one.

**Answer:** `Amel1a`

---

### 3. HR user running scheduled tasks

```spl
index=win_eventlogs HostName="HR_*" schtasks
| table _time UserName HostName CommandLine
```

Look for `schtasks` commands.

**Answer:** `Chris.fort`

---

### 4. HR user downloading payload

```spl
index=win_eventlogs HostName="HR_*" CommandLine="*http*"
| table _time UserName HostName CommandLine
```

Look for a LOLBIN downloading from internet.

**Answer:** `haroon`

---

### 5. LOLBIN used

Same result shows:

```text
certutil.exe
```

**Answer:** `certutil.exe`

---

### 6. Date executed

```spl
index=win_eventlogs UserName="haroon" CommandLine="*certutil*"
| table _time UserName CommandLine
```

Check `_time`.

**Answer:** `2022-03-04`

---

### 7. File-sharing site

Same `CommandLine` shows the URL:

```text
controlc.com
```

**Answer:** `controlc.com`

---

### 8. Downloaded file name

Same command line shows saved file:

```text
benign.exe
```

**Answer:** `benign.exe`

---

### 9. Full URL

```spl
index=win_eventlogs CommandLine="*controlc*"
| table CommandLine
```

**Answer:** `https://controlc.com/e4d11035`

Then open the URL to get the flag:

**Answer:** `THM{KJ&*H^B0}`

