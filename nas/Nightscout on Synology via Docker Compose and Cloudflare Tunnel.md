# Nightscout on Synology via Docker Compose and Cloudflare Tunnel

Runs Nightscout (the CGM remote monitor that feeds AAPS) on a Synology NAS — tested on a
DS923+ with DSM 7.2.2 — and exposes it to the internet through a Cloudflare Tunnel.
No port forwarding, no QuickConnect, no reverse-proxy-and-certificate juggling.

Source repo: [ohjay73/Synology-Nightscout-compose-](https://github.com/ohjay73/Synology-Nightscout-compose-).
The same pattern works for exposing any other NAS app the same way.

## What you need

- A Synology NAS (DSM 7.x; tested on 7.2.2-72806 on a DS923+)
- A purchased domain managed in Cloudflare
- Upstream projects: [nightscout/cgm-remote-monitor](https://github.com/nightscout/cgm-remote-monitor)
  and [nightscout/AndroidAPS](https://github.com/nightscout/AndroidAPS)

## The stack

One Compose project builds three containers:

| Container | Image | Port |
|---|---|---|
| MongoDB | `mongo:5.0` | 27017 |
| Nightscout | `nightscout/cgm-remote-monitor:latest_dev` (15.0.3, current AAPS support) | 1337 |
| mongo-express | `mongo-express:latest` (DB admin UI) | 8081 |

Notes from testing: Mongo 5.0 works on the DS923+; 4.4.18 also works. Mongo 6 and above did
not build successfully — stick with 5.0.

## Set it up

1. **Create the folders** in File Station (or via CLI, where the volume shows as `/volume1`):
   - `/volume1/docker/Mongo/nightscout-mongodb`
   - `/volume1/docker/cloudflare` (for the tunnel connector later)
2. **Create the project:** Container Manager > Project > Create.
   - Project name: anything descriptive
   - Path: `docker/Mongo/nightscout-mongodb`
   - Source: create `docker-compose.yaml`, paste in the compose file from the repo.
3. **Edit these before building** — the build will not work correctly without them:
   - `API_SECRET`: set your own secret (used to authenticate to Nightscout)
   - `CUSTOM_TITLE`: name your site
   - Review the `ENABLE` plugin list and trim it to what you actually use.
4. Click through Next/Next/Done — the project builds the three containers.

## Test it locally

1. Browse to `http://<NAS-IP>:1337` — you should get a working Nightscout site.
2. Authenticate with your API secret, open **Admin Tools**, and create an **Access Token**
   with admin privileges — AAPS needs this to talk to Nightscout.
3. Set up your base profile in Nightscout now; AAPS pulls it from here once connected.

## Expose it with a Cloudflare Tunnel

Cloudflare was removed as a DDNS provider option in DSM 7, so DDNS goes through the
[Cloudflare DDNS script](https://github.com/ohjay73/SynologyDDNSCloudflareMultidomain)
(see the writeup "Synology Cloudflare DDNS for Multidomains and Subdomains"). The tunnel
itself needs no open ports at all.

1. In Cloudflare: search for **Zero Trust** and open it (first visit: create a team name;
   the free plan is fine).
2. **Networks > Tunnels > Create a tunnel**, select **cloudflared**, give it a name.
3. Choose **Docker** and copy the full `docker run` command — you only need the token from it:
   `docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <TOKEN>`
4. **Second Compose project** for the connector: Container Manager > Project > Create,
   path `docker/cloudflare`, with this compose file (token from step 3):
   ```yaml
   version: "3.3"
   services:
     cloudflared:
       image: cloudflare/cloudflared:latest
       command: tunnel run
       environment:
         - TUNNEL_TOKEN=<paste-your-token-here>
   ```
5. Back in Zero Trust: the connector should show **Connected**. Click Next to configure routes:
   - **Subdomain:** e.g. `nightscout`
   - **Domain:** your Cloudflare domain
   - **Service type:** HTTP
   - **URL:** `http://<NAS-IP>:1337`
6. Browse to `https://nightscout.yourdomain` — your Nightscout instance loads.
7. Enter that URL in AAPS. Works with both the v1 and v3 APIs, websockets included.

## Reference

- Full compose files and the original (unedited) walkthrough live in the source repo linked at the top.
- An alternative approach using Synology's own DDNS, Let's Encrypt certificates, port
  forwarding, and a reverse proxy is documented at
  [ohjay73/Synology_NightScout_Easy](https://github.com/ohjay73/Synology_NightScout_Easy) —
  more moving parts and less secure, which is why this guide uses the tunnel instead.
