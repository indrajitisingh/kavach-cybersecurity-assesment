# Project KAVACH — Network Forensics · Workstream A

Assesment Scope:- Network traffic analysis of a 10-minute packet capture containing 51,181 frames from a simulated Meridian FinServe workstation enviroment

> **Risk Rating: 🔴 HIGH** · Confidence: MEDIUM-HIGH · Capture: 51,181 frames · ~10 min window

---

## What Happened

One workstation — **10.1.21.58** — was responsible for nearly all observed traffic. It was doing two things it shouldn't have been:

**Talking to the outside world.** Repeated outbound connections to **whitepepper.su** (`153.92.1.49`), hitting `/api/set_agent` with agent IDs and auth tokens over HTTP and TLS. That's not user browsing — that's automated check-in behavior, consistent with a C2 activity or an unauthorized remote management tool.

**Mapping the inside network.** Heavy SMB traffic to internal server `10.1.21.2`, including LSARPC and SAMR calls used to enumerate accounts and policies. Volume and timing didn't match legitimate admin work.

DNS queries were also going to domains on suspicious TLDs (`.su`, `.sbs`, `.lat`, `.cc`, `.cyou`) — the kind of infrastructure associated with attacker-controlled or rotated services.

---

## Why It's a Problem

The encrypted outbound channel means we can't see what left the network. The account enumeration activity is pre-lateral-movement reconnaissance. Together, these 
observations are consistent with an established compromise involving external communications and internal reconnaissance activity.

Potential exposure: data exfiltration, credential harvesting, lateral movement, and regulatory consequences — all with limited detection visibility.

---

## What To Do

| Priority | Action |
|---|---|
| 🔴 Immediate | Block high-risk TLDs (`.su`, `.sbs`, `.cc`, `.cyou`) at egress |
| 🔴 Immediate | Deploy EDR with beaconing / anomalous outbound detection |
| 🔴 Immediate | Restrict workstation-initiated SMB to core infrastructure |
| 🟠 Short-term | Alert on LSARPC/SAMR volume from user endpoints |
| 🟠 Short-term | Define and drill a rapid isolation playbook |
| 🟡 Structural | Network segmentation + centralized DNS/HTTP/TLS/SMB logging |

---

## Next Steps

Endpoint forensics investigation is required to determine the root cause (malware, browser extension , or user - installed software).
---
*Project KAVACH · Meridian FinServe Simulated Environment*
