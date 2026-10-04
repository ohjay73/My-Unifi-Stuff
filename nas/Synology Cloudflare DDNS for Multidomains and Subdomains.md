# Synology Cloudflare DDNS for Multidomains and Subdomains

Adds Cloudflare as a DDNS provider in Synology DSM (Control Panel > External Access > DDNS),
so the NAS itself keeps your Cloudflare DNS records pointed at your current home IP —
including multiple domains and subdomains in one entry.

Based on the PHP script from [mrikirill/SynologyDDNSCloudflareMultidomain](https://github.com/mrikirill/SynologyDDNSCloudflareMultidomain)
(Cloudflare API v4). Jason's fork: [ohjay73/SynologyDDNSCloudflareMultidomain](https://github.com/ohjay73/SynologyDDNSCloudflareMultidomain).

## What it does

- Updates Cloudflare A records (and AAAA records when IPv6 is available, auto-detected via ipify) straight from DSM's DDNS panel.
- Supports single domains, multiple domains, subdomains, and regional domains in any combination
  (e.g. `dev.my.domain.com.au`, `domain.com.uk`).
- Multiple hostnames go in one entry, separated by `|`:
  `subdomain.mydomain.com|vpn.mydomain.com` (256-character DSM UI limit).
- Dual-stack IPv4 and IPv6.

## Before you begin

1. **Cloudflare API token** (not the Global API key — create one at dash.cloudflare.com/profile/api-tokens) with these permissions:
   - Zone > Zone Settings > Read
   - Zone > Zone > Read
   - Zone > DNS > Edit
   - Zone resources: Include > All zones from an account > your domain(s)
2. **DNS records pre-created** in Cloudflare: the A record(s) — and AAAA record(s) for IPv6 —
   for every domain/zone the script will update. (Proxied can stay on; it just hides your real IP.)
3. **SSH access** on the Synology: Control Panel > Terminal & SNMP > Enable SSH service.
   (SRM: Control Panel > Services > System Services > Terminal > Enable SSH service.)

## Install

1. Enable SSH (above), then connect as root and run the installer:
   ```
   wget https://raw.githubusercontent.com/mrikirill/SynologyDDNSCloudflareMultidomain/master/install.sh -O install.sh && sudo bash install.sh
   ```
2. In DSM: Control Panel > External Access > DDNS > Add:
   - **Service provider:** Cloudflare
   - **Hostname:** not used by the script — any value works
   - **Username:** your domain, or pipe-separated domains: `subdomain.mydomain.com|vpn.mydomain.com`
   - **Password:** your Cloudflare API token
3. Press **Test Connection**, then OK to save.
4. Turn SSH back off when you're done — leaving it on is a security risk.

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| API error on free TLDs (`.cf`, `.ga`, `.gq`, `.ml`, `.tk`) | Cloudflare's API refuses DDNS updates for these TLDs — manage them in the Cloudflare dashboard instead |
| Connection test fails, or nothing in the Cloudflare audit log | Something's mistyped in the DDNS screen — re-check the token, domains, and permissions; API updates show up in the audit log as a `Rec Set` action |
| Cloudflare disappears as a provider after a DSM/SRM update | Updates wipe `/usr/syno/bin/ddns/cloudflare.php` and reset `ddns_provider.conf` — just re-run the installer |

## Debug

Run the script directly to see its logs:

```
/usr/bin/php -d open_basedir=/usr/syno/bin/ddns -f /usr/syno/bin/ddns/cloudflare.php "domain1.com|vpn.domain2.com" "your-Cloudflare-token" "" "your-ip-address"
```

## Notes

- Proxied A records hide your home IP from snoopers; unproxied exposes it but is required for some services — pick per record.
- If you'd rather not run PHP on the NAS at all, the author also publishes a native Kotlin version: [KTSynologyDDNSCloudflareMultidomain](https://github.com/mrikirill/KTSynologyDDNSCloudflareMultidomain).
