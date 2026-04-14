# Phase 6 — Verification, Testing, and Maintenance

**GitHub Issue:** #6  
**Dependencies:** Phases 1–5 (run after all other phases)  
**Estimated time:** 1–2 hours  

---

## Goal

Confirm the full stack is working correctly across all device types, and document the ongoing maintenance routine to keep everything effective.

---

## Post-Setup Verification Checklist

### Network Layer (Phase 1–2)

Run these checks from the AdGuard Home web UI at `http://192.168.1.2`:

- [ ] **Dashboard**: query count is growing, blocked percentage is visible (expect 20–40% after 24h of normal use)
- [ ] **Per-Client Statistics**: all device types visible — phones, tablets, TVs, consoles, smart home devices
- [ ] **Blocklists**: all lists show green (last updated recently)
- [ ] **Custom rules**: no syntax errors shown

Run these from a terminal on your PC:
```bash
# Should resolve normally
nslookup google.com 192.168.1.2

# Should return 0.0.0.0 (blocked)
nslookup doubleclick.net 192.168.1.2
nslookup spotxchange.com 192.168.1.2
nslookup video.fwmrm.net 192.168.1.2
nslookup imasdk.googleapis.com 192.168.1.2
```

### Browser Test

From any device, visit: https://d3ward.github.io/toolz/adblock

Expected score: **80%+ with DNS only**, **90%+ with DNS + browser extension (uBlock Origin)**

### Per-Device Spot Checks

| Device | Test | Expected Result |
|--------|------|----------------|
| Android phone (ReVanced) | Open YouTube, start a video | No pre-roll ad, instant playback |
| Android phone (AdGuard) | Browse news site | No banner ads visible |
| iPhone / iPad | Browse news site in Safari | No banner ads visible |
| Android TV (SmartTube) | Open YouTube, start a video | No ad, instant playback |
| Android TV (Tubi) | Start any video | Ad break skipped or brief spinner then content |
| Gaming console | Browse YouTube | Ads may appear — expected, no Layer 2 available |
| Smart TV | Open any streaming app | Reduced trackers; in-stream ads may persist |
| Desktop browser | Open YouTube in Chrome + uBlock Origin | No ads |

### 48-Hour Streaming Stability Check

After enabling custom rules from Phase 2, monitor:

| Service | Watch For | Action If Triggered |
|---------|-----------|-------------------|
| Hulu | Stream pauses at ad break and never resumes | Remove `adstats.hulu.com` and `pixiedust.hulu.com` from custom rules |
| Prime Video | "Playback error" at ad slots | Remove `fls-na.amazon.com` from custom rules |
| Tubi | 5–10 second spinner then content resumes | Normal — ad server timed out, content plays |
| Peacock | Login page fails to load | Check query log, whitelist any blocked auth domain |

---

## Ongoing Maintenance

### Monthly Tasks

**AdGuard Home blocklist updates:**
- In AdGuard Home → **Filters > DNS Blocklists**
- Click **Update** on each list, or enable auto-update:
  - Each list has an **Update interval** setting — set to `3 days` or `1 week`

**Check for new false positives:**
- Review AdGuard Home **Query Log** occasionally
- If a service stops working, filter the log by that device's IP and look for blocked red entries

### As-Needed Tasks

**ReVanced re-patch (Android YouTube):**

When YouTube updates and ReVanced stops working:
1. Open ReVanced Manager
2. Tap **Patcher** — it detects the new YouTube version
3. Re-patch with default patches
4. Install the new patched APK (~5 minutes)

Frequency: roughly monthly when YouTube updates.

**SmartTube update (Android TV):**

SmartTube has an in-app update checker. When a new version is available:
1. The app will notify you
2. Or check https://github.com/yuliskov/SmartTube/releases for the latest APK
3. Install via ADB: `adb install -r smarttube_stable.apk` (the `-r` flag reinstalls without removing)

**AdGuard for Android / TV updates:**
- AdGuard has a built-in update checker
- Or re-download the APK from adguard.com and install over the existing version

### Raspberry Pi Maintenance

```bash
# Monthly: update the OS
ssh <user>@192.168.1.2
sudo apt update && sudo apt upgrade -y

# Check AdGuard Home service health
sudo systemctl status AdGuardHome

# Check disk usage (should stay well below MicroSD capacity)
df -h /

# Check Pi temperature (should stay below 70°C under normal load)
vcgencmd measure_temp
```

---

## Troubleshooting Reference

| Problem | Diagnosis | Fix |
|---------|-----------|-----|
| A device is not being filtered | Check if device IP shows in AdGuard Home per-client view | Device may have hardcoded DNS (8.8.8.8). Force it to use router DNS in device network settings, or set DNS manually to `192.168.1.2` |
| All DNS stops working | Pi is down or AdGuard Home crashed | SSH to Pi, run `sudo systemctl restart AdGuardHome`. Router falls back to `94.140.14.14` (secondary DNS set in Phase 1) |
| A specific website/app broken | Blocklist false positive | Check query log for that device, whitelist the blocked domain with `@@||domain.com^` |
| AdGuard Home web UI unreachable | Service crashed | `sudo systemctl restart AdGuardHome` |
| Pi won't boot after SD card write error | MicroSD wear | Replace with fresh MicroSD and re-run Phase 1 (consider USB SSD for longer lifespan) |
| ReVanced YouTube shows ads again | YouTube app updated, patch stale | Re-patch via ReVanced Manager |
| SmartTube not available in TV launcher | APK install failed | Re-run `adb install smarttube_stable.apk` |

---

## Backup: Export AdGuard Home Configuration

After completing Phases 1–2 and confirming everything works, export the config so you can restore quickly if the Pi needs to be rebuilt:

In AdGuard Home → **Settings > Configuration**:
- Click **Export configuration**
- Save the `AdGuardHome.yaml` file securely (do not commit to a public repo — it contains your admin credentials)

To restore: re-run the AdGuard Home installer (Phase 1, Step 5), then import the exported config file.

---

## Full System Is Operational

At this point your setup delivers:
- Network-wide DNS ad and tracker blocking for every device automatically
- 100% YouTube ad blocking on Android phones (ReVanced) and Android TV devices (SmartTube)
- Maximum achievable ad reduction on iOS via AdGuard Pro
- Device-level HTTPS filtering on Android phones and Android TV via AdGuard
- Encrypted DNS upstream (Quad9 + Cloudflare DoH) — ISP cannot see your DNS queries
- Partial in-stream ad disruption on Tubi, Pluto TV, and Peacock via SSAI endpoint blocking
