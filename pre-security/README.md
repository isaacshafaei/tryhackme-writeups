## Pre Security Contents:

### Virtual Private Network(VPN):
* **different locations:** it can be use for a user from a acompany that works in different region.
* **Privacy:** when i'm using the public wifi, i have to use VPN to use encryption to protect my traffic.
* **anonymous:** using the right VPN help to be anonymous in network and ISP can't track your traffic.
* **Types:** PPP and PTPP work together which is easy to set up but less secure on the other hand IPsec is more secure but difficult to set up.
---
### ARP and DHCP:
* **ARP:** Adress Resolution Protocol-> allows a device to associate its MAC address with an IP address on the network which has two messages(ARP Request and ARP Reply)
* **DHCP:** Dynamic Host Configuration Protocol-> When a device connects to a network, if it has not already been manually assigned an IP address, it sends out a request (DHCP Discover) to see if any DHCP servers are on the network which has following request and reply messages(DHCP Discover(user)->DCHP offer(server)->DHCP Request(user)->DHCP ACK(server))
---

### DNS:

* **TLD (Top-Level Domain): ** There are two types of TLD `1-gTDL (Generic TLD) 2-ccTLD (country code TLD)`
* **Second Level Domain:** in Kitwp.com the kitwp is the second level domain which is limited to 63 characters and can only use a-z 0-9 and hypens in the middle. and also maximum length of a domain name is 253 characters.
* **DNS Records:** DNS records are domain settings on DNS servers that connect a domain to an IPv4 address (A), IPv6 address (AAAA), another domain (CNAME), email servers (MX), or text-based info like verification and security (TXT).

`Note: Tim Berners-Lee made developed th HTTP`
---
* **Cookies:** are small pieces of data stored in your browser that help websites remember who you are, your settings, and your session between requests. 
 
### HTTP:
| Method      | Purpose                       | Example                           |
| ----------- | ----------------------------- | --------------------------------- |
| **GET**     | Retrieve data                 | View a webpage or fetch user data |
| **POST**    | Send data to create something | Login, register, submit a form    |
| **PUT**     | Replace an existing resource  | Update an entire user profile     |
| **PATCH**   | Partially update a resource   | Change only a user's email        |
| **DELETE**  | Remove a resource             | Delete a user account             |
| **HEAD**    | Get headers only              | Check if a page exists            |
| **OPTIONS** | Show allowed methods          | Discover API capabilities         |

```
GET /users       -> View users
POST /users      -> Create user
PUT /users/1     -> Replace user 1
PATCH /users/1   -> Update part of user 1
DELETE /users/1  -> Delete user 1
```

