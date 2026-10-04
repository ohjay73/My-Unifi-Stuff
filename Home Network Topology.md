# Home Network Topology

Target design for the Edmonton house + new 24x24 garage. This is the network as
planned — items marked **TBD** are the pending decisions.

Legend: `──` wired Ethernet · `══` fiber · `★` PoE powered · `(?)` decision pending

```
Telus fibre (XGS-PON, 10000/10000 Mbps)
  │
Vantiva gateway — bridge mode (192.168.1.254 pre-cutover)
  │ 10G
  │
Ubiquiti UCG-Fiber (LAN 192.168.1.0/24, console 192.168.1.1)
  │
  ├── SFP+ LAN ── UACC-CM-RJ45-MG ── Cat6a ── Synology DS923+ (10G, .211)
  │
  ├── SFP+ (remapped WAN→LAN) ══ BiDi pair ══ OS2 trunk ══► GARAGE
  │     armoured 4-fibre LC-LC, 75–100 ft, 1-1/4" PVC @ 18–24" deep,
  │     exits foundation above grade (core-drilled, sealed), VLAN 30 tagged
  │       │
  │     Garage switch (?) — leading candidate: USW-Flex-2.5G-8 non-PoE (USB-C powered)
  │       ├── NVR (NVR8CH-8P-2AI-S2)
  │       │     └── NVR PoE ── 3× IPC-T54PRO-ZE (garage exteriors, VLAN 30) ★
  │       ├── eero Max 7 (wired once fibre lands)
  │       └── Onkyo TX-NR818 (AVR)
  │
  ├── 2.5G PoE+ (30W, VLAN 30 access) ──★ IPC-T54PRO-ZE, house rear soffit (.130)
  │
  ├── 2.5G ── YuanLey YS25-0402P (4-port 2.5G PoE, unmanaged)
  │       └──★ eero Outdoor 7 (backyard)
  │
  ├── 2.5G ── YuanLey YS25-2400 (24-port 2.5G, unmanaged) ──► BASEMENT (?)
  │       │     to be replaced by managed distribution switch — DECISION PENDING
  │       ├── Office leg ── TL-SG116E ── SODOLA SL-SGT108 ── desk gear
  │       │     └── 2× Telus STBs (Optik + Mediaroom, VLAN 20 optional)
  │       ├── Living-room leg ── Vimin VM-S250801 ── 8 devices (AVR, PS5, Shield, Apple TV…)
  │       ├── 2nd office leg (empty)
  │       └── Candace's office leg
  │
  └── 2.5G ── (spare)
```

## Devices

| Device | Where | Address / link |
|---|---|---|
| Synology DS923+ (E10G22-T1-Mini 10GBASE-T) | Electrical room, direct to UCG-Fiber SFP+ | 10G, hosts SYN_Movies |
| QNAP TS-653D (Plex server) | Basement, 2× 2.5G SMB multichannel | .90 (serves Movies, Kids, TV Shows + SYN_Movies) |
| QNAP TS-464 + 4th NAS | Basement distribution | single-link |
| NVR NVR8CH-8P-2AI-S2 | Garage | cameras on NVR PoE side, LAN on main LAN |
| 4× IPC-T54PRO-ZE | 3 garage exteriors + 1 house rear | VLAN 30 |
| Front-door camera (?) | Dedicated run, front door → electrical-room PoE port | VLAN 30, camera-only, no doorbell — DECISION PENDING |
| Garage interior camera (?) | NW corner planned | was IPC-T54IR-AS, now 2.8mm varifocal recommended — DECISION PENDING |
| Wireless APs (?) | 7× eero currently, bridge mode behind Vantiva | AP decision — DECISION PENDING |
| 2× Telus STBs (Optik + Mediaroom) | Office, behind UCG-Fiber | main LAN; VLAN 20 isolation optional |

## Networks

| Network | Subnet | Notes |
|---|---|---|
| Main LAN | 192.168.1.0/24 | everything by default; UCG-Fiber at .1 |
| VLAN 20 (IPTV, optional) | — | Telus STBs only if multicast hygiene requires it |
| VLAN 30 (cameras) | — | all EmpireTech cameras + NVR camera side |
| Bench (pre-cutover only) | 192.168.10.0/24 | UCG-Fiber double-NATed behind Vantiva during testing |

## Decisions pending

Tracked in the [landing README](README.md#upcoming-decisions): wireless AP choice,
front-door camera pick, and switch selection for the garage + electrical room.
