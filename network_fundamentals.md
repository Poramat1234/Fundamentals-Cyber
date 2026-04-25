# Network Fundamentals — Cybersecurity Notes

## 1. What Is a Network?
A **network** is two or more computing devices connected to share data and resources. Networks are the foundation of cybersecurity: almost every attack and defense happens across one.

### Scale / Types
- **PAN** (Personal Area Network) — e.g., Bluetooth between phone and earbuds
- **LAN** (Local Area Network) — home/office, typically one building
- **MAN** (Metropolitan Area Network) — city-scale
- **WAN** (Wide Area Network) — country/global scale (the Internet is the largest WAN)
- **VPN** (Virtual Private Network) — encrypted tunnel over a public network

---

## 2. The OSI Model (7 Layers)
Mnemonic: **"Please Do Not Throw Sausage Pizza Away"**

| # | Layer | Role | Example Protocols / Devices |
|---|-------|------|-----------------------------|
| 7 | Application | User-facing services | HTTP, HTTPS, DNS, FTP, SMTP |
| 6 | Presentation | Encoding, encryption, compression | TLS/SSL, JPEG, ASCII |
| 5 | Session | Opens/closes sessions | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery, reliability | TCP, UDP |
| 3 | Network | Routing, logical addressing | IP, ICMP, routers |
| 2 | Data Link | Frames, MAC addressing | Ethernet, switches, ARP |
| 1 | Physical | Bits on the wire/air | Cables, Wi-Fi radio, NICs |

## 3. The TCP/IP Model (4 Layers — Real-World Stack)
1. **Application** (HTTP, DNS, SSH, SMTP…)
2. **Transport** (TCP, UDP)
3. **Internet** (IP, ICMP)
4. **Network Access / Link** (Ethernet, Wi-Fi)

---

## 4. Core Protocols

### TCP vs. UDP
| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery + ordering | Best-effort |
| Speed | Slower | Faster |
| Use cases | Web, email, file transfer | DNS, VoIP, gaming, streaming |

**TCP 3-Way Handshake:** `SYN → SYN-ACK → ACK`

### IP Addressing
- **IPv4** — 32-bit, e.g., `192.168.1.10` (~4.3B addresses)
- **IPv6** — 128-bit, e.g., `2001:db8::1`
- **Private ranges** (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- **Loopback:** `127.0.0.1` (IPv4) / `::1` (IPv6)

### Ports (0–65535)
- **Well-known (0–1023):** 20/21 FTP, 22 SSH, 23 Telnet, 25 SMTP, 53 DNS, 80 HTTP, 443 HTTPS, 3389 RDP
- **Registered (1024–49151)**
- **Dynamic/Ephemeral (49152–65535)**

### Key Helper Protocols
- **DNS** — hostname → IP resolution
- **DHCP** — auto-assigns IP addresses to clients
- **ARP** — IP → MAC address mapping inside a LAN
- **ICMP** — diagnostics (`ping`, `traceroute`)
- **NAT** — translates private IPs to a public IP

---

## 5. Network Devices
- **Hub** — dumb repeater (legacy, broadcasts to all ports)
- **Switch** — forwards frames by MAC address (Layer 2)
- **Router** — forwards packets between networks (Layer 3)
- **Firewall** — filters traffic by rules (L3/L4, or L7 NGFW)
- **Access Point (AP)** — wireless bridge to wired LAN
- **Proxy** — intermediary between client and server
- **Load Balancer** — distributes traffic across servers
- **IDS / IPS** — detects / blocks malicious traffic

---

## 6. Network Security Fundamentals

### CIA Triad
- **Confidentiality** — only authorized parties can read data (encryption, access control)
- **Integrity** — data cannot be tampered with undetected (hashing, signatures)
- **Availability** — systems stay reachable when needed (redundancy, DDoS protection)

### Common Controls
- **Firewalls** — packet filter, stateful, next-gen (NGFW)
- **VPNs** — IPsec, SSL/TLS-based (encrypt traffic over untrusted networks)
- **Network Segmentation / VLANs** — isolate zones (e.g., DMZ, guest, internal)
- **Zero Trust** — never trust, always verify — authenticate every request
- **Encryption in transit** — TLS, SSH, WPA2/WPA3

### Common Network Attacks
| Attack | Description | Layer |
|--------|-------------|-------|
| Sniffing / Eavesdropping | Capturing traffic (e.g., Wireshark on open Wi-Fi) | L2/L3 |
| MITM (Man-in-the-Middle) | Attacker sits between two parties | L2–L7 |
| ARP Spoofing | Fake ARP replies to redirect LAN traffic | L2 |
| DNS Spoofing / Cache Poisoning | Forge DNS responses | L7 |
| DoS / DDoS | Flood target to exhaust resources | L3/L4/L7 |
| Port Scanning | Probe for open services (nmap) | L4 |
| Rogue AP / Evil Twin | Malicious Wi-Fi pretending to be legitimate | L1/L2 |
| SYN Flood | Half-open TCP connections exhaust server | L4 |

---

## 7. Essential Tools
- **ping** — test reachability (ICMP)
- **traceroute / tracert** — path to destination
- **ipconfig / ifconfig / ip** — view local network config
- **netstat / ss** — show active connections & listening ports
- **nslookup / dig** — query DNS
- **nmap** — port scanning & host discovery
- **Wireshark / tcpdump** — packet capture & analysis
- **curl / wget** — HTTP requests from CLI

---

## 8. Quick Reference — Subnetting
- **CIDR notation:** `/24` = 255.255.255.0 = 256 addresses (254 usable)
- **/25** = 128 addresses, **/26** = 64, **/27** = 32, **/28** = 16
- Formula: usable hosts = `2^(32 - prefix) - 2`

---

## 9. Study Checklist
- [ ] Can explain every OSI layer with one example protocol
- [ ] Know the difference between TCP and UDP and when each is used
- [ ] Memorize the top 10 well-known ports
- [ ] Understand public vs. private IP and NAT
- [ ] Know what ARP, DNS, DHCP, and ICMP each do
- [ ] Can describe at least 5 common network attacks and their defenses
- [ ] Practice with `ping`, `nslookup`, `netstat`, `nmap`, `Wireshark`

---

*Next topics to cover: cryptography fundamentals, authentication & access control, malware, incident response.*
