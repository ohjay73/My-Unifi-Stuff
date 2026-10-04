# Home Network Topology

Live network for the Edmonton house + new 24x24 garage, current as of the 2026-10-03
cutover (Vantiva bridged, UCG-Fiber is the gateway). Items marked **TBD** are the pending decisions.

Legend: `──` wired Ethernet · `══` fiber · `★` PoE powered · `(?)` decision pending

```
Telus fibre (XGS-PON, 10000/10000 Mbps)
  │
Vantiva gateway — bridge mode since 2026-10-03
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
| Bench (retired 2026-10-03) | 192.168.10.0/24 | UCG-Fiber double-NATed behind Vantiva during testing |

## Decisions pending

Tracked in the [landing README](README.md#upcoming-decisions): wireless AP choice,
front-door camera pick, and switch selection for the garage + electrical room.

## Live client inventory (UniFi topology, 2026-10-03)

Screenshot transcription of every client the UCG-Fiber currently sees. All appear as
direct children of the gateway because the unmanaged switches between them are invisible
to UniFi — the real path is gateway → unmanaged switch → client. List is partial
(~40 of ~45 visible; a few entries sit below the fold).

### Eeros (6 of 7 visible)

| Client | Link |
|---|---|
| eero 99:54 | 2.5 GbE |
| eero f6:d2 | 2.5 GbE |
| eero 59:6d | 2.5 GbE |
| eero 27:94 | 2.5 GbE |
| eero d6:ed | 2.5 GbE |
| eero 87:72 | 2.5 GbE |

### NAS

| Client | Link | Notes |
|---|---|---|
| 90:09:d0:a4:d… | 10 GbE | Synology DS923+ (90:09:d0 = Synology OUI) — 10G link is up |
| MAIN-NAS 4e:… | 2.5 GbE | QNAP TS-653D, NIC 1 of 2 (SMB multichannel pair) |
| MAIN-NAS 4e:… | 2.5 GbE | QNAP TS-653D, NIC 2 of 2 |

### Cameras

| Client | Link | Notes |
|---|---|---|
| VIVOTEK Netw… | 2.5 GbE | VIVOTEK alley camera |
| WYZE_CAKP2… | 2.5 GbE | Wyze Cam Pan v2 |
| IC Realtime ICI… | FE (100M) | IC Realtime camera — unidentified which one |

### Media / living room

| Client | Link |
|---|---|
| NVIDIA Shield … | 2.5 GbE |
| Living-Room 9… | 2.5 GbE |
| Yamaha RX-A1… | 2.5 GbE |
| SDMC Androi… | 2.5 GbE |
| P210M e7:ca | 2.5 GbE — unidentified |
| BoostLite-E0C… | 2.5 GbE — unidentified |

### Computers / phones / tablets

| Client | Link |
|---|---|
| MacBookPro a… | 2.5 GbE |
| iPad fb:74 | 2.5 GbE |
| Ipadminonor2… | 2.5 GbE |
| candace-s-S2… | 2.5 GbE |
| Jason-s-S26-… | 2.5 GbE |
| HP EliteDesk 8… | 2.5 GbE |
| 5CG011DK5K f… | 2.5 GbE (HP serial — second HP box) |
| DESKTOP-6EI… | 2.5 GbE |

### Smart home / IoT / energy

| Client | Link |
|---|---|
| homeassistant… | 2.5 GbE |
| Hubitat Elevati… | 2.5 GbE |
| Sense-N22600… | 2.5 GbE (Sense energy monitor) |
| airthings-view … | 2.5 GbE (Airthings air quality) |
| AmazonAQM-… | 2.5 GbE (Amazon air quality monitor) |
| HS300 09:b2 | 2.5 GbE (Kasa power strip) |
| HS300 c6:ee | 2.5 GbE (Kasa power strip) |
| HS300 cf:46 | 2.5 GbE (Kasa power strip) |
| Rivian f6:10 | 2.5 GbE (Rivian vehicle Wi-Fi) |

### Unidentified

| Client | Link |
|---|---|
| Router_Switch… | 2.5 GbE |
| ADC-220120 0… | 2.5 GbE |
| C42996C99DC… | 2.5 GbE (MAC-style name) |
| 00:62:6e:94:4… | 2.5 GbE (MAC only) |
| AT&T Arris BG… | 2.5 GbE |

Not visible in these shots (possibly below the fold): the two Telus STBs, Apple TV, PS5,
printer, the 7th eero, SolarEdge inverter, Hue Bridge, Forest monitor, and the house
EmpireTech camera (.130).
