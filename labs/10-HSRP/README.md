# Secure Campus Gateway Redundancy (HSRP)

## Overview
This repository contains the documentation and configuration for a **secure campus gateway redundancy** lab built around **HSRP (Hot Standby Router Protocol)**.  
The purpose of this project was to design and validate a highly available default-gateway architecture for a segmented campus LAN, where each VLAN has a resilient first-hop gateway with automatic failover.

This lab was completed as part of my CCNA networking practice to strengthen my understanding of:
- Inter-VLAN gateway design
- First-hop redundancy (FHRP)
- VLAN-aware Layer 2/Layer 3 integration
- STP interaction with redundant gateway paths

---

## Topology
<p align="center">
  <img src="https://github.com/zedparsa/ccna-notes/blob/main/labs/10-HSRP/hsrp.PNG" alt="HSRP Topology" />
</p>

### Design Specifications
- **Architecture**: Two Distribution Switches (Layer 3) providing redundant gateways for a centralized Access Switch.
- **Segmentation**: Dual VLAN environment (VLAN 10 and VLAN 20).
- **Gateway Redundancy**: HSRP (Hot Standby Router Protocol) implemented for high availability.
- **Load Sharing**: Optimized gateway distribution (DS1 active for VLAN 10, DS2 active for VLAN 20).
- **Layer 2 Transport**: IEEE 802.1Q Trunking across all switch-to-switch links.

---

## VLAN & HSRP Mapping
| VLAN | Name | Subnet | Virtual IP (Gateway) | Active Node | Standby Node |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 10 | Users_A | 192.168.10.0/24 | 192.168.10.254 | DS1 | DS2 |
| 20 | Users_B | 192.168.20.0/24 | 192.168.20.254 | DS2 | DS1 |

---

## Configuration Highlights

### 1. HSRP Core Logic (Example: VLAN 10 on DS1)
backtick cisco
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 standby version 2
 standby 10 ip 192.168.10.254
 standby 10 priority 110
 standby 10 preempt
backtick

### 2. Trunking & Encapsulation
To ensure VLAN propagation, all inter-switch links were manually configured as trunks.
backtick cisco
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20
backtick

### 3. Layer 3 Routing
Explicitly enabled on Distribution switches to allow Inter-VLAN communication.
backtick cisco
ip routing
backtick

---

## Troubleshooting & Learnings (Key Takeaways)

1. **HSRP Dependency on Layer 2**: Confirmed that HSRP failover depends entirely on stable Layer 2 trunking. If a VLAN is missing from a trunk, the HSRP nodes will fail to see each other (Duplicate Active state).
2. **Virtual IP vs. Physical SVI**: Learned that hosts must point *only* to the HSRP Virtual IP (VIP). Pointing to a physical SVI IP bypasses the redundancy mechanism.
3. **STP and Path Selection**: Observed how Spanning Tree Protocol (STP) blocks redundant links to prevent loops. Realized that for optimal performance, the STP Root Bridge should be aligned with the HSRP Active gateway for a specific VLAN.
4. **Encapsulation Conflict**: Resolved the "Trunk encapsulation Auto" error by explicitly defining `dot1q` before forcing trunk mode on multilayer switches.

---

## Verification Commands
Used for validating HSRP states and path redundancy:
- `show standby brief` (To verify Active/Standby roles)
- `show ip interface brief` (To verify SVI status)
- `show interfaces trunk` (To ensure VLANs 10 and 20 are allowed)
- `show spanning-tree vlan <id>` (To identify root ports and blocked paths)

---

## Repository Structure
- `/configs/`: Running configurations for DS1, DS2, and AS1.
- `/packet-tracer/`: Original `.pkt` simulation file.
- `/verification/`: Logs of HSRP failover tests and connectivity status.

