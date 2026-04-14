# Phase 3 — Android Ad Blocking (Layer 2)

**GitHub Issue:** #3  
**Dependencies:** Phase 1 complete  
**Applies to:** Android phones, tablets  
**Can run in parallel with:** Phase 4, Phase 5  

---

## Goal

Supplement network-level DNS blocking (Phase 1–2) with device-level solutions on Android to achieve full YouTube ad blocking and improve in-app ad blocking for all other applications.

---

## Part A — YouTube: ReVanced

ReVanced is a patched version of the official YouTube app that blocks all ad types: pre-roll, mid-roll, banner, end-screen, and unskippable ads. It installs as a separate app alongside the official YouTube app.

### Installation

1. On the Android device, enable **Install from unknown sources** for your browser:
   - Settings > Apps > (your browser) > Install unknown apps > Allow

2. Download **ReVanced Manager** APK from the official site:
   - URL: https://revanced.app (official download link is on the site)
   - Install the APK

3. Open ReVanced Manager:
   - Tap **Patcher > Select an application**
   - Choose **YouTube** (it will auto-detect the installed version)
   - If YouTube is not installed or the version is incompatible, the app will prompt you to download a compatible version via APKMirror

4. Select patches:
   - Use the **Default** patch set (includes ad blocking, SponsorBlock, return YouTube dislike, etc.)
   - Tap **Patch**

5. When patching is complete, tap **Install** — YouTube ReVanced installs as `YouTube ReVanced` alongside the original app

6. Sign in with your Google account — ReVanced supports Google sign-in normally

### What ReVanced Blocks

- All pre-roll and mid-roll video ads
- Unskippable 15/30 second ads
- Banner and overlay ads within videos
- Promoted videos in search results (configurable)
- SponsorBlock integration: also skips in-video sponsor segments, intros, outros

### Maintenance

YouTube updates push regularly. When the official YouTube app updates, ReVanced may stop working or prompt a re-patch. To update:
1. Open ReVanced Manager
2. Tap **Patcher** — it will detect the new YouTube version
3. Re-patch and reinstall (takes ~5 minutes)

ReVanced Manager can also auto-check for updates.

---

## Part B — All Apps: AdGuard for Android

AdGuard for Android creates a local VPN that filters all app traffic on the device itself — adding a device-level layer on top of the network-level DNS (Phase 1). It catches ad requests that slip past DNS (e.g., ads served over HTTPS from domains shared with legitimate content).

### Installation

AdGuard is **not available on the Google Play Store** (Google's policy). Install directly:

1. On the device, go to: https://adguard.com/en/adguard-android/overview.html
2. Download the APK
3. Install (allow from browser if prompted)

### Configuration

1. Open AdGuard > tap **Enable Protection**
2. Grant VPN permission when prompted
3. Go to **Settings > DNS Filtering**:
   - Enable DNS filtering
   - Set DNS server to your Pi: **Custom** → `192.168.1.2`
   - This makes AdGuard use your Pi for DNS (Phase 1 + AdGuard work together)
4. Go to **Settings > Filtering**:
   - Enable HTTPS filtering (required to block ads in HTTPS traffic)
   - Install the AdGuard certificate when prompted (Settings will guide you)
5. Under **Filters**, enable:
   - AdGuard Base filter
   - AdGuard Mobile Ads filter
   - AdGuard Tracking Protection filter
   - EasyList
   - EasyPrivacy

### Important: AdGuard + Pi DNS Coexistence

AdGuard's local VPN handles DNS on the device. To ensure it uses your Pi rather than overriding it:
- In AdGuard > Settings > DNS Filtering > DNS server: set to `192.168.1.2` (your Pi)
- This means: AdGuard intercepts traffic AND uses Pi-hole for DNS lookups — both layers active simultaneously

### Streaming App Coverage

With AdGuard for Android + Pi DNS active:
- Tubi / Pluto TV: ~55–65% in-stream ad blocking
- Hulu: ~15–25% (SSAI limit)
- Prime Video: ~15–25% (SSAI limit)
- All non-SSAI apps: ~85–95%

---

## Verification

- [ ] Open YouTube ReVanced — videos play immediately with no ads
- [ ] Open a news website in Chrome — banner ads absent
- [ ] Open Tubi — ad breaks visibly shorter or skipped
- [ ] AdGuard dashboard shows active filtering and request count growing
- [ ] AdGuard DNS setting shows `192.168.1.2` (Pi)
- [ ] AdGuard Home web UI (`http://192.168.1.2`) shows the device in per-client stats

---

## Next Steps

- [Phase 4 — iOS/iPadOS](phase-4-layer2-apple.md)
- [Phase 5 — TV devices](phase-5-layer2-tv.md)
- [Phase 6 — Verification](phase-6-verification.md)
