## 📖 Part 1 — Routing Fundamentals

### 📝 Summary:
Routing is the process by which a router forwards packets toward their destinations based on information stored in its routing table.  
A router’s forwarding decisions depend entirely on its Routing Information Base (RIB).  
Static routes let administrators manually define paths — providing reliability and control, but without adaptability to topology changes.

### 🎯 Objectives:
- Understand router forwarding logic based on the routing table.
- Learn how routes are added, selected, and prioritized.
- Differentiate between *directly connected*, *static*, and *learned* routes.
- Configure static routes using next-hop IP, exit interface, or both.
- Learn Longest Prefix Match and Administrative Distance rules.
- Understand default and floating static routes.
- Recognize AD values for all routing protocols.
- Verify route behavior and load balancing decisions.

### 🧩 Topology:
A simple lab with two or three routers in a line (R1 ↔ R2 ↔ R3) is sufficient to practice all static and connected route concepts.

---

### 🛠️ Step-by-Step:

```cisco
Router# show ip route
Router# show ip route static
```

These commands display the routing table in full and filter it for static routes only.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `show ip route` | Router IOS | Displays all routes with source codes and metrics |
| `show ip route static` | Router IOS | Shows static routes only from the routing table |

### Route Codes — Understanding RIB Symbols

| Code | Meaning | Description |
|---|---|---|
| `L` | Local | IP assigned to router interface (host route /32) |
| `C` | Connected | Directly connected network on active interface |
| `S` | Static | Manually configured static route |
| `S*` | Static Default | Default route pointing to 0.0.0.0/0 |
| `R` | RIP | Route learned via RIP protocol |
| `D` | EIGRP | Route learned via Enhanced IGRP |
| `O` | OSPF | Route learned via OSPF |
| `B` | BGP | Route learned via Border Gateway Protocol |

💡 **Tip:** You can use `show ip route` to see these symbols in the left column (before each prefix).

---

### Adding Static Routes

```cisco
Router(config)# ip route <destination-ip> <subnet-mask> <next-hop-ip>
```

Or using exit interface:

```cisco
Router(config)# ip route <destination-ip> <subnet-mask> <exit-interface>
```

#### Explanation of Options
- **Next-Hop IP:** IP address of the router that will forward your packet next.  
- **Exit Interface:** Outbound interface used to reach the destination directly.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `ip route <dest> <mask> <next-hop-ip>` | Router IOS (config mode) | Adds static route pointing to next-hop address |
| `ip route <dest> <mask> <interface>` | Router IOS (config mode) | Adds static route using a specific outgoing interface |


### Recursive Lookup and Route Resolution

When a static route is configured using a **next-hop IP address**, the router must determine **which interface can reach that next-hop**.
This process is called **recursive lookup**.

Example:

A static route points to next-hop `10.1.1.2`, but the router does not yet know which interface leads there.  
So the router searches its routing table again to find a route to `10.1.1.2`.
That second lookup resolves the **exit interface**.

Example logic:

`Destination → Next-hop → Exit interface`

This additional lookup is why some engineers prefer specifying the exit interface in certain scenarios.

### Directly Connected and Local Routes

Whenever an interface is configured with an IP address and becomes **up/up**, the router automatically installs two routes in the routing table.

1️⃣ **Connected Route (C)**  
Represents the network reachable through that interface.

2️⃣ **Local Route (L)**  
Represents the router’s own interface IP address as a /32 host route.

Example:

Interface: `192.168.1.1/24`

Routes added automatically:

- `C 192.168.1.0/24`
- `L 192.168.1.1/32`

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `show ip interface brief` | Router IOS | Displays interface status and assigned IP addresses |
| `show ip route connected` | Router IOS | Displays directly connected routes |


### Longest Prefix Match (LPM)

Routers choose the route with the **longest prefix**, meaning the most specific route.

Example:

- `192.168.0.0/16`
- `192.168.1.0/24`

Traffic destined for `192.168.1.x` matches the **/24 route** because it has a longer prefix.
This rule **always takes priority over Administrative Distance**.

Routing decision order:

1️⃣ Longest Prefix Match  
2️⃣ Administrative Distance  
3️⃣ Metric


### Administrative Distance (AD)

Administrative Distance indicates **how trustworthy a routing source is**.
Lower AD values are preferred.

| Source | AD |
|---|---|
| Connected | `0` |
| Static | `1` |
| EIGRP summary | `5` |
| External BGP | `20` |
| Internal EIGRP | `90` |
| IGRP | `100` |
| OSPF | `110` |
| RIP | `120` |
| EIGRP external | `170` |
| Unknown | `255` |

A route with AD `255` is considered **unreachable and will never be installed**.


### Static Route with Administrative Distance

```cisco
Router(config)# ip route <destination> <mask> <next-hop/interface> <AD>
```

Example:

```cisco
Router(config)# ip route 10.10.10.0 255.255.255.0 192.168.1.2 5
```

This installs a static route with AD value **5**.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `ip route <dst> <mask> <nh/interface> <AD>` | Router IOS (config mode) | Creates static route with custom AD |

### Default Route

A **default route** is used when no more specific route exists in the routing table.
It acts as a **gateway of last resort**.

Default route prefix:

`0.0.0.0/0`

Configuration:

```cisco
Router(config)# ip route 0.0.0.0 0.0.0.0 <next-hop>
```

Example:

```cisco
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

After configuration, the routing table will display:

`S* 0.0.0.0/0`

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `ip route 0.0.0.0 0.0.0.0 <next-hop>` | Router IOS (config mode) | Configures a default route |
| `show ip route` | Router IOS | Displays gateway of last resort |


### Floating Static Route

A **Floating Static Route** is a backup route that is installed only when the primary route fails.
This is achieved by configuring a **higher Administrative Distance**.

Example:

Primary route:

```cisco
Router(config)# ip route 10.10.10.0 255.255.255.0 192.168.1.1
```

Backup route:

```cisco
Router(config)# ip route 10.10.10.0 255.255.255.0 192.168.2.1 200
```

If the primary route disappears, the backup route becomes active.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `ip route <dst> <mask> <nh> 200` | Router IOS (config mode) | Creates floating backup route |


### Load Balancing (Equal-Cost Paths)

If a router learns **multiple routes with equal cost**, it can install multiple paths in the routing table.
This is called **Equal-Cost Multi-Path (ECMP)**.

Example scenario:

Two routes to the same network:

- `via Router A`
- `via Router B`

If both have the same metric and AD, the router may install both.
Traffic will be distributed between them.

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `show ip route` | Router IOS | Displays multiple equal-cost routes if present |


### ✅ Verification

```cisco
Router# show ip route
Router# show running-config | include ip route
Router# show ip interface brief
```

#### Command Table
| Command | Where | Purpose |
|---|---|---|
| `show ip route` | Router IOS | Verifies routes installed in routing table |
| `show running-config \| include ip route` | Router IOS | Shows configured static routes |
| `show ip interface brief` | Router IOS | Displays interface states and IP addresses |


### ⚠️ Note

Static routes provide **predictable control** over traffic paths but do not adapt automatically to topology changes.
In large networks, **dynamic routing protocols** (RIP, OSPF, EIGRP, BGP) are used to automatically learn and update routes.
However, static routes remain important for:

- `Default routes`
- `Small networks`
- Backup paths
- Security‑controlled routing
