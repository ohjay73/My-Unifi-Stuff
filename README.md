# Homelab Walkthroughs

A landing pad for my step-by-step homelab how-tos. Each page is a self-contained walkthrough
for one part of the home setup — UniFi gear, NAS boxes, and whatever else needs documenting.
Start here, pick the page you need, and follow it in order.

## UniFi

| Page | What it covers |
|---|---|
| [UCG-Fiber Bench Setup Walkthrough — Ubiquiti Cloud Gateway Fiber](unifi/UCG-Fiber%20Bench%20Setup%20Walkthrough%20—%20Ubiquiti%20Cloud%20Gateway%20Fiber.md) | Bench bring-up of the Ubiquiti UCG-Fiber behind the Vantiva: cabling, the UniFi app and browser setup paths, the temp 192.168.10.0/24 LAN, and the subnet plan for final cutover |
| [TELUS Mediaroom IPTV on UniFi Cloud Gateway (UCG-Fiber)](unifi/TELUS%20Mediaroom%20IPTV%20on%20UniFi%20Cloud%20Gateway%20(UCG-Fiber).md) | Fixing the ~4m55s live-TV freeze on TELUS Mediaroom/Optik STBs behind the UCG-Fiber: the IGMPv3 multicast group-state timeout and its fix |

## NAS

| Page | What it covers |
|---|---|
| [UniFi Teleport Access to Synology DSM & Docker Services](nas/UniFi%20Teleport%20Access%20to%20Synology%20DSM.md) | Fixing ERR_CONNECTION_ABORTED when reaching DSM (5000/5001) and Docker services (Sonarr, Plex) over the Teleport VPN: static route in DSM plus firewall permissions |
| [Plex Cross-NAS Walkthrough — Synology DS923+ + QNAP](nas/Plex%20Cross-NAS%20Walkthrough%20—%20Synology%20DS923+%20+%20QNAP.md) | Making the QNAP-hosted Plex server see the Synology's SYN_Movies share: SMB on the DS923+, File Station remote mount on the QNAP, Plex libraries, testing, and keeping the mount alive |
| [Synology Cloudflare DDNS for Multidomains and Subdomains](nas/Synology%20Cloudflare%20DDNS%20for%20Multidomains%20and%20Subdomains.md) | Adding Cloudflare as a DDNS provider in DSM: one entry updating multiple domains/subdomains via the Cloudflare API, install steps, and troubleshooting |
| [Nightscout on Synology via Docker Compose and Cloudflare Tunnel](nas/Nightscout%20on%20Synology%20via%20Docker%20Compose%20and%20Cloudflare%20Tunnel.md) | Nightscout for AAPS on a DS923+ in Container Manager (MongoDB + Nightscout + mongo-express), exposed through a Cloudflare Tunnel with no open ports |

## Coming next

These are the planned pages for the rest of the build — they don't exist yet:

- **Vantiva bridge cutover** — sequenced plan with a rollback step at each stage, LAN back to 192.168.1.0/24
- **Telus STB / IGMP proxy test** — the two-stage live-TV multicast test behind the UCG-Fiber
- **NVR firewall rules** — allow the NVR to the house camera and the internet, deny to the NAS and other private ranges
- **Camera lockdown** — P2P/cloud off, VLAN 30, the five EmpireTechs as remotes on the NVR
- **USW-Pro-Max-16-PoE distribution** — basement port map, VLANs, and the 2.5G uplink to the UCG-Fiber
- **Garage leg** — fiber trunk, USW-Flex-2.5G-8, NVR, eero Max 7, and the Onkyo

## Adding a new page

1. Add the `.md` file to the right folder (`unifi/`, `nas/`, or a new folder — named the same as its title).
2. Add one row under that folder's section in the table above, with a link and a one-line summary.
3. If it's one of the planned pages, move it out of "Coming next" when it's written.
