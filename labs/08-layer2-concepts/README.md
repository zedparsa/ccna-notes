# Secure Research Campus LAN (Layer 2 Foundation)

## Overview
This repository contains the documentation and configuration for a foundational **Layer 2 Secure Campus LAN** project. This lab was designed as part of my CCNA curriculum (Chapter 14) to master VLAN segmentation, Trunking, and security hardening on Cisco switches. 

The primary objective was to move beyond basic connectivity and implement a production-like hardened Layer 2 infrastructure. This lab served as a critical prerequisite for my upcoming research project on **Intrusion Detection Systems (IDS)** by establishing a segmented environment with strict VLAN controls.

## Topology
<p align="center">
  <img src="https://github.com/zedparsa/ccna-notes/blob/main/labs/08-layer2-concepts/VLAN.PNG" alt="Port Security Topology" />
</p>


### Design Specifications
- **Architecture**: Collapsed Core (1 Distribution Switch + 3 Access Switches).
- **Segmentation**: 5 distinct functional VLANs (10, 20, 30, 40, 50).
- **Hardening Strategy**:
    - **Static Trunking**: Manual configuration of `switchport mode trunk` to prevent DTP-based VLAN Hopping attacks.
    - **DTP Disabling**: Explicit `switchport nonegotiate` applied on all trunk links.
    - **Native VLAN**: Standardized on VLAN 999 to avoid mismatch errors and enhance security.
    - **VLAN Pruning**: Only necessary VLANs are allowed across the trunks.

## VLAN Mapping
| VLAN ID | Name | Subnet | Purpose |
| :--- | :--- | :--- | :--- |
| 10 | Students | 10.10.10.0/24 | End-user access |
| 20 | Security | 10.10.20.0/24 | Security cameras/devices |
| 30 | Research | 10.10.30.0/24 | Research equipment/workstations |
| 40 | Servers | 10.10.40.0/24 | Centralized server resources |
| 50 | Guest | 10.10.50.0/24 | Guest Wi-Fi / Visitors |

## Troubleshooting & Learnings (Key Takeaways)
This lab presented several real-world challenges that provided deeper insight into Cisco IOS operations:
1.  **VLAN Database Integrity**: Encountered connectivity issues between Access switches because VLANs were only defined on access ports, not in the `vlan.dat` file of the Distribution switch. *Lesson: Always ensure end-to-end VLAN availability in the distribution layer.*
2.  **Native VLAN Mismatch**: Faced `CDP-4-NATIVE_VLAN_MISMATCH` logs due to inconsistent Native VLAN settings. Resolved by standardizing on VLAN 999 across all links.
3.  **Trunking Primitives**: Understood the necessity of explicit manual trunking to prevent potential VLAN hopping and spoofing attacks.

## Repository Structure
- `/configs/`: Running configurations for all switches (DIST-SW, ACC-SW1, ACC-SW2, ACC-SW3).
- `/packet-tracer/`: Original `.pkt` file for simulation.
- `/verification/`: Text logs of essential show commands (`show vlan brief`, `show interfaces trunk`).

---
*Created by: Parsa (zedparsaa)*  
*Focus: CCNA | Network Security | Academic Research*
