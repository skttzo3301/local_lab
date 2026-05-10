# Homelab Recovery Context — LLM Ingestion Version

Generated: 2026-05-10

Purpose: high-fidelity reconstruction of the user's last-known homelab configuration for transfer into another ChatGPT project or LLM context.

This file is intentionally explicit and dense. It includes confirmed facts, inferred facts, conflicts, and unknowns. Do not treat inferred/uncertain items as confirmed without verification.

---

## 0. Reconstruction Confidence Model

Use these labels:

- CONFIRMED: explicitly known from previous project context.
- OBSERVED: derived from actual tests or observed behavior.
- INFERRED: likely based on behavior/topology, but not directly confirmed.
- CONFLICTING: multiple prior contexts disagree or represent different stages.
- PLANNED: intended design, not necessarily implemented.
- UNKNOWN: not present in the captured project context.

---

## 1. User Intent / Current Task

The user has not visited the homelab project for a while and wants a maximum-fidelity summary of the current/last-known configuration.

The user plans to plug one more PC alongside the main PC currently being used. The user explicitly does not want hardware recommendations in this recovery document. They want the existing/latest configuration summarized: network anatomy, IPs, firewall rules, switch configuration, machine types/specs, and any known details needed to recover context in another project.

This document should be treated as a baseline memory dump, not a target redesign.

---

## 2. High-Level Current Physical Topology

CONFIRMED latest topology:

```text
ISP1 + ISP2
   → pfSense / OPNsense-capable firewall appliance
   → TP-Link Omada SG3428X-M2 managed L3-capable switch
   → devices / endpoints:
        - main PC
        - planned additional PC
        - future/suspended Kubernetes nodes
        - separate Linux utility / jump / Docker host
```

More specific physical topology from prior context:

```text
ISP routers
  → pfSense
      WAN  = igc0 = ISP1
      OPT1 = igc1 = ISP2
      LAN  = igc3 = trunk parent toward switch
      OPT2 = VLAN100 on igc3
  → TP-Link SG3428X-M2
      Port 1 = uplink/trunk to pfSense LAN/igc3
      Ports 2–28 = endpoint/access ports
  → main PC and future lab hosts
```

CONFIRMED port mapping:

```text
pfSense WAN  → igc0 → ISP1
pfSense OPT1 → igc1 → ISP2
pfSense LAN  → igc3 → trunk parent toward TP-Link switch
pfSense OPT2 → VLAN100 on igc3
```

Important clarification:

- OPT1 is ISP2 / second WAN.
- VLAN100 is OPT2, not OPT1.
- LAN/igc3 is the trunk parent toward the switch.

---

## 3. Hardware Inventory — Known Devices and Specs

## 3.1 pfSense / OPNsense Firewall Appliance

Status: CONFIRMED as installed/used, but exact hardware model has some conflict across older/newer context.

Role:

```text
Firewall edge device
WAN failover
NAT
Perimeter firewall
Default gateway for VLAN100 clients
DHCP for VLAN100 likely/used
```

Known/remembered specs, with conflict preserved:

### Newer remembered description

```text
Device class: fanless N305-class firewall appliance
NICs: 4–6 × 2.5GbE Intel NICs
Use: pfSense / OPNsense-capable firewall/router
```

### Older/project-memory description

```text
CPU: Intel N2940 quad-core
NICs: 4 × RJ45 1GbE
AES-NI: supported
Use: pfSense / OPNsense-capable firewall/router
```

CONFLICTING interpretation:

```text
There are two remembered descriptions for the firewall hardware:
1. N305-class, 4–6 × 2.5GbE Intel NICs.
2. N2940 quad-core, 4 × 1GbE RJ45, AES-NI.

Treat the currently installed firewall as confirmed, but verify the exact CPU/NIC hardware in the pfSense dashboard or device order history.
```

Known interface mapping:

```text
igc0 = WAN / ISP1
igc1 = OPT1 / ISP2
igc3 = LAN trunk parent to TP-Link switch
VLAN100 on igc3 = OPT2
```

The `igc*` interface names strongly suggest Intel NICs and are consistent with a multi-port Intel-based firewall appliance.

Known pfSense management/access behavior:

```text
VLAN100 pfSense IP: 10.100.0.1
HTTPS to 10.100.0.1:443 from PC 10.100.0.10: works
ICMP ping to 10.100.0.1: does not work / likely blocked
Legacy pfSense/gateway IP: 10.10.0.1 was reachable by web UI previously
```

---

## 3.2 TP-Link Omada SG3428X-M2 Switch

Status: CONFIRMED installed/used.

Known model:

```text
TP-Link Omada SG3428X-M2
```

Known hardware/specs:

```text
24 × 2.5GbE RJ45 ports
4 × 10GbE SFP+ ports
Managed switch
L2+ / L3-capable
802.1Q VLAN support
LACP support
Static routing support
SVI support
ACL support
Omada SDN capable
```

Role:

```text
Central LAN switch
VLAN-aware fan-out to endpoints
Port 1 uplink/trunk to pfSense
Potential/static L3 switching via SVIs
Endpoint access ports for PC and future nodes
```

Known/remembered switch config:

```text
Port 1:
  Role: pfSense uplink/trunk
  Tagged VLANs: VLAN100
  Untagged VLAN: VLAN1
  PVID: 1

Ports 2–28:
  Role: endpoint/access ports
  Untagged VLAN: VLAN100
  PVID: 100

Port 3:
  PC example
  PVID: 100
  Untagged VLAN100
```

Known management addresses:

```text
Legacy/recovery management: 10.10.0.2
Expected/intended VLAN100 SVI/mgmt: 10.100.0.2
```

Observed behavior:

```text
From PC 10.100.0.10:
  Web UI at 10.10.0.2 works.
  Web UI at 10.100.0.2 did not work at last check.
```

Interpretation:

```text
Switch is still reliably manageable via old/legacy 10.10.0.2.
VLAN100 SVI/mgmt address 10.100.0.2 may exist conceptually/intended, but was not reachable via web UI at last observation.
```

---

## 3.3 Main PC

Status: CONFIRMED present and connected.

Role:

```text
User's current workstation / personal PC.
Used to access pfSense and TP-Link web UIs.
Not intended to host infrastructure long-term.
Should not host infrastructure; only local code/governance interfaces.
```

Known network state:

```text
Current IP: 10.100.0.10
Network: 10.100.0.0/24
Connected to TP-Link switch on an access/untagged VLAN100 port.
Gateway expected: 10.100.0.1
```

Known specs from prior context:

```text
CPU/core count: 32 cores
RAM: 64 GB
```

Unknown main PC details:

```text
Exact CPU model: UNKNOWN
GPU: UNKNOWN for main PC
Storage: UNKNOWN
NIC speed/model: UNKNOWN
Switch port number: UNKNOWN
OS: UNKNOWN in homelab context, though user often uses Linux and also has Windows/macOS elsewhere
```

Important preference/constraint:

```text
Main PC should not be considered an infrastructure host.
It should remain personal/workstation/governance/client-side only.
```

---

## 3.4 Additional PC Planned to Plug In

Status: PLANNED / not configured in captured context.

Role:

```text
Additional PC to be plugged alongside the main PC because the project is suspended.
No recommendation requested.
```

Known specs:

```text
UNKNOWN
```

Expected network behavior if placed like main PC:

```text
Switch port should behave like endpoint access port:
  Untagged VLAN100
  PVID 100

Expected network:
  10.100.0.0/24
  Gateway 10.100.0.1
```

Do not assume it is a Kubernetes node unless the user later says so.

---

## 3.5 Separate Linux Utility / Jump / Docker Host

Status: CONFIRMED as part of intended homelab architecture; already purchased per project instructions, but exact current connection status/specs are not fully captured.

Role options previously described:

```text
Linux utility host
Jump host
Docker-only auxiliary services
CI/tools host
Monitoring/admin/support host
Not part of the 3-node Kubernetes cluster
```

Known from fixed project constraint:

```text
There is 1 separate Linux host already purchased.
It runs Docker and auxiliary services.
It is separate from the 3 physical Kubernetes nodes.
```

Known/remembered hardware from prior context:

```text
Machine class: 16-core machine
RAM: 64 GB
GPU: RTX 3080 Ti
Intended role: infra/governance host
```

Potential relationship:

```text
This 16-core / 64 GB / RTX 3080 Ti machine may be the separate infra/governance/Linux/Docker host.
However, treat the mapping as likely but not absolutely confirmed unless the user confirms.
```

Unknown:

```text
Exact CPU model: UNKNOWN
Storage: UNKNOWN
NIC model/speed: UNKNOWN
Switch port: UNKNOWN
Current IP: UNKNOWN
Hostname: UNKNOWN
Whether currently powered/connected: UNKNOWN
```

Important:

```text
This host is not supposed to be one of the 3 Kubernetes physical nodes.
```

---

## 3.6 Kubernetes Nodes

Status: PLANNED / project suspended / partially specified in old context.

Fixed intended architecture:

```text
3 physical Kubernetes nodes
All physical machines; no nested virtualization as the base design
HA-capable control plane + worker role distribution
Separate Linux Docker/utility host outside cluster
```

Prior concrete machine inventory:

```text
3 × Lenovo machines intended as cluster nodes
  - 2 machines with 32 GB RAM
  - 1 machine with 16 GB RAM
```

Additional node-related memory:

```text
Nodes are mini-PC / Tiny class.
Lenovo ThinkCentre M75q Gen5 was referenced.
Example MPN: 12RQ000XGE
Example spec referenced: 16 GB RAM, 512 GB NVMe
```

CONFLICT / evolution:

```text
Older inventory says 3 Lenovo machines:
  - 2 × 32 GB RAM
  - 1 × 16 GB RAM

Later memory says nodes are mini-PC/Tiny class, e.g. Lenovo ThinkCentre M75q Gen5 12RQ000XGE, 16 GB RAM, 512 GB NVMe.

This may mean:
  - one specific model was considered/referenced,
  - one or more nodes have that spec,
  - or the original 3-node inventory was being refined.
Do not assume all 3 nodes are identical unless verified.
```

Known/desired cluster composition:

```text
3 physical nodes total.
Each node can act as control plane + worker for HA-capable learning cluster.
No nested virtualization required/assumed.
```

Unknown per-node inventory:

```text
Hostnames: UNKNOWN
Exact CPU models: UNKNOWN
Exact RAM per final installed node: partially known only
Exact disk sizes per final installed node: partially known/example only
NIC speed/model: UNKNOWN
Switch ports: UNKNOWN
Current IPs: UNKNOWN
Whether installed/powered: UNKNOWN
```

Potential IP plan from earlier proposed design, not confirmed implemented:

```text
Example planned subnet: 10.20.0.0/24
Gateway: 10.20.0.1
platform: 10.20.0.10
k8s-node1: 10.20.0.11
k8s-node2: 10.20.0.12
k8s-node3: 10.20.0.13
personal PC second NIC: 10.20.0.100
MetalLB / VM / storage ranges were suggested
```

Important: the above `10.20.0.0/24` plan is older/proposed and should not override the observed active VLAN100 network.

---

## 3.7 Earlier / Alternate Switch Mention

There is an older context mentioning:

```text
TP-Link TL-SG3210XHP-M2 managed 2.5GbE PoE switch
```

But the later/current confirmed switch is:

```text
TP-Link Omada SG3428X-M2
```

Interpretation:

```text
Treat SG3428X-M2 as current.
Treat TL-SG3210XHP-M2 as older/planning context or superseded reference.
```

---

## 4. Network/VLAN Design — Implemented and Planned

## 4.1 Implemented/Observed VLANs

### VLAN100

Status: CONFIRMED active.

Purpose labels used:

```text
Management / Transit
Transit / Data
Active workstation/client network
```

Subnet:

```text
10.100.0.0/24
```

Known IPs:

```text
10.100.0.1  = pfSense VLAN100 gateway / interface
10.100.0.2  = intended TP-Link switch SVI/management on VLAN100
10.100.0.10 = main PC DHCP reservation/current address
```

Observed:

```text
Main PC 10.100.0.10 can access internet.
Main PC can access pfSense HTTPS at 10.100.0.1:443.
Main PC cannot ping 10.100.0.1, but TCP/443 works.
Main PC cannot access switch web UI at 10.100.0.2 at last check.
```

DHCP desired/remembered:

```text
VLAN100 DHCP example: 10.100.0.50–10.100.0.200
First ~50 addresses reserved/static.
Main PC has DHCP reservation 10.100.0.10.
```

Firewall desired/remembered:

```text
VLAN100 net → any allowed
Outbound NAT via pfSense WAN
```

### VLAN1 / Legacy Recovery

Status: CONFIRMED still present/reachable.

Subnet:

```text
10.10.0.0/29
```

Known IPs:

```text
10.10.0.1 = legacy pfSense/gateway/recovery address
10.10.0.2 = TP-Link switch legacy management address
```

Observed:

```text
Switch web UI at 10.10.0.2 works from PC currently on 10.100.0.10.
10.10.0.1 web UI was reachable previously.
Ping to 10.10.0.1 or 10.100.0.1 may fail while HTTPS works.
```

Switch port relationship:

```text
Port 1 has VLAN1 untagged, PVID 1, to pfSense.
VLAN1 likely retained for recovery/migration safety.
```

---

## 4.2 Planned / Recommended Production VLAN Layout From Earlier Design

This was a proposed/finalized addressing plan in memory, not necessarily implemented.

```text
Transit VLAN 100
VLAN10 Platform/Mgmt: 10.10.10.0/23
VLAN20 K8s:          10.10.20.0/23
VLAN30 Personal:     10.10.30.0/?? or similar
```

Known exact from memory:

```text
VLAN10 Platform/Mgmt = 10.10.10.0/23
VLAN20 K8s = 10.10.20.0/23
VLAN30 Personal = mentioned but full CIDR truncated in memory
```

Important:

```text
Do not treat VLAN10/20/30 as currently implemented unless verified.
The active confirmed network is VLAN100 10.100.0.0/24 plus legacy VLAN1 10.10.0.0/29.
```

---

## 5. Switch Configuration — Last Known

Device:

```text
TP-Link Omada SG3428X-M2
```

Port 1:

```text
Role: uplink/trunk to pfSense igc3
Tagged VLAN100
Untagged VLAN1
PVID 1
```

Ports 2–28:

```text
Role: endpoint/access ports
Untagged VLAN100
PVID 100
```

Port 3:

```text
Known example PC port
PVID 100
Untagged VLAN100
```

Implication for plugging in another PC:

```text
Use an endpoint/access port matching Ports2–28 behavior:
  VLAN100 untagged
  PVID 100
```

Management IP behavior:

```text
Switch UI reliable at 10.10.0.2.
Switch UI not reachable at 10.100.0.2 during last check.
```

Possible explanations for 10.100.0.2 issue:

```text
VLAN100 SVI not configured.
SVI configured but web management not bound to it.
ACL/management access policy blocks it.
No route/default gateway on switch.
ARP/routing/state issue.
Browser/HTTPS binding issue.
```

---

## 6. pfSense Configuration — Last Known

Device role:

```text
Edge firewall
NAT
WAN failover
DHCP for VLAN100 likely
Gateway for VLAN100
```

Interfaces:

```text
WAN  = igc0 = ISP1
OPT1 = igc1 = ISP2
LAN  = igc3 = trunk parent
OPT2 = VLAN100 on igc3
```

VLAN100 interface:

```text
Name: OPT2
Parent: igc3
VLAN ID: 100
IP: 10.100.0.1/24
DHCP: intended/enabled with pool example 10.100.0.50–10.100.0.200
Firewall: allow VLAN100 net → any, at least enough for internet and pfSense HTTPS
NAT: outbound NAT to WAN working
```

WAN design:

```text
ISP1 primary
ISP2 failover
Preferred failover/failback behavior was discussed.
State flush on gateway transition was discussed.
Exact gateway group settings not fully captured.
```

Observed access from main PC:

```text
10.100.0.10 → 10.100.0.1:443 = works
10.100.0.10 → 10.100.0.1 ICMP = fails
10.100.0.10 → internet = works
```

Interpretation:

```text
VLAN100 connectivity and pfSense routing/NAT work.
ICMP failure is likely firewall/rule behavior, not proof of outage.
```

---

## 7. Firewall Rules / Effective Policy

Exact pfSense rule table: UNKNOWN.

Effective/desired rules from context:

```text
VLAN100 net → any = allowed
VLAN100 net → pfSense VLAN100 address TCP/443 = allowed
VLAN100 net → internet = allowed via NAT
ICMP to pfSense gateway = not allowed or not responding
```

Known behavior matrix:

```text
Source: main PC 10.100.0.10

Destination/protocol:
  10.100.0.1 TCP/443  → success
  10.100.0.1 ICMP     → failure
  10.10.0.1 web UI    → reachable previously
  10.10.0.1 ICMP      → failure
  10.10.0.2 web UI    → success
  10.100.0.2 web UI   → failure
  Internet            → success
```

Practical rule:

```text
Do not diagnose this network using ping alone.
Use TCP tests such as Test-NetConnection -Port 443 and browser HTTPS.
```

---

## 8. Routing Model

Current likely routing:

```text
VLAN100 clients default gateway: 10.100.0.1 (pfSense)
Internet-bound: client → switch → pfSense → active WAN
```

Design preference remembered:

```text
Switch-centric design:
  Switch handles VLANs/SVIs/inter-VLAN routing.
  pfSense handles edge NAT, WAN failover, perimeter firewall.
  ISP1 primary, ISP2 failover.
```

But current implementation evidence:

```text
pfSense is the active VLAN100 default gateway at 10.100.0.1.
Switch SVI 10.100.0.2 was expected but not reachable by web UI.
Inter-VLAN routing ownership between switch and pfSense is not fully confirmed.
```

Legacy reachability issue:

```text
PC 10.100.0.10 can access switch 10.10.0.2.
This implies some working path between VLAN100 and legacy 10.10.0.0/29.
Possible owners:
  - pfSense routing
  - TP-Link static routing/SVI
  - native VLAN/trunk recovery behavior
  - temporary or residual routing
```

---

## 9. IP Inventory

```text
10.100.0.1  pfSense VLAN100 gateway/interface, HTTPS works, ping fails
10.100.0.2  intended TP-Link VLAN100 SVI/management, web UI failed at last check
10.100.0.3  infrastructure IP mentioned, exact device unknown
10.100.0.10 main PC, DHCP reservation/current address
10.10.0.1   legacy pfSense/gateway/recovery address, web UI reachable previously
10.10.0.2   TP-Link legacy management address, web UI reachable
```

Older/proposed IP plan:

```text
10.20.0.0/24 planned example subnet
10.20.0.1   gateway
10.20.0.10  platform
10.20.0.11  k8s-node1
10.20.0.12  k8s-node2
10.20.0.13  k8s-node3
10.20.0.100 personal PC second NIC
```

Future/proposed VLAN plan:

```text
VLAN10 Platform/Mgmt: 10.10.10.0/23
VLAN20 K8s:          10.10.20.0/23
VLAN30 Personal:     mentioned, exact CIDR unclear
```

---

## 10. UPS / Power Context

Known user constraint:

```text
UPS should protect cluster only:
  - Kubernetes nodes
  - switch
  - router/firewall

UPS should NOT protect:
  - jump host
  - main PC
```

Previously discussed UPS models:

```text
APC Back-UPS Pro BR900GI: 900 VA / 540 W
APC Back-UPS Pro BR1200GI: 1200 VA / 720 W
CyberPower CP900 EPFCLCD alternative
CyberPower CP1300 EPFCLCD alternative
```

This document does not recommend a purchase; it records prior context only.

---

## 11. Important Operational Observations

Observed Windows command result:

```powershell
Test-NetConnection 10.100.0.1 -Port 443
```

Result:

```text
ComputerName     : 10.100.0.1
RemoteAddress    : 10.100.0.1
RemotePort       : 443
InterfaceAlias   : Ethernet
SourceAddress    : 10.100.0.10
TcpTestSucceeded : True
```

Meaning:

```text
The main PC is on VLAN100.
pfSense is reachable at 10.100.0.1 over HTTPS.
The network is not broken just because ping fails.
```

Browser targets:

```text
pfSense current VLAN100 UI:
https://10.100.0.1

TP-Link currently reachable UI:
https://10.10.0.2

TP-Link intended VLAN100 UI, not working at last check:
https://10.100.0.2
```

---

## 12. Known Unknowns to Verify Later

```text
Exact firewall appliance model and CPU:
  conflict between N305-class 4–6x2.5GbE and N2940 4x1GbE.

Exact pfSense gateway group:
  ISP1/ISP2 priorities, trigger level, failback, state-kill behavior.

Exact DHCP scope:
  likely 10.100.0.50–10.100.0.200 but verify.

Exact pfSense firewall rules:
  effective behavior known, full rule table unknown.

Exact switch SVI config:
  10.100.0.2 intended but web UI not reachable.

Exact switch default route/static routes:
  unknown.

Exact management access control on TP-Link:
  unknown.

Exact main PC switch port:
  unknown, but port behaves as VLAN100 untagged/PVID 100.

Exact additional PC specs and port:
  unknown.

Exact Linux utility/Docker host identity:
  likely 16-core/64GB/RTX3080Ti infra/governance machine, but verify.

Exact Kubernetes node inventory:
  3 Lenovo machines known; 2x32GB and 1x16GB RAM from older context.
  Also Lenovo ThinkCentre M75q Gen5 12RQ000XGE 16GB/512GB NVMe referenced.
  Final per-node CPU/RAM/disk/IP/port not confirmed.
```

---

## 13. Minimal Transfer Summary for Another LLM

The user has a suspended physical homelab. Current topology is dual ISP → pfSense/OPNsense firewall appliance → TP-Link Omada SG3428X-M2 managed switch → PCs/future nodes. pfSense interface mapping from screenshots: WAN=igc0 ISP1, OPT1=igc1 ISP2, LAN=igc3 trunk parent to switch, OPT2=VLAN100 on igc3. VLAN100 is active on 10.100.0.0/24 with pfSense 10.100.0.1, intended switch SVI 10.100.0.2, main PC 10.100.0.10 DHCP reservation. Legacy VLAN1/recovery is 10.10.0.0/29 with 10.10.0.1 and switch management 10.10.0.2. Switch SG3428X-M2 specs: 24x2.5GbE RJ45, 4x10G SFP+, VLAN/LACP/static routing/SVI/ACL/Omada SDN capable. Switch config: port1 to pfSense trunk tagged VLAN100, untagged VLAN1, PVID1; ports2–28 endpoint ports untagged VLAN100 PVID100; port3 was a PC example. From PC 10.100.0.10, internet works; pfSense HTTPS 10.100.0.1:443 works; ping to gateways may fail; switch UI works at 10.10.0.2 but not 10.100.0.2. Hardware inventory: main PC 32 cores/64GB, should not host infra; separate infra/Linux/Docker/jump host likely 16 cores/64GB/RTX3080Ti; 3 Lenovo Kubernetes node machines planned, older inventory 2x32GB RAM + 1x16GB RAM, also ThinkCentre M75q Gen5 12RQ000XGE 16GB/512GB NVMe referenced. Firewall appliance exact spec conflicts: newer memory says fanless N305-class with 4–6x2.5GbE Intel NICs; older says N2940 quad-core 4x1GbE AES-NI. User wants factual recovery, not recommendations.
