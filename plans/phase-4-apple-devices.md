# Phase 4 — Apple Device Coverage (iPhone, iPad, Apple TV)

**GitHub Issue:** #4  
**Dependencies:** Phases 1, 2, and 3 complete  
**Device action required:** None  
**Applies to:** iPhone, iPad, Apple TV  

> ⚠️ **Constraints**
> - No jailbreaking of any Apple device — iOS, iPadOS, or tvOS — is required, used, or acceptable
> - No per-device app installation or configuration of any kind
> - All filtering is applied at the network layer only
> - Apple TV is a first-class citizen of this solution — it is fully covered with zero device interaction

---

## How Apple Devices Are Covered

All Apple devices (iPhone, iPad, Apple TV) receive ad and tracker blocking automatically through the network stack established in Phases 1–3:

1. **Phase 1** configured the Netgear M70 router to hand out the Pi's IP (`192.168.1.2`) as the DNS server to all devices via DHCP
2. **Phase 2** configured the Pi to block ad networks, trackers, and SSAI ad decision servers at the DNS level
3. **Phase 3** blocks iCloud Private Relay and DNS-over-TLS so Apple devices cannot route around the Pi

When an Apple device connects to the home network WiFi, it receives the Pi as its DNS server automatically. No configuration is needed on the device. This applies equally to:
- iPhone (iOS)
- iPad (iPadOS)
- Apple TV (tvOS) — including 4K models
- HomePod, HomePod mini
- Mac computers (macOS) — also covered automatically

---

## Apple TV — Explicit Coverage

Apple TV runs **tvOS**, Apple's locked television platform. It is treated identically to an iPhone at the network layer:

- No sideloading of apps is possible on tvOS (this is not required by this solution)
- No jailbreaking is needed or used
- Apple TV receives the Pi IP as its DNS server via DHCP — automatic, zero interaction
- iCloud Private Relay (if enabled on the associated Apple ID) is blocked at the network level by Phase 3
- All DNS queries from Apple TV flow through AdGuard Home, applying all blocklists

**Apple TV is fully protected with absolutely no action taken on the device.**

---

## What the Network Layer Achieves on Apple Devices

| Ad / Tracker Type | Coverage | Notes |
|-------------------|:--------:|-------|
| Tracking pixels, telemetry, analytics | ~90–95% | DNS-level, highly effective |
| Banner and overlay ads (apps and websites) | ~85–95% | DNS-level |
| YouTube in-stream ads (YouTube app) | ~25–35% | SSAI ad decision server blocking; Phase 2 SSAI rules apply |
| Hulu in-stream ads | ~15–30% | Innovid + FreeWheel blocking; may cause brief black segment instead of ad |
| Peacock in-stream ads | ~20–35% | FreeWheel + SpotX blocking |
| Prime Video in-stream ads | ~10–20% | Amazon SSAI — hardest to disrupt via DNS |
| Tubi / Pluto TV in-stream ads | ~40–55% | More CSAI-based — effective with FreeWheel blocking |
| In-app ads (games, free apps) | ~70–85% | Ad SDK network domains blocked |

These rates apply equally to iPhones, iPads, and Apple TV.

---

## tvOS Platform Context

Apple TV runs a locked operating system. The constraints of tvOS are:

- No App Store alternatives exist for YouTube that block ads (no SmartTube equivalent for tvOS)
- No user-installable DNS filtering apps (no AdGuard equivalent for all-app filtering on tvOS)
- The native YouTube app on Apple TV will show some ads — the network layer handles what it can
- The native streaming apps (Hulu, Peacock, etc.) on Apple TV will have in-stream ads partially disrupted as described in the table above

**This is not a limitation specific to this solution** — it applies to any network-level approach. The ceiling for tvOS in-stream ad blocking is the same as for any other locked platform (gaming consoles, Smart TVs). No jailbreak is needed, used, or would be permitted even if it could increase effectiveness.

---

## iCloud Private Relay Context

If an Apple ID on the network has iCloud Private Relay enabled (requires iCloud+ subscription), the device will attempt to route DNS and web traffic through Apple's relay servers, bypassing the Pi.

Phase 3 blocks this at the router level by:
- Blocking `mask.icloud.com`, `mask-h2.icloud.com`, and related Private Relay domains in AdGuard Home
- Optionally blocking Apple's Private Relay IP ranges at the router firewall

When Private Relay is blocked, the device falls back to standard DNS (the Pi). The user sees a notification in Settings > Apple ID > iCloud > Private Relay that Private Relay is "not available on this network" — this is expected and correct behavior. No device action is needed; the network enforces this automatically.

---

## Verification

This phase requires no action — only confirmation that Phases 1–3 have taken effect on Apple devices.

In AdGuard Home (`http://192.168.1.2`) → **Query Log**:
- [ ] Filter by the Apple TV's IP — DNS queries should be visible (confirms Phase 3 worked)
- [ ] Filter by an iPhone's IP — DNS queries should be visible
- [ ] Confirm blocked entries (in red) appear for ad/tracker domains

On an Apple TV:
- [ ] Open any app and browse — tracker/banner ads should be reduced
- [ ] Open a streaming app (Tubi, Pluto TV) — ad breaks may be shorter or show brief black segments (indicates SSAI ad decision server was blocked)

---

## Next Step

- [Phase 5 — TV, Console & Smart Home Coverage](phase-5-tv-consoles.md)
