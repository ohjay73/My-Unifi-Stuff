# UCG-Fiber Bench Setup Walkthrough — Ubiquiti Cloud Gateway Fiber

This is the write-up of the bench bring-up walked through on Sep 28. The goal: configure the
UCG-Fiber off to the side (double-NATed behind the Vantiva) before any final cutover, so the
rest of the house never goes down.

```
Bench topology:

  Vantiva (192.168.1.254) ─── UCG-Fiber 10 GbE RJ45 WAN port
                                    │
                        UCG-Fiber LAN port 3 ─── unmanaged switch ─── PC
```

Two separate cables, two separate domains. The unmanaged switch is LAN-side only — it must
never also connect to the Vantiva network, or it bridges the UCG-Fiber's WAN and LAN domains
and creates competing DHCP paths.

## Step 1 — Cable it right

1. Plug the Vantiva leg into the **10 GbE RJ45 WAN port** — the one outside the numbered 1–4 block.
   (The Vantiva's 10G port feeds it; the UCG-Fiber's WAN pulls a 192.168.1.x address from Vantiva DHCP.)
2. Plug your setup PC (or a dumb unmanaged switch with the PC on it) into **LAN port 3**.
   No config needed on the unmanaged switch — it acts as just a wire between the PC and the router.

> **The gotcha that actually bit on Sep 28:** the cable was first plugged into UCG-Fiber port 1
> (a 2.5G LAN port) instead of the WAN port. That bridged the UCG-Fiber's LAN DHCP server onto
> the Vantiva's 192.168.1.x network — poisoning DHCP on the main network — and the console
> couldn't be found. Fix: unplug from port 1 immediately and move the cable to the 10 GbE RJ45
> WAN port.

## Step 2 — Set up via the UniFi mobile app (Bluetooth)

1. Power on the UCG-Fiber and wait for it to settle (2–3 minutes from cold).
2. On your phone, open the UniFi app and start **Set Up a New Console** — the app finds the
   console over Bluetooth.
3. When the wizard asks for the local network, **do not accept the 192.168.1.0/24 default.**
   Set it to a temporary **192.168.10.0/24** (gateway **192.168.10.1**). Reason: the WAN side is
   pulling a 192.168.1.x address from the Vantiva during bench, so the LAN can't use the same
   subnet.
4. Finish the wizard. After it completes, the console lives at `https://192.168.10.1`.
5. You can adopt the console with a Ubiquiti account for remote management, or use the offline
   local-admin path — both work. See the account note below.

> **Sep 28 reality check:** the app flow got wedged partway — the app detected the console but
> couldn't establish a connection. The working recovery was to factory-reset the console and
> redo setup (see Step 4), and the confirmed-working access path ended up being
> **UCG-Fiber LAN port 3 → unmanaged switch → PC** from a browser.

## Step 3 — Set up from a browser (the path that worked)

1. With the PC on the LAN side (port 3 → unmanaged switch → PC), browse to `https://192.168.1.1`.
2. Accept the certificate warning (expected — it's a self-signed factory cert).
3. If you already completed the wizard with the temp LAN, the console is now at
   `https://192.168.10.1` instead.
4. The PC should obtain an address from the UCG-Fiber's DHCP server — no static IP needed.

**Verify you're in:**

- [ ] Console loads at its LAN IP.
- [ ] Dashboard shows the WAN interface with a 192.168.1.x address from the Vantiva.
- [ ] The LAN network shows 192.168.10.0/24.
- [ ] A device on the bench LAN can reach the internet through the double NAT.

## Step 4 — Factory reset (when setup is wedged)

If the app or wizard gets stuck halfway (console detected but not connectable, or a
half-configured state):

1. Find the reset hole on the console.
2. Hold the reset button for ~10 seconds.
3. Wait 2–3 minutes for it to reboot fully — this wipes the half-configured state.
4. Redo setup via a fresh app/Bluetooth flow or the browser path above, with the WAN cable in
   the 10 GbE RJ45 WAN port and the temp LAN at 192.168.10.0/24.

## The subnet plan — bench vs. final cutover

| Phase | LAN subnet | Console | Notes |
|---|---|---|---|
| Bench (now) | 192.168.10.0/24 | `https://192.168.10.1` | Double-NATed behind the Vantiva; WAN pulls 192.168.1.x |
| Final cutover | 192.168.1.0/24 | `https://192.168.1.1` | Vantiva goes to bridge mode; LAN moves back so the camera (.130), NAS (.211), and other statics keep working |

## UI account vs. local-only

Not yet decided — both are viable:

- **Ubiquiti account:** remote management via unifi.ui.com, push notifications, Teleport, cloud backups.
- **Local-only (offline admin):** console manageable at its LAN IP forever; remote access via
  WireGuard (already the explored direction for camera/NVR access) instead of the cloud.

Nothing about the account choice changes the port map or the SFP+ allocations below.

## The port map this setup feeds into

Once the bench is up, this is the final allocation it was all heading toward:

- **10G RJ45 WAN** ← Vantiva 10G (bridge mode at cutover)
- **SFP+ LAN** → UACC-CM-RJ45-MG → Cat6a → Synology DS923+ at 10G
- **Remapped SFP+ (WAN→LAN)** → BiDi pair → garage fiber trunk (OS2, pre-terminated armored 4-fiber LC-LC, house→garage pull)
- **2.5G PoE+ port** (30W, VLAN 30 access) → house camera (re-patch from the YuanLey YS25-0402P)
- **2.5G** → YuanLey YS25-0402P uplink (backyard eero Outdoor 7, main LAN)
- **2.5G** → YuanLey YS25-2400
- **2.5G** → living-room / Vimin chain

Both SFP+ ports are allocated, so there is no free SFP+ for a DAC trunk to the future
USW-Pro-Max-16-PoE — the correct uplink there is 2.5G RJ45-to-RJ45.

## Open / not confirmed

- The full setup wizard was never confirmed completed in the Sep 28 window — only console
  access ("ok im in" via the port-3 path).
- On Sep 29 the bench LAN showed as 192.168.2.x on both the phone and the gateway (not the
  planned 192.168.10.0/24), with local Chrome access working but the UniFi mobile app reporting
  the site offline. The remote-app/cloud access question is still open.
- The Vantiva bridge cutover is still pending; the LAN move back to 192.168.1.0/24 happens then.
- IGMP proxy for the Telus STBs was started on the bench (native IGMP Proxy + LAN-side snooping);
  whether bridge mode passes IPTV multicast to the UCG-Fiber is still untested.
