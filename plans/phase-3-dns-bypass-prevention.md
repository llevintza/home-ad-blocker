# Phase 3 — Encrypted DNS Bypass Prevention

**GitHub Issue:** #3  
**Dependencies:** Phase 1 complete  
**Applies to:** All devices automatically — no per-device action required  
**Must complete before:** Phase 4, Phase 5  

> ⚠️ **Constraints**
> - No jailbreaking of any device (iOS, iPadOS, tvOS, Android)
> - No per-device app installation or configuration
> - All filtering is applied at the network layer only

> **Router compatibility:** These steps apply to any home router with firewall/port blocking capabilities. The solution is not tied to a specific brand or model. Where menu paths differ by brand, multiple examples are provided.

---

## Why This Phase Is Critical

Without this phase, the Raspberry Pi DNS sinkhole is **invisible to modern devices**. Here is why:

| Device / Feature | Bypass Mechanism | Effect Without This Phase |
|-----------------|-----------------|--------------------------|
| iPhones, iPads, Apple TV (iOS/tvOS) | **iCloud Private Relay** — routes DNS through Apple's servers | Pi DNS completely bypassed; zero filtering applied |
| Android 9+ phones and tablets | **Private DNS** (DoT) — hardcoded in Android settings | If set, Pi DNS is bypassed entirely |
| Chrome browser | **DNS-over-HTTPS** — built-in, enabled by default in some regions | DNS queries go to Google, not the Pi |
| Firefox browser | **DNS-over-HTTPS** — Cloudflare by default | DNS queries bypass the Pi |
| Any device / app | **Hardcoded DNS** — some apps hardcode `8.8.8.8` directly | Bypasses router-assigned DNS |

This phase closes all these bypass routes at the **router firewall and DNS level** — no device is touched.

---

## How It Works

The router is configured to:
1. **Block port 853** (DNS-over-TLS) outbound — devices cannot establish encrypted DoT tunnels to bypass the Pi
2. **Block DoH provider IPs on port 443** — prevents browser/app DoH from bypassing the Pi
3. **Block iCloud Private Relay domains** (in AdGuard Home) — Apple devices fall back to standard DNS (the Pi)
4. **Optionally redirect all port 53** outbound to the Pi — catches any device with a hardcoded DNS server IP

The Pi's own upstream DoH (to Quad9 and Cloudflare) is not affected — that traffic originates *from the Pi*, not from client devices.

---

## Step 1 — Block DNS-over-TLS (Port 853) on the Router

DNS-over-TLS runs on TCP/UDP port 853. Blocking this port forces any device that tries to use DoT (including Android's Private DNS feature) to fall back to the DHCP-assigned DNS server (the Pi).

**Where to configure outbound port blocking by router brand:**

| Router Brand | Menu Path |
|-------------|-----------|
| Netgear (Orbi/Nighthawk) | Advanced > Security > Block Services > Add > Custom (TCP+UDP, port 853) |
| TP-Link | Advanced > Security > Access Control (add rule for TCP+UDP port 853, direction: outbound) |
| Asus | Firewall > Network Services Filter > add TCP+UDP port 853 outbound |
| Ubiquiti UniFi | Network > Settings > Firewall > LAN Out > add rule blocking port 853 |
| pfSense / OPNsense | Firewall > Rules > LAN > add block rule for destination port 853 |
| Eero / Google Nest WiFi | These platforms have limited port blocking; skip to Step 2 and rely on iCloud Private Relay blocking and Pi DHCP DNS |

**Generic configuration:**
- Protocol: TCP + UDP
- Port: 853
- Direction: Outbound (LAN to WAN)
- Action: Block / Deny
- Apply to: All devices (entire LAN subnet)

---

## Step 2 — Block Known DoH Provider IPs on Port 443

DNS-over-HTTPS uses standard HTTPS port 443 to known IP addresses. Block outbound connections to these specific IPs:

| Provider | IPv4 to Block |
|----------|--------------|
| Cloudflare | `1.1.1.1`, `1.0.0.1` |
| Google | `8.8.8.8`, `8.8.4.4` |
| Quad9 | `9.9.9.9`, `149.112.112.112` |
| OpenDNS | `208.67.222.222`, `208.67.220.220` |
| AdGuard DNS | `94.140.14.14`, `94.140.15.15` |

Add these as outbound IP+port block rules in your router's firewall:
- Destination IP: each address above
- Destination port: 443
- Protocol: TCP
- Direction: Outbound
- Action: Block

> **Important:** This blocks HTTPS *to those specific IPs only*. Regular HTTPS traffic to websites (netflix.com, google.com, etc.) uses different IPs and is completely unaffected.

> **On Eero/Nest WiFi:** These routers may not support IP-level firewall rules. In this case, rely on Step 3 (AdGuard custom rules) and accept that browser DoH may partially bypass filtering — browser-level bypass is less impactful than device-level bypass.

---

## Step 3 — Block iCloud Private Relay (AdGuard Home Custom Rules)

iCloud Private Relay routes all DNS and web traffic through Apple's relay servers, making the Pi invisible to iPhones, iPads, and Apple TVs.

In AdGuard Home web UI → **Filters > Custom Filtering Rules**, add:

```
# iCloud Private Relay — blocking forces Apple devices back to Pi DNS
||mask.icloud.com^
||mask-h2.icloud.com^
||mask-api.icloud.com^
||doh.dns.apple.com^
||dnsstar.apple.com^
||applovin.apple.com^
```

Click **Save**.

When these domains are blocked, Apple devices display "Private Relay is not available on this network" in Settings and fall back to standard DNS (the Pi). **No action is required on any Apple device** — the network enforces this automatically.

---

## Step 4 — Optionally Redirect All Port 53 Traffic to the Pi

Some apps and smart home devices bypass DHCP-assigned DNS by hardcoding a DNS IP (e.g., always querying `8.8.8.8:53`). A NAT redirect rule intercepts any outbound DNS query and sends it to the Pi instead.

**Support varies by router:**

| Router | Port 53 Redirect Support | Method |
|--------|------------------------|--------|
| Netgear Orbi/Nighthawk | Limited — not all firmware versions | Port Forwarding may partially work: Advanced > Setup > Port Forwarding, map external port 53 → Pi `192.168.1.2:53` |
| TP-Link (Archer) | Partial | Virtual Server: port 53 → `192.168.1.2` |
| Asus | Via firewall script | Advanced > Firewall > network services |
| Ubiquiti UniFi | Full support | Firewall rule: NAT DNAT any destination port 53 → `192.168.1.2:53` |
| pfSense / OPNsense | Full support | NAT > Port Forward: LAN, TCP/UDP, dst port 53, redirect to `192.168.1.2:53` |
| Eero / Google Nest WiFi | Not supported | Not available on these platforms |

If your router does not support this, the solution still works for all DHCP-aware devices (the vast majority). Hardcoded DNS in smart home/IoT devices is a less common scenario and is partially mitigated by blocklists.

---

## Step 5 — Verify Android Private DNS Is Overridden

Android 9+ has a **Private DNS** setting (Settings > Network & Internet > Advanced > Private DNS). When set to "Automatic" or a hostname, Android uses DoT, which bypasses the Pi.

Step 1 (blocking port 853) handles this automatically: when Android's DoT connection fails, it falls back to the DHCP-assigned DNS server (the Pi). No device changes are needed.

---

## Verification Checklist

```bash
# From a client device on the network:

# 1. Port 853 is blocked outbound (should time out or be connection refused)
nc -vz 1.1.1.1 853
# Expected: connection refused or timeout (NOT "Connected")

# 2. Pi's own upstream DoH still works (run this from the Pi via SSH)
curl -s -o /dev/null -w "%{http_code}" \
  "https://dns10.quad9.net/dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB" \
  -H "accept: application/dns-message"
# Expected: 200

# 3. iCloud Private Relay domains are blocked
nslookup mask.icloud.com 192.168.1.2
# Expected: 0.0.0.0 (blocked)
```

In AdGuard Home → **Client Statistics** (or Query Log):
- [ ] iPhone IP appears with DNS query count > 0 → Private Relay blocked, Pi DNS active
- [ ] iPad IP appears with DNS query count > 0
- [ ] Apple TV IP appears with DNS query count > 0
- [ ] Android phone/tablet IP appears → Private DNS (port 853) is overridden

On an iPhone connected to the home network:
- [ ] Settings > Apple ID > iCloud > Private Relay shows **"Private Relay is not available on this network"**

---

## Expected Outcome

After this phase, every device — regardless of its OS, DNS settings, or Apple/Android privacy features — uses the Pi as its DNS resolver. No user action is needed on any device. The network enforces DNS routing transparently.

---

## Next Steps

- [Phase 4 — Apple Device Coverage](phase-4-apple-devices.md)
- [Phase 5 — TV, Console & Smart Home Coverage](phase-5-tv-consoles.md)
