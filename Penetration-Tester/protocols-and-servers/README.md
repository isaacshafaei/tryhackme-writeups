### Telnet — Short Note

* **Telnet:** Remote terminal protocol for administering systems.
* **Default port:** **23/TCP**
* **Main problem:** Sends **credentials and data in plaintext** → insecure.
* **Modern alternative:** **SSH**, which encrypts all traffic.
* **Today:** Mostly found on **legacy systems, IoT devices, network equipment, or misconfigured systems**.
* **Recon use:** The Telnet **client** can connect to any TCP port for manual interaction and banner grabbing.

### Useful Protocols & Default Ports

| Protocol   | Port | Purpose                      |
| ---------- | ---: | ---------------------------- |
| **FTP**    |   21 | File transfer                |
| **SSH**    |   22 | Secure remote access         |
| **Telnet** |   23 | Unencrypted remote access    |
| **SMTP**   |   25 | Sending email                |
| **DNS**    |   53 | Domain name resolution       |
| **HTTP**   |   80 | Web traffic                  |
| **POP3**   |  110 | Receiving email              |
| **IMAP**   |  143 | Receiving/managing email     |
| **HTTPS**  |  443 | Encrypted web traffic        |
| **SMB**    |  445 | Windows file/printer sharing |
| **RDP**    | 3389 | Windows remote desktop       |

**Key point:** `telnet <IP>` with default settings connects to **TCP port 23**.
---
### HTTP — Short Note

* **HTTP:** Protocol used by browsers to communicate with web servers and transfer web pages/files.
* **HTTP port:** **80/TCP**
* **HTTPS port:** **443/TCP**
* **HTTP:** Unencrypted/cleartext.
* **HTTPS:** HTTP + **TLS encryption**.
* **Manual HTTP request with Telnet:**

  ```text
  telnet <MACHINE_IP> 80
  GET /index.html HTTP/1.1
  Host: telnet
  ```

  Press **Enter twice** after the `Host` header.
* **Useful header:** `Server:` may reveal the **web server software, version, and sometimes OS**.
* **Common web servers:** Nginx, Apache, IIS.
* **HTTP versions:** HTTP/1.1 (text-based), HTTP/2, HTTP/3 (QUIC/UDP).

**TryHackMe task:** Connect to port 80 and request `/flag.thm`:

```text
GET /flag.thm HTTP/1.1
Host: telnet
```

The content returned by the server is the answer.

*** for example below is an example:
### Telnet + HTTP — Short Note

* **Telnet** can create a raw TCP connection to a server.
* Connect to an HTTP server on port `80`:

```bash
telnet 10.112.149.39 80
```

* Manually send an HTTP GET request:

```http
GET /flag.thm HTTP/1.1
Host: 10.112.149.39
Connection: close
```

* Press **Enter twice** to send the request.
* The server responds with the contents of `flag.thm`.

**Key idea:** Telnet gives you a raw connection, allowing you to manually send an HTTP request instead of using a browser.
---
### FTP — Short Note

| Topic             | Key Point                                                      |
| ----------------- | -------------------------------------------------------------- |
| **FTP**           | File Transfer Protocol; transfers files between systems        |
| **FTP Port**      | **21** (control), **20** (active-mode data)                    |
| **Security**      | Unencrypted → credentials and data sent in cleartext           |
| **SFTP**          | Secure alternative over **SSH, port 22**                       |
| **FTPS**          | FTP over TLS → **990** (implicit) or **21** (explicit)         |
| **SCP**           | Secure file copying over **SSH, port 22**                      |
| **Anonymous FTP** | Login with `anonymous` or `ftp`; always check during pentests  |
| **Active Mode**   | Server connects back to client for data → port **20**          |
| **Passive Mode**  | Client connects to server's high data port → firewall-friendly |
| **FTP Commands**  | `USER`, `PASS`, `SYST`, `PASV`, `TYPE`, `STAT`, `ls`, `get`    |
| **Download File** | `ftp TARGET_IP` → login → `ls` → `get filename`                |
| **Common Server** | `vsftpd`, `ProFTPD`, `Pure-FTPd`                               |
| **Recommended**   | Use **SFTP** instead of plain FTP                              |

**Key pentesting point:** Check for **anonymous login** and exposed sensitive files.
---
### SMTP — Short Note

| Topic              | Key Point                                                      |
| ------------------ | -------------------------------------------------------------- |
| **SMTP**           | Sends email between clients and mail servers                   |
| **Port 25**        | Server-to-server SMTP (MTA ↔ MTA)                              |
| **Port 587**       | Email submission (MUA → MSA), usually authenticated + STARTTLS |
| **Port 465**       | SMTP with implicit TLS                                         |
| **POP3**           | Receives/downloads emails                                      |
| **IMAP**           | Accesses and synchronizes emails on the server                 |
| **Main commands**  | `HELO/EHLO`, `MAIL FROM`, `RCPT TO`, `DATA`, `QUIT`            |
| **Spoofing**       | SMTP alone does not verify the sender's identity               |
| **Security**       | Plain SMTP is unencrypted; use TLS                             |
| **Anti-spoofing**  | **SPF, DKIM, DMARC**                                           |
| **Pentest checks** | Open relay, email spoofing, SMTP misconfiguration              |

**Basic connection:**

```bash
telnet MACHINE_IP 25
```

**Useful security point:** SMTP is commonly involved in **phishing**, **email spoofing**, and **open relay** vulnerabilities.
---

