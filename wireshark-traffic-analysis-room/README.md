wireshark traffic analysys:
Detecting Nmap Scans with Wireshark
A quick reference guide for identifying common Nmap network mapping signatures using Wireshark display filters.

Core TCP Flag Filters
SYN only: tcp.flags == 2 or tcp.flags.syn == 1

ACK only: tcp.flags == 16 or tcp.flags.ack == 1

SYN, ACK: tcp.flags == 18 or (tcp.flags.syn == 1) and (tcp.flags.ack == 1)

RST only: tcp.flags == 4 or tcp.flags.reset == 1

RST, ACK: tcp.flags == 20 or (tcp.flags.reset == 1) and (tcp.flags.ack == 1)

FIN only: tcp.flags == 1 or tcp.flags.fin == 1

Scan Signatures & Detection
1. TCP Connect Scan (-sT)
Behavior: Completes the full 3-way handshake. Executed by non-privileged users.

Signature: Window size is typically > 1024 bytes (expects data).

Wireshark Filter: tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024

2. TCP SYN Scan (-sS)
Behavior: Incomplete handshake (tears down the connection with an RST before final ACK). Requires privileged access.

Signature: Window size is typically <= 1024 bytes (does not expect data).

Wireshark Filter: tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024

3. UDP Scan (-sU)
Behavior: Stateless protocol mapping. Open ports typically provide no response.

Signature: Closed ports return an ICMP Type 3, Code 3 message (Destination/Port Unreachable), which encapsulates the original UDP request data.

Wireshark Filter: icmp.type==3 and icmp.code==3
