<p align="center">
  <img src="https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white" alt="Cisco Packet Tracer" />
  <img src="https://img.shields.io/badge/VLSM-Subnetting-FF6F00?style=for-the-badge" alt="VLSM" />
  <img src="https://img.shields.io/badge/RIPv2-Dynamic_Routing-28A745?style=for-the-badge" alt="RIPv2" />
  <img src="https://img.shields.io/badge/Static-4_Types-D00000?style=for-the-badge" alt="Static Routing" />
  <img src="https://img.shields.io/badge/DHCP-Auto_Config-5586A4?style=for-the-badge" alt="DHCP" />
  <img src="https://img.shields.io/badge/License-Academic-lightgrey?style=for-the-badge" alt="License" />
</p>

<h1 align="center">🏥 Smart Healthcare Network Design</h1>

<p align="center">
  <strong>A multi-site hospital network connecting 810 devices across 6 medical units — designed for reliability, scalability, and zero-downtime failover.</strong>
</p>

<p align="center">
  A complete enterprise network simulation built in <strong>Cisco Packet Tracer</strong> for a healthcare system spanning 6 specialized units. The design applies <strong>VLSM subnetting</strong> from a single /16 block, demonstrates all <strong>4 types of static routing</strong> (next-hop, exit-interface, recursive, floating), implements <strong>RIPv2 dynamic routing</strong> with static redistribution, and configures <strong>DHCP servers</strong> for automatic device addressing — all with a failover path that activates when the primary link goes down.
</p>

---

## 📌 Why This Project?

Networking courses teach subnetting formulas and routing protocols in isolation. This project ties them together into a single, realistic deployment — a healthcare network where address waste matters (810 devices from one /16 block), where routing decisions have clinical implications (a pharmacy must always reach the central hub), and where redundancy isn't optional.

The simulation demonstrates:

- **VLSM address planning** — 6 subnets carved from 94.21.0.0/16 with zero overlap and minimal waste, sized to each unit's device count
- **Hybrid routing architecture** — static routes for predictable, low-overhead paths combined with RIPv2 for dynamic convergence and `redistribute static` for unified reachability
- **4 types of static routing in one network** — next-hop (CH→DC), exit-interface (EU→RC), recursive (PH→CH via RL), and floating static (RC→CH via EU, AD 130) — each chosen for the link it serves
- **DHCP relay and server configuration** — centralized pools with gateway and DNS assignment for plug-and-play device onboarding
- **Failover design** — floating static route with elevated administrative distance activates a backup path automatically when the primary serial link drops

---

## 🏗️ Network Topology

```
                              ┌──────────┐
                   S0/0/0     │    DC    │     150 devices
              ┌──────────────▶│  Router  │     94.21.2.0/24
              │  94.21.5.1/2  │          │
              │               └────┬─────┘
              │                    │ G0/0 → Switch → Email Server
              │                    │        94.21.2.1
         ┌────┴─────┐
         │    CH    │  300 devices   ◄── Central Hub
         │  Router  │  94.21.0.0/23
         │          │  DNS + Web + Email Servers
         └┬───┬────┘
          │   │
   S0/0/1 │   │ S0/1/0
          │   │
          ▼   ▼
    ┌─────────┐    ┌─────────┐          ┌──────────────┐
    │   RL    │    │   EU    │ S0/0/1   │     RC       │ 60 devices
    │ Router  │    │ Router  │─────────▶│   Router     │ 94.21.4.128/26
    │         │    │         │          │              │
    └──┬──┬──┘    └──┬──┬──┘          └──┬───┬──────┘
       │  │          │  │                │   │
  G0/0 │  │ G0/1     │  │ G0/1     G0/0 │   │ S0/0/0
       │  │          │  │               │   │
       ▼  ▼          ▼  ▼               ▼   ▼
    Switch  Core   Switch  Core      Switch  PH Router
    120 dev Switch  80 dev Switch    Email   ◄── via serial
    RL unit         EU unit          RC unit  94.21.5.13/14
    DHCP                            Floating static
    Email                           backup → EU

                    ┌─────────┐
                    │   PH    │  100 devices
                    │ Router  │  94.21.3.128/25
                    │         │  DHCP + Email
                    └──┬──┬──┘
                  G0/0 │  │ G0/1
                       │  │
                    Switch Core Switch
                    PH unit
                    Recursive static → CH via RL
```

All 6 routers are **Cisco 2911** with serial + gigabit interfaces. Switches are **Cisco 2960-24TT**. A **Core Switch** interconnects RL, PH, and EU via gigabit links for local east-west traffic.

---

## 📊 VLSM Addressing Scheme

All subnets are carved from **94.21.0.0/16** using Variable Length Subnet Masking to minimize address waste:

| Unit | Subnet | CIDR | Usable Hosts | Devices | Utilization |
|------|--------|------|-------------|---------|-------------|
| **CH** (Central Hub) | 94.21.0.0 | /23 | 510 | 300 | 59% |
| **DC** (Diagnostic Center) | 94.21.2.0 | /24 | 254 | 150 | 59% |
| **RL** (Research Lab) | 94.21.3.0 | /25 | 126 | 120 | 95% |
| **PH** (Pharmacy) | 94.21.3.128 | /25 | 126 | 100 | 79% |
| **EU** (Emergency Unit) | 94.21.4.0 | /25 | 126 | 80 | 63% |
| **RC** (Rehab Center) | 94.21.4.128 | /26 | 62 | 60 | 97% |

**WAN Links** use point-to-point /30 subnets from the **94.21.5.0/24** block.

---

## 🔀 Router Interface Table

| Router | Interface | IP Address | Connects To |
|--------|-----------|------------|-------------|
| **CH** | G0/0 | 94.21.0.1/23 | CH LAN |
| | S0/0/0 | 94.21.5.1/30 | → DC (S0/0/0) |
| | S0/0/1 | 94.21.5.5/30 | → RL (S0/0/0) |
| | S0/1/0 | 94.21.5.9/30 | → EU (S0/0/0) |
| **DC** | G0/0 | 94.21.2.1/24 | DC LAN |
| | S0/0/0 | 94.21.5.2/30 | → CH (S0/0/0) |
| **RL** | G0/0 | 94.21.3.1/25 | RL LAN |
| | G0/1 | 94.21.5.25/30 | → Core Switch |
| | S0/0/0 | 94.21.5.6/30 | → CH (S0/0/1) |
| **PH** | G0/0 | 94.21.3.129/25 | PH LAN |
| | G0/1 | 94.21.5.26/30 | → Core Switch |
| | S0/0/0 | 94.21.5.14/30 | → RC (S0/0/0) |
| **EU** | G0/0 | 94.21.4.1/25 | EU LAN |
| | G0/1 | 94.21.5.27/30 | → Core Switch |
| | S0/0/0 | 94.21.5.10/30 | → CH (S0/1/0) |
| | S0/0/1 | 94.21.5.18/30 | → RC (S0/0/1) |
| **RC** | G0/0 | 94.21.4.129/26 | RC LAN |
| | S0/0/0 | 94.21.5.13/30 | → PH (S0/0/0) |
| | S0/0/1 | 94.21.5.17/30 | → EU (S0/0/1) |

---

## 🖥️ Server Configuration

| Server | Unit | IP Address | Service |
|--------|------|------------|---------|
| DNS Server | CH | 94.21.0.2 | Domain name resolution (network-wide) |
| Web Server | CH | 94.21.0.3 | Centralized web services |
| CH Email | CH | 94.21.0.4 | Email for Central Hub |
| DC Email | DC | 94.21.2.2 | Email for Diagnostic Center |
| RL Email | RL | 94.21.3.3 | Email for Research Lab |
| RL DHCP | RL | 94.21.3.2 | DHCP for RL devices |
| PH Email | PH | 94.21.3.131 | Email for Pharmacy |
| PH DHCP | PH | 94.21.3.130 | DHCP for PH devices |
| EU Email | EU | 94.21.4.2 | Email for Emergency Unit |
| RC Email | RC | 94.21.4.130 | Email for Rehab Center |

---

## 🛣️ Routing Architecture

The network uses a **hybrid routing strategy** — static routes where paths are predictable, RIPv2 where dynamic convergence is needed, and a floating static route for automatic failover.

### 4 Types of Static Routing

| Type | Where Used | Route | Why This Type |
|------|-----------|-------|---------------|
| **Next-Hop** | CH → DC | `ip route 94.21.2.0 255.255.255.0 94.21.5.2` | Single serial link to DC — next-hop IP is unambiguous and avoids recursive lookup |
| **Exit-Interface** | EU → RC | `ip route 94.21.4.128 255.255.255.192 S0/0/1` | Point-to-point serial — specifying the exit interface avoids ARP on serial links |
| **Recursive** | PH → CH (via RL) | `ip route 94.21.0.0 255.254.0.0 94.21.5.25` | PH has no direct link to CH — the route resolves through RL's Core Switch interface |
| **Floating Static** | RC → CH (via EU) | `ip route 94.21.0.0 ... 94.21.5.18 130` | AD 130 > RIPv2's 120 — this backup path activates only when the primary route through PH disappears |

### RIPv2 Dynamic Routing

Configured on **CH** as the hub router:

```
router rip
 version 2
 network 94.21.0.0
 no auto-summary
 redistribute static
```

`redistribute static` injects all static routes into RIPv2 updates, so downstream routers learn paths to statically-routed subnets without manual configuration on every router.

### Failover Scenario

**Normal operation:** RC reaches CH through the path RC → PH → RL → CH (learned via RIPv2, AD 120).

**Primary link failure:** If the RC↔PH serial link drops, the RIPv2 route is withdrawn. The floating static route (AD 130) to CH via EU activates automatically — RC → EU → CH — with zero manual intervention.

---

## ⚙️ DHCP Configuration

Two dedicated DHCP servers provide automatic addressing:

| DHCP Server | Location | Pool | Gateway | DNS |
|-------------|----------|------|---------|-----|
| RL DHCP (94.21.3.2) | Research Lab | 94.21.3.0/25 | 94.21.3.1 | 94.21.0.2 |
| PH DHCP (94.21.3.130) | Pharmacy | 94.21.3.128/25 | 94.21.3.129 | 94.21.0.2 |

Other units (CH, DC, EU, RC) use **static IP assignment** for their devices, as these units host critical servers that require fixed addresses.

---

## 🚀 Getting Started

### Prerequisites

- **Cisco Packet Tracer 8.0+** (free with Cisco Networking Academy account)

### 1. Clone the Repository

```bash
git clone https://github.com/swapnilrob/Smart-Healthcare-Network-Design.git
cd Smart-Healthcare-Network-Design
```

### 2. Open the Simulation

1. Launch Cisco Packet Tracer
2. Open `Final.pkt`
3. The full network topology loads with all configurations pre-applied

### 3. Verify Connectivity

Test end-to-end reachability from any PC:

```
PC> ping 94.21.0.2        # Ping CH DNS Server (cross-network)
PC> ping 94.21.0.3        # Ping CH Web Server
PC> nslookup example.com  # Test DNS resolution
```

### 4. Test Failover

1. Click the serial link between **RC** and **PH** routers
2. Delete the link (simulating failure)
3. From any RC device, ping CH (94.21.0.1) — traffic now routes through **EU** via the floating static route
4. Restore the link — RIPv2 reconverges and the primary path resumes

---

## 📁 Project Structure

```
Smart-Healthcare-Network-Design/
├── Final.pkt                       # Cisco Packet Tracer simulation (complete network)
├── docs/
│   ├── Project_Specification.pdf   # Network requirements and design spec
│   ├── VLSM_Address_Config.pdf     # VLSM tree, IP tables, router configurations
│   └── Work_Distribution.pdf       # Team member contributions
├── topology/
│   └── network_topology.png        # Network topology screenshot
└── README.md                       # This file
```

---

## 🔧 Concepts Demonstrated

| Networking Concept | Implementation |
|-------------------|---------------|
| **VLSM Subnetting** | 6 variable-size subnets from 94.21.0.0/16 — /23 to /26 based on device count |
| **Next-Hop Static** | CH→DC direct path via next-hop IP address |
| **Exit-Interface Static** | EU→RC via outgoing serial interface |
| **Recursive Static** | PH→CH resolved through intermediate RL gateway |
| **Floating Static** | RC backup path via EU with AD 130 (activates on primary failure) |
| **RIPv2** | Dynamic routing on CH with `redistribute static` for network-wide reachability |
| **DHCP** | Centralized address pools on RL and PH with gateway/DNS auto-assignment |
| **DNS/Email/Web** | Service deployment across 10 servers with centralized DNS resolution |
| **Redundant Paths** | Dual connectivity for RC ensures no single point of failure |
| **WAN Design** | Point-to-point /30 serial links for inter-router communication |

---

## 👥 Team

| Name | Contributions |
|------|---------------|
| Audrija Chowdhury (22299421) | VLSM tree design — hand-computed subnet splits from /16 base; Packet Tracer topology layout — physical placement of 6 routers, 7 switches, and all device connections; DHCP server configuration — pool setup on RL and PH servers with gateway and DNS parameters |
| Swapnil Rob (22299138) | Server configuration — DNS, Web, and Email server deployment across all 6 units with static IP assignment; IP addressing — complete address table for router interfaces, servers, and WAN links; dynamic routing — RIPv2 configuration on CH with `redistribute static` and `no auto-summary` |
| Nafisa Hasan (22299141) | Static routing — implementation of all 4 types (next-hop on CH, exit-interface on EU, recursive on PH, floating static on RC with AD 130); server addressing and configuration; project documentation — VLSM tables, configuration command reference, and assumptions |

---

## 📄 Course Context

Developed for **CSE 421 — Computer Networks** at **BRAC University**. The project demonstrates practical application of enterprise network design concepts including VLSM address planning, hybrid static/dynamic routing architectures, DHCP auto-configuration, DNS-based service discovery, multi-site WAN connectivity with serial links, and automatic failover through floating static routes with administrative distance manipulation.

---

## 🧠 What We Learned

- **VLSM prevents address waste** — a flat /16 gives 65,534 hosts for 810 devices (1.2% utilization); VLSM carves subnets to fit, pushing utilization above 60% on every subnet and above 95% on the tightest ones (RL, RC)
- **Each static routing type exists for a reason** — next-hop is clearest for single paths, exit-interface avoids unnecessary ARP on point-to-point links, recursive handles multi-hop resolution, and floating static provides backup without manual intervention
- **`redistribute static` bridges two routing worlds** — without it, RIPv2-only routers can't reach statically-routed subnets; with it, CH acts as a translation layer between the static and dynamic domains
- **Administrative distance is the failover mechanism** — setting the floating static to AD 130 (above RIPv2's 120) means the backup path is always installed in the routing table but never preferred until the primary disappears
- **Healthcare networks demand redundancy** — a pharmacy that can't reach the central database means prescriptions can't be verified; the floating static on RC ensures no single serial link failure isolates a unit
