### File Transfer & Hosting (Short Notes)

**1. Download files — `wget`**
Download files from the web via HTTP.

```bash
wget <URL>
```

Example:

```bash
wget https://site.com/file.txt
```

---

**2. Transfer files securely — `scp` (SSH)**
Copy files between local and remote machines.

Local → Remote:

```bash
scp file.txt user@IP:/path/file.txt
```

Remote → Local:

```bash
scp user@IP:/path/file.txt local.txt
```

---

**3. Serve files from your machine — Python HTTP server**
Start a simple web server in current directory:

```bash
python3 -m http.server
```

Download from another machine:

```bash
wget http://IP:8000/file
```

**Note:** Server runs until stopped and requires exact filename.
---
### Linux Processes — Short Notes

* **Process** = running program managed by the kernel, identified by **PID** (Process ID).

* View processes:

  ```bash
  ps       # current session
  ps aux   # all users + system processes
  top      # live process monitoring
  ```

* Manage processes:

  ```bash
  kill PID
  ```

  Signals:

  * `SIGTERM` → graceful stop
  * `SIGKILL` → force kill
  * `SIGSTOP` → pause process

* Process startup:

  * Processes run inside **namespaces** (resource isolation)
  * **systemd (PID 0 in this note)** manages and starts child processes at boot

* Start services on boot:

  ```bash
  systemctl start|stop|enable|disable|status service
  ```

* Background / Foreground:

  ```bash
  command &   # run in background
  Ctrl + Z    # suspend/background
  fg          # return to foreground
  ```
---
### Linux Processes — Very Short Notes

* **Process** = running program managed by kernel with unique **PID** (increments as processes start).

* View processes:

  ```bash
  ps       # current user
  ps aux   # all processes
  top      # live monitoring
  ```

* Manage processes:

  ```bash
  kill PID
  ```

  Signals:

  * `SIGTERM` → graceful stop
  * `SIGKILL` → force stop
  * `SIGSTOP` → pause

* Startup:

  * Processes are isolated using **namespaces**
  * **systemd** starts and manages services at boot

* Service control:

  ```bash
  systemctl start|stop|enable|disable|status service
  ```

* Background / Foreground:

  ```bash
  command &   # background
  Ctrl+Z      # suspend
  fg          # foreground
  ```
---------
### Crontab — Short Notes

* **Cron** = Linux service that runs tasks automatically on a schedule
* **Crontab** = file used to define scheduled tasks (cron jobs)

---

### Crontab format (6 fields)

```
MIN HOUR DOM MON DOW CMD
```

* MIN → minute
* HOUR → hour
* DOM → day of month
* MON → month
* DOW → day of week
* CMD → command to run

---

### Example

```bash id="8q2w9m"
0 */12 * * * cp -R /home/user/Documents /var/backups/
```

→ runs backup every 12 hours

---

### Special symbol

* `*` = “any value”

---

### Manage crontab

```bash id="n4jv2a"
crontab -e   # edit tasks
```

---

### Key idea

Used to **automate tasks after boot or on a schedule** (backup, updates, scripts, etc.)

---
### Packages & Repositories — Short Notes

* **APT repository** = online source where Linux software is stored and shared
* Developers publish software to repos; users install via **apt**

---

### What `apt` does

* Installs software
* Updates software
* Manages dependencies automatically

```bash id="k9xq2a"
apt install package
apt update
apt remove package
```

---

### Adding software sources

* You can add extra (community/3rd-party) repositories
* Done via:

  ```bash id="m2p8sd"
  add-apt-repository
  ```

  or manually in `/etc/apt/sources.list.d/`

---

### Security (GPG keys)

* Repositories are verified using **GPG keys**
* Ensures software is trusted and not tampered with

---

### Install flow

1. Add GPG key
2. Add repository
3. Run:

   ```bash id="v1zq7m"
   apt update
   apt install <software>
   ```

---

### Remove software

```bash id="c7xk3p"
apt remove <software>
```

or remove repo + package manually.
---
### Linux Logs — Very Short Notes

* Logs are stored in:

  ```bash
  /var/log
  ```

* Used to monitor system, services, and security events

---

### Examples

* Apache logs → web traffic + errors
* fail2ban → brute-force protection logs
* UFW → firewall activity

---

### Common log types

* **access log** → requests to services
* **error log** → system/service errors
* auth logs → login attempts

---

### Key idea

Logs help **debug issues, monitor performance, and detect attacks**.

