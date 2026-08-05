### Nmap — Short Notes

* **Nmap (Network Mapper):** Open-source network scanning tool by **Gordon Lyon (Fyodor)**.
* Used for:

  * Finding live hosts
  * Scanning ports
  * Detecting services/versions
  * Vulnerability scanning (NSE scripts)

### Host Discovery Methods:

* **ARP Scan:** Finds hosts in the same LAN using broadcast requests.
* **ICMP Scan:** Uses ping, timestamp, and address mask requests.
* **TCP Ping:** Uses SYN/ACK packets to detect online hosts.
* **UDP Ping:** Uses ICMP "port unreachable" responses.

**Flow:** Host Discovery → Port Scanning → Service Detection → Vulnerability Analysis.
