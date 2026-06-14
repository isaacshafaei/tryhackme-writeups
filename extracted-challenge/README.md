**CTF Writeup – Extracted (Short Note)**

### Goal

Investigate a **PCAP network capture**, recover the **KeePass master password**, open the database, and retrieve the flag.

### What Happened (Attack Flow)

1. **Traffic analysis in Wireshark**

   * Used **Statistics → Conversations** to identify suspicious traffic.
   * Found large transfers from:

     * `10.10.45.95 → 10.10.94.106`
     * Ports **1337, 1338, 1339**

2. **Malicious PowerShell discovered**

   * HTTP GET downloaded a `.ps1` script.
   * Extracted via **File → Export Objects → HTTP**.

3. **PowerShell attack behavior**

   * Downloaded **ProcDump** (Sysinternals tool).
   * Checked if **KeePass** was running.
   * Created a **memory dump (1337.dmp)** of the KeePass process.
   * Located **Database1337.kdbx**.
   * Applied:

     * **XOR encryption**

       * `0x41` → memory dump
       * `0x42` → database
     * **Base64 encoding**
   * Exfiltrated files to attacker:

     * Port **1337 → memory dump**
     * Port **1338 → KeePass database**

---

### Investigation Process

#### Recover files

Extract exfiltrated data from PCAP:

```bash
tshark -r traffic.pcapng -Y "tcp.dstport == 1337" -T fields -e data | xxd -r -p > dump.b64
tshark -r traffic.pcapng -Y "tcp.dstport == 1338" -T fields -e data | xxd -r -p > db.b64
```

#### Reverse attacker protection

* **Base64 decode**
* **XOR decrypt**
* Restore:

  * `1337.dmp`
  * `Database1337.kdbx`

#### Recover KeePass password

* Identified **CVE-2023-32784**
* Exploit extracts KeePass master password traces from memory dump:

```bash
keepass-dump-extractor 1337.dmp
```

* Generate candidates:

```bash
keepass-dump-extractor 1337.dmp -f all
```

* Crack database:

```bash
keepass2john Database1337.kdbx
john databasehash.txt --wordlist=wordlist.txt
```

* Open DB:

```bash
kpcli --kdb Database1337.kdbx
```

Retrieve flag from **You Win! → Database1337**

---

### Final Answers

* **Password (partial):** `NoWaYIcanF0rGetThis123`
* **Missing character:** `?`
* **Flag:** `THM{B3tt3r_Upd4t3_Y0ur_K33p455}`

---

### Key DFIR Lessons

* Start with **Statistics → Conversations** to find unusual traffic.
* **Export HTTP objects** to recover malware/scripts.
* Obfuscation ≠ encryption → **Base64 + XOR is reversible**.
* Memory dumps can expose secrets.
* Know common tools:

  * **Wireshark**
  * **tshark**
  * **ProcDump**
  * **John**
  * **KeePass dump extractor**
* Always check for known **CVEs** before brute forcing.

### Future Investigation Checklist

**Traffic → Extract → Decode → Decrypt → Reconstruct → Exploit → Recover**
---------------
**ProcDump** is a Microsoft **Sysinternals** tool used to create a **memory dump** of a running process.

A **memory dump (.dmp)** is a snapshot of everything stored in a program’s RAM at a specific moment.

Think of it like:

```text
Running Program → RAM → ProcDump → process.dmp
```

### Why people use ProcDump (legitimate use)

* Debug application crashes
* Analyze memory issues
* Investigate performance problems
* Collect forensic evidence

Example:

```bash
procdump.exe -ma chrome.exe dump.dmp
```

Meaning:

* `-ma` → full memory dump
* `chrome.exe` → target process
* `dump.dmp` → output file

### In your CTF

The attacker used:

```powershell
procdump.exe -ma <KeePass_PID> 1337.dmp
```

This means:

1. Find **KeePass** process
2. Dump all KeePass memory
3. Save it as `1337.dmp`
4. Search memory for secrets (master password)

Why this worked:

* KeePass had the **master password temporarily stored in RAM**
* The attacker later extracted it using the KeePass vulnerability

### Important DFIR lesson

If malware runs **ProcDump + LSASS + KeePass + browser processes**, that’s a strong sign of:

* Credential dumping
* Secret extraction
* Data exfiltration

Common targets:

* **lsass.exe** → Windows credentials
* **KeePass.exe** → password vaults
* **chrome.exe / firefox.exe** → cookies & sessions

