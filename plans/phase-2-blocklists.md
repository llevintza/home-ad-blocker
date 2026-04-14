# Phase 2 — AdGuard Home Blocklist + Custom Rule Configuration

**GitHub Issue:** #2  
**Dependencies:** Phase 1 complete (AdGuard Home running)  
**Estimated time:** 30–45 minutes  

> ⚠️ **Constraints**
> - No jailbreaking of any device (iOS, iPadOS, tvOS, Android)
> - No per-device app installation or configuration
> - All filtering is applied at the network layer only

---

## Goal

Add comprehensive blocklists and streaming-specific custom DNS rules to AdGuard Home to maximize ad and tracker blocking across all device types and platforms.

---

## Step 1 — Add Core Blocklists

In AdGuard Home web UI → **Filters > DNS Blocklists > Add blocklist > Add a custom list**

Add each of the following:

| List Name | URL |
|-----------|-----|
| AdGuard DNS filter | `https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt` |
| EasyList | `https://easylist.to/easylist/easylist.txt` |
| EasyPrivacy | `https://easylist.to/easylist/easyprivacy.txt` |
| OISD Full | `https://big.oisd.nl` |
| AdAway hosts | `https://adaway.org/hosts.txt` |
| hagezi Pro | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt` |
| hagezi Threat Intelligence Feeds | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt` |

After adding each, click **Save**. Then click **Update** on all lists.

---

## Step 2 — Add Streaming-Specific Blocklists

Still in **Filters > DNS Blocklists**, add:

| List Name | URL |
|-----------|-----|
| AdGuard Mobile Ads | `https://raw.githubusercontent.com/AdguardTeam/AdguardFilters/master/MobileFilter/sections/adservers.txt` |
| hagezi Ultimate | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/ultimate.txt` |

> **Note on hagezi Ultimate**: This is an aggressive list and may cause occasional false positives (blocking a domain a service legitimately needs). Monitor for 48 hours after adding. If a service breaks, check AdGuard Home's **Query Log** to see what domain is being blocked and add it to the **Custom Filtering Rules** as an unblock rule (prefix with `@@`).

---

## Step 3 — Add Custom DNS Blocking Rules

In AdGuard Home web UI → **Filters > Custom Filtering Rules**

Paste all of the following:

```
# ── FreeWheel / Comcast (used by Tubi, Peacock, Pluto TV, Hulu) ──
||video.fwmrm.net^
||freewheel.tv^
||tags.tiqcdn.com^
||adserver.adtechus.com^

# ── SpotX ad network (Peacock, Pluto TV, Tubi) ──
||spotxchange.com^
||spotx.tv^
||srv.spotxchange.com^
||search.spotxchange.com^

# ── Google IMA SDK (YouTube, many apps) ──
||imasdk.googleapis.com^
||pubads.g.doubleclick.net^

# ── Hulu telemetry and ad decision servers ──
||adstats.hulu.com^
||pixiedust.hulu.com^
||ads.hulustream.com^

# ── Amazon Prime Video ad SDK ──
||fls-na.amazon.com^
||aax.amazon-adsystem.com^
||advertising.amazon.com^

# ── Generic ad networks ──
||scorecardresearch.com^
||quantserve.com^
||moatads.com^
||adsafeprotected.com^
||krxd.net^
||casalemedia.com^
||advertising.com^
||adnxs.com^
||rubiconproject.com^
||openx.net^
||contextweb.com^
||lijit.com^
||sharethrough.com^

# ── Tracking / analytics ──
||amplitude.com^
||segment.io^
||segment.com^
||mixpanel.com^
||fullstory.com^
||hotjar.com^
||mouseflow.com^
||heapanalytics.com^

# ── Google DAI / Dynamic Ad Insertion (SSAI — used by many streaming platforms) ──
||dai.google.com^
||googleads.g.doubleclick.net^
||securepubads.g.doubleclick.net^

# ── Innovid (SSAI platform — used by Hulu, Peacock, A&E, History Channel, etc.) ──
||rtr.innovid.com^
||csm.dai.innovid.com^
||pb.innovid.com^
||a.innovid.com^
||innovid.com^

# ── Springserve (SSAI/programmatic — Tubi and others) ──
||springserve.com^
||*.springserve.com^

# ── Magnite / Rubicon Project (programmatic SSAI decisions) ──
||prebid-server.rubiconproject.com^
||fastlane.rubiconproject.com^

# ── The Trade Desk (demand-side SSAI decisions) ──
||match.adsrvr.org^
||insight.adsrvr.org^
||sync.adsrvr.org^

# ── Index Exchange ──
||js-sec.indexww.com^
||ssum-sec.casalemedia.com^

# ── LiveRamp (identity resolution for ad targeting) ──
||idsync.rlcdn.com^
||api.rlcdn.com^

# ── Yospace (SSAI middleware — used by broadcast streaming services) ──
||csm.yospace.com^
||yospace.com^

# ── Amazon Publisher Services (APS — Prime Video SSAI decisions) ──
||c.amazon-adsystem.com^
||s.amazon-adsystem.com^
||aax-us-east.amazon-adsystem.com^

# ── Unruly / Tremor Video ──
||targeting.unrulymedia.com^
||tremormedia.com^
||unrulymedia.com^

# ── iCloud Private Relay (forces Apple devices back to Pi DNS) ──
||mask.icloud.com^
||mask-h2.icloud.com^
||mask-api.icloud.com^
||doh.dns.apple.com^
||dnsstar.apple.com^
```

Click **Save**.

> **SSAI blocking behavior note:** When an SSAI ad decision server is blocked, the streaming service backend cannot retrieve the ad manifest. Depending on the platform's error handling:
> - **Best case**: the ad slot is skipped entirely and content resumes immediately
> - **Typical case**: the player waits for a timeout (2–5 seconds of silence/black) then resumes content
> - **Worst case**: the stream stalls indefinitely — roll back the specific domain causing the stall

---

## Step 4 — Monitor for Streaming Service Issues (48-Hour Window)

After applying the custom rules, watch for these specific failure modes:

| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| Hulu stream pauses at ad break and never resumes | `adstats.hulu.com` or `pixiedust.hulu.com` rule causing Hulu to stall rather than skip | Remove those two rules from custom list |
| Prime Video shows a "playback error" at ad slots | Amazon ad rule too aggressive | Remove `fls-na.amazon.com` rule |
| Tubi / Pluto TV shows a buffering spinner at ad break for 5–10 seconds then continues | Expected — ad server blocked, service is waiting for timeout before continuing playback | This is acceptable behavior |
| A streaming service login/account page won't load | Check Query Log in AdGuard Home for blocked domains | Add the domain as `@@||domain.com^` to unblock |

To check what is being blocked for a specific device:
1. Go to AdGuard Home → **Query Log**
2. Filter by the device's IP
3. Look for red (blocked) entries around the time of the issue

---

## Step 5 — Enable DNSSEC and Parallel Requests

In AdGuard Home → **Settings > DNS Settings**:
- [x] Enable DNSSEC
- [x] Enable Parallel Requests
- Upstream DNS mode: **Parallel**

Click **Apply**.

---

## Verification

- [ ] All blocklists show green status in **Filters > DNS Blocklists**
- [ ] Custom rules saved successfully (no syntax errors shown)
- [ ] Visit `https://d3ward.github.io/toolz/adblock` from a connected device — score should be 80%+
- [ ] `nslookup doubleclick.net 192.168.1.2` returns `0.0.0.0`
- [ ] `nslookup spotxchange.com 192.168.1.2` returns `0.0.0.0`
- [ ] Tubi or Pluto TV shows shorter/skipped ad breaks (ad-decision server blocked)
- [ ] 48-hour monitoring window: Hulu and Prime Video streams play without indefinite stalls

---

## Rollback: If a Service Breaks

To temporarily unblock a domain without removing it from lists, add to custom rules:
```
@@||domain-that-broke.com^
```

This whitelists the domain even if it appears in a blocklist.

---

## Next Steps

Phases 3, 4, and 5 can now be run in parallel:
- [Phase 3 — Android](phase-3-layer2-android.md)
- [Phase 4 — iOS/iPadOS](phase-4-layer2-apple.md)
- [Phase 5 — TV devices](phase-5-layer2-tv.md)
