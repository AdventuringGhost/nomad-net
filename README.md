# nomad-net

Cisco Packet Tracer simulation of the physical network layer underpinning [Nomad Edge](https://github.com/AdventuringGhost/nomad-edge-factory) — a Kubernetes-based edge compute platform designed for intermittent rural connectivity.

## What This Is

Nomad Edge runs Kubernetes workloads at the edge. This repo is the network that makes that possible: a simulated enterprise-grade LAN/WAN topology modelling the physical layer of a rural industrial site (Lloydminster, SK / Cold Lake, AB region).

The Packet Tracer simulation covers the last-mile problem — how do you maintain reliable, segmented, secure connectivity when your uplink is a cellular modem or VSAT link and your site spans a yard, a shop, and a mobile unit?

## Topology

```
[ISP / WAN Uplink]
        |
  Cisco 2911 Router
  - NAT/PAT (inside: 10.0.0.0/8, outside: public)
  - ACLs: permit-list inbound, deny-all default
  - Static routes + default route to ISP
        |
  [Layer 3 Core Switch]
  - Inter-VLAN routing
  - SVI per segment
        |
  .-------------------.
  |          |        |
VLAN 10   VLAN 20  VLAN 30
 Mgmt    Edge Compute  Wireless
  |           |           |
 OOB       Nomad Edge    WLC
 hosts      nodes      .--+--.
                      AP-1  AP-2
                    (LWAP mesh coverage)
```

### VLANs

| VLAN | Name | Purpose |
|------|------|---------|
| 10 | MGMT | Out-of-band management — switches, router, WLC |
| 20 | EDGE | Edge compute nodes running Nomad Edge workloads |
| 30 | WIRELESS | Wireless clients, IoT endpoints, mobile units |

### Devices

- **Cisco 2911 Router** — WAN termination, NAT/PAT, ACL enforcement, DHCP relay
- **Layer 3 Core Switch** — Inter-VLAN routing, SVI interfaces, trunk/access port segmentation
- **Layer 2 Access Switch** — Edge node connectivity, VLAN tagging
- **Cisco WLC** — Lightweight AP controller; manages two LWAP access points for mesh wireless coverage across the site
- **Lightweight APs (x2)** — Centrally managed via WLC; SSIDs mapped to VLAN 30

## Design Decisions

**Why NAT/PAT at the router, not the switch?**
Keeps the WAN boundary clean. The 2911 is the single choke point for all ingress/egress. ACLs at the WAN interface drop everything not explicitly permitted before it enters the LAN.

**Why a WLC instead of autonomous APs?**
Nomad Edge nodes roam. A WLC with LWAP ensures wireless clients stay on the right VLAN as they move between coverage zones — no manual SSID reconfiguration per AP.

**Why simulate in Packet Tracer instead of GNS3/EVE-NG?**
Packet Tracer is sufficient for validating VLAN segmentation, routing, and ACL logic without requiring IOS image licensing. The goal is topology validation and documentation, not full protocol emulation.

**ACL strategy**
Inbound on WAN interface: deny all by default, permit specific management ports. VLAN-to-VLAN: EDGE nodes cannot initiate connections to MGMT (prevents lateral movement if a workload is compromised). WIRELESS cannot reach EDGE directly — must route through the router ACL.

## Relationship to Nomad Edge

This network is the substrate. Nomad Edge (`nomad-edge-factory`) provisions the K3s/Kubernetes layer that runs on the VLAN 20 nodes. The connectivity guarantees — segmentation, WAN failover tolerance, wireless client isolation — are what make the edge compute layer viable in a rural industrial context.

When Nomad Edge detects a WAN partition, it degrades gracefully to local-only mode. The network topology here ensures that local cluster-internal traffic (VLAN 20) never has to traverse the WAN interface, so partition events do not cascade into compute failures.

## Files

```
nomad-net.pkt    # Packet Tracer simulation (open in Cisco Packet Tracer 8.x+)
```

## Related

- [nomad-edge-factory](https://github.com/AdventuringGhost/nomad-edge-factory) — the edge compute layer this network supports
- [Portfolio case study](https://adventuringghost.com/projects/nomad-net/) — architecture walkthrough and design rationale