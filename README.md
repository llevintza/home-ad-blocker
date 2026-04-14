# Home Ad Blocker

Network-wide ad and tracker blocking for all home devices using a Raspberry Pi 4 running AdGuard Home, with per-device supplementary solutions where DNS alone cannot reach.

## Architecture

```
Internet
    │
    ▼
Netgear M70 Mesh Router
    │  (DNS handed to all clients via DHCP → Pi IP)
    ▼
Raspberry Pi 4 (4GB) — AdGuard Home DNS sinkhole
    │  (upstream: encrypted DoH to Quad9 + Cloudflare)
    ▼
All devices on network (phones, tablets, TVs, consoles, smart home)
```

Devices that need deeper YouTube/streaming ad blocking also run client-side solutions (Layer 2) — see the phase plans below.

---

## Quick Start

1. **Immediate win (no hardware needed):** Set your Netgear router's DNS to `94.140.14.14` / `94.140.15.15` (AdGuard public DNS). Most banner ads and trackers will be blocked across all devices within minutes.
2. **Full setup:** Follow the phases in order, starting with [Phase 1](plans/phase-1-adguard-setup.md).

---

## Feasibility & Success Rate

### Why Brave Can Block Ads That a Router Cannot

Brave operates at the **application layer** inside the device — it inspects every HTTP/HTTPS request, matches URL patterns against blocklists, and can inject JavaScript to dismantle ad player logic before it runs.

A router or DNS server operates at a fundamentally different level. It can only intercept domain name lookups (DNS queries). If an ad is served from a dedicated domain (e.g., `ads.example.com`), blocking that DNS query prevents the device from finding the ad server. But it cannot inspect the content of encrypted HTTPS traffic, match URL path patterns within a domain, or intercept ads embedded directly into the video stream.

### The SSAI Problem (Server-Side Ad Insertion)

Modern premium streaming platforms — **Hulu, Peacock, Tubi, Prime Video, Paramount+** — use **Server-Side Ad Insertion (SSAI)**. Instead of the app making a separate network request to an ad server (which DNS could block), the streaming service's own server stitches the ad video clip directly into the content stream before sending it to your device.

Your device receives one seamless HLS/DASH video stream. There is no separate "ad request" to intercept — the ad and the content arrive as the same bytes from the same server. This is why ad blockers began failing on Hulu and similar services around 2019–2021; the platforms specifically engineered SSAI to defeat DNS and browser-level blocking.

### Success Rate by Platform

| Platform | In-Stream Ads (DNS only) | In-Stream Ads (DNS + Layer 2) | Trackers / Banners |
|----------|:---:|:---:|:---:|
| YouTube (Android) | ~25–35% | **~100%** (ReVanced) | ~95% |
| YouTube (Android TV / Fire TV) | ~25–35% | **~100%** (SmartTube) | ~95% |
| YouTube (iOS / iPadOS) | ~25–35% | ~40–50% (iOS ceiling) | ~95% |
| YouTube (gaming console) | ~25–35% | ~25–35% (no Layer 2 available) | ~90% |
| Tubi / Pluto TV | ~40–55% | ~55–65% | ~95% |
| Hulu (ad-supported) | ~10–20% | ~15–25% | ~90% |
| Prime Video (ad tier) | ~10–20% | ~15–25% | ~90% |
| Peacock (free tier) | ~20–35% | ~25–40% | ~90% |
| Paramount+ | ~25–40% | ~30–45% | ~90% |
| Banner / overlay ads (all apps) | ~85–95% | ~90–98% | — |
| Mobile in-app ads (games / free apps) | ~70–85% | ~80–90% | ~95% |
| Smart TV native apps (Samsung, LG) | ~30–50% | ~30–50% (locked OS) | ~85% |
| Desktop browser ads | ~85–95% | ~95–99% (+ uBlock Origin) | ~99% |

### Honest Bottom Line

This solution will:
- **Nearly eliminate** tracking, telemetry, banner ads, and mobile in-app ads across all devices
- **Partially disrupt** in-stream ads on Tubi and Pluto TV (~40–55% with DNS alone)
- **Minimally impact** in-stream ads on Hulu, Prime Video, and Peacock (~10–20%) — SSAI is the hard limit
- **Fully block** YouTube ads on Android and Android TV with per-device apps (ReVanced / SmartTube)

> The only 100% solution for Hulu, Peacock, and Prime Video in-stream ads is subscribing to their paid ad-free tiers.

---

## Implementation Phases

| Phase | Description | Plan File | GitHub Issue |
|-------|-------------|-----------|-------------|
| 0 | Repository setup | *(this file)* | — |
| 1 | Raspberry Pi 4 setup + AdGuard Home installation | [phase-1-adguard-setup.md](plans/phase-1-adguard-setup.md) | [#1](../../issues/1) |
| 2 | AdGuard Home blocklist + custom rule configuration | [phase-2-blocklists.md](plans/phase-2-blocklists.md) | [#2](../../issues/2) |
| 3 | Android ad blocking — ReVanced + AdGuard app | [phase-3-layer2-android.md](plans/phase-3-layer2-android.md) | [#3](../../issues/3) |
| 4 | iOS / iPadOS ad blocking — AdGuard Pro | [phase-4-layer2-apple.md](plans/phase-4-layer2-apple.md) | [#4](../../issues/4) |
| 5 | TV ad blocking — SmartTube + AdGuard TV via ADB | [phase-5-layer2-tv.md](plans/phase-5-layer2-tv.md) | [#5](../../issues/5) |
| 6 | Verification, testing, and maintenance guide | [phase-6-verification.md](plans/phase-6-verification.md) | [#6](../../issues/6) |

Phases 3, 4, and 5 are independent and can be executed in parallel.
