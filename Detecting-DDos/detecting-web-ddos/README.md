### Splunk Investigation Commands

**Botnet Size:**
`index="main" status=503 | stats dc(clientip)`

**Most Common User-Agent:**
`index="main" status=503 | top limit=1 useragent`

**Peak Requests Per Second:**
`index="main" status=503 | timechart span=1s count | stats max(count)`

**Most Frequent URI:**
`index="main" status=503 | top limit=1 uri`

---

### README Note

**Project: DDoS Traffic Analysis**
Used Splunk to identify attack metrics within web access logs.

* **Normalization:** Filtered `status=503` to isolate malicious traffic.
* **Visualization:** Applied `timechart span=1s` to determine peak throughput.
* **Attribution:** Used `stats` and `top` to identify the botnet size, target URIs, and primary user-agents.
---
### **DDoS Prevention & Mitigation**

#### **1. Application-Level Defense**

* **Secure Coding:** Implement strict **input validation** on forms and search fields to prevent complex, resource-intensive queries from overloading the system.
* **Challenges:** Deploy **CAPTCHAs** or **JavaScript challenges** to differentiate between human users and automated bots.

#### **2. Network & Infrastructure Defense**

* **Content Delivery Network (CDN):** Caches content at the edge to reduce origin server load.  It provides **load balancing** to distribute traffic and absorbs massive volumes of malicious requests.
* **Web Application Firewall (WAF):** Inspects incoming traffic against threat intelligence. Key feature: **Rate-limiting** (e.g., capping login attempts per minute per IP).

#### **3. Large-Scale Mitigation**

* Major providers leverage distributed global infrastructure to absorb multi-terabit attacks by filtering traffic before it reaches critical services.

#### **4. Common Evasion Tactics**

Attackers attempt to bypass defenses using:

* **Cache Busting:** Appending random query parameters (e.g., `?a=123`) to force the origin server to process requests instead of the CDN.
* **Traffic Obfuscation:** Spoofing headers (user-agents, referrers) and using diverse geographic botnets to evade WAF rules.

