# Hypotheses — Network Workstream A
**PCAP:** `2026-01-31-traffic-analysis-exercise.pcap`

**Tool:** tshark 4.2.2 + Wireshark on Ubuntu 24
**Date:** 2026-06-08

> **Methodology:** Three competing hypotheses were formed before drawing any conclusion. Each was tested against the data. The deliverable is the reasoning, not just the answer.

---

## Hypothesis 1 — Malware C2 Beaconing (Browser Hijack / Infostealer)

### Statement
Host `10.1.21.58` is infected with malware that has injected itself into both Chrome and Edge browser processes. The malware uses the browser agent strings as a cover identity and periodically checks in to its command-and-control server at `whitepepper.su` (153.92.1.49) via the `/api/set_agent` endpoint, transmitting a persistent agent ID and auth token.

### What would confirm it
- The agent ID (`[REDACTED-AGENT-ID]`) and token (`[REDACTED-TOKEN]`) remain static across all requests — a hardcoded identifier is characteristic of malware, not legitimate software
- Both Chrome and Edge report to the **same** endpoint with the **same** ID and token simultaneously, which no legitimate dual-browser configuration would produce
- The `.su` TLD (Soviet Union legacy domain) has no legitimate business justification for a financial enterprise network
- Communication is plaintext HTTP in 2026 — legitimate monitoring software universally uses HTTPS
- The `act=log` POST parameter suggests the malware is uploading activity logs to the C2

### What would refute it
- If a legitimate software vendor (e.g. an RMM tool vendor, browser extension developer) can be identified that uses this exact endpoint pattern with agent ID + token over plaintext HTTP
- If the domain `whitepepper.su` resolves to a known corporate or vendor asset
- If the token rotates between sessions (indicating OAuth-style legitimate auth rather than hardcoded malware identifier)

### Evidence from capture
| Evidence Item | Packet Context | Weight |
|---|---|---|
| GET /api/set_agent?id=[REDACTED-ID]&agent=Chrome | HTTP to 153.92.1.49 | HIGH |
| POST /api/set_agent?...&agent=Chrome&act=log | HTTP to 153.92.1.49 | HIGH |
| GET /api/set_agent?id=[REDACTED-ID]&agent=Edge | Same ID, different agent string | HIGH |
| POST /api/set_agent?...&agent=Edge&act=log | Same token across both agents | HIGH |
| All 51,115 of 51,181 packets originate from 10.1.21.58 | tshark filter ip.addr==10.1.21.58 | MEDIUM |
| Plaintext HTTP — no TLS to 153.92.1.49 | Protocol hierarchy: HTTP 16 frames, TLS to other hosts only | HIGH |
| .su TLD — no legitimate business entity in financial sector | Domain registration context | MEDIUM |

### Verdict
**SUPPORTED — HIGH CONFIDENCE**

The static agent ID used identically by both Chrome and Edge is the strongest indicator. No legitimate browser extension or monitoring software shares a single hardcoded credential pair across two separate browser processes simultaneously. The `act=log` POST parameter suggests active data collection and upload. This hypothesis is the most consistent with all observed evidence.

---

## Hypothesis 2 — Legitimate Remote Monitoring & Management (RMM) Agent

### Statement
Host `10.1.21.58` has a legitimate RMM or browser management agent installed (e.g. a corporate endpoint management tool, a browser extension management platform, or an MDM agent). The `/api/set_agent` endpoint and agent ID are expected behaviour for this class of software. The IT team is aware of this installation.

### What would confirm it
- The domain `whitepepper.su` is registered to a known software vendor
- The agent ID is dynamically assigned per device during enrolment (not hardcoded malware)
- HTTPS is used in the full deployment but this particular version or environment uses HTTP for a specific reason (e.g. internal proxy, legacy version)
- SMB2 and LSARPC activity is consistent with IT management polling from a domain controller, not lateral movement

### What would refute it
-No evidence was found that whitepepper .su belongs to a legitmate commerical RMM vendor. — this TLD has been consistently associated with malicious infrastructure since at least 2012
- The same agent ID and token appear for both Chrome and Edge — legitimate browser management tools issue per-browser or per-device identifiers, not a single shared credential
- Legitimate RMM agents use HTTPS with certificate pinning; plaintext HTTP would expose the auth token
-NO publicy available vendor documentation or product information was identified for 'whitepepper.su'.

### Evidence from capture
| Evidence Item | Assessment |
|---|---|
| SMB2 volume (22,301 frames) | Could indicate legitimate IT management polling |
| LSARPC/SAMR activity | Could be domain controller policy sync |
| Kerberos (16 frames) | Consistent with normal domain authentication |
| /favicon.ico fetch after agent check-in | Legitimate agents sometimes fetch branding assets |

### Verdict
**REFUTED — LOW CONFIDENCE**

While SMB2 , kerberos , and LSARPC activity can occur in legimated enterprise enviroments , the observed combination of a static agent identifier, shared token 
across multiple browser contexts , plaintext HTTP communication , and the external domain 'whitepepper.su' is not consistent with normal enterpise RMM deployment 
patterns. Therefore, Hypothesis 2 is not supported by the available network evidence.
---

## Hypothesis 3 — Insider Threat / Deliberate Data Exfiltration Tool

### Statement
A user on host `10.1.21.58` deliberately installed a tool — possibly a data harvesting or browser activity logging tool — to exfiltrate browsing data, credentials, or activity logs to an external service they control or are paid to report to. The `act=log` parameter suggests structured log upload. The use of both Chrome and Edge agents indicates the tool is harvesting from both browsers.

### What would confirm it
- POST body content (from TCP stream reconstruction) contains structured browser history, saved passwords, cookies, or form data
- The tool was installed voluntarily by the user (no lateral movement evidence pointing to external compromise)
- The agent ID `[REDACTED-AGENT-ID]` can be attributed to a specific user account via endpoint logs
- LSARPC/SAMR activity originates from the same host and represents the user probing domain accounts

### What would refute it
- If the tool was installed without user interaction (dropper, drive-by download) — this would reclassify as H1 (malware)
- If the POST bodies contain only browser metadata (user-agent strings, tab counts) rather than sensitive content
- If the host shows no signs of voluntary software installation in SMB2 or filesystem activity

### Evidence from capture
| Evidence Item | Assessment |
|---|---|
| POST /api/set_agent?...&act=log (Frame=1234 ,TCP Stream 17) | `act=log` is explicit logging action — data upload is occurring |
| Both Chrome and Edge agents reporting | Suggests systematic harvesting across browser profiles |
| Static agent ID + token | Could be a user-specific credential issued by the operator of whitepepper.su |
| LSARPC 4,576 frames + SAMR 2,538 frames | If user-initiated, could indicate deliberate account enumeration |
| No inbound connections to 10.1.21.58 | No evidence of external attacker pushing commands inbound |

### Verdict
**PARTIALLY SUPPORTED — MEDIUM CONFIDENCE**

This hypothesis cannot be fully refuted from network evidence alone. The absence of inbound C2 commands (which would indicate remote operator control) slightly favours deliberate insider installation over external compromise. However, the distinction between H1 (malware) and H3 (insider tool) requires endpoint forensics — registry run keys, installation logs, browser extension manifests — which are outside the scope of this PCAP analysis. From network evidence only: the behaviour is malicious regardless of whether the actor is internal or external.

**For the Meridian FinServe scenario:** H3 is the more dangerous finding — a malicious insider with LSARPC/SAMR enumeration capability and active browser data exfiltration represents a combined network + web application threat surface, which directly feeds into Workstream C synthesis.

---

## Cross-Hypothesis Summary

| Criterion | H1: Malware C2 | H2: Legitimate RMM | H3: Insider Tool |
|---|---|---|---|
| .su TLD | Consistent | Refutes | Consistent |
| Static agent ID across Chrome + Edge | Consistent | Refutes | Consistent |
| Plaintext HTTP for auth token | Consistent | Refutes | Consistent |
| act=log POST parameter | Consistent | Neutral | Strongly consistent |
| LSARPC/SAMR volume | Lateral movement indicator | IT management | Deliberate enumeration |
| No inbound C2 commands | Neutral | Neutral | Slightly favours H3 |
| **Overall** | **HIGH confidence** | **REFUTED** | **MEDIUM confidence** |


**Final Position**

Based solely on network evidence, Hypothesis 1 is the strongest explanation.

Host `10.1.21.58` repeatedly communicates with `153.92.1.49` using static identifiers across multiple browser contexts and uploads activity data through the `act=log` parameter over plaintext HTTP. These behaviors are most consistent with browser-focused malware or an information-stealing capability.

Hypothesis 2 is not supported by the observed authentication, domain reputation, and communication patterns.

Hypothesis 3 remains plausible regarding how the software was introduced onto the host, but confirming insider involvement requires endpoint artifacts that are outside the scope of this PCAP analysis.
---

*Output feeds directly into: `iocs.csv` (IOC extraction) → `architecture/` (before/after hardening diagram)*
