# Homelab Current Configuration Recovery Snapshot

Generated: 2026-05-10

This is a human-readable recovery/reference document for the current suspended homelab project.

It is written to preserve as much known information as possible, including machine specs, network anatomy, IPs, switch configuration, pfSense configuration, observed behavior, and unresolved uncertainties.

> Important: this is not a redesign and not a shopping recommendation.  
> It is a reconstruction of the latest known state from previous project context.

---

## 1. Executive Summary

Your homelab currently appears to be built around this physical chain:

```text
ISP1 + ISP2
   ↓
pfSense / OPNsense-capable firewall appliance
   ↓
TP-Link Omada SG3428X-M2 managed switch
   ↓
Main PC + future/paused lab machines
```

The active working client network is:

```text
VLAN100
Subnet: 10.100.0.0/24
pfSense gateway: 10.100.0.1
Main PC: 10.100.0.10
```

The switch is still practically reachable through its older/legacy management address:

```text
TP-Link switch UI: https://10.10.0.2
```

The expected newer VLAN100 switch address was not reachable at the last check:

```text
Expected TP-Link VLAN100 address: https://10.100.0.2
Status: not working at last observation
```

Most important operational fact:

```text
Ping is not reliable as a health check in the current setup.
HTTPS/TCP works even where ICMP ping fails.
```

---

## 2. Current Physical Network Diagram

```text
                      ┌────────────────────┐
                      │       ISP 1         │
                      └─────────┬──────────┘
                                │
                                │ WAN / igc0
                                │
┌────────────────────┐    ┌─────▼────────────────────────┐
│       ISP 2         │    │ pfSense / OPNsense appliance │
└─────────┬──────────┘    │                              │
          │               │ WAN  = igc0 = ISP1           │
          │ OPT1 / igc1   │ OPT1 = igc1 = ISP2           │
          └──────────────►│ LAN  = igc3 = trunk parent   │
                          │ OPT2 = VLAN100 on igc3       │
                          └─────┬────────────────────────┘
                                │
                                │ igc3 trunk / switch uplink
                                │
                          ┌─────▼────────────────────────┐
                          │ TP-Link Omada SG3428X-M2     │
                          │ Port 1 = pfSense uplink      │
                          │ Ports 2–28 = endpoints       │
                          └─────┬────────────────────────┘
                                │
        ┌───────────────────────┼────────────────────────┐
        │                       │                        │
┌───────▼────────┐     ┌────────▼─────────┐     ┌────────▼─────────┐
│ Main PC         │     │ Additional PC     │     │ Future lab nodes  │
│ 10.100.0.10     │     │ planned           │     │ K8s / Docker host │
│ VLAN100         │     │ likely VLAN100    │     │ suspended/project │
└────────────────┘     └──────────────────┘     └──────────────────┘
```

---

## 3. Machine and Hardware Inventory

## 3.1 Firewall Appliance

### Role

The firewall appliance is the network edge.

It provides:

- WAN connectivity.
- ISP1 / ISP2 multi-WAN.
- NAT.
- pfSense firewall policy.
- VLAN100 gateway.
- Likely DHCP for VLAN100.
- Web UI management.

### Known interface mapping

```text
WAN  → igc0 → ISP1
OPT1 → igc1 → ISP2
LAN  → igc3 → trunk parent toward switch
OPT2 → VLAN100 on igc3
```

Important correction from the previous troubleshooting:

```text
OPT1 is ISP2.
VLAN100 is OPT2.
```

### Known IPs

```text
10.100.0.1 = pfSense VLAN100 gateway
10.10.0.1  = legacy/recovery pfSense/gateway address from old network
```

### Known access behavior

```text
https://10.100.0.1 works from the main PC.
Ping to 10.100.0.1 may fail.
This does not mean pfSense is unreachable.
```

### Hardware specs — conflict preserved

There are two remembered versions of the firewall hardware.

Version A, newer remembered context:

```text
Fanless N305-class firewall appliance
4–6 × 2.5GbE Intel NICs
pfSense / OPNsense capable
```

Version B, older project context:

```text
Intel N2940 quad-core appliance
4 × RJ45 1GbE
AES-NI capable
pfSense / OPNsense capable
```

Recovery note:

```text
The installed firewall is confirmed.
The exact CPU/NIC generation should be verified in pfSense or from the device order history.
Do not silently assume N2940 or N305 without checking.
```

---

## 3.2 TP-Link Switch

### Confirmed model

```text
TP-Link Omada SG3428X-M2
```

### Known specs

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

### Role

The switch is the central LAN aggregation point.

It currently handles:

- VLAN-aware switching.
- pfSense uplink.
- Endpoint ports.
- Legacy management access.
- Possibly L3/SVI functions, though the exact routing state must be verified.

### Known switch port configuration

```text
Port 1:
  Connected to pfSense LAN / igc3
  Trunk/uplink role
  Tagged VLAN100
  Untagged VLAN1
  PVID 1

Ports 2–28:
  Endpoint/access ports
  Untagged VLAN100
  PVID 100

Port 3:
  Example PC port
  Untagged VLAN100
  PVID 100
```

### Known switch management behavior

```text
https://10.10.0.2 works.
https://10.100.0.2 did not work at last check.
```

Interpretation:

```text
The switch still has a reliable management path through the legacy network.
The intended VLAN100 management/SVI address is not confirmed working.
```

---

## 3.3 Main PC

### Role

The main PC is your current workstation.

It is used to:

- Access pfSense web UI.
- Access TP-Link switch web UI.
- Administer the lab.
- Run local/governance/client-side work.

Important preference:

```text
The main PC should not host homelab infrastructure.
It is not intended to be a Kubernetes node or always-on infra host.
```

### Known network state

```text
IP address: 10.100.0.10
Network: 10.100.0.0/24
Likely gateway: 10.100.0.1
Switch port behavior: untagged/access VLAN100
```

### Known specs

```text
CPU/core count: 32 cores
RAM: 64 GB
```

### Unknown main PC specs

```text
Exact CPU model: unknown
GPU: unknown
Storage: unknown
NIC model/speed: unknown
Exact switch port: unknown
```

---

## 3.4 Additional PC You Plan to Plug In

### Status

This PC is planned to be plugged in alongside the main PC.

No final configuration is captured.

### Known specs

```text
Unknown
```

### Expected network attachment, based on the current switch design

```text
Use an endpoint/access switch port:
  Untagged VLAN100
  PVID 100

Expected network:
  10.100.0.0/24

Expected gateway:
  10.100.0.1
```

Do not assume this PC is a Kubernetes node unless you explicitly decide that later.

---

## 3.5 Separate Linux / Docker / Utility Host

### Role

The wider homelab design includes a separate Linux host outside the Kubernetes cluster.

It is meant for:

- Docker-only auxiliary services.
- Jump/admin functions.
- Tooling.
- CI/support services.
- Monitoring/admin utilities.
- Governance or infrastructure-related workloads.

### Known / likely specs from previous context

```text
CPU/core count: 16 cores
RAM: 64 GB
GPU: RTX 3080 Ti
Role: infra/governance host
```

Important qualification:

```text
This 16-core / 64 GB / RTX 3080 Ti machine is likely the separate infra/Linux/Docker utility host,
but the mapping should be verified if exact inventory matters.
```

### Unknowns

```text
Exact CPU model: unknown
Storage: unknown
NIC speed/model: unknown
Current IP: unknown
Hostname: unknown
Switch port: unknown
Whether currently powered/connected: unknown
```

---

## 3.6 Kubernetes Node Machines

### Intended cluster shape

The homelab Kubernetes design is:

```text
3 physical Kubernetes nodes
1 separate Linux/Docker utility host
Main PC remains outside infrastructure
```

The Kubernetes nodes are intended to be physical machines, not nested virtualization.

### Known older machine inventory

```text
3 × Lenovo machines intended as Kubernetes nodes

Node RAM split:
  2 × machines with 32 GB RAM
  1 × machine with 16 GB RAM
```

### Later referenced machine class/model

```text
Mini-PC / Tiny class
Example referenced: Lenovo ThinkCentre M75q Gen5
Example MPN: 12RQ000XGE
Example spec: 16 GB RAM, 512 GB NVMe
```

### Important uncertainty

There is not enough captured context to say all three Kubernetes nodes are identical.

Most accurate statement:

```text
The cluster was planned around 3 Lenovo mini/Tiny-class machines.
Older inventory says 2 nodes have 32 GB RAM and 1 node has 16 GB RAM.
A Lenovo ThinkCentre M75q Gen5 12RQ000XGE with 16 GB RAM and 512 GB NVMe was referenced.
Final per-node CPU/RAM/disk/IP/port mapping is not confirmed.
```

### Unknown per-node values

```text
Hostnames: unknown
Exact CPU models: unknown
Final RAM per node: partially known only
Disk size per node: partially known/example only
NIC speed/model: unknown
Switch ports: unknown
Current IPs: unknown
Installed OS state: unknown
Kubernetes install state: unknown
```

---

## 4. VLANs and IP Addressing

## 4.1 Active / Implemented Network: VLAN100

```text
VLAN ID: 100
Purpose: Management / Transit / active data network
Subnet: 10.100.0.0/24
```

Known IPs:

| IP | Device / Role | Status |
|---|---|---|
| `10.100.0.1` | pfSense VLAN100 gateway | Working over HTTPS |
| `10.100.0.2` | Intended TP-Link VLAN100 SVI/management | Not reachable at last check |
| `10.100.0.3` | Mentioned infrastructure IP | Exact device unknown |
| `10.100.0.10` | Main PC | Confirmed active |

Likely DHCP plan:

```text
DHCP pool example: 10.100.0.50–10.100.0.200
First ~50 addresses reserved/static
Main PC reservation: 10.100.0.10
```

### Observed VLAN100 behavior

```text
Main PC 10.100.0.10 reaches internet.
Main PC reaches pfSense UI at 10.100.0.1.
Main PC does not necessarily ping 10.100.0.1.
Main PC could not reach switch UI at 10.100.0.2 at last check.
```

---

## 4.2 Legacy / Recovery Network: VLAN1

```text
VLAN ID: 1
Purpose: legacy / recovery / old management
Subnet: 10.10.0.0/29
```

Known IPs:

| IP | Device / Role | Status |
|---|---|---|
| `10.10.0.1` | Legacy pfSense/gateway address | Web UI reachable previously |
| `10.10.0.2` | TP-Link legacy management address | Web UI reachable |

Observed:

```text
From PC 10.100.0.10, the switch UI is reachable at 10.10.0.2.
```

This means the legacy management path is still alive.

---

## 4.3 Older / Proposed Lab Addressing

This was discussed earlier and should not be confused with the current active VLAN100 state.

```text
Example subnet: 10.20.0.0/24
Gateway:        10.20.0.1
Platform:       10.20.0.10
k8s-node1:      10.20.0.11
k8s-node2:      10.20.0.12
k8s-node3:      10.20.0.13
Personal PC 2nd NIC: 10.20.0.100
```

Also previously discussed as a more structured VLAN plan:

```text
Transit VLAN100
VLAN10 Platform/Mgmt: 10.10.10.0/23
VLAN20 K8s:          10.10.20.0/23
VLAN30 Personal:     mentioned, exact CIDR not fully preserved
```

Recovery note:

```text
Do not treat the 10.20.0.0/24 plan or VLAN10/20/30 plan as implemented unless verified.
The implemented/observed network is VLAN100 10.100.0.0/24 plus legacy VLAN1 10.10.0.0/29.
```

---

## 5. pfSense Configuration Snapshot

## 5.1 Interface mapping

| pfSense Interface | Physical / VLAN Interface | Role |
|---|---|---|
| WAN | `igc0` | ISP1 |
| OPT1 | `igc1` | ISP2 |
| LAN | `igc3` | Trunk parent to TP-Link switch |
| OPT2 | VLAN100 on `igc3` | VLAN100 gateway |

## 5.2 VLAN100 interface

```text
Interface: OPT2
Parent: igc3
VLAN ID: 100
IP: 10.100.0.1/24
Role: gateway for VLAN100
```

## 5.3 DHCP

Likely / desired:

```text
VLAN100 DHCP enabled
Example pool: 10.100.0.50–10.100.0.200
Main PC reserved at 10.100.0.10
```

Exact DHCP scope must be verified.

## 5.4 Firewall behavior

Effective behavior:

```text
VLAN100 → internet works.
VLAN100 → pfSense HTTPS works.
VLAN100 → pfSense ICMP/ping fails or is blocked.
```

Likely rule:

```text
Allow VLAN100 net → any
```

Exact pfSense firewall rule table was not captured.

## 5.5 WAN failover

Known intent:

```text
ISP1 primary
ISP2 failover
```

Previously discussed:

```text
Failover/failback behavior
State flushing on gateway transition
Gateway group behavior
```

Exact final gateway group settings are not preserved.

---

## 6. TP-Link Switch Configuration Snapshot

## 6.1 Switch model/specs

```text
TP-Link Omada SG3428X-M2
24 × 2.5GbE RJ45
4 × 10GbE SFP+
Managed / L2+ / L3-capable
802.1Q VLANs
LACP
Static routing
SVIs
ACLs
Omada SDN
```

## 6.2 Port config

| Port(s) | Role | VLAN behavior |
|---|---|---|
| Port 1 | pfSense uplink | Tagged VLAN100, untagged VLAN1, PVID 1 |
| Ports 2–28 | Endpoint ports | Untagged VLAN100, PVID 100 |
| Port 3 | PC example | Untagged VLAN100, PVID 100 |

## 6.3 Management addresses

| Address | Meaning | Status |
|---|---|---|
| `10.10.0.2` | Legacy switch management | Works |
| `10.100.0.2` | Intended VLAN100 SVI/management | Did not work at last check |

Possible reasons `10.100.0.2` did not work:

```text
VLAN100 SVI not actually configured.
Web management not bound to VLAN100 SVI.
ACL or management access policy blocks it.
Switch default route/static route missing.
ARP/state issue.
HTTPS service not listening on that interface.
```

---

## 7. Access and Troubleshooting Facts

## 7.1 Known successful test

From the main PC:

```powershell
Test-NetConnection 10.100.0.1 -Port 443
```

Observed result:

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
The PC is on VLAN100.
pfSense is reachable at 10.100.0.1 over HTTPS.
TCP access works even if ping fails.
```

## 7.2 Browser targets

```text
pfSense:
https://10.100.0.1

TP-Link switch currently working:
https://10.10.0.2

TP-Link switch expected but not working:
https://10.100.0.2
```

## 7.3 Connectivity matrix

| Source | Destination | Protocol | Result |
|---|---|---|---|
| Main PC `10.100.0.10` | Internet | TCP/UDP | Works |
| Main PC `10.100.0.10` | pfSense `10.100.0.1` | HTTPS / 443 | Works |
| Main PC `10.100.0.10` | pfSense `10.100.0.1` | ICMP | Fails / blocked |
| Main PC `10.100.0.10` | legacy gateway `10.10.0.1` | Web UI | Worked previously |
| Main PC `10.100.0.10` | legacy gateway `10.10.0.1` | ICMP | Fails / blocked |
| Main PC `10.100.0.10` | switch `10.10.0.2` | Web UI | Works |
| Main PC `10.100.0.10` | switch `10.100.0.2` | Web UI | Did not work |

---

## 8. Routing Model

## 8.1 What is confirmed

```text
VLAN100 clients use pfSense 10.100.0.1 as gateway.
Internet access from main PC works.
pfSense NAT and WAN path work for VLAN100.
```

## 8.2 Design preference from previous work

The remembered target design is switch-centric internally:

```text
TP-Link switch:
  VLANs
  SVIs
  possible inter-VLAN routing

pfSense:
  NAT
  WAN failover
  edge/perimeter firewall
```

But the current observed configuration has pfSense as the active VLAN100 gateway.

## 8.3 Legacy path issue

The main PC on `10.100.0.10` can reach the switch at `10.10.0.2`.

Possible explanations:

```text
pfSense routes between VLAN100 and legacy network.
TP-Link switch routes between VLAN100 and VLAN1.
Port/native VLAN recovery behavior allows it.
Temporary/residual route exists.
```

This should be treated as working but not fully explained until verified on the devices.

---

## 9. UPS / Power Context

Known requirement from previous context:

```text
UPS should protect:
  - Kubernetes nodes
  - switch
  - router/firewall

UPS should not protect:
  - main PC
  - jump host / utility host
```

Previously discussed UPS options:

```text
APC Back-UPS Pro BR900GI  = 900 VA / 540 W
APC Back-UPS Pro BR1200GI = 1200 VA / 720 W
CyberPower CP900 EPFCLCD
CyberPower CP1300 EPFCLCD
```

No current UPS installation state is confirmed in this recovery snapshot.

---

## 10. What To Preserve When Moving This Context To Another Project

Use this as the condensed project memory:

```text
Suspended physical homelab. Current topology: dual ISP → pfSense/OPNsense firewall appliance → TP-Link Omada SG3428X-M2 → main PC/future lab hosts. pfSense interface mapping: WAN=igc0 ISP1, OPT1=igc1 ISP2, LAN=igc3 trunk parent to switch, OPT2=VLAN100 on igc3. Active VLAN100 is 10.100.0.0/24 with pfSense 10.100.0.1 and main PC 10.100.0.10. TP-Link intended VLAN100 SVI/mgmt 10.100.0.2 did not work; switch is currently reachable via legacy 10.10.0.2. Legacy VLAN1/recovery subnet is 10.10.0.0/29 with 10.10.0.1 and 10.10.0.2. Switch SG3428X-M2 has 24x2.5GbE RJ45 and 4x10G SFP+, supports VLAN/LACP/static routing/SVI/ACL/Omada. Switch port1 to pfSense: tagged VLAN100, untagged VLAN1, PVID1. Ports2–28 endpoint ports: untagged VLAN100, PVID100. Port3 was a PC example. Main PC: 32 cores, 64GB RAM, 10.100.0.10, should not host infra. Separate infra/Linux/Docker host likely 16 cores, 64GB RAM, RTX3080Ti. Kubernetes nodes: 3 Lenovo mini/Tiny-class machines planned; older inventory 2x32GB RAM + 1x16GB RAM; ThinkCentre M75q Gen5 12RQ000XGE 16GB/512GB NVMe referenced. Firewall hardware exact model conflicts: N305-class 4–6x2.5GbE Intel NICs vs older N2940 4x1GbE AES-NI. Current behavior: internet works from main PC; pfSense HTTPS works; ping to gateways may fail; use TCP tests instead of ping.
```

---

## 11. Highest-Priority Unknowns To Verify Later

These are not questions you need to answer now; they are the recovery checklist for when the lab is powered and inspected.

### Firewall

```text
Exact firewall CPU/model: N305-class or N2940?
Exact NIC count/speed.
Exact gateway group settings for ISP1/ISP2.
Exact VLAN100 DHCP pool.
Exact firewall rule table.
```

### Switch

```text
Confirm SG3428X-M2 running config.
Confirm port1 VLAN membership.
Confirm ports2–28 are all untagged VLAN100/PVID100.
Confirm whether 10.100.0.2 SVI exists.
Confirm why 10.100.0.2 web UI does not work.
Confirm switch default route/static routes.
Confirm whether Omada controller or standalone mode is used.
```

### Machines

```text
Exact main PC CPU model/GPU/storage/NIC.
Exact additional PC specs.
Exact Linux/Docker utility host identity and IP.
Exact Kubernetes node models, CPU, RAM, disk, NICs.
Exact switch port per host.
Hostnames and MAC addresses.
```

---

## 12. Final State Statement

The last-known working state is:

```text
The lab has a working VLAN100 client network behind pfSense.
The main PC is active as 10.100.0.10.
pfSense is active as VLAN100 gateway 10.100.0.1.
Internet access works.
pfSense HTTPS works.
TP-Link switch management still works through legacy 10.10.0.2.
The expected TP-Link VLAN100 address 10.100.0.2 was not reachable at last check.
The Kubernetes cluster and auxiliary hosts are planned/suspended and not fully inventoried in the captured context.
```
