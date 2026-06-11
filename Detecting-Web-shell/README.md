## Web Shell

A **web shell** is a malicious script uploaded to a web server that allows attackers to execute commands remotely.

### Purpose

* Gain initial access through vulnerable file uploads
* Maintain persistence
* Perform:

  * Reconnaissance
  * Privilege escalation
  * Lateral movement
  * Data exfiltration

### Common Deployment Methods

* Unrestricted file upload
* Poor file validation
* Server misconfiguration
* Existing system compromise

Example:

```php
<?php system($_GET['cmd']); ?>
```

Attackers upload files like `.php` or `.aspx` shells to execute commands through a browser.

### Real-World Examples

* **Hafnium** used Exchange vulnerabilities to upload `.aspx` web shells.
* **Conti ransomware** attackers used web shells for network discovery and persistence.

**Summary:** Web shells provide attackers remote control over compromised web servers.
---
````markdown
## Web Shell Anatomy

- Web shells abuse legitimate system execution functions to run commands remotely.
- Common PHP functions abused:
  - `shell_exec()`
  - `exec()`
  - `system()`
  - `passthru()`

### How it works
1. User sends a command (e.g. `?cmd=whoami`)
2. Script stores it in a variable
3. Executes it on the server
4. Displays the output

Example:
```php
<?php echo shell_exec($_GET['cmd']); ?>
````

Web shells can range from simple command execution scripts to advanced tools with file managers and authentication.

Commands passed through URLs should be URL-encoded:

```
ls -la → ls%20-la
```

```
```
---
### Web Server Logs — Web Shell Detection (Short Note)

Web server logs (Apache/Nginx) help detect web shell activity by analyzing requests, IPs, methods, and patterns.

**Main indicators:**

* Repeated GET/POST requests or rapid probing
* POST after GET scans (possible upload attempt)
* Repeated access to the same file (shell interaction)
* Suspicious HTTP methods:

  * GET = recon / interaction
  * POST = upload/control
  * PUT/DELETE = file changes
  * OPTIONS/HEAD = reconnaissance

**Other red flags:**

* Strange or outdated User-Agents (e.g., curl, wget, old browsers)
* External or unknown IPs
* Long or encoded query strings (`cmd=`, Base64 payloads)
* Missing referrer (sometimes direct malicious access)

**Attack pattern:**
Scanning → finding upload point → uploading file → executing shell (often same IP/UA, mixed 404/200 responses)

**Correlation:**

* Web logs show requests
* `auditd` confirms file creation/execution (`ausearch -k web_shell`)
* Combining both reveals full attack chain

**SIEM:**
Centralizes logs and correlates events to quickly detect suspicious activity.
---

### File System & Network Analysis — Short Note

Web shells are usually stored in web directories or injected into web apps.

**File system analysis:**

* Common paths: `/var/www/html/` (Apache), `/usr/share/nginx/html/` (Nginx)
* Attackers may use `/uploads/`, `/tmp/`, etc.
* Look for suspicious files: `.php`, `.jsp`, double extensions like `image.jpg.php`
* Search tools:

  * `find` → recently modified files
  * `grep` → malicious code like `eval(`

**Network traffic analysis:**

* Detects attacker behavior via packet inspection (PCAP/Wireshark)
* Indicators:

  * Unusual HTTP methods (GET, POST, PUT)
  * Suspicious User-Agents / IPs
  * Encoded payloads or commands in requests
  * Web server spawning system commands

**Useful Wireshark filters:**

* `http.request.method == "PUT"` → PUT requests
* `http.request.uri contains ".php"` → suspicious files
* `http.user_agent` → unusual clients

Combining file, log, and network analysis helps confirm web shell attacks.
---

