# 🔐 CCNA Lab — Device Access Security
Practice securing Cisco device management access in Packet Tracer: banners, Telnet vs SSH, local users, and AAA authentication using a RADIUS server (Server-PT). Optional: NTP.

### ⚠️ Disclaimer (Lab Only)
All credentials, IPs, and keys in this lab are for learning purposes only. Do not reuse them in real networks.

### 🧩 Topology

<p align="center">
  <img src="https://github.com/zedparsa/ccna-notes/blob/main/labs/03-port-security/port-security.png" alt="Port Security Topology" />
</p>

- 1x Router: `R1`
- 1x Switch: `SW1` (2960)
- 1x Laptop: `Laptop0` (client)
- 1x Server: `Server0` (AAA / RADIUS)

### 🗺️ IP Addressing Plan
- `SW1 (VLAN 1)` : `192.168.10.1/24`
- `R1 (G0/0)` : `192.168.10.2/24`
- `Laptop0` : `192.168.10.3/24`
- `Server0` : `192.168.10.4/24`
- Default gateway for hosts: `192.168.10.2`

### 🔑 Lab Credentials
#### 👤 Local user (device login)
- Username: `Parsa`
- Password: `4444`

#### 🔐 RADIUS shared secret (key)
- RADIUS key: `p2082`

Note: The RADIUS key must match on both the router and the RADIUS server configuration.
