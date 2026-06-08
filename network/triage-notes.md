# Triage Notes — Network Workstream A
**PCAP:** `2026-01-31-traffic-analysis-exercise.pcap`
**Tool:** tshark 4.2.2-1.1build3 on Ubuntu 24 (VMware)
**Triage date:** 2026-06-08

---

## 1. Capture Bounds

| Field | Value |
|---|---|
| Total frames | 51,181 |
| Total bytes | 26,423,689 (~25.2 MB) |
| Capture start | Jan 28, 2026 04:34:03.908927000 IST |
| Capture end | Jan 28, 2026 04:44:27.834350000 IST |
| Capture duration | ~10 minutes 24 seconds |
| Primary internal host | 10.1.21.58 |
| Primary external host | 153.92.1.49 (whitepepper.su) |

---

## 2. Protocol Hierarchy (from `tshark -qz io,phs`)

| Protocol | Frames | Bytes | % of Total Frames |
|---|---|---|---|
| eth | 51,181 | 26,423,689 | 100% |
| ip | 51,127 | 26,420,449 | 99.9% |
| **tcp** | **50,296** | **26,128,605** | **98.3%** |
| nbss | 22,304 | 4,597,767 | 43.6% |
| smb2 | 22,301 | 4,597,281 | 43.6% |
| tls | 9,433 | 11,596,660 | 18.4% |
| dcerpc | 120 | 33,659 | 0.23% |
| kerberos | 16 | 4,986 | 0.03% |
| ldap | 95 | 35,562 | 0.19% |
| http | 16 | 6,765 | 0.03% |
| urlencoded-form | 2 | 2,346 | <0.01% |
| udp | 818 | 289,114 | 1.6% |
| dns | 418 | 54,879 | 0.8% |
| dhcp | 4 | 1,423 | <0.01% |
| nbns | 26 | 2,860 | 0.05% |
| quic | 315 | 219,539 | 0.6% |
| icmp | 13 | 2,730 | 0.03% |
| arp | 54 | 3,240 | 0.1% |
| data | 151 | 195,360 | 0.3% |

**Key observation:** TCP dominates at 98.3% of frames. SMB2 (43.6%) and TLS (18.4%) are the two largest application-layer protocols by volume. HTTP accounts for only 16 frames — anomalously low for a workstation in a corporate environment — yet the HTTP frames observed are the most suspicious in the capture.

---

## 3. Notable Protocols — Baseline vs. Anomaly

### SMB2 (22,301 frames — 43.6%)
SMB2 at this volume on a single workstation is consistent with either:
- Active file share access to a domain server
- Lateral movement reconnaissance using SMB
- Automated enumeration (SAMR calls visible: 2,538 frames)

Sub-protocols observed under SMB2: DCERPC, SAMR, LSARPC (4,576 frames), DSSETUP, NBSS. The  LSARPC and SAMR traffic may indicate account enumeration, policy queries , or normal Active Directory interaction and should be investigated in context with endpoint activity.
### TLS (9,433 frames — 18.4%)
TLS traffic is present throughout the cpature. The observed communications with whitepepper.su includes plaintext HTTP requests for agent registration and status 
reporting. The use of unencrypted HTTP for external agent communication is unusual compared to modern enterpise applications, which commonly utilize HTTPS.

### HTTP (16 frames only)
Despite being a Windows workstation, only 16 HTTP frames were captured. All meaningful HTTP traffic is directed to `whitepepper.su` (153.92.1.49) via `/api/set_agent`. This endpoint receives both GET (polling) and POST (data submission) requests.

### DNS (418 UDP frames)
DNS volume is moderate. Specific queries to be documented in IOC list. `msftconnecttest.com` and `clients2.google.com` queries observed — these are standard Windows connectivity checks (NCSI) and are baseline-normal.

### LDAP / CLDAP (95 + 14 frames)
LDAP traffic present; tshark threw recursion_depth warnings on CLDAP packets 38, 39, 51, 194, 229, 22933, 50982. These are dissector bugs, not malicious. However, LDAP queries from a workstation to domain infrastructure are worth noting as they could indicate domain reconnaissance.

### Kerberos (16 frames)
Low volume, consistent with normal domain authentication ticket activity.

---

## 3b. TCP Conversations (from `tshark -qz conv,tcp | head -15`)

| Source | Destination | Frames (←) | Bytes (←) | Frames (→) | Bytes (→) | Total Frames | Total Bytes | Duration (s) | Notes |
|---|---|---|---|---|---|---|---|---|---|
| 10.1.21.58:54840 | 104.21.46.67:443 | 11,470 | 16 MB | 8,252 | 629 KB | **19,722** | **16 MB** | 22.39 | Largest conversation — Cloudflare TLS; likely CDN-hosted content |
| 10.1.21.58:50768 | 10.1.21.2:445 | 11,010 | 2,183 KB | 10,987 | 2,373 KB | **21,997** | 4,557 KB | 574.96 | SMB to internal server 10.1.21.2 — longest duration in capture |
| 10.1.21.58:61834 | **153.92.1.49:443** | 677 | 42 KB | 1,044 | 1,477 KB | 1,721 | **1,520 KB** | 2.81 | **TLS session to whitepepper.su - suspected encrypted communication** |
| 10.1.21.58:61826 | **153.92.1.49:443** | 198 | 14 KB | 294 | 404 KB | 492 | 419 KB | 2.16 | **Second TLS session to whitepepper.su** |
| 10.1.21.58:63273 | **153.92.1.49:443** | 165 | 12 KB | 195 | 263 KB | 360 | 275 KB | 1.81 | **Third TLS session to whitepepper.su** |
| 10.1.21.58:59569 | 192.178.220.102:443 | 230 | 108 KB | 188 | 105 KB | 418 | 214 KB | 30.07 | Google infrastructure — normal |
| 10.1.21.58:61840 | 10.1.21.2:445 | 176 | 31 KB | 192 | 44 KB | 368 | 75 KB | 16.26 | Second SMB session to internal server |
| 10.1.21.58:54191 | 104.21.48.156:443 | 130 | 50 KB | 157 | 166 KB | 287 | 217 KB | 24.76 | Cloudflare TLS — normal CDN |
| 10.1.21.58:60576 | 104.17.25.14:443 | 130 | 151 KB | 95 | 8,374 B | 225 | 160 KB | 22.45 | Cloudflare TLS — normal CDN |
| 10.1.21.58:53253 | 23.47.48.57:443 | 99 | 122 KB | 96 | 7,735 B | 195 | 130 KB | 108.90 | Akamai TLS — long duration, low volume — normal CDN keepalive |

**Critical observation from conversations:** `whitepepper.su` (153.92.1.49) appears in **three separate TLS sessions on port 443** in addition to the plaintext HTTP on port 80. This means the C2 infrastructure at 153.92.1.49 operates on both HTTP (plaintext agent check-in) and HTTPS (encrypted sessions). The three TLS sessions to 153.92.1.49 carry 2,573 total frames and approximately 2.2 MB of encrypted data — the contents of which cannot be inspected without the server's private key. This elevates the signifinance of the finding because the observed plaintext HTTP agent check-in may represent only a portion of the overall 
communications with the external infrastructure . Additional analysis would be required to determine the purpose of the encrypted session.
---

## 4. Top Talkers — Observed

| Internal Host | External Host | Protocol | Notes |
|---|---|---|---|
| 10.1.21.58 | 153.92.1.49 | HTTP | Primary suspicious conversation — C2 candidate |
| 10.1.21.58 | 23.55.178.249 | HTTP | msftconnecttest.com — Windows NCSI (normal) |
| 10.1.21.58 | 142.250.115.113 | HTTP | clients2.google.com — Chrome/Edge time sync (normal) |
| 10.1.21.58 | 239.255.255.250 | SSDP/M-SEARCH | UPnP discovery multicast — normal workstation behaviour |

**99.9% of IP traffic originates from or terminates at 10.1.21.58.** This is consistent with a single-workstation capture window, not a network tap.

---

## 5. HTTP Requests Observed (full list from tshark)

| Source | Destination | Method | Host | URI | Notes |
|---|---|---|---|---|---|
| 10.1.21.58 | 23.55.178.249 | GET | msftconnecttest.com | /connecttest.txt | Windows NCSI — baseline normal |
| 10.1.21.58 | 142.250.115.113 | GET | clients2.google.com | /time/1/current?cup2key=... | Chrome/Edge time sync — baseline normal |
| 10.1.21.58 | 153.92.1.49 | GET | whitepepper.su | /api/set_agent?id=[REDACTED-ID]&token=[REDACTED-TOKEN]&description=&agent=Chrome | **SUSPICIOUS — agent registration GET** |
| 10.1.21.58 | 153.92.1.49 | POST | whitepepper.su | /api/set_agent?id=[REDACTED-ID]&token=[REDACTED-TOKEN]&description=&agent=Chrome&act=log | **SUSPICIOUS — agent activity POST** |
| 10.1.21.58 | 153.92.1.49 | GET | whitepepper.su | /favicon.ico | Favicon fetch — secondary request after agent check-in |
| 10.1.21.58 | 153.92.1.49 | GET | whitepepper.su | /api/set_agent?id=[REDACTED-ID]&token=[REDACTED-TOKEN]&description=&agent=Edge | **SUSPICIOUS — Edge agent registration GET** |
| 10.1.21.58 | 153.92.1.49 | POST | whitepepper.su | /api/set_agent?id=[REDACTED-ID]&token=[REDACTED-TOKEN]&agent=Edge&act=log | **SUSPICIOUS — Edge agent activity POST** |

**Observed agent identifiers in HTTP parameters:**
- `id`: `[REDACTED-AGENT-ID]`
- `token`: `[REDACTED-TOKEN]`
- `agent=Chrome` and `agent=Edge` — both browser agents are reporting to the same external endpoint with the same ID and token

---

## 6. Anomaly Summary

| Observation | Baseline Expected | What Was Seen | Verdict |
|---|---|---|---|
| Outbound HTTP to .su TLD | Not expected in financial corporate environment | Repeated GET/POST to whitepepper.su | **ANOMALOUS** |
| C2-style URI with id + token params | Not expected | /api/set_agent with static agent ID and auth token | **ANOMALOUS** |
| Both Chrome and Edge reporting to same external endpoint | Not expected | Same id/token pair used by both browser agents | **ANOMALOUS** |
| Plaintext HTTP for external agent traffic | Expected: HTTPS | All whitepepper.su traffic is unencrypted HTTP | **ANOMALOUS** |
| SMB2 + LSARPC + SAMR at high volume | Possible if legitimate file share access | LSARPC 4,576 frames; SAMR 2,538 frames — account/policy enumeration | **INVESTIGATE** |
| NCSI connectivity checks | Normal | msftconnecttest.com, clients2.google.com | **BASELINE NORMAL** |
| UPnP M-SEARCH multicast | Normal on Windows | 239.255.255.250 M-SEARCH frames | **BASELINE NORMAL** |

---

## 7. Triage Command Reference

Commands run to produce this triage:

```bash
# Protocol hierarchy
tshark -r 2026-01-31-traffic-analysis-exercise.pcap -qz io,phs

# HTTP requests with URI
tshark -r 2026-01-31-traffic-analysis-exercise.pcap \
  -Y "http.request" \
  -T fields \
  -e ip.src -e ip.dst -e http.request.method \
  -e http.host -e http.request.uri

# Filter on primary host
tshark -r 2026-01-31-traffic-analysis-exercise.pcap \
  -Y "ip.addr == 10.1.21.58" | wc -l
# Result: 51,115 packets — 99.9% of capture
```

---
