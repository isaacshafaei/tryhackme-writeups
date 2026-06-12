### Denial-of-Service (DoS) :
**DoS** attacks make websites or applications unavailable by overwhelming resources and preventing normal use.

* **DoS:** Attack from a **single machine**
* **DDoS:** Attack from **many compromised devices (botnet)** → much larger impact

**Common targets:** Web applications (**OSI Layer 7**)

**Examples:**

* Flooding forms/search requests
* Sending malformed or huge inputs
* Overloading login systems

**Common attack types:**

* **Slowloris** → many incomplete HTTP requests
* **HTTP Flood** → massive request volume
* **Cache Bypass** → force origin server processing
* **Oversized Query** → expensive requests
* **Login/Form Abuse** → overload authentication
* **Input Validation Abuse** → exploit poor input handling
---
### DoS/DDoS Detection in Web Logs 

Web server logs help detect **DoS/DDoS** attacks by finding unusual traffic patterns.

**Key indicators:**

* High request rate to one page (`/login`, `/search`)
* Suspicious User-Agents (`curl`, scripts)
* Traffic from many locations (botnet)
* Sudden request bursts
* Increased **5xx errors** (especially `503`)
* Abusive queries (`limit=999999`)

**Common targets:**
`/login`, `/search`, `/api`, `/register`, `/contact`, `/checkout`

**Typical attack flow:**
Repeated requests → server overload → users receive **503 Service Unavailable**.
---
