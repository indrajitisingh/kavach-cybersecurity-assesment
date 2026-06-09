# Network Forensics Report — Workstream A
**Project:** KAVACH · Meridian FinServe Security Assessment
**PCAP:** `2026-01-31-traffic-analysis-exercise.pcap`
**Date:** 2026-06-08
**Status:** Complete — A.1 through A.5

---

## A.1 Source PCAP — Selection & Analogue Justification

**Selected capture:** `2026-01-31-traffic-analysis-exercise.pcap`
**Source corpus:** Malware-Traffic-Analysis.net (public corpus, free use for educational purposes)

### Why this capture is a defensible analogue for Meridian FinServe

The Meridian FinServe trigger was a 72-hour window of anomalous east-west and outbound traffic from a server segment that historically generates predictable, low-variance flows. This capture exhibits identical characteristics:

| Meridian FinServe Trigger | This Capture |
|---|---|
| Anomalous outbound traffic from a stable segment | 99.9% of 51,181 frames originate from single host DESKTOP-ES9F3ML (10.1.21.58) |
| East-west traffic to internal infrastructure | SMB2 sessions to internal server 10.1.21.2 with LSARPC/SAMR enumeration |
| Outbound traffic to external unknown destination | Repeated HTTP/TLS sessions to whitepepper.su (153.92.1.49)- an external .su TLD domain with no observed business
   justification in this enviroment|
| On-call team could not isolate root cause | Traffic pattern combines C2 beaconing, DNS to multiple suspicious TLDs, and SMB lateral movement — not attributable to a single cause without deep analysis |
| Low-variance segment showing sudden anomaly | DHCP confirms single Windows workstation (DESKTOP-ES9F3ML); normal baseline includes Microsoft/Google domains; anomaly is the .su/.cc/.lat/.sbs DNS queries and /api/set_agent HTTP pattern |

The capture represents a Windows 11 workstation (`DESKTOP-ES9F3ML`, hostname confirmed via DHCP) on a small office network with an Active Directory domain (`win11office.com`, `mshome.net`), which maps directly to Meridian FinServe's branch office workstation environment connecting to co-located data center infrastructure.

---

## A.2 Triage Pass Summary

**Full triage documented in:** `triage-notes.md`

| Metric | Value |
|---|---|
| Total frames | 51,181 |
| Total bytes | 26,423,689 (~25.2 MB) |
| Capture start | Jan 28, 2026 04:34:03 IST |
| Capture end | Jan 28, 2026 04:44:27 IST |
| Duration | 10 minutes 24 seconds |
| Primary internal host | 10.1.21.58 (DESKTOP-ES9F3ML) |
| Internal server | 10.1.21.2 (SMB/AD) |
| Primary external threat | 153.92.1.49 (whitepepper.su) |

**Protocol distribution (dominant):**
- TCP: 50,296 frames (98.3%)
- SMB2/NBSS: 22,301 frames (43.6%) — file share + named pipe enumeration
- TLS: 9,433 frames (18.4%) — encrypted sessions including to C2
- DNS: 418 UDP frames — 70+ unique domains queried, 7 suspicious
- HTTP: 16 frames — all meaningful HTTP is C2 traffic to whitepepper.su

**Baseline normal domains identified:** Microsoft NCSI, Google time sync, Microsoft/Google CDN, Office 365, Edge telemetry, Skype config — all expected for a Windows 11 workstation with Chrome and Edge installed.

---

## A.3 Hypothesis-Driven Deep Dive Summary

**Full hypothesis documentation in:** `hypotheses.md`

Three competing hypotheses were formed and tested:

| Hypothesis | Verdict | Confidence |
|---|---|---|
| H1: Malware C2 beaconing via browser process injection | **SUPPORTED** | HIGH |
| H2: Legitimate RMM or browser management agent | **REFUTED** | — |
| H3: Insider-installed data exfiltration tool | PARTIALLY SUPPORTED | MEDIUM |

**Final position:** H1 and H3 are not mutually exclusive. Network evidence is consistent with C2-style beaconing and potential data exfiltration activity.
Confirmation of the exact data transmitted would require endpoint or payload inspection.

**Key reasoning:**
- Static agent credential pair used identically across Chrome and Edge — no consistent with typical enterprise software deployment patterns
- No evidence was identified that whitepepper.su serves a legitimate business function within the observed enviroment
- `act=log` POST parameter is consistent with structured logging or data submission activity rather than a simple connectivity check. 
---

## A.4 Indicator Extraction Summary

**Full IOC list in:** `iocs.csv`

### High-Confidence IOCs

| Type | Indicator | MITRE |
|---|---|---|
| IP | 153.92.1.49 | T1071.001 |
| Domain | whitepepper.su | T1071.001 |
| URI pattern | /api/set_agent | T1071.001 |
| HTTP param | act=log | T1041 |
| SMB file | AutoRun.inf | T1091 |
| Named pipe | lsarpc, samr | T1003.004 |

### Suspicious DNS Domains (anomalous, not baseline-normal)

| Domain | TLD Assessment | Risk |
|---|---|---|
| whitepepper.su | .su — legacy TLD frequently observed in malicious infrastructure investigation | **HIGH** |
| arch.filemegahab4.sbs | .sbs — new gTLD, high malware prevalence; random subdomain pattern | **HIGH** |
| media.megafilehub4.lat | .lat — high malware prevalence; related to filemegahab4 | **HIGH** |
| communicationfirewall-security.cc | .cc — typosquat pattern; fake security branding | **HIGH** |
| whooptm.cyou | .cyou — high malware prevalence; random string | **MEDIUM** |
| holiday-forever.cc | .cc — suspicious naming pattern | **MEDIUM** |
| hiyter.com | Unknown vendor; no business context | **MEDIUM** |

**Note:** `arch.filemegahab4.sbs` and `media.megafilehub4.lat` share the `megafilehub4` root — this is likely a multi-domain C2 infrastructure operating across TLDs, consistent with domain generation algorithm (DGA) or deliberate redundancy.

### SMB Lateral Movement Indicators

| Indicator | Type | Significance |
|---|---|---|
| AutoRun.inf accessed on 10.1.21.2 | SMB2 file access | Classic autorun-based persistence/spreading mechanism |
| lsarpc named pipe | SMB2 named pipe | LSA policy enumeration — account privilege discovery |
| samr named pipe | SMB2 named pipe | SAM database enumeration — user account harvesting |
| desktop.ini / Desktop.ini | SMB2 file access | Directory enumeration pattern |

**LSARPC (4,576 frames) + SAMR (2,538 frames)** represent substantial account and policy-realted activity . The voulme observed warrants fruther investigation
to determine whether it reflects legitimate administration or reconnaissance activity.
---

## A.5 Architecture Analysis

**Diagram files:** `architecture/before.svg` · `architecture/after.svg`

### Current State (Before) — Observed Vulnerabilities

Based on traffic analysis, the Meridian FinServe analogous segment presents the following architecture weaknesses:

```
INTERNET
    │
    │  ← No egress filtering observed
    │  ← DNS queries to .su/.cc/.lat/.sbs TLDs not blocked
    │  ← HTTP to arbitrary external IPs permitted
    │
[WORKSTATION SEGMENT]
    │
    DESKTOP-ES9F3ML (10.1.21.58)
    │   Windows 11 · Chrome + Edge installed
    │   ← No evidence of endpoint controls preventing te observed outbound communication
    │   ← Outbound HTTP communciation from browser-associated traffic was observed.
    │   ← DHCP hostname exposed: DESKTOP-ES9F3ML
    │
    │  SMB2 (port 445) — unrestricted east-west
    │  LSARPC + SAMR named pipes accessible
    │
[INTERNAL SERVER]
    10.1.21.2
    │   Active Directory domain controller (win11office.com)
    │   ← AutoRun.inf accessible via SMB share
    │   ← Named pipe enumeration not restricted
    │   ← SMB signing status was not verified during this analysis
```

**Identified architectural gaps:**

1. **No egress DNS filtering** — 7 suspicious TLDs queried without blocking
2. **No HTTP egress allowlist** — workstation can reach arbitrary external IPs on port 80
3. **Unrestricted SMB east-west** — workstation accesses domain controller SMB with no apparent segmentation
4. **Named pipe exposure** — LSARPC and SAMR accessible from workstation to DC without restriction
5. **No network-layer C2 detection** — 10-minute beaconing window generated zero apparent alerts
6. **AutoRun.inf on accessible share** — persistence vector available via SMB

### Proposed Hardened State (After) — Defense Diff

The diff between before and after is the deliverable. Each control addresses a specific finding:

| Layer | Control | Finding Addressed | Effort |
|---|---|---|---|
| Perimeter | Egress DNS firewall blocking .su/.cc/.lat/.sbs/.cyou TLDs | whitepepper.su, filemegahab4, communicationfirewall-security.cc | S |
| Perimeter | HTTP egress allowlist — deny arbitrary outbound port 80 | /api/set_agent C2 channel | M |
| Perimeter | TLS inspection for sessions to unknown external IPs | 3 encrypted TLS sessions to 153.92.1.49:443 | L |
| Segmentation | Firewall rule: workstations cannot initiate SMB to DC directly | AutoRun.inf access, LSARPC/SAMR enumeration | M |
| Segmentation | Named pipe filtering — restrict SAMR/LSARPC to admin hosts only | 7,000+ frames of account enumeration | M |
| Application | Browser extension allowlist — prevent unauthorized extensions | Browser agent injection via Chrome/Edge | M |
| Observability | DNS query logging to SIEM with TLD anomaly alerting | 7 suspicious domains queried with no detection | S |
| Observability | SMB named pipe access alerting on LSARPC/SAMR from workstations | Account enumeration | S |
| Endpoint | EDR deployment with C2 callback pattern detection | /api/set_agent beaconing | L |
| Response | Isolation playbook for workstations showing C2 beacon pattern | DESKTOP-ES9F3ML not isolated during 10-min window | S |

**Trade-offs noted:**
- TLS inspection (L effort) requires certificate deployment and may break certificate pinning in some applications
- SMB segmentation may break legacy file-share workflows if workstations legitimately access DC shares — requires audit before enforcement
- Browser extension allowlist requires MDM (Intune/SCCM) and ongoing maintenance overhead

---

## Summary of Findings

| Finding | Confidence | Severity | MITRE |
|---|---|---|---|
| Active C2 beaconing to whitepepper.su via /api/set_agent | HIGH | CRITICAL | T1071.001 |
| Encrypted C2 channel — 3 TLS sessions to 153.92.1.49:443, ~2.2MB | HIGH | CRITICAL | T1071.001 |
| Browser-associated communication observed using — Chrome and Edge both identifiers| HIGH | HIGH | T1071.001 |
| DNS queries to 7 suspicious domains across malicious TLDs | HIGH | HIGH | T1071.004 |
| SMB lateral movement — AutoRun.inf access on internal server | HIGH | HIGH | T1091 |
| Domain account enumeration — LSARPC/SAMR 7,000+ frames | HIGH | HIGH | T1003.004 |
| Multi-domain C2 infrastructure — filemegahab4.sbs + .lat | MEDIUM | HIGH | T1583.001 |
| No egress filtering — all anomalous traffic permitted outbound | HIGH | HIGH | TA0010 |

**Overall assessment:** The 10-minutes capture window contains multiple indicators consistent with suspicious or potentially malicious activity.
Observed communication includes HTTP and TLS session with external infrastructure , unusual DNS activity, and SMB-based account enumeration behaviour.
Additional endpoint investigation would be required to confirm compromise and dtermine scope.

---



