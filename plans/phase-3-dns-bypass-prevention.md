# Phase 3 — Encrypted DNS Bypass Prevention

**GitHub Issue:** #3  
**Dependencies:** Phase 1 complete  
**Applies to:** All devices automatically — no per-device action required  
**Must complete before:** Phase 4, Phase 5  

> ⚠️ **Constraints**
> - No jailbreaking of any device (iOS, iPadOS, tvOS, Android)
> - No per-device app installation or configuration
> - All filtering is applied at the network layer only

---

## Why This Phase Is Critical

Without this phase, the Raspberry Pi DNS sinkhole is **invisible to modern devices**. Here is why:

| Device / Feature | Bypass Mechanism | Effect Without This Phase |
|-----------------|-----------------|--------------------------|
| iPhones, iPads, Apple TV (iOS/tvOS) | **iCloud Private Relay** — routes DNS through Apple's servers | Pi DNS is completely bypassed; no filtering applied |
| Android 9+ phones and tablets | **Private DNS** (DNS-over-TLS) — hardcoded in Android settings | If user has Private DNS set, Pi DNS is bypassed |
| Chrome browser | **DNS-over-HTTPS** — built-in, enabled by default in some regions | DNS queries go to Google's servers, not the Pi |
| Firefox browser | **DNS-over-HTTPS** — Cloudflare by default | DNS queries bypass the Pi |
| Any device | **Hardcoded DNS** — some apps/devices use `8.8.8.8` directly | Bypasses router-assigned DNS entirely |

This phase closes all these bypass routes at the **router firewall level** — no device is touched.

---

## How It Works

The router is configured to:
1. **Redirect** all outbound DNS (port 53) queries from any device to the Pi — even if a device tries to use a different DNS server IP
2. **Block** DNS-over-TLS (port 853) — devices cannot establish encrypted DNS tunnels to bypass the Pi
3. **Block** iCloud Private Relay IP ranges — Apple TV and iPhones fall back to standard DNS (the Pi)
4. **Block** connections to known DoH provider IPs on port 443 — prevents browsers and apps from using encrypted DNS to bypass filtering

The Pi's own upstream DoH connections are not affected — the Pi itself uses encrypted DNS to Quad9 and Cloudflare, but this traffic originates *from the Pi*, not from client devices.

---

## Step 1 — Configure Netgear Router Firewall Rules

Log in to the Netgear admin panel at `orbilogin.com` or `192.168.1.1`.

### A — Block DNS-over-TLS (Port 853)

Go to **Advanced > Security > Block Services** (or **Firewall Rules**):

Add a service block rule:
- Service name: `DNS-over-TLS`
- Protocol: TCP + UDP
- Port range: 853–853
- Apply to: All devices (or all IPs in the 192.168.1.0/24 range)
- Direction: Outbound

This prevents any device from establishing a DoT tunnel to external servers (Cloudflare 1.1.1.1, Google 8.8.8.8, etc.).

### B — Redirect All Outbound Port 53 to the Pi

Some devices or apps hardcode DNS server IPs (e.g., `8.8.8.8`) instead of using the DHCP-assigned server. This rule intercepts those and redirects them to the Pi.

On Netgear, this requires a **port forwarding / NAT redirect** rule. The exact menu path varies by Netgear firmware version:

- Go to **Advanced > Setup > Port Forwarding / Port Triggering**
- Add a rule:
  - Service name: `DNS Redirect`
  - External port: 53
  - Internal IP: `192.168.1.2` (the Pi)
  - Internal port: 53
  - Protocol: UDP

> **Note**: Not all Netgear Orbi firmware versions support true NAT redirection for DNS. If this option is not available, the alternative is to set AdGuard Home as the only DNS server (primary and secondary both set to `192.168.1.2`) in the router DHCP settings — this covers DHCP-aware devices but not hardcoded DNS.

### C — Block Known DoH Provider IPs

Add outbound block rules for the following IP addresses on port 443. This prevents Chrome, Firefox, and apps from using encrypted DoH to bypass the Pi.

| Provider | IP Addresses to Block |
|----------|-----------------------|
| Cloudflare | `1.1.1.1`, `1.0.0.1`, `2606:4700:4700::1111`, `2606:4700:4700::1001` |
| Google | `8.8.8.8`, `8.8.4.4`, `2001:4860:4860::8888`, `2001:4860:4860::8844` |
| Quad9 | `9.9.9.9`, `149.112.112.112` |
| OpenDNS | `208.67.222.222`, `208.67.220.220` |

In Netgear admin panel → **Advanced > Security > Block Sites** or **Firewall Rules**:
- Block outbound TCP port 443 to each of the IPs above

> This only blocks DoH to these specific IPs. Regular HTTPS traffic to websites on port 443 is not affected because those go to different IP addresses.

---

## Step 2 — Block iCloud Private Relay

iCloud Private Relay is Apple's privacy feature that routes all DNS and web traffic through Apple's servers, bypassing local DNS entirely. This makes the Pi invisible to iPhones, iPads, and Apple TVs that have Private Relay enabled.

### Option A — Block at DNS Level (AdGuard Home)

In AdGuard Home → **Settings > General Settings**:
- Enable **"Block Apple Private Relay"** (if available in your AdGuard Home version)

Alternatively, add to **Filters > Custom Filtering Rules**:
```
# iCloud Private Relay blocking
||mask.icloud.com^
||mask-h2.icloud.com^
||mask-api.icloud.com^
||doh.dns.apple.com^
||dnsstar.apple.com^
```

When these domains are blocked, Apple devices fall back to the standard DNS assigned by the router (the Pi).

### Option B — Block Apple Private Relay IP Ranges at the Router

Apple publishes the IP ranges used by Private Relay at:
`https://mask-api.icloud.com/egress-ip-ranges.csv`

Adding firewall rules to block outbound connections to these IP ranges forces Apple devices to use regular networking (and therefore the Pi DNS). Given the number of ranges, this is best done in AdGuard Home's custom rules or the router's IP block list if it supports bulk IP blocking.

---

## Step 3 — Configure AdGuard Home Upstream for DoH Bypass

In AdGuard Home → **Settings > DNS Settings**, add a **DNS rewrites** or custom rule to redirect any device attempting to query known public DNS IPs:

Under **Settings > DNS Settings > Private reverse DNS servers**, ensure AdGuard Home handles internal DNS correctly.

The upstream DoH configuration (set in Phase 1) ensures the Pi's own DNS queries are encrypted and go to Quad9/Cloudflare — this is unaffected by the blocking rules above because those rules apply to client devices, not to the Pi itself.

---

## Step 4 — Handle Android Private DNS

Android 9+ devices have a **Private DNS** setting (Settings > Network > Advanced > Private DNS) that, if set to a hostname or "Automatic," will use DoT/DoH and bypass the Pi.

This is handled by Step 1A (blocking port 853) — when DoT connections fail, Android falls back to the DHCP-assigned DNS (the Pi). No device-level changes are needed.

---

## Verification

After completing this phase, confirm that bypass routes are closed:

```bash
# From your PC, test that port 853 is blocked outbound
# (should time out or be refused)
nc -vz 1.1.1.1 853

# From the Pi, verify the Pi itself can still reach DoH upstream
curl -s "https://dns10.quad9.net/dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB" -H "accept: application/dns-message" | head -c 20
```

In AdGuard Home → **Query Log**:
- Filter by an iPhone or Apple TV's IP address
- Confirm DNS queries ARE appearing (Private Relay blocked, device using Pi DNS)
- If the device's queries are not appearing in the log, Private Relay is still active — re-check Step 2

In AdGuard Home → **Client Statistics**:
- All device IP addresses should appear with query counts
- A device showing zero queries means it is bypassing the Pi — investigate which mechanism

---

## Expected Outcome

After this phase, every device on the network — regardless of its operating system, DNS settings, or privacy features — will have its DNS queries processed by the Pi. iCloud Private Relay on Apple TV and iPhones will be blocked at the network level, forcing those devices to use the Pi as their DNS resolver. No user action is needed on any device.

---

## Next Steps

- [Phase 4 — Apple Device Coverage](phase-4-apple-devices.md)
- [Phase 5 — TV, Console & Smart Home Coverage](phase-5-tv-consoles.md)
