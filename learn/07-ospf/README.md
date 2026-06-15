## 📖 Part 3 — OSPF (Open Shortest Path First)

### 📝 Summary:
OSPF is a robust, link-state Interior Gateway Protocol (IGP) designed for scalability and fast convergence.  
Unlike distance-vector protocols (like RIP) that share their entire routing table, OSPF routers build a complete "map" of the network using Link-State Advertisements (LSAs) and calculate the shortest path using the Dijkstra (SPF) algorithm.

### 🎯 Objectives:
- Understand OSPF neighbor adjacency requirements.
- Master the DR/BDR election process.
- Configure OSPF using network statements or direct interface activation.
- Manage OSPF metrics and reference bandwidth.
- Implement passive interfaces to secure network segments.
- Understand multi-area hierarchical design (ABR/ASBR).

### 🧩 Topology:
Multi-router topology (e.g., Star or Mesh) with multiple areas (Backbone Area 0 and secondary areas like Area 10 or 20) to demonstrate ABR/ASBR roles.

---

### 🛠️ Step-by-Step:

#### 1. Configuring OSPF
There are two common methods to enable OSPF on an interface:

**Method A: Network Command**
```cisco
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
```

**Method B: Interface Activation (Modern approach)**
```cisco
Router(config)# interface gi0/0
Router(config-if)# ip ospf 1 area 0
```

#### 2. Fine-tuning Cost and Passive Interfaces
To control updates and influence path selection:
```cisco
Router(config)# router ospf 1
Router(config-router)# auto-cost reference-bandwidth 1000         # Sets ref-BW to 1Gbps
Router(config-router)# passive-interface default                   # Block updates everywhere
Router(config-router)# no passive-interface gi0/0                  # Allow updates on specific link
```

#### Command Table
| Command | Purpose |
|---|---|
| router ospf <id> | Enters OSPF configuration mode |
| router-id <A.B.C.D> | Manually sets the OSPF Router ID |
| network <net> <wildcard> area <x> | Activates OSPF on target subnet |
| ip ospf cost <n> | Manually defines the cost of an interface |
| passive-interface <int> | Suppresses hello packets on interface |

---

### ✅ Verification:
```cisco
Router# show ip ospf neighbor          # Check adjacency and DR/BDR roles  
Router# show ip ospf interface brief   # View OSPF-enabled interfaces and costs  
Router# show ip ospf database          # Display LSDB content  
Router# show ip protocols              # Verify OSPF settings and timers  
Router# debug ip ospf events           # Real-time OSPF operations (careful in labs)
```

---

### ⚠️ Note:

#### 1️⃣ Router ID Selection Priority
The OSPF Router ID (RID) uniquely identifies each router and is selected automatically in this order:

1. **Configured Manually:** using `router-id`  
2. **Highest Loopback IP** (active interface)  
3. **Highest Physical Interface IP**

Once OSPF has started, the RID remains fixed until OSPF restarts.

---

#### 2️⃣ DR / BDR Election Rules
In multi-access networks (e.g. Ethernet segments), OSPF reduces update traffic by electing a **Designated Router (DR)** and a **Backup Designated Router (BDR)**.

- **Election Criteria:**
  - Highest `OSPF Priority` (0–255, default 1)
  - If tie → highest `Router ID`
- **Destinations:**
  - DR sends updates → 224.0.0.6  
  - Other routers send Hellos → 224.0.0.5
- **Note:** The election is **non-preemptive**. New routers joining later will not replace the existing DR until it goes down.

---

#### 3️⃣ OSPF Neighbor Adjacency Requirements
Routers must agree on these parameters to become neighbors:
- Same IP network/subnet (layer 3 adjacency)
- Identical **Hello interval** (default 10s on Ethernet)
- Identical **Dead interval** (default 40s)
- Same **Area ID**
- Same **Area type** (normal / stub / NSSA)
- Same **Authentication password** (if configured)

Failing any of these will prevent adjacency.

---

#### 4️⃣ Hello Packet Summary
Hello packets serve as OSPF’s keepalive and neighbor discovery mechanism.  
Contain fields like:
- Router ID  
- Hello / Dead Intervals  
- Subnet Mask  
- Area ID  
- DR/BDR IPs  
- Authentication info (if any)

Default multicast destination: **224.0.0.5**

---

#### 5️⃣ Interface Cost and Reference Bandwidth
OSPF cost = `Reference Bandwidth / Interface Bandwidth`

Default reference bandwidth = 100 Mbps.  
You can customize it globally:
```cisco
Router(config)# router ospf 1
Router(config-router)# auto-cost reference-bandwidth <Mbps>
```

Loopback interfaces always have a default cost of **1**.  
You can verify per-interface cost:
```cisco
Router# show ip ospf interface brief
```

Manual cost override:
```cisco
Router(config)# interface Gi0/0
Router(config-if)# ip ospf cost 10
```

---

#### 6️⃣ Multi-Area OSPF
OSPF uses a **hierarchical area design** for scalability:

- **Backbone Area (0):** Core of the OSPF domain (mandatory).  
- **ABR (Area Border Router):** Connects one or more non-backbone areas to Area 0.  
- **ASBR (Autonomous System Boundary Router):** Redistributes routes from other routing domains or protocols (e.g., RIP → OSPF).

Example:
Area 10 ↔ Area 0 ↔ Area 20
- Router connecting Area 0 and Area 10 = ABR  
- Router importing external routes = ASBR

---

#### 7️⃣ Loopback Interface as Router-ID / Management
Loopbacks are ideal for stable router identification:
```cisco
Router(config)# interface loopback0
Router(config-if)# ip address 1.1.1.1 255.255.255.255
Router(config-if)# ip ospf 1 area 0
```

By default OSPF advertises loopback as `/32`.  
To advertise the real subnet:
backtick cisco
Router(config-if)# ip ospf network point-to-point
backtick

---

### 🧠 Conceptual Analogy:
Imagine routers as post offices in a city:
- Without DR/BDR, every office floods updates to every other office — chaotic.  
- With DR/BDR, each office tells only the **Designated Post Office**, which updates everyone else efficiently.  
This is precisely the **DR/BDR optimization logic** in OSPF.

