# Phase 4 — iOS / iPadOS Ad Blocking (Layer 2)

**GitHub Issue:** #4  
**Dependencies:** Phase 1 complete  
**Applies to:** iPhones, iPads  
**Can run in parallel with:** Phase 3, Phase 5  

---

## Goal

Supplement network-level DNS blocking (Phase 1–2) with device-level solutions on iOS/iPadOS. Apple's App Store policies impose hard limits on what is achievable — this phase delivers the maximum possible ad reduction within those constraints.

---

## iOS Limitations — Important Context

Apple's App Store policies prevent apps from:
- Intercepting and modifying traffic from other apps (no app-level VPN filtering like AdGuard Android)
- Distributing modified versions of existing apps (no ReVanced equivalent)
- Installing custom CA certificates silently (users must manually trust them)

As a result:
- No full YouTube app-level ad blocking is available on iOS without jailbreaking
- HTTPS filtering of other apps is not possible at the granularity available on Android
- The practical ceiling for iOS is ~40–50% YouTube ad reduction and ~85–90% for other ad types

---

## Part A — AdGuard Pro for iOS

AdGuard Pro is the most effective ad blocking tool available on iOS within App Store rules. It uses a local DNS proxy (not a full VPN) to filter DNS queries on the device.

### Installation

1. Open the App Store on the iPhone/iPad
2. Search for **AdGuard — ad blocker & privacy** (by Adguard Software Limited)
3. Purchase **AdGuard Pro** (~$2.99 one-time or subscription) — the free version does not include DNS filtering for all apps

> Alternative: **AdGuard** (free version) covers Safari only. AdGuard Pro covers all apps via DNS filtering.

### Configuration

1. Open AdGuard Pro
2. Tap **Protection** and enable it
3. Go to **Settings > DNS Protection**:
   - Enable **DNS Protection**
   - Tap **DNS Server** > **Add Custom DNS Server**
   - Name: `Home Pi`
   - Server: `192.168.1.2`
   - Tap **Save** and select it as the active server
4. Go to **Filters** (under DNS Protection) and enable:
   - AdGuard DNS filter
   - AdGuard Mobile Ads
   - EasyPrivacy
   - OISD Full

5. Under **Safari Protection** (separate from DNS):
   - Enable content blockers for Safari
   - Enable all available AdGuard filter categories

### What This Achieves on iOS

- All DNS-level blocking flows through your Pi first, then AdGuard Pro adds local DNS filtering on top
- Safari ad blocking via content blocker extension (very effective for web browsing)
- Partial reduction of in-app ads via DNS filtering
- YouTube app ads: reduced but not eliminated (~40–50%)

---

## Part B — YouTube on iOS

No full YouTube ad-blocking app exists on iOS without jailbreaking. The best available options, in order of effectiveness:

| Option | Effectiveness | Notes |
|--------|-------------|-------|
| AdGuard Pro DNS filtering (Part A above) | ~40–50% | Reduces but doesn't eliminate |
| Accepting limitations on iOS | — | For full YouTube ad blocking, use an Android device or Android TV stick |

If YouTube ad-free viewing is critical on an Apple device, the practical solution is to use an Apple TV with a connected Android TV stick (Chromecast GTV or Fire TV Stick) for YouTube playback via SmartTube — see [Phase 5](phase-5-layer2-tv.md).

---

## Verification

- [ ] AdGuard Pro shows DNS Protection active
- [ ] AdGuard Pro shows custom DNS server `192.168.1.2` selected
- [ ] AdGuard Home web UI (`http://192.168.1.2`) shows the iOS device in per-client stats
- [ ] Open Safari and browse a news site — banner ads absent
- [ ] Open a free game — in-app banner ads reduced (not all will be blocked)
- [ ] YouTube app: ads may still appear — this is expected on iOS

---

## Next Steps

- [Phase 3 — Android](phase-3-layer2-android.md) (if not done)
- [Phase 5 — TV devices](phase-5-layer2-tv.md)
- [Phase 6 — Verification](phase-6-verification.md)
