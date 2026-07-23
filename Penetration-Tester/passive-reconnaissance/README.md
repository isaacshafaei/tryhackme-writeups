## Reconnaissance

Reconnaissance is the first stage of an attack, used to gather information about a target.

### Passive Reconnaissance

Uses only public information without directly contacting the target.

Examples:

* Public DNS records
* Certificate transparency logs
* Job postings and social media
* Shodan and Censys
* Public GitHub repositories
* News and leaked documents

### Active Reconnaissance

Directly interacts with the target and may be detected.

Examples:

* Ping and ARP discovery
* Port scanning with Nmap
* Web and API fuzzing
* Phishing or phone calls
* Physical approaches

Passive recon is stealthier. Active recon requires explicit authorisation because it can trigger security alerts and legal issues.

**Important:** Direct interaction with an employee also counts as active reconnaissance.
---

## WHOIS and RDAP

**WHOIS** retrieves domain registration information using TCP port `43`.

```bash
whois example.com
```

It may reveal:

* Registrar
* Registration, update, and expiry dates
* Name servers
* Domain status
* Abuse contact details

Personal registrant data is often hidden by privacy services.

**RDAP** is the modern replacement for WHOIS. It uses HTTPS and returns structured JSON.

```bash
curl -s https://rdap.verisign.com/com/v1/domain/example.com | jq .
```

### Important Results for TryHackMe

* **Registration date:** `2018-07-05`
* **Registrar:** Namecheap
* **Name server provider:** Cloudflare
* **`clientTransferProhibited`:** Domain transfer is locked
---
## DNS Lookup

DNS queries can reveal IP addresses, mail servers, aliases, and verification records.

### Common Records

* `A`: IPv4 address
* `AAAA`: IPv6 address
* `CNAME`: Domain alias
* `MX`: Mail servers
* `SOA`: DNS zone information
* `TXT`: SPF, DMARC, verification strings, or flags

Use `dig` as the preferred tool:

```bash
dig thmlabs.com TXT
```

Or:

```bash
nslookup -type=TXT thmlabs.com
```

**TXT flag:**

```text
THM{a5b83929888ed36acb0272971e438d78}
```

([notestime.in][1])

[1]: https://notestime.in/cyber-security/advanced-penetration-testing-for-beginners/pentesting-tryhackme-labs?utm_source=chatgpt.com "TryHackMe Labs: Beginner to Advanced Pentesting Practice"

