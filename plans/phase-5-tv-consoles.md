# Phase 5 — TV, Console & Smart Home Device Coverage

**GitHub Issue:** #5  
**Dependencies:** Phases 1, 2, and 3 complete  
**Device action required:** None  
**Applies to:** Android TV, Chromecast, Fire TV, Smart TVs, PS5, Xbox, smart home devices  

> ⚠️ **Constraints**
> - No jailbreaking of any device — iOS, iPadOS, tvOS, or Android — is required, used, or acceptable
> - No sideloading of apps, no ADB connections, no per-device configuration of any kind
> - All filtering is applied at the network layer only
> - Every device class listed here is covered automatically upon connecting to the network

---

## How All Devices Are Covered

Every device in this phase receives ad and tracker blocking automatically through Phases 1–3:

1. The Netgear M70 router hands out the Pi IP (`192.168.1.2`) as DNS to all devices via DHCP
2. AdGuard Home on the Pi blocks ad networks, trackers, and SSAI ad decision servers at DNS level
3. Router firewall rules (Phase 3) block outbound port 853 (DoT) and known DoH provider IPs — Android's Private DNS feature cannot bypass the Pi

No device needs to be opened, configured, modified, or connected to via ADB. Connecting to the WiFi is the only user action.

---

## Device-by-Device Coverage

### Android TV / Chromecast with Google TV / NVIDIA Shield

**How DNS is enforced:** DHCP assigns Pi DNS automatically. Phase 3 blocks Android's Private DNS (port 853), preventing bypass.

**Coverage:**

| Ad Type | Rate | Notes |
|---------|:----:|-------|
| Trackers / telemetry | ~90–95% | DNS-level |
| Banner / overlay ads in apps | ~85–95% | DNS-level |
| YouTube in-stream ads | ~25–35% | SSAI ad decision server blocking (Phase 2) |
| Tubi / Pluto TV in-stream ads | ~40–55% | FreeWheel + Springserve blocking |
| Hulu in-stream ads | ~15–30% | Innovid + FreeWheel blocking |
| Peacock in-stream ads | ~20–35% | FreeWheel + SpotX blocking |
| Prime Video in-stream ads | ~10–20% | Amazon own SSAI stack |

**Android Private DNS note:** If a user has previously configured Private DNS on a device (Settings > Network > Private DNS), Phase 3 blocks the DoT connection so Android falls back to DHCP-assigned DNS (the Pi). No action is needed on the device.

---

### Amazon Fire TV Stick / Fire TV Cube

**How DNS is enforced:** DHCP assigns Pi DNS. Fire TV does not use Private DNS — no additional bypass concern.

**Coverage:** Identical to Android TV table above. Amazon Prime Video ads are the hardest to disrupt because Amazon controls its own SSAI stack end-to-end.

---

### Apple TV (tvOS)

Covered fully in [Phase 4](phase-4-apple-devices.md). Listed here for completeness.

**Summary:** Automatic via DHCP + Phase 3 iCloud Private Relay blocking. No device interaction required.

---

### Samsung Smart TVs (Tizen OS)

**How DNS is enforced:** DHCP assigns Pi DNS. Tizen has no DoH/DoT implementation — standard DNS only.

**Coverage:**

| Ad Type | Rate | Notes |
|---------|:----:|-------|
| Trackers / telemetry | ~85–90% | DNS-level |
| Banner / overlay ads | ~50–65% | DNS-level — some Tizen ads are served locally |
| In-app YouTube ads | ~25–35% | SSAI ad decision server blocking |
| Streaming app in-stream ads | ~20–40% | FreeWheel/Innovid blocking where applicable |

Samsung Tizen TVs have no mechanism to bypass DHCP-assigned DNS. They are among the more cooperative devices for DNS-level filtering.

---

### LG Smart TVs (webOS)

**How DNS is enforced:** DHCP assigns Pi DNS. webOS uses standard DNS.

**Coverage:** Similar to Samsung Tizen. Some LG TVs use their own ad network (`lgad.lge.com`, `lgappstv.com`) — these are included in the hagezi blocklists added in Phase 2.

---

### Vizio SmartCast / Roku TVs / Other Smart TVs

**How DNS is enforced:** DHCP assigns Pi DNS. These platforms do not use DoH/DoT.

**Coverage:**

| Ad Type | Rate | Notes |
|---------|:----:|-------|
| Trackers / telemetry | ~85–95% | DNS-level |
| ACR (automatic content recognition) | ~80–90% | Significant privacy gain — these TVs fingerprint viewing habits |
| Banner / overlay ads | ~40–60% | DNS-level |
| In-stream ads | ~20–40% | Platform dependent |

**Note on Roku:** Roku has its own ad platform and some ads are served through Roku's own servers. Blocking `ads.roku.com` and `cooper.roku.com` disrupts many of these. These domains are included in the hagezi blocklists.

---

### PlayStation 5 (PS5) / PlayStation 4

**How DNS is enforced:** DHCP assigns Pi DNS. PlayStation does not use DoH/DoT — straightforward DNS.

**Coverage:**

| Ad Type | Rate | Notes |
|---------|:----:|-------|
| Trackers / telemetry | ~85% | DNS-level |
| YouTube app in-stream ads | ~25–35% | SSAI blocking |
| Streaming app ads | ~20–40% | Platform dependent |
| PlayStation Store promotional interstitials | ~30–50% | Some served from blockable domains |

---

### Xbox Series X/S / Xbox One

**How DNS is enforced:** DHCP assigns Pi DNS. Xbox does not use DoH/DoT.

**Coverage:** Similar to PS5. Xbox's ad ecosystem uses Microsoft/Xandr ad infrastructure — `xandr.com` and related domains are included in the EasyList blocklist added in Phase 2.

---

### Smart Home / IoT Devices (Nest, Ring, Alexa, Smart bulbs, etc.)

**How DNS is enforced:** DHCP assigns Pi DNS. IoT devices use standard DNS — no bypass mechanisms.

**Coverage:**

| Ad Type | Rate | Notes |
|---------|:----:|-------|
| Telemetry / data collection | ~90–95% | DNS-level — significant privacy gain |
| Alexa ad insertion | ~60–75% | Some Amazon ad domains blockable |
| Third-party tracker SDKs | ~85–95% | DNS-level |

IoT devices are among the biggest beneficiaries of DNS-level filtering because they constantly phone home to telemetry servers. This phase passively blocks most of that background data collection with no action required.

---

## Consolidated Success Rate Summary

| Device | In-Stream Video Ads | Trackers & Banners |
|--------|:-------------------:|:-----------------:|
| Android TV / Chromecast GTV | ~25–40% | ~90–95% |
| Amazon Fire TV | ~25–40% | ~90–95% |
| Apple TV (tvOS) | ~25–35% | ~90–95% |
| Samsung Smart TV (Tizen) | ~25–40% | ~85–90% |
| LG Smart TV (webOS) | ~25–40% | ~85–90% |
| Vizio / Roku | ~20–40% | ~85–95% |
| PS5 / PS4 | ~25–35% | ~85% |
| Xbox Series / One | ~25–35% | ~85% |
| Smart home / IoT | N/A | ~90–95% |

These rates reflect network-layer-only filtering with zero device changes. No device in this table requires modification, jailbreaking, sideloading, or individual configuration.

---

## Verification

No device action is required. Confirm coverage via AdGuard Home:

- [ ] In AdGuard Home → **Client Statistics**: all TV and console devices appear with query activity
- [ ] In AdGuard Home → **Query Log**: filter by a Smart TV's IP — tracker domains (LG/Samsung analytics, ACR) appear as blocked
- [ ] Test on a streaming device: open Tubi or Pluto TV and observe shortened or skipped ad breaks

---

## Next Step

- [Phase 6 — Verification & Maintenance](phase-6-verification.md)
