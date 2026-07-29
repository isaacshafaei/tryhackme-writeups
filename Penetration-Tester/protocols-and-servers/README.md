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

