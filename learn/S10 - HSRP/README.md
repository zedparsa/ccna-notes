### 📝 Summary:
In a high-availability network, the **Default Gateway** is a single point of failure. If the router acting as the gateway fails, all hosts lose connectivity to external networks.  
**FHRP (First Hop Redundancy Protocol)** solves this by allowing multiple routers to act as a single **Virtual Router**.

**HSRP (Hot Standby Router Protocol)** is a Cisco-proprietary protocol that ensures network redundancy by grouping two or more routers into a virtual unit with a shared **Virtual IP (VIP)** and **Virtual MAC (VMAC)**.

### 🎯 Objectives:
- Understand the need for gateway redundancy.
- Compare HSRP, VRRP, and GLBP.
- Configure HSRP active and standby routers.
- Manipulate HSRP priority and understand the Preempt feature.
- Implement HSRP authentication for security.
- Verify HSRP states and operations.

### 🧩 Topology:
Two Multilayer Switches (DSW1 and DSW2) or Routers acting as gateways for VLAN 10.  
- **VIP:** 192.168.10.254  
- **DSW1:** Active (Priority 110)  
- **DSW2:** Standby (Priority 100)

### 🛠️ Step-by-Step:

#### 1. FHRP Protocols Overview

| Protocol | Owner | Characteristics |
| :--- | :--- | :--- |
| **HSRP** | Cisco | 1 Active, 1 Standby, Others Listen. Uses Hello messages. |
| **VRRP** | Open Standard | 1 Master, Others Backup. Similar to HSRP. |
| **GLBP** | Cisco | Load balances traffic across up to 4 routers simultaneously. |


#### 2. How HSRP Works (VIP & VMAC)
Instead of pointing hosts to a physical router's IP, we point them to a **Virtual IP (VIP)**.  
The routers in the HSRP group also share a **Virtual MAC address**:
- **HSRP v1:** `0000.0c07.acXX` (where XX is the group ID in hex)
- **HSRP v2:** `0000.0c9f.fXXX` (where XXX is the group ID in hex)


#### 3. Basic HSRP Configuration
You must configure the same Group Number and Virtual IP on both routers.

```cisco
Router(config)# interface vlan 10
Router(config-if)# standby 10 ip 192.168.10.254
Router(config-if)# standby 10 version 2
```

| Command | Where | Purpose |
|---|---|---|
| `standby <group> ip <vip>` | Interface Mode | Activates HSRP and sets the Virtual IP address |
| `standby version 2` | Interface Mode | Enables HSRP version 2 (supports more groups and IPv6) |

#### 4. Influencing Election (Priority & Preempt)
By default, the router with the **higher IP** becomes Active. However, we usually use **Priority** (Default: 100, Range: 0-255).  
**Preempt:** If a higher-priority router reboots and comes back online, it will **not** take back the Active role unless `preempt` is configured.

```cisco
Router(config-if)# standby 10 priority 110
Router(config-if)# standby 10 preempt
```

| Command | Where | Purpose |
|---|---|---|
| `standby <group> priority <val>` | Interface Mode | Sets priority (higher value wins the Active role) |
| `standby <group> preempt` | Interface Mode | Allows a higher-priority router to take over the Active role |

#### 5. Optimization (Timers)
The default Hello time is 3 seconds, and Hold time is 10 seconds. You can speed up the failover process:

```cisco
Router(config-if)# standby 10 timers 1 3
```

| Command | Where | Purpose |
|---|---|---|
| `standby <group> timers <h> <ht>` | Interface Mode | Sets Hello and Hold timers (in seconds or milliseconds) |


#### 6. HSRP Security (Authentication)
To prevent an attacker from introducing a fake high-priority router into the network (MITM attack), use authentication.

```cisco
Router(config-if)# standby 10 authentication md5 key-string P@ssw0rd123
```

| Command | Where | Purpose |
|---|---|---|
| `standby <group> authentication <key>` | Interface Mode | Sets a password for HSRP packets (prevents rogue HSRP neighbors) |


#### 7. Load Sharing with HSRP (MHSRP)
HSRP doesn't load balance by default, but you can achieve "Manual Load Balancing" by making Router A active for VLAN 10 and Router B active for VLAN 20.

### ✅ Verification:

**1. Check HSRP Status (Detailed):**
```cisco
Router# show standby
```

**2. Check HSRP Status (Summary):**
```cisco
Router# show standby brief
```

**3. Check Interface IP Status:**
```cisco
Router# show ip interface brief | exclude unassigned
```

| Command | Where | Purpose |
|---|---|---|
| `show standby` | Privileged EXEC | Shows detailed HSRP state, timers, and Virtual MAC |
| `show standby brief` | Privileged EXEC | Displays a summary table of Active/Standby states |
| `show ip interface brief` | Privileged EXEC | Verifies IP addresses and interface status |

### ⚠️ Note:

1. **Active vs. Standby Election:** The first router to boot up usually becomes Active regardless of priority. Use `preempt` to ensure your preferred router is always Active.
2. **HSRP version differences:** Version 1 uses multicast `224.0.0.2`, while Version 2 uses `224.0.0.102`. They are **not** compatible.
3. **Security Tip:** Always use MD5 authentication. Clear-text passwords can be easily sniffed and used to hijack the Active router role.
4. **Gratuitous ARP:** When a failover occurs, the new Active router sends a Gratuitous ARP to update the MAC tables of all switches in the LAN so they know the new path to the VMAC.
`
