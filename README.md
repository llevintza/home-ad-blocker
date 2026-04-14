# Home Ad Blocker

> **Design Constraints — Non-Negotiable**
> - **No jailbreaking** of any device (iOS, iPadOS, tvOS, Android) is required, used, or acceptable under any circumstance
> - **No per-device configuration is required.** Devices connect to the network and are protected automatically
> - **All ad blocking is enforced at the network layer** (Netgear M70 router + Raspberry Pi 4)

Network-wide ad and tracker blocking for all home devices using a Raspberry Pi 4 running AdGuard Home. Every device on the network — iPhones, iPads, Apple TVs, Android phones, Android TVs, Smart TVs, gaming consoles, and smart home devices — is covered automatically with no individual setup.

---

## Architecture

```
Internet
    │
    ▼
Netgear M70 Mesh Router
    │  ← Firewall layer:
    │     • Blocks outbound port 853 (DNS-over-TLS)
    │     • Blocks outbound port 53 to non-Pi destinations
    │     • Blocks iCloud Private Relay IP ranges
    │     • Blocks DoH to known providers (Cloudflare, Google, Quad9)
    │     This forces ALL devices through the Pi — no device can bypass it
    ▼
Raspberry Pi 4 (4GB) — AdGuard Home DNS sinkhole
    │  Upstream: encrypted DoH to Quad9 + Cloudflare (Pi-initiated only)
    │  Blocklists: ads, trackers, SSAI ad decision servers, telemetry
    ▼
ALL devices — zero configuration required on any device
iPhone  ·  iPad  ·  Apple TV  ·  Android phone  ·  Android TV
Fire TV  ·  Chromecast  ·  Smart TV  ·  PS5  ·  Xbox  ·  Smart home
```

---

## Quick Start

1. **Immediate partial win (no hardware needed yet):** Set your Netgear router's DNS to `94.140.14.14` / `94.140.15.15` (AdGuard public DNS). Most banner ads and trackers will be blocked across all devices within minutes.
2. **Full setup:** Follow the phases in order, starting with [Phase 1](plans/phase-1-adguard-setup.md).

---

## Feasibility & Network-Layer Ceiling

### Why Brave Can Block Ads That a Router Cannot

Brave operates at the **application layer inside the device** — after TLS encryption has been terminated. It sees the plaintext of every HTTP request, matches URL patterns against blocklists, and can inject JavaScript to dismantle ad player logic before it runs.

A router sees only TLS-encrypted traffic. It can intercept DNS lookups (domain name queries) but cannot inspect the content of HTTPS streams. It cannot distinguish an ad video segment from a content video segment when both arrive as the same encrypted bytes from the same server.

### The SSAI Problem

Modern streaming platforms — Hulu, Peacock, Tubi, Prime Video, Paramount+, and others — use **Server-Side Ad Insertion (SSAI)**. The streaming service's own server stitches ad video clips directly into the content stream before sending it to the device. There is no separate ad network request to intercept — the ad and the content are the same stream.

**What this plan does about SSAI:**
Even with SSAI, the streaming backend must contact an **ad decision server** (VAST/VMAP/DAI endpoint) to determine what ad to insert. This call is a separate DNS-resolvable request. Blocking it at the DNS level causes the SSAI system to either:
- Skip the ad slot entirely (best case)
- Insert a brief black/silent segment and resume (acceptable)
- Stall waiting for a timeout before continuing (worst case — specific rules can be rolled back)

This plan blocks all known SSAI ad decision servers: Google DAI, Innovid, FreeWheel, SpotX, Springserve, Magnite, and others.

### The DNS Bypass Problem

Modern devices actively try to circumvent local DNS servers:
- **iCloud Private Relay** (iOS/iPadOS/tvOS): routes DNS through Apple's servers, bypassing the Pi entirely
- **Android Private DNS** (Android 9+): uses DoT/DoH, bypasses the Pi
- **Browser DoH** (Chrome, Firefox): uses encrypted DNS, bypasses the Pi

**Phase 3** closes all these bypass routes at the router firewall level — transparently, without touching any device.

### Success Rate (Network Layer Only — No Device Changes)

| Platform / Device | In-Stream Ads | Trackers & Banners | Notes |
|-------------------|:---:|:---:|-------|
| YouTube (any device) | ~25–35% | ~95% | SSAI ad decision server blocking |
| Tubi / Pluto TV | ~40–55% | ~95% | More CSAI-based; FreeWheel blocking effective |
| Hulu (ad-supported) | ~15–30% | ~90% | Innovid + FreeWheel blocking; stall risk — monitor |
| Peacock (free) | ~20–35% | ~90% | FreeWheel + SpotX blocking |
| Prime Video (ad tier) | ~10–20% | ~90% | Amazon own SSAI — hardest to disrupt |
| Paramount+ | ~25–40% | ~90% | Mix of FreeWheel + Google DAI |
| Apple TV (tvOS) | ~25–35% | ~90–95% | Same network ceiling as all devices |
| Banner / overlay ads (all devices) | ~85–95% | — | DNS blocking highly effective |
| Mobile in-app ads (games / free apps) | ~70–85% | ~95% | Ad SDK domains blocked |
| Android TV / Fire TV / Chromecast | ~25–40% | ~90–95% | DNS-only; no device changes |
| Smart TVs (Samsung, LG, Vizio) | ~30–50% | ~85–90% | DNS-only |
| Gaming consoles (PS5, Xbox) | ~25–35% | ~85% | DNS-only |
| Smart home / IoT devices | N/A | ~90–95% | Highly effective for telemetry blocking |
| Desktop browser ads | ~85–95% | ~99% | DNS + optional uBlock Origin extension |

### Honest Bottom Line

This solution will:
- **Nearly eliminate** tracking, telemetry, banner ads, and mobile in-app ads across every device automatically
- **Significantly disrupt** in-stream ads on Tubi and Pluto TV (~40–55%)
- **Partially disrupt** in-stream ads on Hulu, Peacock, and Paramount+ (~20–35%)
- **Minimally disrupt** Prime Video in-stream ads (~10–20%) — Amazon controls the full SSAI stack
- **Apply equally to all devices** — Apple TV, iPhones, gaming consoles, and every other device get the same treatment with zero per-device configuration

> For streaming platforms where in-stream ads persist despite this solution, the only 100% alternative is subscribing to their paid ad-free tier. No network-level solution can fully replicate application-layer blocking (like Brave) without performing HTTPS inspection, which requires per-device certificate trust — contradicting the no-device-change constraint.

---

## Implementation Phases

| Phase | Description | Plan File | GitHub Issue |
|-------|-------------|-----------|-------------|
| 1 | Raspberry Pi 4 setup + AdGuard Home installation | [phase-1-adguard-setup.md](plans/phase-1-adguard-setup.md) | [#1](../../issues/1) |
| 2 | Blocklist + SSAI ad decision server configuration | [phase-2-blocklists.md](plans/phase-2-blocklists.md) | [#2](../../issues/2) |
| 3 | Encrypted DNS bypass prevention (DoH/DoT/Private Relay) | [phase-3-dns-bypass-prevention.md](plans/phase-3-dns-bypass-prevention.md) | [#3](../../issues/3) |
| 4 | Apple device coverage (iPhone, iPad, Apple TV) | [phase-4-apple-devices.md](plans/phase-4-apple-devices.md) | [#4](../../issues/4) |
| 5 | TV, console & smart home coverage | [phase-5-tv-consoles.md](plans/phase-5-tv-consoles.md) | [#5](../../issues/5) |
| 6 | Verification, testing, and maintenance guide | [phase-6-verification.md](plans/phase-6-verification.md) | [#6](../../issues/6) |

Phases 4 and 5 require no device actions — they are coverage documentation and verification only. Run them after Phase 3 is complete.
