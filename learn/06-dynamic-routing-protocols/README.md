## 📘 Part 1 — Dynamic Routing Protocols

### 📝 Summary
Dynamic routing protocols allow routers to automatically exchange routing information with each other.  
Unlike static routes which are manually configured and do not change unless an administrator modifies them, dynamic protocols learn routes, detect failures, and select optimal paths based on metrics.

This section introduces:
- Dynamic vs Static Routing  
- Loopback Interface fundamentals  
- RIP version 2 configuration  
- Routing protocol classification (IGP vs EGP)

### 🎯 Objectives
- Understand the difference between static and dynamic routing.
- Learn the purpose and behavior of loopback interfaces.
- Configure a loopback interface using IOS commands.
- Configure RIP version 2 with best practices (version 2, no auto-summary).
- Interpret RIP routes in the routing table.
- Understand IGP vs EGP protocol categories.

### 🧩 Topology
A simple two‑router or three‑router linear topology (R1 ↔ R2 ↔ R3) is enough to demonstrate:
- Loopback interfaces
- RIP route propagation
- Hop count behavior
- Route installation in the RIB


### 📍 Dynamic Routing vs Static Routes

#### Static Routes
Manually configured by the administrator.  
Advantages:
- Predictable  
- Secure  
- No unnecessary updates  

Disadvantages:
- No automatic failover  
- Poor scalability  
- High administrative overhead in large networks  

#### Dynamic Routing
Routers automatically:
- Discover networks  
- Advertise routes  
- Detect topology changes  
- Reroute traffic dynamically  

Advantages:
- Scalable  
- Self‑healing  
- Less operational overhead  

Disadvantages:
- More CPU/memory usage  
- Potential routing loops if misconfigured  
- Less predictable compared to static routes  


### 🧱 Loopback Interface

A **loopback interface** is a software‑based virtual interface that is always **up/up** unless manually shutdown.

#### Why loopbacks are used?
- They provide a **stable, never‑down** IP for routers.
- Commonly used for router IDs in dynamic routing protocols.
- Useful for testing, management, and logical addressing.
- Helpful when no physical interfaces are currently available.

#### Configuration

```cisco
Router(config)# interface loopback <number>
Router(config-if)# ip address <ip> <subnet-mask>
```

#### Notes
- Loopback interfaces are **up by default**.
- You can shut them if needed:

```cisco
Router(config-if)# shutdown
```

- They appear in `show ip interface brief`.

#### Verification

```cisco
Router# show ip interface brief | exclude unassigned
```

This displays all active interfaces, including loopbacks.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `interface loopback <num>` | Router IOS (config) | Creates a loopback interface |
| `ip address <ip> <mask>` | Router IOS (config-if) | Assigns IP to loopback |
| `shutdown` | Router IOS (config-if) | Disables loopback manually |
| `show ip interface brief` | Router IOS | Displays interface status |


### 🔄 RIP Dynamic Routing

RIP (Routing Information Protocol) is one of the oldest IGPs.  
It is a **distance‑vector** protocol using **hop count** as its metric.

Maximum hop count: **15**  
If hop count reaches 16 → network is **unreachable**.


### 🛠️ RIP v2 Configuration

```cisco
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
Router(config-router)# network <ip-address>
```

#### Why version 2?
- Supports subnet masks (classless)
- Multicasts updates to 224.0.0.9 instead of broadcasting
- Supports VLSM and CIDR

#### Why no auto-summary?
- Prevents classful route summarization  
- Ensures correctness in discontiguous networks  
- Required for modern subnetting topologies


### 🔍 Checking Routes

After RIP configuration, use:

```cisco
Router# show ip route
```

Learned RIP routes begin with:

`R <network>`

Example:

`R 10.10.10.0/24 [120/2] via 192.168.1.2`

Interpretation:
- `R` = learned from RIP  
- `120` = Administrative Distance of RIP  
- `2` = hop count metric  

### 🧪 Verification Commands

```cisco
Router# show ip protocols
Router# show ip route rip
Router# debug ip rip
```

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `router rip` | Router IOS | Enters RIP configuration mode |
| `version 2` | Router IOS (router-config) | Enables RIPv2 |
| `no auto-summary` | Router IOS | Disables classful summarization |
| `network <ip>` | Router IOS | Advertises network into RIP |
| `show ip protocols` | Router IOS | Shows routing protocol settings |
| `show ip route rip` | Router IOS | Displays RIP-learned routes |


### 🧠 Theoretical Concepts

#### Routing Protocol Categories

Dynamic routing protocols are divided into two major families:

#### 1. Interior Gateway Protocols (IGPs)
Used **inside** an autonomous system (enterprise networks).

Examples:
- `RIP`
- `OSPF`
- `IS-IS`
- `EIGRP`

Characteristics:
- Fast convergence (modern protocols)
- Optimized for internal communication


#### 2. Exterior Gateway Protocols (EGPs)
Used **between** autonomous systems on the Internet.

Example:
- BGP (Border Gateway Protocol) — the only mainstream EGP

Characteristics:
- Extremely scalable  
- Policy‑based routing  
- Used by ISPs and backbone networks  
