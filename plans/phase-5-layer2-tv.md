# Phase 5 — TV Ad Blocking: SmartTube + AdGuard TV via ADB

**GitHub Issue:** #5  
**Dependencies:** Phase 1 complete  
**Applies to:** Android TV, Chromecast with Google TV, Amazon Fire TV, Smart TVs, Gaming Consoles  
**Can run in parallel with:** Phase 3, Phase 4  

---

## Goal

Deploy ad-blocking solutions on TV-connected devices. Android TV-based devices (Chromecast GTV, Fire TV Stick, NVIDIA Shield, Android TV boxes) support sideloading apps via ADB. Smart TVs and gaming consoles are locked and rely entirely on Phase 1–2 DNS filtering.

---

## Prerequisites

On your PC:
- Install **ADB (Android Debug Bridge)**:
  - Windows: Download [Platform Tools from Google](https://developer.android.com/tools/releases/platform-tools)
  - Linux/macOS: `sudo apt install adb` or `brew install android-platform-tools`
- Ensure PC and TV device are on the **same WiFi network**

---

## Part A — YouTube: SmartTube (Android TV / Fire TV / Chromecast GTV)

SmartTube is a full native YouTube replacement for Android TV-class devices. It blocks 100% of ads and has SponsorBlock integration.

### Supported Devices

- Chromecast with Google TV
- Amazon Fire TV Stick (all generations)
- NVIDIA Shield
- Any Android TV box
- Mi Box S
- Sony / Philips / TCL Android TVs (sideloading may need enabling in settings)

### Step 1 — Enable Developer Mode on the TV Device

**Chromecast with Google TV / Android TV:**
1. Settings > System > About > scroll to **Build** and tap it 7 times
2. Developer options will appear in Settings > System
3. Enable **USB Debugging** and **Network Debugging** (ADB over network)

**Fire TV Stick:**
1. Settings > My Fire TV > About > tap **Build** 7 times
2. Settings > My Fire TV > Developer Options
3. Enable **ADB Debugging** and **Apps from Unknown Sources**

### Step 2 — Find the TV Device's IP Address

On the TV device:
- Android TV: Settings > Network & Internet > (your network) — note the IP
- Fire TV: Settings > My Fire TV > About > Network — note the IP

### Step 3 — Connect via ADB

On your PC:
```bash
adb connect <tv-ip-address>:5555
# Example: adb connect 192.168.1.50:5555
```

Confirm connection:
```bash
adb devices
# Should show: 192.168.1.50:5555    device
```

If prompted on the TV screen to authorize the ADB connection, select **Always allow from this computer**.

### Step 4 — Download and Install SmartTube

Download the latest SmartTube APK:
- Official releases: https://github.com/yuliskov/SmartTube/releases
- Download `smarttube_stable.apk` (not the beta unless you prefer it)

Install via ADB:
```bash
adb install smarttube_stable.apk
```

### Step 5 — Launch SmartTube

SmartTube will appear in the Apps section of the TV launcher. Sign in with your Google account — it supports normal Google account login. All your subscriptions and history are accessible.

**SmartTube features:**
- 100% YouTube ad blocking (pre-roll, mid-roll, unskippable)
- SponsorBlock (skips sponsor segments, intros, outros)
- Return YouTube Dislike
- 4K / 8K / HDR playback support
- Customizable player controls designed for TV remotes

---

## Part B — All Apps: AdGuard for Android TV

AdGuard TV creates a local VPN on Android TV / Fire TV devices, adding device-level filtering on top of Phase 1 DNS blocking.

### Download

```bash
# Download AdGuard APK for Android TV to your PC first
# Official download: https://adguard.com/en/adguard-android-tv/overview.html
```

### Install via ADB

```bash
adb install adguard_tv.apk
```

### Configure on the TV

1. Open AdGuard on the TV (use your remote)
2. Enable protection
3. Grant VPN permission
4. Go to **Settings > DNS Filtering**:
   - Enable DNS filtering
   - Set custom DNS: `192.168.1.2` (your Pi)
5. Enable HTTPS filtering if prompted and install the certificate

This adds device-level filtering for Tubi, Pluto TV, and other streaming apps on the TV device.

---

## Part C — Gaming Consoles (PS5, Xbox Series)

Gaming console operating systems are fully locked. No sideloading, no custom DNS apps, no VPN tools.

**What Phase 1 DNS achieves on consoles:**
- Blocks ad tracking and telemetry
- Blocks banner/overlay ads where served from dedicated domains
- In-stream YouTube and streaming app ads will persist

**Practical recommendation:**
- For YouTube and streaming on your TV screen: connect a Chromecast GTV or Fire TV Stick to the same TV's HDMI input and use SmartTube + AdGuard TV from there instead of the console's built-in apps
- Use the console for gaming; let the Android TV stick handle streaming

---

## Part D — Smart TVs (Samsung Tizen, LG webOS, Vizio SmartCast)

Smart TV operating systems do not allow sideloading or custom app installs beyond their official stores. No ADB access is available.

**What Phase 1 DNS achieves on Smart TVs:**
- Blocks tracker/telemetry domains (~85%)
- Blocks some banner/overlay ad domains (~50–60%)
- In-stream video ads in built-in apps will persist

**Practical recommendation:**
- Use the Smart TV as a display only — connect an Android TV stick (Chromecast GTV, Fire TV Stick) to handle all streaming apps via SmartTube + AdGuard TV
- Configure the Android TV stick's DNS to `192.168.1.2` (or rely on router DHCP to hand it the Pi IP automatically)

---

## Verification

- [ ] `adb connect <tv-ip>:5555` returns `connected`
- [ ] SmartTube installed and visible in TV launcher
- [ ] SmartTube: open YouTube — videos play immediately with no ads
- [ ] AdGuard TV installed and protection enabled
- [ ] AdGuard TV DNS set to `192.168.1.2`
- [ ] AdGuard Home per-client view shows the TV device with DNS queries flowing through the Pi
- [ ] Tubi or Pluto TV on the TV device: ad breaks visibly shorter or skipped

---

## Next Step

- [Phase 6 — Verification & Maintenance](phase-6-verification.md)
