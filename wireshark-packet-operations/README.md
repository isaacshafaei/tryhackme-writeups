**Wireshark – Statistics (Short Note)**

The **Statistics** menu in Wireshark gives a quick overview of captured network traffic and helps analysts identify patterns, protocols, endpoints, and suspicious activity.

* **Resolved Addresses:** Displays resolved IP addresses and hostnames using DNS responses in the capture.
* **Protocol Hierarchy:** Shows protocol distribution in a tree view with packet counts and percentages.
* **Conversations:** Displays communication between two endpoints (Ethernet, IPv4, IPv6, TCP, UDP).
* **Endpoints:** Lists unique devices/addresses involved in the capture.
* **Name Resolution:** Converts MAC, IP, and port values into readable names (configured in Preferences).
* **GeoIP Mapping:** Maps IP locations using a MaxMind database (requires setup and internet access).

These statistics help analysts build investigation hypotheses and quickly understand network activity.
-----
**Wireshark – IPv4/IPv6, DNS & HTTP Statistics (Short Note)**

* **IPv4 / IPv6 Statistics:** Filters and displays statistics for packets using a specific IP version, helping analysts focus on IPv4 or IPv6-related events.
  *(Statistics → IPvX Statistics)*

* **DNS Statistics:** Analyzes DNS traffic and shows packet counts and percentages. Includes details such as **rcode, opcode, class, query types, services, and query statistics** to understand DNS activity.
  *(Statistics → DNS)*

* **HTTP Statistics:** Analyzes HTTP traffic and displays request/response statistics, including **HTTP codes and original requests** to review web activity.
  *(Statistics → HTTP)*

These statistics help analysts isolate protocol-specific behavior and investigate network events more efficiently.
-------
**Wireshark – Packet Filtering (Short Note)**

Wireshark provides two filtering methods:

* **Capture Filters:** Applied **before capture** to save only selected traffic. Cannot be changed during capture.
  Example: `tcp port 80`

* **Display Filters:** Applied **during analysis** to show only relevant packets without changing captured data. Supports **3000+ protocols**.
  Example: `tcp.port == 80`

**Display Filter Operators**

* `==` → Equal (`ip.src == 10.10.10.100`)
* `!=` → Not equal *(prefer `!(value)` instead)*
* `>` `<` `>=` `<=` → Comparison operators

**Logical Operators**

* `AND / &&` → Both conditions must match
* `OR / ||` → Either condition matches
* `NOT / !` → Excludes condition

**Filter Toolbar Features**

* Filters use **lowercase**
* Supports **autocomplete** with protocol fields (`.` notation)
* Color indicators:

  * **Green** → Valid filter
  * **Red** → Invalid filter
  * **Yellow** → Works but not recommended

Packet filtering helps analysts isolate and investigate relevant network traffic efficiently.
--------------------
**Wireshark – Protocol Filters (Short Note)**

Wireshark protocol filters allow packet analysis at different OSI layers using **display filters**.

### IP Filters (Network Layer)

Used to filter traffic based on IP information such as **address, version, TTL, flags, and checksum**.

Examples:

* `ip` → Show all IP packets
* `ip.addr == 10.10.10.111` → Packets containing this IP
* `ip.addr == 10.10.10.0/24` → Packets in subnet
* `ip.src == X` → Source IP only
* `ip.dst == X` → Destination IP only

**Note:** `ip.addr` ignores direction, while `ip.src/ip.dst` considers it.

### TCP / UDP Filters (Transport Layer)

Used to filter traffic using **ports, sequence numbers, flags, and protocol details**.

Examples:

* `tcp.port == 80` → TCP port 80
* `udp.port == 53` → UDP port 53
* `tcp.srcport == 1234` → Source TCP port
* `tcp.dstport == 80` → Destination TCP port

### HTTP / DNS Filters (Application Layer)

Used to inspect application-specific traffic and payload details.

Examples:

* `http` → All HTTP packets
* `http.response.code == 200` → HTTP 200 responses
* `http.request.method == "GET"` → HTTP GET requests
* `dns` → All DNS packets
* `dns.flags.response == 0` → DNS requests
* `dns.flags.response == 1` → DNS responses
* `dns.qry.type == 1` → DNS Type A queries

### Display Filter Expressions

Wireshark provides **Analyse → Display Filter Expression** to help build filters by showing:

* Available protocol fields
* Accepted value types
* Predefined values

Coloring rules can also be applied to highlight filtered results.
--------------
**Wireshark – Advanced Filtering (Short Note)**

Advanced filters in Wireshark help analysts perform deeper packet analysis using operators and functions.

### Advanced Filter Operators

* **`contains`** → Searches for a specific value inside packets (**case-sensitive**)
  Example:
  `http.server contains "Apache"`

* **`matches`** → Uses **regular expressions** to search patterns (**case-insensitive**)
  Example:
  `http.host matches "\.(php|html)"`

* **`in`** → Checks if a value exists within a set/range
  Example:
  `tcp.port in {80 443 8080}`

### Filter Functions

* **`upper()`** → Converts text to uppercase
  Example:
  `upper(http.server) contains "APACHE"`

* **`lower()`** → Converts text to lowercase
  Example:
  `lower(http.server) contains "apache"`

* **`string()`** → Converts non-string values into strings
  Example:
  `string(frame.number) matches "[13579]$"`

### Productivity Features

* **Bookmarks:** Save frequently used filters for quick reuse.
* **Filter Buttons:** Apply predefined filters with one click.
* **Profiles:** Store custom settings (filters, colouring rules, layouts) for different investigation scenarios.

Profiles can be managed through:
**Edit → Configuration Profiles** or **Status Bar → Profile**.

