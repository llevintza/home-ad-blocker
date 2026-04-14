# Phase 1 — Raspberry Pi 4 Setup + AdGuard Home Installation

**GitHub Issue:** #1  
**Dependencies:** None  
**Estimated time:** 1–2 hours  

---

## Goal

Install AdGuard Home on a Raspberry Pi 4 (4GB) and configure the Netgear M70 router to use the Pi as the DNS server for all devices on the network.

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
3. Click the gear icon (advanced settings) **before** writing:
   - Enable SSH
   - Set hostname: `adguard` (or your preference)
   - Set username and password (remember these)
   - Leave WiFi unconfigured — you will use Ethernet
4. Write the image and eject the card

---

## Step 2 — First Boot

1. Insert MicroSD into the Pi
2. Connect Pi to the router via Ethernet cable
3. Power on the Pi
4. Wait ~60 seconds for the first boot to complete
5. Find the Pi's IP address from the Netgear admin panel:
   - Log in at `orbilogin.com` or `192.168.1.1`
   - Go to **Attached Devices** — look for the hostname you set (e.g., `adguard`)
   - Note the current IP (e.g., `192.168.1.100`) — this is temporary

---

## Step 3 — Assign a Static IP via DHCP Reservation

Still in the Netgear admin panel:

1. Go to **Advanced > Setup > LAN Setup**
2. Under **Address Reservation**, click **Add**
3. Select the Pi from the device list (or enter its MAC address manually)
4. Assign IP: `192.168.1.2`
5. Click **Add** and then **Apply**
6. Reboot the Pi (`sudo reboot`) — it will come back with the reserved IP

---

## Step 4 — SSH Into the Pi

From your PC:
```bash
ssh <your-username>@192.168.1.2
```

Update the OS before installing anything:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 5 — Install AdGuard Home

```bash
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

The installer will:
- Download and install the AdGuard Home binary
- Set it up as a systemd service (starts automatically on boot)
- Bind to port 3000 for the initial setup wizard

Verify it is running:
```bash
sudo systemctl status AdGuardHome
```

---

## Step 6 — Run the Setup Wizard

From any browser on your network, go to:
```
http://192.168.1.2:3000
```

In the wizard:
1. **Admin web interface**: keep default (port 80)
2. **DNS server**: bind to `192.168.1.2`, port `53`
3. Create your admin username and password
4. Complete the wizard

After setup, the web UI will be at `http://192.168.1.2`

---

## Step 7 — Configure Encrypted Upstream DNS

In AdGuard Home web UI → **Settings > DNS Settings**:

**Upstream DNS servers** (replace any defaults with):
```
https://dns10.quad9.net/dns-query
https://cloudflare-dns.com/dns-query
```

**Bootstrap DNS servers:**
```
9.9.9.9
1.1.1.1
```

**Options to enable:**
- [x] Parallel requests
- [x] DNSSEC

Click **Apply**.

---

## Step 8 — Point the Router DNS to the Pi

In Netgear admin panel → **Advanced > Setup > Internet Setup**:

| Setting | Value |
|---------|-------|
| Primary DNS | `192.168.1.2` (your Pi) |
| Secondary DNS | `94.140.14.14` (AdGuard public — fallback only if Pi is down) |

Click **Apply**. The router will push this DNS to all devices via DHCP on their next lease renewal. To force it immediately, disconnect and reconnect each device to WiFi.

---

## Immediate Interim Step (Do This Before the Pi Is Ready)

While setting up the Pi, apply basic filtering now:

In Netgear admin → **Advanced > Setup > Internet Setup**, set DNS to:
- Primary: `94.140.14.14`
- Secondary: `94.140.15.15`

This activates AdGuard's free cloud DNS for all devices immediately. Revert to the Pi IP once setup is complete.

---

## Verification

- [ ] SSH works: `ssh <user>@192.168.1.2`
- [ ] AdGuard Home web UI loads at `http://192.168.1.2`
- [ ] AdGuard Home service is running: `sudo systemctl status AdGuardHome`
- [ ] Pi shows in AdGuard Home's **Settings > DNS Settings** — upstream test returns green
- [ ] On a connected device, run: `nslookup google.com 192.168.1.2` — should resolve successfully
- [ ] On a connected device, run: `nslookup doubleclick.net 192.168.1.2` — should return `0.0.0.0` (blocked)

---

## Next Step

Proceed to [Phase 2 — Blocklist Configuration](phase-2-blocklists.md)
