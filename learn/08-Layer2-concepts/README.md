## 📖 Part 1 — Campus LAN Architecture

### 📝 Summary:
Campus LAN architecture is a hierarchical design model used in enterprise networks to improve scalability, performance, and manageability.  
Instead of connecting all devices randomly, the network is divided into logical layers, each responsible for specific tasks.

This model simplifies troubleshooting, improves fault isolation, and allows the network to scale as the organization grows.

### 🎯 Objectives:
- Understand the hierarchical campus LAN design model.
- Identify the responsibilities of each layer.
- Learn how traffic flows inside enterprise networks.
- Understand the concept of collapsed core architecture.

### 🧩 Topology:
Typical enterprise campus network containing access switches connected to distribution switches, which then connect to core switches that form the backbone of the network.


### 🛠️ Step-by-Step:

Campus LAN hierarchical design usually contains four layers:

**1️⃣ Access Layer**
This is the layer where end devices connect to the network.

Examples:
- PCs
- Printers
- IP Phones
- Wireless Access Points

Responsibilities:
- Device connectivity
- VLAN assignment
- Port security
- Basic switching operations


**2️⃣ Distribution Layer (Aggregation Layer)**

This layer aggregates multiple access switches and performs policy-based operations.

Responsibilities:
- Inter-VLAN routing
- Access control policies
- Filtering and QoS policies
- Route summarization


**3️⃣ Core Layer**

The core layer acts as the high-speed backbone of the network.

Responsibilities:
- Very fast packet forwarding
- High availability
- Minimal latency
- No heavy filtering or complex policies


**4️⃣ Edge Layer**

The edge layer represents the boundary between the internal network and external networks.

Examples:
- Internet connections
- WAN links
- Firewalls
- VPN gateways


### Collapsed Core Design

In small or medium networks, the **core layer and distribution layer can be merged** into a single layer.

This design is called:

Collapsed Core

Advantages:
- Lower cost
- Simpler topology
- Easier management

### ⚠️ Note:
The hierarchical design greatly improves scalability.  
Large enterprise networks almost always follow this model because it separates responsibilities between layers and prevents network complexity.

---
---
---

## 📖 Part 2 — VLAN (Virtual Local Area Network)

### 📝 Summary:
A VLAN (Virtual Local Area Network) is a logical segmentation of a Layer 2 network.  
It allows administrators to divide a physical switch into multiple broadcast domains.

Instead of every device belonging to one large network, VLANs allow devices to be grouped logically regardless of their physical location.

### 🎯 Objectives:
- Understand the concept of VLANs and broadcast domains.
- Learn why VLAN segmentation improves network performance and security.
- Create and manage VLANs on switches.
- Assign switch ports to specific VLANs.
- Verify VLAN configuration using IOS commands.

### 🧩 Topology:
One or more switches with multiple end devices connected to different VLANs (for example VLAN 10 for HR and VLAN 20 for IT).


### 🛠️ Step-by-Step:

### Why VLANs are Important

In a normal switch network without VLANs:

- All devices belong to the same broadcast domain.
- Broadcast traffic is sent to every device.

This can cause:
- Network congestion
- Security issues
- Difficult network management

By creating VLANs, we can:

- Reduce broadcast domains
- Improve security
- Organize the network logically

Example:
- VLAN 10 → HR department
- VLAN 20 → IT department
- VLAN 30 → Finance department

Devices in different VLANs **cannot communicate directly** without Layer 3 routing.


### Checking MAC Address Table

```cisco
Switch# show mac address-table
```

This command displays learned MAC addresses and the VLAN in which they were learned.

### Viewing VLAN Information

```cisco
Switch# show vlan brief
```

This command shows:
- VLAN ID
- VLAN name
- Status
- Assigned ports

By default, all switch ports belong to **VLAN 1**, which is the default VLAN.


### Creating a VLAN

Main method:

```cisco
Switch(config)# vlan 10
```

The switch creates VLAN 10 and assigns a default name.

### Changing VLAN Name

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name HR
```

This assigns a descriptive name to the VLAN.

### Assigning a VLAN to a Single Port

```cisco
Switch(config)# interface fa0/1
Switch(config-if)# switchport access vlan 10
```

This assigns interface Fa0/1 to VLAN 10.

### Assigning VLAN to Multiple Ports

```cisco
Switch(config)# interface range fa0/1 - 5
Switch(config-if-range)# switchport access vlan 10
```

This assigns interfaces Fa0/1 through Fa0/5 to VLAN 10.

### Automatic VLAN Creation

If you assign a port to a VLAN that does not yet exist:

```cisco
Switch(config)# interface fa0/10
Switch(config-if)# switchport access vlan 20
```

The switch will **automatically create VLAN 20**.

### VLAN Database File

If you check the flash storage:

```cisco
Switch# show flash:
```

You will see a file named: `vlan.dat`  

This file stores the VLAN database of the switch.

### ✅ Verification:

```cisco
Switch# show vlan brief
Switch# show mac address-table
Switch# show flash:
```

These commands allow you to verify:
- Existing VLANs
- MAC address learning
- VLAN database storage

### ⚠️ Note:

Devices inside the same VLAN can communicate directly at Layer 2.  
However, communication between different VLANs requires **Layer 3 routing**, usually implemented using:

- Router-on-a-Stick
- Layer 3 Switch (SVI)

---
---
---

## 📖 Part 3 — Trunking (802.1Q)

### 📝 Summary:
When you have multiple VLANs across multiple switches, you need a way for switches to distinguish which VLAN a frame belongs to.  
A **Trunk** is a point-to-point link between two switches (or a switch and a router) that carries traffic for **multiple VLANs** simultaneously.  
To differentiate between VLANs, switches use **Encapsulation** (adding a "tag" to the frame).

### 🎯 Objectives:
- Understand the necessity of Trunk links in multi-switch environments.
- Learn the IEEE 802.1Q (Dot1Q) encapsulation standard.
- Configure Trunk ports on Cisco switches.
- Understand the role of Native VLAN.
- Verify trunk configuration.

### 🧩 Topology:
Two switches (SW1 and SW2) connected via a single physical link. Both switches have multiple VLANs (10, 20) defined, and that single link must carry traffic for all those VLANs.

### 🛠️ Step-by-Step:

#### 1. Why Trunk?
If you have VLAN 10 users in Building A and VLAN 10 users in Building B, you cannot use separate cables for every VLAN.  
Trunking allows you to send traffic for all VLANs over **one single physical link** while keeping the traffic logically separated using tags.

#### 2. Encapsulation (Dot1Q)
When a frame leaves a switch via a trunk port, the switch inserts a 4-byte **VLAN Tag** into the Ethernet header.  
The industry standard for this is **IEEE 802.1Q (Dot1Q)**.

#### 3. Configuration
On most modern Cisco switches, you must explicitly set the encapsulation type before enabling trunk mode.

```cisco
Switch(config)# interface gig0/1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```

**Crucial Point:** Trunking must be configured on **both ends** of the link between the two switches.

### 🔍 Native VLAN
In Dot1Q, there is a special VLAN called the **Native VLAN** (default is VLAN 1).  
Traffic belonging to the Native VLAN is **not tagged** when it crosses the trunk.  

- If a switch receives an untagged frame on a trunk, it automatically assumes it belongs to the Native VLAN.
- For security reasons, it is a best practice to change the Native VLAN from 1 to an unused VLAN ID.

### ✅ Verification:

**1. Viewing VLANs**
Note that trunk ports **do not appear** in `show vlan brief`.

**2. Viewing Trunk Status**
```cisco
Switch# show interfaces trunk
```

This command displays:
- The trunk interface
- Encapsulation type (802.1q)
- Status (trunking)
- Native VLAN
- VLANs allowed on the trunk

### ⚠️ Note:

**1. Trunking Modes:**
- `switchport mode trunk`: Forces the port to be a trunk.
- `switchport mode access`: Forces the port to be an access port.
- `switchport nonegotiate`: Disables Dynamic Trunking Protocol (DTP) packets (recommended for security).

**2. Security Tip:**
- **Never use VLAN 1** as your data VLAN or Native VLAN.
- If the Native VLAN on one side of the trunk is different from the other side, you will get a **VLAN Mismatch Error**, causing traffic issues.

---
---
---

## 📖 Part 4 — Dynamic Trunking Protocol (DTP)

### 📝 Summary:
**DTP (Dynamic Trunking Protocol)** is a Cisco proprietary protocol that allows two switches to negotiate whether a link should become an **Access** port or a **Trunk** port automatically.  
While it simplifies configuration, it is often disabled in high-security environments to prevent "VLAN Hopping" attacks.

### 🎯 Objectives:
- Understand DTP operational modes.
- Analyze the DTP negotiation matrix.
- Learn how to verify the administrative vs. operational state of a port.
- Implement security best practices by disabling DTP.

### 🧩 Topology:
Two switches (SW-A and SW-B) connected directly. We will observe how changing the DTP mode on one side affects the operational state of the link.


### 🛠️ Step-by-Step:

#### 1. DTP Negotiable Modes
There are four primary modes for a switch port:

| Mode | Description |
| :--- | :--- |
| **Access** | Forces the port to be a permanent non-trunk port. |
| **Trunk** | Forces the port to be a permanent trunk and negotiates to convert the neighbor. |
| **Dynamic Auto** | Default on most switches. It waits for the other side to "ask" to become a trunk. |
| **Dynamic Desirable** | Actively asks the other side to become a trunk. |


#### 2. The DTP Negotiation Matrix
This table explains what the **Operational Mode** of the link will be based on the **Administrative Mode** of both sides:

| | Dynamic Auto | Dynamic Desirable | Trunk | Access |
| :--- | :---: | :---: | :---: | :---: |
| **Dynamic Auto** | Access | **Trunk** | **Trunk** | Access |
| **Dynamic Desirable** | **Trunk** | **Trunk** | **Trunk** | Access |
| **Trunk** | **Trunk** | **Trunk** | **Trunk** | **Conflict/Error** |
| **Access** | Access | Access | **Conflict/Error** | Access |

**Key Takeaways from the Matrix:**
- If both sides are **Dynamic Auto**, the link becomes **Access** (because neither side starts the negotiation).
- **Dynamic Desirable** is "aggressive"; if the other side is Auto, Desirable, or Trunk, the link becomes a **Trunk**.
- **Access** always wins unless the other side is forced to **Trunk** (which creates a configuration mismatch).


#### 3. Configuration Commands

**Setting a port to Actively Negotiate:**
```cisco
Switch(config)# interface gig0/1
Switch(config-if)# switchport mode dynamic desirable
```

**Disabling DTP (The Professional/Secure Way):**
To prevent a port from sending DTP frames, use the `nonegotiate` command. This is only possible if the port is manually set to Trunk or Access.
```cisco
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate
```


### ✅ Verification:

**1. Detailed Switchport Status:**
This is the most important command to see what is happening "under the hood".
```cisco
Switch# show interface fa0/1 switchport
```

Look for these lines in the output:
- **Administrative Mode:** What you configured (e.g., dynamic auto).
- **Operational Mode:** What the port actually became (e.g., static access).
- **Negotiation of Trunking:** Shows if DTP is On or Off.

**2. Verifying DTP packets:**
```cisco
Switch# show dtp
```
(This shows how many DTP packets are being sent/received).


### ⚠️ Note: Best Practice for Security

In a professional environment (Offensive Security mindset):
1. **Never leave ports in Dynamic Auto.** An attacker can send a DTP "Desirable" packet and turn their PC link into a Trunk, gaining access to all VLANs (VLAN Hopping).
2. **Static Configuration:** Always use `switchport mode trunk` and `switchport nonegotiate` on switch-to-switch links.
3. **Access Ports:** On ports connected to end-devices (PCs, Laptops), use `switchport mode access` and disable DTP.

---
---
---

## 📖 Part 5 — VLAN Management: Allowed VLANs & VTP

### 📝 Summary:
VLAN management involves controlling which VLANs can cross a trunk link and automating VLAN creation across multiple switches. 
- **Allowed VLANs:** A security and performance feature to restrict specific VLANs from traversing a trunk.
- **VTP (VLAN Trunking Protocol):** A Cisco proprietary protocol that synchronizes the VLAN database (vlan.dat) across all switches in a VTP domain.

### 🎯 Objectives:
- Learn how to prune VLANs manually using the Allowed list.
- Understand VTP modes (Server, Client, Transparent).
- Understand the dangers of the VTP Revision Number.
- Learn how to reset a switch's VLAN database before adding it to a network.

### 🧩 Topology:
A Core Switch (VTP Server) connected to multiple Access Switches (VTP Clients) via Trunk links.


### 🛠️ Step-by-Step:

#### 1. Restricting VLANs on a Trunk (Allowed List)
By default, a trunk link allows **all** VLANs (1-4094). To improve security and reduce unnecessary broadcast traffic, we manually specify which VLANs are allowed.

backtick cisco
Switch(config)# interface gig0/1
Switch(config-if)# switchport trunk allowed vlan 10,20,30
backtick

**Pro Tip:** If you want to add a VLAN later without removing the existing ones, use:
`switchport trunk allowed vlan add 40`


#### 2. VTP (VLAN Trunking Protocol) Concepts
VTP allows you to create a VLAN on one switch (Server) and have it propagate to all other switches automatically.

**VTP Modes:**
- **Server (Default):** Can create, modify, and delete VLANs. Sends advertisements.
- **Client:** Cannot change VLANs. It listens to advertisements and updates its database.
- **Transparent:** Does not synchronize its database with the server. It only forwards VTP advertisements to other switches. Useful when you want a switch to have its own unique local VLANs.

**VTP Configuration:**
backtick cisco
Switch(config)# vtp domain CiscoLab
Switch(config)# vtp password P@ssw0rd
Switch(config)# vtp mode server  (or client/transparent)
Switch(config)# vtp version 2
backtick


#### 3. The Revision Number (The "Switch Killer")
Every time the VLAN database changes on a Server, the **Configuration Revision Number** increases by 1.
- If a switch receives a VTP advertisement with a **higher** Revision Number, it immediately overwrites its own `vlan.dat` file.
- **The Danger:** If you plug in an old "Server" switch from a warehouse that has a Revision Number of 100 into a fresh network with Revision 10, the old switch will **wipe out** all VLANs in the entire network.


#### 4. VTP Pruning
VTP Pruning increases available bandwidth by restricting flooded traffic (broadcast/unknown unicast) to those trunk links that the traffic must use to reach the destination devices.

backtick cisco
Switch(config)# vtp pruning
backtick


#### 5. Clearing the Database (Decommissioning a Switch)
Before adding an old switch to your network, you **must** wipe its configuration and VLAN database to reset the Revision Number to 0.

backtick cisco
Switch# erase startup-config
Switch# delete flash:vlan.dat
Switch# reload
backtick

### ✅ Verification:

**To check VTP status, Revision Number, and Domain:**
backtick cisco
Switch# show vtp status
backtick

**To see the configured VTP password:**
backtick cisco
Switch# show vtp password
backtick

**To verify which VLANs are actually crossing the trunk:**
backtick cisco
Switch# show interfaces trunk
backtick

### ⚠️ Note:

1. **VTP Version 3:** Unlike versions 1 and 2, version 3 allows better password encryption and supports Extended VLANs (above 1005).
2. **Security Risk:** VTP is generally discouraged in modern high-security networks because it is easy to accidentally wipe out all VLANs. Many engineers prefer **VTP Mode Off** or **Transparent**.
3. **Synchronization:** VTP only works over **Trunk** links. It will not synchronize over Access links.

