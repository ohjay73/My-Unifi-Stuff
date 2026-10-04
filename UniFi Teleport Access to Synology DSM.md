# UniFi Teleport Access to Synology DSM & Docker Services

When accessing a Synology NAS (e.g., DS923+) over Ubiquiti UniFi Teleport VPN, the connection may fail with `ERR_CONNECTION_ABORTED` on DSM (port 5000/5001) and Docker services (e.g., Sonarr on 8989, Plex on 32400)[cite: 1, 3].

This happens because Teleport assigns client devices to a separate dynamic VPN subnet (typically `192.168.3.0/24`)[cite: 1]. Without explicit route definitions and firewall permissions, Synology DSM drops return packets or fails to route them back across the gateway[cite: 1, 4].

---

## Network Topology Reference

* **UniFi Gateway / Router:** `192.168.2.1`[cite: 5]
* **Synology NAS (Primary/10GbE Interface):** `192.168.2.211` (`LAN 3`)[cite: 1, 5]
* **Teleport VPN Client Subnet:** `192.168.3.0/24` (Client IP: `192.168.3.1`)[cite: 1]

---

## 1. Add Static Route in DSM (Critical Fix)

By default, DSM may not know how to route return traffic back to an external dynamic VPN subnet when using non-primary interfaces (such as an upgraded 10GbE PCIe adapter on LAN 3)[cite: 5].

1. Log in to **DSM** via your local network.
2. Go to **Control Panel** > **Network** > **Static Route** tab[cite: 5].
3. Click **Create** and configure:
   * **Network Destination:** `192.168.3.0`[cite: 1]
   * **Netmask:** `255.255.255.0` (or `24`)
   * **Gateway:** `192.168.2.1`[cite: 5]
   * **Interface:** Select your active interface (e.g., `LAN 3` / 10GbE Mini)[cite: 5]
4. Click **OK** to save and enable the route.

---

## 2. Allow Teleport Subnet in Synology Firewall

If the DSM firewall is enabled, it will block incoming traffic originating outside the local `192.168.2.0/24` subnet[cite: 1, 4].

1. In DSM, go to **Control Panel** > **Security** > **Firewall** tab[cite: 4].
2. Click **Edit Rules** on your active profile (e.g., `default` or `All interfaces`)[cite: 4].
3. Click **Create** and set:
   * **Ports:** `All`[cite: 4]
   * **Protocol:** `All`[cite: 4]
   * **Source IP:** Select **Specific IP** > **Subnet**[cite: 4]
     * **IP Address:** `192.168.3.0`[cite: 4]
     * **Subnet Mask:** `255.255.255.0`[cite: 4]
   * **Action:** `Allow`[cite: 4]
4. Click **OK**[cite: 4].
5. **Important:** Drag the new allow rule to the **top** of the list so it evaluates before any deny rules[cite: 4].
6. Click **OK** and then **Apply**[cite: 4].

---

## 3. Verify Default Gateway & Routing Configuration

Ensure DSM's global routing points to the UniFi Gateway:

1. Go to **Control Panel** > **Network** > **General** tab[cite: 5].
2. Verify **Default gateway** displays `192.168.2.1 (LAN 3)`[cite: 5].
3. Click **Advanced Settings**[cite: 5]:
   * Check **Enable Multiple Gateway**.
   * Check **Reply to ARP requests if the target IP address is a local address configured on the incoming interface**.
4. Click **Apply**.

---

## 4. Verification & Testing

1. Disconnect your mobile device from home Wi-Fi and enable mobile data[cite: 1].
2. Open the **WiFiman** app and toggle **Teleport** to **On**[cite: 1].
3. Confirm your assigned IP is in the `192.168.3.x` range[cite: 1].
4. Test access in a mobile browser:
   * DSM Base UI: `http://192.168.2.211:5000`
   * Sonarr: `http://192.168.2.211:8989`[cite: 1]
   * Plex: `http://192.168.2.211:32400/web`
