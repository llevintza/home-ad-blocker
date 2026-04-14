# Phase 6 — Verification, Testing, and Maintenance

**GitHub Issue:** #6  
**Dependencies:** All phases (1–5) should be complete before running this phase  
**Device action required:** None  

> ⚠️ **Constraints**
> - No jailbreaking of any device (iOS, iPadOS, tvOS, Android)
> - No per-device app installation or configuration
> - All filtering is applied at the network layer only

---

## Goal

Confirm the full network-level ad blocking stack is working correctly across all device types, verify that DNS bypass routes are closed, and document the ongoing maintenance routine.

---

## Post-Setup Verification Checklist

### 1. AdGuard Home Dashboard

Open `http://192.168.1.2` from any browser on the network.

- [ ] **Dashboard**: query count is growing; blocked percentage shows (expect 20–40% after 24h of normal household use)
- [ ] **Blocklists**: all lists show green status with recent update timestamps
- [ ] **Custom rules**: no syntax errors shown (yellow warning icon would indicate an issue)
- [ ] **Upstream DNS**: test upstream servers — all should return green in Settings > DNS Settings

### 2. DNS Resolution Tests (from PC terminal)

```bash
# These should resolve normally (not blocked)
nslookup google.com 192.168.1.2
nslookup youtube.com 192.168.1.2
nslookup netflix.com 192.168.1.2

# These should return 0.0.0.0 (blocked)
nslookup doubleclick.net 192.168.1.2
nslookup spotxchange.com 192.168.1.2
nslookup video.fwmrm.net 192.168.1.2
nslookup dai.google.com 192.168.1.2
nslookup rtr.innovid.com 192.168.1.2
nslookup innovid.com 192.168.1.2
nslookup springserve.com 192.168.1.2
nslookup mask.icloud.com 192.168.1.2
```

### 3. Ad Block Test (from any connected device browser)

Visit: `https://d3ward.github.io/toolz/adblock`

Expected score: **80%+** with DNS only. This score will vary because it tests browser-level blocking — the DNS layer handles what it can at the network level.

### 4. Per-Device DNS Verification (Critical — Phase 3 Confirmation)

In AdGuard Home → **Query Log** and **Client Statistics**:

- [ ] **iPhone / iPad**: the device's IP appears in per-client stats with DNS query activity → confirms iCloud Private Relay is blocked
- [ ] **Apple TV**: the device's IP appears in per-client stats → confirms Private Relay blocked, Pi DNS active
- [ ] **Android phone/tablet**: device appears in stats → confirms Private DNS (port 853) bypass is blocked
- [ ] **Android TV / Chromecast / Fire TV**: device appears in stats
- [ ] **Smart TVs** (Samsung, LG, etc.): device appears in stats
- [ ] **Gaming consoles** (PS5, Xbox): device appears in stats
- [ ] **Smart home / IoT devices**: devices appear in stats

**If any device does NOT appear in the query log**, it is bypassing the Pi. Troubleshoot using the table in the Troubleshooting section below.

### 5. iCloud Private Relay Verification

On an iPhone or iPad connected to the home network:
- Go to **Settings > Apple ID > iCloud > Private Relay**
- If Phase 3 is working correctly, the status will show **"Private Relay is not available on this network"** or a similar message indicating it is blocked
- This is expected and correct — the network is enforcing DNS filtering

On an Apple TV:
- Settings > Privacy > iCloud Private Relay (if the option exists) should similarly show unavailable

### 6. Streaming Service Spot Checks (48-Hour Window)

After Phase 2 SSAI rules are active, monitor these services:

| Service | Watch For | Action If Triggered |
|---------|-----------|-------------------|
| Tubi / Pluto TV | Ad break shows brief black screen then content resumes | Normal — ad decision server timed out |
| Tubi / Pluto TV | Ad break skips entirely | Ideal outcome |
| Hulu | Stream pauses at ad break and never resumes | Remove `adstats.hulu.com` and `pixiedust.hulu.com` from custom rules |
| Prime Video | "Playback error" at ad slots | Remove `fls-na.amazon.com` from custom rules |
| Peacock | Login/account page fails | Check query log, whitelist the blocked auth domain |
| Any service | Black segment of 2–5 seconds then content | Acceptable — ad decision timed out cleanly |

---

## Ongoing Maintenance

### Monthly Tasks

**Update AdGuard Home blocklists:**
- AdGuard Home → **Filters > DNS Blocklists**
- Click **Update** on each list, or set auto-update interval per list (recommended: **3 days**)

**Review false positives:**
- Occasionally check AdGuard Home → **Query Log** for any legitimate services being blocked
- If something stops working, filter the log by that device's IP and look for red (blocked) entries near the time of the issue
- Unblock with: `@@||the-domain.com^` in custom rules

### Raspberry Pi OS Maintenance

```bash
ssh <user>@192.168.1.2

# Monthly OS update
sudo apt update && sudo apt upgrade -y

# Check AdGuard Home service
sudo systemctl status AdGuardHome

# Disk usage check (MicroSD wear monitoring)
df -h /

# Temperature check (should stay below 70°C)
vcgencmd measure_temp
```

### AdGuard Home Configuration Backup

After completing all phases and verifying everything works, export the AdGuard Home configuration:

- AdGuard Home → **Settings > Configuration**
- Click **Export configuration**
- Save `AdGuardHome.yaml` securely (do not commit to a public repo — it contains your admin credentials)

To restore after a Pi rebuild: run the Phase 1 installer, then import this file.

---

## Troubleshooting Reference

| Problem | Diagnosis | Fix |
|---------|-----------|-----|
| A device doesn't appear in AdGuard Home client stats | Device is bypassing the Pi DNS | Check: (1) iCloud Private Relay — Phase 3 Step 2; (2) Android Private DNS — Phase 3 Step 4; (3) Hardcoded DNS app — Phase 3 Step 1B |
| iPhone shows "Private Relay not available" but still no queries in log | Private Relay blocked but device using mobile data (not WiFi) | Ensure device is on home WiFi, not cellular |
| All DNS stops working for all devices | Pi is down or AdGuard Home crashed | SSH to Pi: `sudo systemctl restart AdGuardHome`. Router falls back to secondary DNS `94.140.14.14` automatically |
| Specific website/app broken after Phase 2 | Blocklist false positive | Check query log for that device's IP → whitelist the blocked domain: `@@||domain.com^` |
| Hulu stream stalls at ad break | Innovid/Hulu rules too aggressive | Remove `innovid.com` rules from custom rules; leave FreeWheel rules intact |
| Tubi/Pluto TV stream stalls (not just brief spinner) | FreeWheel timeout too long for this service | Remove `video.fwmrm.net` rule; re-add only the Innovid rules |
| AdGuard Home web UI unreachable | Service crashed | `sudo systemctl restart AdGuardHome` |
| Pi won't boot | MicroSD wear/corruption | Replace MicroSD with new card; re-run Phase 1 (import backup config from Step above) |
| Smart TV or console still getting many ads | SSAI cannot be fully blocked at DNS level | Expected — see feasibility table in README. The DNS layer achieves what is technically possible |

---

## System Is Operational

When all checklist items above are green, the network ad blocking system is fully operational:

- Every device on the network has DNS filtered through the Pi automatically
- No device has been modified, jailbroken, or individually configured
- iCloud Private Relay and Android Private DNS bypasses are closed at the network level
- Trackers, telemetry, banner ads, and mobile in-app ads are blocked network-wide (~85–95%)
- SSAI ad decision servers are blocked, disrupting in-stream ads on supported platforms
- Apple TV, iPhones, iPads are covered with zero device interaction
