## Active Reconnaissance

**Active reconnaissance** directly interacts with a target by sending packets, making connections, or probing services. Unlike passive recon, it leaves traces in logs and security tools.

### Passive vs Active

* **Passive:** DNS, WHOIS, CT logs, Shodan — no direct contact with the target.
* **Active:** Browsing websites, `ping`, `traceroute`, `telnet`, `netcat` — directly interacts with the target.

### Tools

* **Browser:** Inspect headers, JavaScript, and certificates.
* **Ping:** Check host reachability and infer OS from TTL.
* **Traceroute / mtr:** Map network paths and intermediate hops.
* **Telnet:** Legacy banner grabbing.
* **Netcat (nc):** Banner grabbing and port probing.

⚠️ **Important:** Only perform active reconnaissance with explicit legal authorisation.

**Key idea:** Active reconnaissance is more detectable because it directly communicates with the target.
---
## Browser Reconnaissance

A web browser is a useful tool for active reconnaissance because its traffic looks like normal browsing.

### Key Points

* **HTTP:** TCP port 80
* **HTTPS:** TCP port 443
* **HTTP/3:** Uses QUIC over UDP 443

### Developer Tools

* **Network:** View requests, responses, headers, cookies, and status codes.
* **Console:** Run JavaScript and inspect errors/DOM.
* **Sources:** Inspect HTML, JavaScript, and CSS for API endpoints, directories, and exposed information.
* **Application:** Inspect cookies, Local Storage, and Session Storage.
* **Security:** Check certificates and SANs for related domains/subdomains.

### Useful Extensions

* **FoxyProxy:** Manage proxies.
* **User-Agent Switcher:** Change browser identity.
* **Wappalyzer:** Identify technologies used by a website.

**Key idea:** Developer Tools, especially the **Network** and **Sources** tabs, can reveal information that is not visible on the webpage.
---

## Ping

**Ping** checks whether a host is reachable and responding using **ICMP**.

* **Echo Request:** ICMP Type 8
* **Echo Reply:** ICMP Type 0
* **Linux/macOS:** `ping -c 5 IP`
* **Windows:** `ping -n 5 IP`
* **IPv4/IPv6:** `-4` / `-6`
* **Data size option:** `-s`
* **ICMP header size:** 8 bytes

### TTL

TTL can help with OS fingerprinting:

* Linux: typically **64**
* Windows: typically **128**

Routers reduce TTL by 1 per hop, so the received value may be lower.

### No Reply

Possible causes:

* Host is offline
* ICMP is blocked by firewall
* Network route is unavailable
* NAT or cloud infrastructure blocks ICMP

**Windows Firewall blocks ping by default:** Yes (Y)
**10 pings to the target:** 10 replies
---
### Traceroute — Short Note

* **Traceroute** shows the path packets take from your computer to a destination.
* It discovers each **hop (router)** using the **TTL** field.
* For each TTL value, it sends **3 probe/test packets by default**.
* `TTL = 1` → first router responds.
* `TTL = 2` → second router responds.
* It continues until the **destination** is reached.
* **Multiple IPs in one hop:** The 3 probes may take different paths due to **load balancing**.
* `*` = A probe received **no response** from that hop.
* The route can change due to **dynamic routing, load balancing, and failover**.

**Remember:**

```text
Hop number = TTL value
IP address = responding router
3 probes = 3 test packets sent for that TTL
* = no response
```
---
### TELNET & Banner Grabbing — Short Note

* **TELNET:** Remote CLI protocol using **TCP port 23**.
* **Security issue:** Sends usernames, passwords, and data in **cleartext**.
* **Secure alternative:** **SSH**, which encrypts communication.
* **Banner grabbing:** Connecting to a TCP port to read the service's **banner**, revealing software name and version.
* **Example:** `telnet <IP> 80` → send an HTTP request → check the `Server` header.
* **Why useful:** Software versions can be checked against known vulnerabilities (**CVE, Exploit-DB**).
* **Alternatives:** `netcat (nc)` and `curl`.
* **TLS services:** Use `curl`, `openssl s_client`, or `ncat --ssl` because TELNET cannot handle encryption.

**Answers:**

* Server: **Apache**
* Version: **2.4.61**
---

