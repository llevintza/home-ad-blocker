# Phase 1 — Raspberry Pi 4 Setup + AdGuard Home Installation

**GitHub Issue:** #1  
**Dependencies:** None — start here  
**Estimated time:** 1–2 hours  

> ⚠️ **Constraints**
> - No jailbreaking of any device (iOS, iPadOS, tvOS, Android)
> - No per-device app installation or configuration
> - All filtering is applied at the network layer only

---

## Goal

Install AdGuard Home on a Raspberry Pi 4 (4GB) and configure the home router to distribute the Pi's IP as the DNS server to every device on the network automatically via DHCP.

This solution is **router-agnostic** — it works with any home router (Netgear, TP-Link, Asus, Linksys, Eero, Ubiquiti, Google Nest WiFi, etc.) that supports custom DNS in DHCP settings and DHCP address reservation.

---

## Assumed Network Topology

| Element | Value |
|---------|-------|
| Home router LAN IP | `192.168.1.1` (adjust to match your router — may be `192.168.0.1` or other) |
| Pi reserved IP | `192.168.1.2` |
| LAN subnet | `192.168.1.0/24` |
| AdGuard Home web UI | `http://192.168.1.2` (after setup) |
| AdGuard Home DNS port | `53` on `192.168.1.2` |

> If your router uses a different subnet (e.g. `192.168.0.x`), substitute accordingly throughout all phases.

---

## Hardware Checklist

Before starting, confirm you have:
- [ ] Raspberry Pi 4 Model B (4GB)
- [ ] MicroSD card — 32GB minimum, Class 10 (Samsung Endurance or SanDisk Endurance recommended for always-on use)
- [ ] MicroSD card reader (to flash from your PC)
- [ ] Ethernet cable (connect Pi directly to the router — do not use WiFi for a DNS server)
- [ ] USB-C power supply — official Raspberry Pi 15W adapter recommended

---

## Step 1 — Flash Raspberry Pi OS

1. Download and install **Raspberry Pi Imager** from https://raspberrypi.com/software
2. In the imager:
   - **Device**: Raspberry Pi 4
   - **OS**: Raspberry Pi OS Lite (64-bit) — no desktop environment needed
   - **Storage**: your MicroSD card
3. Click the gear icon (⚙ advanced settings) **before** writing:
   - Enable SSH
   - Set hostname: `adguard` (or your preference)
   - Set a username and password (remember these — you will SSH with them)
   - Leave WiFi unconfigured — use Ethernet
4. Write the image and eject the card

---

## Step 2 — First Boot and Find the Pi's IP

1. Insert MicroSD into the Pi
2. Connect Pi to the router via Ethernet cable
3. Power on the Pi
4. Wait ~60 seconds for the first boot to complete
5. Find the Pi's IP address from your router's admin panel:

**How to access your router's admin panel:**
| Router Brand | Typical URL | Alternative |
|-------------|-------------|-------------|
| Netgear (Orbi/Nighthawk) | `orbilogin.com` or `routerlogin.net` | `192.168.1.1` |
| TP-Link | `tplinkwifi.net` | `192.168.0.1` |
| Asus | `router.asus.com` | `192.168.1.1` |
| Eero | Eero mobile app | — |
| Google Nest WiFi | Google Home app | — |
| Linksys | `linksyssmartwifi.com` | `192.168.1.1` |
| Ubiquiti UniFi | UniFi Network app or `192.168.1.1` | — |
| Generic | `192.168.1.1` or `192.168.0.1` | Check label on router |

Once logged in, look for **Connected Devices**, **Attached Devices**, or **Device List** — find the hostname you set (e.g., `adguard`) and note its current IP.

---

## Step 3 — Assign a Static IP via DHCP Reservation

DHCP reservation ties the Pi's MAC address to a fixed IP so it always gets `192.168.1.2`.

**Where to find this setting by router brand:**
| Router Brand | Menu Path |
|-------------|-----------|
| Netgear (Orbi/Nighthawk) | Advanced > Setup > LAN Setup > Address Reservation |
| TP-Link | Advanced > Network > LAN > DHCP Server > Address Reservation |
| Asus | LAN > DHCP Server > Manually Assigned IP |
| Eero | Eero app > Network > Connected Devices > (select Pi) > Reserve IP |
| Google Nest WiFi | Google Home app > WiFi > Devices > (select Pi) > Reserve IP |
| Linksys | Smart WiFi > Local Network > DHCP Reservations |
| Ubiquiti UniFi | Network > Clients > (select Pi) > Fixed IP |

**Steps (generic):**
1. Navigate to DHCP reservation settings
2. Add a new reservation using the Pi's MAC address → assign IP `192.168.1.2`
3. Save/Apply
4. Reboot the Pi: `sudo reboot` (over SSH at its current IP)
5. After ~30 seconds, SSH in at the new fixed IP: `ssh <user>@192.168.1.2`

---

## Step 4 — Update the Pi OS

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 5 — Install AdGuard Home

```bash
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

The installer:
- Downloads and installs the AdGuard Home binary to `/opt/AdGuardHome/`
- Registers it as a systemd service (auto-starts on boot)
- Binds to port 3000 for the initial setup wizard

Verify the service is running:
```bash
sudo systemctl status AdGuardHome
# Expected: ● AdGuardHome.service - Active: active (running)
```

---

## Step 6 — Run the Setup Wizard

From any browser on your network:
```
http://192.168.1.2:3000
```

In the wizard:
1. **Admin web interface**: keep port 80 (default)
2. **DNS server**: bind to `192.168.1.2`, port `53`
3. Create your admin username and a strong password
4. Complete the wizard — the web UI will now be at `http://192.168.1.2`

---

## Step 7 — Configure Encrypted Upstream DNS

In AdGuard Home web UI → **Settings > DNS Settings**:

**Upstream DNS servers** (delete defaults and enter):
```
https://dns10.quad9.net/dns-query
https://cloudflare-dns.com/dns-query
```

**Bootstrap DNS servers:**
```
9.9.9.9
1.1.1.1
```

**Options:**
- [x] Enable Parallel requests
- [x] Enable DNSSEC

Click **Apply**, then click **Test upstreams** — both should show green.

---

## Step 8 — Point Router DHCP DNS to the Pi

This is the key step that makes every device on the network automatically use the Pi for DNS.

**Where to find DNS settings in your router:**
| Router Brand | Menu Path |
|-------------|-----------|
| Netgear (Orbi/Nighthawk) | Advanced > Setup > Internet Setup > Domain Name Server (DNS) Address |
| TP-Link | Advanced > Network > Internet > DNS Server (or Advanced > DHCP Server) |
| Asus | WAN > Internet Connection > DNS Server |
| Eero | Eero app > Network Settings > DNS |
| Google Nest WiFi | Google Home app > WiFi > DNS |
| Linksys | Smart WiFi > Connectivity > Local Network > DNS |
| Ubiquiti UniFi | Network > Settings > Networks > LAN > DHCP > DNS Server |

**Set these values:**
| Setting | Value |
|---------|-------|
| Primary DNS | `192.168.1.2` (the Pi) |
| Secondary DNS | `94.140.14.14` (AdGuard public DNS — fallback if Pi is unreachable) |

Save/Apply. Devices will pick up the new DNS on their next DHCP lease renewal. To apply immediately: disconnect and reconnect the device to WiFi.

---

## Immediate Interim Step (Apply Now While Pi Is Being Set Up)

Before the Pi is ready, apply basic cloud-based filtering on your router today:

Set router DNS to:
- Primary: `94.140.14.14`
- Secondary: `94.140.15.15`

This activates AdGuard's free public DNS for all devices immediately. Revert to `192.168.1.2` once the Pi is configured.

---

## Verification Checklist

Run these checks before proceeding to Phase 2:

```bash
# 1. SSH connects to the Pi
ssh <user>@192.168.1.2

# 2. AdGuard Home service is active
sudo systemctl status AdGuardHome

# 3. AdGuard Home web UI is reachable
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.2
# Expected output: 200 or 302

# 4. DNS resolution works through the Pi
nslookup google.com 192.168.1.2
# Expected: Non-authoritative answer with an IP address

# 5. Pi upstream test passes
# In AdGuard Home web UI → Settings > DNS Settings → click "Test upstreams"
# Expected: all upstreams return green checkmarks

# 6. A device on the network is using the Pi as DNS
# On a phone: disconnect WiFi, reconnect, then check DNS in network settings
# Should show 192.168.1.2 as the DNS server
```

- [ ] SSH to `192.168.1.2` succeeds
- [ ] `sudo systemctl status AdGuardHome` → active (running)
- [ ] `http://192.168.1.2` loads AdGuard Home dashboard
- [ ] Upstream DNS test: green for both Quad9 and Cloudflare
- [ ] `nslookup google.com 192.168.1.2` → resolves successfully
- [ ] A connected device reports `192.168.1.2` as its DNS server in network settings

---

## Next Step

Proceed to [Phase 2 — Blocklist Configuration](phase-2-blocklists.md)
