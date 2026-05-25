# nomad-net

Edge-to-cloud network topology for off-grid stewardship operations — simulated in Cisco Packet Tracer.

This is the physical and logical network layer that underpins the [Nomad Edge](https://adventuringghost.com) architecture: a full-stack simulation from AWS cloud connection down to renewable power distribution and IoT sensor endpoints.

---

## Topology Overview

```
AWS Cloud (Cloud-PT)
    |
    +-- ISR4321 -- TheBrain/Gateway
            |
            +-- 2960-24TT -- TheNervousSystem (core distribution switch)
            |       +-- Server-PT -- Server
            |       +-- MCU-PT -- Liver (microcontroller / edge logic)
            |       +-- SBC-PT -- Heart (single-board compute)
            |       +-- PowerDistributor1
            |       |       +-- Solar Panels (IoT3, IoT6, IoT20, Solar1)
            |       |       +-- Power Meter (IoT10)
            |       +-- MCU-PT -- PowerDistributor2
            |               +-- Wind Turbines (Turbine1, Turbine2, Turbine3, Turbine4)
            |               +-- Batteries (Battery1, Battery2, Battery3)
            |
            +-- 2960-24TT -- HerdAccessSW
                    +-- Laptop-PT -- Steward Laptop
                    +-- SBC-PT -- DataCollector
                    +-- Thing Tag1
                    +-- Thing Tag2
```

---

## Segments

**Cloud / WAN**
ISR4321 gateway (`TheBrain`) maintains the uplink to AWS via a simulated Cloud-PT connection. This is the egress point for telemetry sync, remote management, and cloud-side processing when connectivity is available. The design assumes intermittent WAN — edge nodes operate autonomously when the link is down.

**Core Distribution — TheNervousSystem**
2960-24TT switch acting as the central fabric. All edge compute, power management controllers, and IoT endpoints terminate here. VLAN segmentation separates operational traffic (power/sensor data) from steward access.

**Edge Compute**
- `Heart` (SBC-PT) — primary edge compute node, runs local services and agent processes
- `Liver` (MCU-PT) — microcontroller handling local orchestration and sensor aggregation
- `Server` — local server for on-site data storage and services

**Power Management**
Dual power distribution controllers manage the renewable energy fabric:
- `PowerDistributor1` — solar array (4 panels, 119W each) + power metering
- `PowerDistributor2` — wind turbine array (4 turbines) + battery bank (3 units)

This isn't decorative — the power topology reflects a real off-grid constraint: compute and networking must survive grid independence.

**Herd / Field Access — HerdAccessSW**
Isolated access switch for field-facing devices: steward laptop, data collector SBC, and IoT asset tags (Thing Tag1/2). Kept separate from the core fabric to limit blast radius if a field device is compromised.

---

## Design Principles

- **Offline-first** — the topology is designed to function without cloud connectivity. Local compute, local storage, local power.
- **Segmentation by function** — power management, core compute, and field access are on separate switches, not a flat network.
- **ISR4321 as the trust boundary** — all cloud-bound traffic routes through a single controlled gateway, enabling ACL enforcement and traffic inspection at the edge.
- **Renewable-aware infrastructure** — power distribution is modeled as a first-class network concern, not an afterthought.

---

## Files

| File | Description |
|---|---|
| `Nomad-Net_caseStudy2.pkt` | Full Packet Tracer simulation — open with Cisco Packet Tracer 8.x+ |

---

## Context

nomad-net is the physical layer of a larger architecture. The cloud and application layers are covered by the broader [Nomad Edge](https://adventuringghost.com) project, which includes AWS infrastructure (IoT Core, S3, Lambda), Terraform IaC, and the edge agent stack.

---

Built by [Skipper](https://adventuringghost.com) · Cold Lake, AB