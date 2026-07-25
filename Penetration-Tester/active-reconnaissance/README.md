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

