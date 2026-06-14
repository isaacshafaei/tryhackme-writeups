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
