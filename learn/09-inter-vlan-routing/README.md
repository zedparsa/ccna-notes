## 📖 Part 1 — ROAS

### 📝 Summary:
Inter-VLAN Routing is the process of allowing devices in different VLANs to communicate with each other.

By default, VLANs are separate Layer 2 broadcast domains.  
Devices inside the same VLAN can communicate directly, but devices in different VLANs cannot communicate unless a Layer 3 device performs routing between them.

There are two common ways to implement Inter-VLAN Routing:

- **ROAS (Router-on-a-Stick):** Using one physical router interface with multiple subinterfaces.
- **SVI (Switch Virtual Interface):** Using a Layer 3 switch to route between VLANs.

### 🎯 Objectives:
- Understand why routing is required between VLANs.
- Configure Router-on-a-Stick using subinterfaces.
- Configure 802.1Q encapsulation on router subinterfaces.
- Understand the role of the Native VLAN in ROAS.
- Configure Inter-VLAN Routing using SVIs on a Layer 3 switch.
- Verify Inter-VLAN connectivity using IOS commands.

### 🧩 Topology:
A switch has multiple VLANs configured:

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| VLAN 10 | HR | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | IT | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 999 | Native | No user devices | Optional |

The switch connects to a router using a trunk link.  
The router performs routing between VLAN 10 and VLAN 20 using subinterfaces.

### 🛠️ Step-by-Step:

#### 1. Why Inter-VLAN Routing Is Needed

A VLAN is a separate broadcast domain.  
This means that VLAN 10 and VLAN 20 are logically isolated from each other.

For example:

- A PC in VLAN 10 can communicate with another PC in VLAN 10.
- A PC in VLAN 20 can communicate with another PC in VLAN 20.
- A PC in VLAN 10 cannot communicate with a PC in VLAN 20 without routing.

To allow communication between VLANs, we need a Layer 3 device such as:

- A router
- A Layer 3 switch

## Method 1 — ROAS (Router-on-a-Stick)

#### 2. Concept

ROAS stands for **Router-on-a-Stick**.

In this design, one physical router interface is connected to a switch using a trunk link.  
Because the router must route traffic for multiple VLANs over one physical interface, we divide that physical interface into multiple logical interfaces called **subinterfaces**.

Each subinterface represents one VLAN.

Example:

| Router Subinterface | VLAN | IP Address |
|---|---|---|
| Gig0/0/0.10 | VLAN 10 | 192.168.10.1 |
| Gig0/0/0.20 | VLAN 20 | 192.168.20.1 |
| Gig0/0/0.999 | Native VLAN 999 | Optional |

The IP address configured on each subinterface becomes the **default gateway** for devices in that VLAN.

#### 3. Configure VLANs on the Switch

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name IT
Switch(config-vlan)# exit
Switch(config)# vlan 999
Switch(config-vlan)# name NATIVE
```

| Command | Where | Purpose |
|---|---|---|
| `vlan 10` | Switch IOS global configuration | Creates VLAN 10 |
| `name HR` | Switch IOS VLAN configuration | Assigns the name HR to VLAN 10 |

#### 4. Assign Access Ports to VLANs

```cisco
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```

| Command | Where | Purpose |
|---|---|---|
| `interface fa0/1` | Switch IOS global configuration | Enters interface configuration mode for Fa0/1 |
| `switchport mode access` | Switch IOS interface configuration | Forces the port to operate as an access port |
| `switchport access vlan 10` | Switch IOS interface configuration | Assigns the port to VLAN 10 |

#### 5. Configure the Switch Port Connected to the Router as a Trunk

```cisco
Switch(config)# interface gig0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 999
Switch(config-if)# switchport trunk allowed vlan 10,20,999
```

| Command | Where | Purpose |
|---|---|---|
| `interface gig0/1` | Switch IOS global configuration | Enters the interface connected to the router |
| `switchport mode trunk` | Switch IOS interface configuration | Forces the port to operate as a trunk |
| `switchport trunk native vlan 999` | Switch IOS interface configuration | Changes the native VLAN from VLAN 1 to VLAN 999 |
| `switchport trunk allowed vlan 10,20,999` | Switch IOS interface configuration | Allows only VLANs 10, 20, and 999 across the trunk |

#### 6. Enable the Physical Router Interface

Before creating subinterfaces, the physical router interface must be enabled.

```cisco
Router(config)# interface gig0/0/0
Router(config-if)# no shutdown
```

| Command | Where | Purpose |
|---|---|---|
| `interface gig0/0/0` | Router IOS global configuration | Enters the physical interface connected to the switch |
| `no shutdown` | Router IOS interface configuration | Enables the physical router interface |

#### 7. Create a Subinterface for VLAN 10

```cisco
Router(config)# interface gig0/0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

| Command | Where | Purpose |
|---|---|---|
| `interface gig0/0/0.10` | Router IOS global configuration | Creates subinterface Gig0/0/0.10 |
| `encapsulation dot1Q 10` | Router IOS subinterface configuration | Associates this subinterface with VLAN 10 using 802.1Q tagging |
| `ip address 192.168.10.1 255.255.255.0` | Router IOS subinterface configuration | Assigns the default gateway IP address for VLAN 10 |

#### 8. Create a Subinterface for VLAN 20

backtick cisco
Router(config)# interface gig0/0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
backtick

#### Command Table

| Command | Where | Purpose |
|---|---|---|
| `interface gig0/0/0.20` | Router IOS global configuration | Creates subinterface Gig0/0/0.20 |
| `encapsulation dot1Q 20` | Router IOS subinterface configuration | Associates this subinterface with VLAN 20 using 802.1Q tagging |
| `ip address 192.168.20.1 255.255.255.0` | Router IOS subinterface configuration | Assigns the default gateway IP address for VLAN 20 |

---

#### 9. Configure the Native VLAN on the Router

If you change the native VLAN on the switch trunk, you must also inform the router.

```cisco
Router(config)# interface gig0/0/0.999
Router(config-subif)# encapsulation dot1Q 999 native
```

| Command | Where | Purpose |
|---|---|---|
| `interface gig0/0/0.999` | Router IOS global configuration | Creates a subinterface for the native VLAN |
| `encapsulation dot1Q 999 native` | Router IOS subinterface configuration | Tells the router that VLAN 999 is the native VLAN and should be treated as untagged traffic |


---
---
---

## 📖 Part 2 — SVI

#### 10. Concept

SVI stands for **Switch Virtual Interface**.

An SVI is a virtual Layer 3 interface created on a multilayer switch.  
Instead of using an external router, the Layer 3 switch routes traffic between VLANs internally.

This method is faster and more scalable than ROAS because the routing happens inside the switch hardware.

In SVI-based Inter-VLAN Routing:

- Each VLAN has one SVI.
- Each SVI has an IP address.
- That IP address becomes the default gateway for devices in that VLAN.
- IP routing must be enabled on the Layer 3 switch.

#### 11. Configure VLANs on the Layer 3 Switch

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name IT
```

| Command | Where | Purpose |
|---|---|---|
| `vlan 10` | Layer 3 Switch IOS global configuration | Creates VLAN 10 |
| `name HR` | Layer 3 Switch IOS VLAN configuration | Assigns the name HR to VLAN 10 |


#### 12. Create SVIs for Each VLAN

```cisco
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
```

| Command | Where | Purpose |
|---|---|---|
| `interface vlan 10` | Layer 3 Switch IOS global configuration | Creates or enters the SVI for VLAN 10 |
| `ip address 192.168.10.1 255.255.255.0` | Layer 3 Switch IOS interface configuration | Assigns the default gateway IP address for VLAN 10 |
| `no shutdown` | Layer 3 Switch IOS interface configuration | Enables the SVI |
| `interface vlan 20` | Layer 3 Switch IOS global configuration | Creates or enters the SVI for VLAN 20 |
| `ip address 192.168.20.1 255.255.255.0` | Layer 3 Switch IOS interface configuration | Assigns the default gateway IP address for VLAN 20 |

#### 13. Enable Layer 3 Routing on the Switch

```cisco
Switch(config)# ip routing
```

| Command | Where | Purpose |
|---|---|---|
| `ip routing` | Layer 3 Switch IOS global configuration | Enables routing between VLAN interfaces on a multilayer switch |


#### 14. Configure Access Ports on the Layer 3 Switch

```cisco
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```

| Command | Where | Purpose |
|---|---|---|
| `interface fa0/1` | Layer 3 Switch IOS global configuration | Enters interface configuration mode for Fa0/1 |
| `switchport mode access` | Layer 3 Switch IOS interface configuration | Forces the port to operate as an access port |
| `switchport access vlan 10` | Layer 3 Switch IOS interface configuration | Assigns Fa0/1 to VLAN 10 |

### ✅ Verification:

#### 1. Verify VLANs

```cisco
Switch# show vlan brief
```

| Command | Where | Purpose |
|---|---|---|
| `show vlan brief` | Switch IOS privileged EXEC mode | Displays VLANs, VLAN names, status, and assigned access ports |

#### 2. Verify Trunk Links

```cisco
Switch# show interfaces trunk
```

| Command | Where | Purpose |
|---|---|---|
| `show interfaces trunk` | Switch IOS privileged EXEC mode | Displays trunk interfaces, native VLAN, allowed VLANs, and trunking status |


#### 3. Verify Router Subinterfaces

```cisco
Router# show ip interface brief
Router# show running-config
```

| Command | Where | Purpose |
|---|---|---|
| `show ip interface brief` | Router IOS privileged EXEC mode | Displays interface status and assigned IP addresses |
| `show running-config` | Router IOS privileged EXEC mode | Displays the active configuration, including subinterfaces and encapsulation |

#### 4. Verify Layer 3 Switch Routing

```cisco
Switch# show ip interface brief
Switch# show ip route
```

| Command | Where | Purpose |
|---|---|---|
| `show ip interface brief` | Layer 3 Switch IOS privileged EXEC mode | Displays SVI status and IP addresses |
| `show ip route` | Layer 3 Switch IOS privileged EXEC mode | Displays the routing table of the Layer 3 switch |

#### 5. Test Connectivity

```cisco
PC> ping 192.168.20.10
PC> ping 192.168.10.1
PC> tracert 192.168.20.10
```

| Command | Where | Purpose |
|---|---|---|
| `ping 192.168.20.10` | PC command prompt | Tests end-to-end connectivity to a device in another VLAN |
| `ping 192.168.10.1` | PC command prompt | Tests connectivity to the local default gateway |
| `tracert 192.168.20.10` | PC command prompt | Shows the Layer 3 path toward the destination |

### ⚠️ Note:

1. **Routers do not understand VLANs by default.**  
   A normal physical router interface receives Ethernet frames, but it does not automatically know which VLAN each frame belongs to.  
   That is why ROAS requires subinterfaces and 802.1Q encapsulation.

2. **The router physical interface does not usually get an IP address in ROAS.**  
   IP addresses are configured on subinterfaces, not on the main physical interface.

3. **The switch port connected to the router must be a trunk.**  
   If this port is left in access mode, only one VLAN can pass through it.

4. **The native VLAN must match on both sides.**  
   If the switch uses VLAN 999 as the native VLAN, the router must also use:
   `encapsulation dot1Q 999 native`

5. **SVI requires a Layer 3 switch.**  
   A normal Layer 2 switch can create VLAN interfaces mainly for management, but it cannot route between VLANs unless it supports Layer 3 routing.

6. **For SVI to work, the VLAN must exist and be active.**  
   If there is no active port in that VLAN, the SVI may stay down.

7. **ROAS vs SVI:**
   - ROAS is simple and common in small labs.
   - SVI is faster and more scalable in enterprise networks.
   - In real campus networks, Inter-VLAN Routing is usually done on multilayer switches using SVIs.
   
