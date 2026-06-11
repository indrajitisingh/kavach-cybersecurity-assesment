# 🛡️ Project KAVACH — Cybersecurity Assessment Report

> **CONFIDENTIAL · SECURITY ASSESSMENT REPORT**

**Meridian FinServe — End-to-End Cybersecurity Assessment**
*Network Forensics · Web Application Security · Threat Modeling · Defense-in-Depth*

---

| Prepared by | Assessment Date | Report Version | Overall Risk |
|---|---|---|---|
| Team Zero Trace | June 2026 | 1.0 — Final | 🔴 **CRITICAL** |

---

## Engagement Statistics

| Workstreams | Total Findings | Critical | High |Medium | MITRE Techniques | OWASP Categories |
|:-----------:|:--------------:|:--------:|:----:|:----------------:|:----------------:|
| 2 | 9 | **3** | **5** | 1 | 9 | 4 |

---

## Assessment Team

| Name | Role |
|---|---|
| **Indrajit Singh** | Team Lead & Lead Security Analyst |
| **Chaitanya** | Evidence & Repository Coordinator |
| **M. Dubey** | Security Documentation Reviewer |
| **Pushpendra Sahu** | Reporting & Compliance Support |

> ⚠️ *Simulated Exercise — For Educational & Portfolio Purposes Only — Meridian FinServe is a Fictional Organisation*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope & Methodology](#2-scope--methodology)
3. [Findings Summary](#3-findings-summary)
4. [Workstream A — Network Forensics](#4-workstream-a--network-forensics)
5. [Workstream B — Web Application Security](#5-workstream-b--web-application-security)
6. [Attack Path Analysis](#6-attack-path-analysis)
7. [Defense-in-Depth Recommendations](#7-defense-in-depth-recommendations)
8. [Prioritised Remediation Plan](#8-prioritised-remediation-plan)
9. [MITRE ATT&CK Coverage](#9-mitre-attck-coverage)
10. [Conclusions](#10-conclusions)

---

## 1. Executive Summary

Project KAVACH was a structured, end-to-end cybersecurity assessment of the Meridian FinServe environment conducted by Team Zero Trace. The engagement spanned two technical workstreams — network forensics and web application security — supported by threat modeling, attack path analysis, and defense-in-depth design.

The assessment identified an a worksattion exhibiting behaviour consistent with command-and -control (C2) communication and internal reconnaissance activity 
operating within the same enviorment as a customer- facing web application containing five explotiable vulnerabilities , including two rated critical.

| Overall Risk Rating | Network Forensics Risk | Web Application Risk |
|:---:|:---:|:---:|
| 🔴 **CRITICAL** | 🟠 **HIGH** | 🟠 **HIGH** |

The assessment identified multiple high-confidence indicators of compromise and critical application-layer weaknesss that collectively present a credible 
multi-stage attack scenario.

---

## 2. Scope & Methodology

### 2.1 Assessment Scope

| Workstream | Scope |
|---|---|
| **Workstream A** | Network traffic analysis — 10-minute PCAP (51,181 frames) from a compromised internal workstation environment. Focus: C2 identification, DNS anomaly detection, SMB lateral movement indicators, IOC extraction. |
| **Workstream B** | Web application security assessment of the B2C Meridian FinServe customer portal. Coverage: authentication, authorisation, input validation, session handling, file upload handling, and data access controls. |
| **Supporting** | Threat modeling, multi-workstream attack path analysis, defense-in-depth control design, and executive risk reporting. |

### 2.2 Methodology & Tools

| Category | Tools & Techniques |
|---|---|
| Network Analysis | Wireshark, Tshark — protocol distribution, beacon interval analysis, JA3 hash analysis |
| Traffic Filtering | SMB named-pipe isolation, DNS anomaly filtering, HTTP pattern extraction |
| Threat Intelligence | JA3 hash databases, MITRE ATT&CK framework, TLD reputation analysis |
| Web App Testing | Burp Suite, manual injection testing, business-logic analysis, OWASP Top 10 mapping |
| Documentation | Markdown evidence logging, Git version control, structured finding register |

### 2.3 Testing Approach

Network forensics followed a hypothesis-driven triage model. Three competing hypotheses (malware C2, legitimate RMM agent, and insider data exfiltration) were formed at the outset and tested against the evidence. Web application testing was risk-prioritised: authentication and injection surfaces were assessed first, followed by business logic, access control, and file handling.

Findings from both workstreams were cross-referenced at the conclusion of technical analysis to identify compounding risk and construct the attack path narrative.

---

## 3. Findings Summary

### 3.1 Workstream A — Network Forensics

| ID | Finding | Severity | MITRE Technique |
|---|---|:---:|---|
| F-A1 | C2 Beaconing — Compromised Internal Workstation (10.1.21.58) to `whitepepper.su` via `/api/set_agent` | 🔴 Critical | T1071 — Application Layer Protocol |
| F-A2 | SMB Reconnaissance — LSARPC/SAMR Enumeration (7,000+ frames to internal DC 10.1.21.2) | 🟠 High | T1087 — Account Discovery |
| F-A3 | DNS Queries to High-Risk TLDs (`.su`, `.sbs`, `.cc`, `.lat`, `.cyou`) — 7 suspicious domains | 🟠 High | T1071.004 — DNS |
| F-A4 | AutoRun.inf Access Over SMB — Lateral Spread Persistence Indicator on internal server | 🟡 Medium | T1091 / T1570 — Lateral Tool Transfer |

### 3.2 Workstream B — Web Application Security

| ID | Finding | Severity | OWASP Category |
|---|---|:---:|---|
| F-01 | SQL Injection — Login Endpoint (boolean-based, stack trace leakage) | 🔴 Critical | A03:2021 — Injection |
| F-04 | Unrestricted File Upload — Web-accessible directory accepting `.php`/`.aspx` | 🔴 Critical | A05:2021 — Security Misconfiguration |
| F-02 | Stored XSS — Admin Dashboard via Customer Message Field | 🟠 High | A03:2021 — Injection |
| F-03 | Authentication Failure — No Rate Limiting + Username Enumeration via Password Reset | 🟠 High | A07:2021 — Identification & Authentication Failures |
| F-05 | IDOR — Sequential Customer Account IDs (`/account/1042`, `/account/1043`…) | 🟠 High | A01:2021 — Broken Access Control |

---

## 4. Workstream A — Network Forensics

### 4.1 Traffic Analysis Overview

| Metric | Value |
|---|---|
| Total Frames | 51,181 frames across 10 minutes 24 seconds |
| Total Data Volume | 26,423,689 bytes (~25.2 MB) |
| Primary Suspect Host | `10.1.21.58` (DESKTOP-ES9F3ML) — responsible for 99.9% of all observed frames |
| Internal Server Target | `10.1.21.2` — Active Directory / SMB |
| Primary External Threat | `153.92.1.49` (`whitepepper.su`) — infrastructure exhibting characteristics consistent with C2 activity |
| SMB/NBSS Volume | 22,301 frames (43.6%) — named-pipe enumeration activity |
| Suspicious DNS Domains | 7 high-risk domains across `.su`, `.sbs`, `.lat`, `.cc`, `.cyou` TLDs |

### 4.2 Key Findings Detail

**F-A1 — C2 Beaconing (Critical)**
Workstation `10.1.21.58` established repeated outbound HTTP and TLS connections to `whitepepper.su` (`153.92.1.49`), accessing the `/api/set_agent` endpoint with static agent credentials across both Chrome and Edge browser processes. This pattern is inconsistent with any identifiable legitimate enterprise software. Three encrypted TLS sessions totalling approximately 2.2 MB were observed, representing potential data exfiltration that cannot be content-inspected without endpoint access.

**F-A2 — SMB Reconnaissance (High)**
4,576 LSARPC frames and 2,538 SAMR frames were observed from `10.1.21.58` to the internal domain controller at `10.1.21.2`. LSARPC provides access to Local Security Authority policy data; SAMR enables enumeration of user accounts, groups, and password policy settings. The volume and pattern are not consistent with legitimate administrative activity.

**F-A3 — DNS to High-Risk TLDs (High)**
Seven suspicious domains were queried, including `whitepepper.su`, `arch.filemegahab4.sbs`, `media.megafilehub4.lat`, and `communicationfirewall-security.cc`. The `filemegahab4` root appearing across both `.sbs` and `.lat` TLDs is consistent with multi-domain C2 infrastructure or deliberate domain redundancy. None of these queries were blocked by any observed control.

**F-A4 — AutoRun.inf SMB Access (Medium)**
Access to `AutoRun.inf` on the internal SMB share at `10.1.21.2` was observed. While AutoRun functionality is disabled in modern Windows environments by default, the access pattern is consistent with a lateral spread mechanism or persistence probe.

### 4.3 Hypothesis Analysis

| Hypothesis | Evidence Basis | Verdict |
|---|---|:---:|
| H1: Malware C2 via browser injection | Static credentials across Chrome and Edge; `/api/set_agent` URI pattern; automated check-in behaviour | ✅ SUPPORTED |
| H2: Legitimate RMM / management agent | No enterprise RMM product matches the observed URI and credential pattern; no business justification for `whitepepper.su` | ❌ REFUTED |
| H3: Insider-installed exfiltration tool | `act=log` POST parameter consistent with structured data submission; cannot rule out without endpoint forensics | ⚠️ PARTIAL |

> *Final position: H1 and H3 are not mutually exclusive. Confirmation of transmitted data content requires endpoint forensic investigation.*

---

## 5. Workstream B — Web Application Security

### 5.1 Testing Surface

The Meridian FinServe customer portal is a B2C financial services web application with live customer records, account data, and PII. Testing covered authentication flows, session management, input handling, file upload processing, and object-level access controls. The approach was risk-prioritised: critical authentication and injection surfaces were assessed first.

### 5.2 Critical Findings

**F-01 — SQL Injection (Critical)**
The application's login endpoint passes user-supplied input directly to database queries without parameterisation. Boolean-based injection was confirmed, enabling authentication bypass and, with further exploitation, full database extraction. The application also returns stack-trace error output in failure responses, providing direct insight into the underlying database structure and technology stack to an attacker.

**F-04 — Unrestricted File Upload (Critical)**
File upload functionality accepts files without type validation. The application permitted upload of `.php` and `.aspx` files to a web-accessible directory. This is a direct path to Remote Code Execution (RCE) — an attacker who uploads a web shell gains interactive command execution on the underlying server.

### 5.3 High Severity Findings

**F-02 — Stored XSS (High)**
User-supplied content in the customer message field is rendered in the administrative dashboard without output encoding. Malicious script stored via a customer-facing input executes in an administrator's browser context, enabling session token theft and full account takeover.

**F-03 — Authentication Failure (High)**
The password reset endpoint permits username enumeration through differential response behaviour. No rate limiting is implemented on authentication endpoints, enabling unrestricted credential brute-forcing.

**F-05 — IDOR (High)**
Customer account records are accessed via sequential numeric IDs (`/account/1042`, `/account/1043`). No server-side authorisation check validates that the requesting user owns the target account. Any authenticated user can traverse the ID space and retrieve other customers' financial records.

---

## 6. Attack Path Analysis

The most significant output of this engagement is not any individual finding — it is the chain. The network and web application workstreams do not represent parallel, independent sets of problems. They represent sequential phases of a single breach scenario.

```
┌──────────────────────────────────────────────────────────────────────┐
│                        ATTACK PATH SUMMARY                           │
└──────────────────────────────────────────────────────────────────────┘

  Phase 1 │ INITIAL COMPROMISE
          │ C2 beaconing confirmed from 10.1.21.58 → whitepepper.su
          │ Encrypted TLS channel (~2.2 MB) — data already transiting C2
          │ Attacker has an established foothold
          ↓

  Phase 2 │ RECONNAISSANCE
          │ 4,576 LSARPC + 2,538 SAMR frames
          │ Active Directory accounts, groups, password policies enumerated
          │ DNS queries to .su/.lat/.sbs/.cc confirm multi-domain C2 redundancy
          ↓

  Phase 3 │ LATERAL MOVEMENT
          │ AutoRun.inf access on internal SMB share — persistence/propagation probe
          │ Enumerated credentials provide candidate accounts for adjacent systems
          ↓

  Phase 4 │ WEB APPLICATION EXPLOITATION
          │ SQLi on login endpoint → authentication bypass
          │ Unrestricted file upload → web shell → RCE
          │ IDOR on /account/ → full customer record traversal
          ↓

  Phase 5 │ CUSTOMER DATA EXPOSURE
          │ Financial records, PII, account credentials accessible
          │ Regulatory notification obligations may be triggered
          │ Financial services context amplifies breach impact
```

| Phase | Detail |
|---|---|
| **Phase 1 — Initial Compromise** | C2 beaconing confirmed from `10.1.21.58` to `whitepepper.su` via `/api/set_agent`. Encrypted TLS channel (~2.2 MB) suggests data is already transiting the C2. Attacker has an established foothold. |
| **Phase 2 — Reconnaissance** | 4,576 LSARPC + 2,538 SAMR frames enumerate Active Directory accounts, groups, and password policies. DNS queries to `.su`/`.lat`/`.sbs`/`.cc` TLD infrastructure confirm multi-domain C2 redundancy. |
| **Phase 3 — Lateral Movement** | AutoRun.inf access on the internal SMB share indicates a persistence or propagation probe. Enumerated credentials from Phase 2 provide candidate accounts for lateral movement to adjacent systems. |
| **Phase 4 — App Exploitation** | SQLi on the login endpoint enables authentication bypass. Unrestricted file upload to a web-accessible directory provides RCE. IDOR on `/account/` enables traversal of the full customer record set. |
| **Phase 5 — Data Exposure** | Customer financial records, PII, and account credentials are accessible. In a regulated financial services context this represents direct exposure under applicable data protection obligations. Regulatory notification obligations may be triggered. |

The attack path demonstrates that the two workstreams are not independent risk registers — they are complementary phases of a single, actionable compromise scenario.

---

## 7. Defense-in-Depth Recommendations

Findings were mapped to a six-layer control architecture rather than presented as a flat remediation list. Each layer addresses specific findings and can be implemented independently, with the combined effect of closing all identified attack paths.

| Layer | Control Domain | Recommended Action | Findings Addressed |
|:---:|---|---|:---:|
| **L1** | Endpoint Detection | Deploy EDR with C2 beaconing pattern detection and browser extension allowlisting via MDM (Intune/SCCM) | F-A1, F-A4 |
| **L2** | Network Segmentation | Firewall rule: workstations cannot initiate SMB (port 445) to domain controllers directly. Restrict SAMR/LSARPC named pipe access to authorised admin hosts only. | F-A2, F-A3 |
| **L3** | DNS Filtering & Threat Intel | Egress DNS firewall blocking `.su`/`.cc`/`.lat`/`.sbs`/`.cyou` TLDs. DNS query logging to SIEM with TLD anomaly alerting. HTTP egress allowlist denying arbitrary outbound port 80. | F-A1, F-A3 |
| **L4** | Web Application Controls | Parameterised queries everywhere (no exceptions). Strict file-type allowlist with server-side validation. Output encoding on all user-supplied content. WAF deployment. | F-01, F-02, F-04 |
| **L5** | Identity & Access Controls | MFA on all customer-facing authentication. Rate limiting on login and password-reset endpoints. Server-side authorisation check on every object-level access request. | F-03, F-05 |
| **L6** | Detection & Response | SIEM rules: SMB named-pipe access alerting from workstations, TLD anomaly detection, beaconing interval detection. Isolation playbook for hosts exhibiting C2 patterns. | F-A1, F-A2, F-A3 |

---

## 8. Prioritised Remediation Plan

| Priority | Action | Finding | Effort |
|:---:|---|:---:|:---:|
| 🔴 **P1 — Now** | Isolate `10.1.21.58` — block outbound HTTP/TLS to `153.92.1.49` and `whitepepper.su` pending endpoint forensics | F-A1 | S — Hours |
| 🔴 **P1 — Now** | Block high-risk TLDs (`.su`, `.sbs`, `.cc`, `.cyou`) at DNS egress firewall | F-A3 | S — Hours |
| 🔴 **P1 — Now** | Replace all dynamic SQL queries with parameterised queries / prepared statements on the login endpoint | F-01 | M — Days |
| 🔴 **P1 — Now** | Disable file upload to web-accessible directory; implement strict server-side type allowlist (MIME + extension) | F-04 | M — Days |
| 🟠 **P2 — Soon** | Add server-side authorisation check on every `/account/` object access; remove sequential ID exposure | F-05 | M — Days |
| 🟠 **P2 — Soon** | Implement output encoding on all user-supplied content rendered in the admin dashboard | F-02 | S — Days |
| 🟠 **P2 — Soon** | Add rate limiting to login and password-reset endpoints; normalise response behaviour to prevent enumeration | F-03 | S — Days |
| 🔵 **P3 — Plan** | Deploy EDR, restrict workstation SMB to DC, implement SIEM alerting on LSARPC/SAMR and beacon patterns | F-A2, F-A4 | L — Weeks |

> *Effort guide: **S** = Small (hours to 1 day) · **M** = Medium (2–5 days) · **L** = Large (1–3 weeks)*
> *P1 actions should be completed before any further customer data processing is accepted.*

---

## 9. MITRE ATT&CK Coverage

| Technique ID | Name | Workstream |
|:---:|---|:---:|
| T1071 | Application Layer Protocol (C2 over HTTP/TLS) | A — Network Forensics |
| T1071.004 | DNS — Suspicious TLD queries to attacker infrastructure | A — Network Forensics |
| T1087 | Account Discovery via LSARPC/SAMR named pipes | A — Network Forensics |
| T1018 | Remote System Discovery | A — Network Forensics |
| T1091 | Replication Through Removable Media (AutoRun.inf) | A — Network Forensics |
| T1570 | Lateral Tool Transfer | A — Network Forensics |
| T1190 | Exploit Public-Facing Application (SQLi, File Upload) | B — Web Application |
| T1078 | Valid Accounts — Authentication Bypass via SQLi | B — Web Application |
| T1059 | Command and Scripting Interpreter (File Upload → RCE) | B — Web Application |

---

## 10. Conclusions

KAVACH delivered a complete, evidence-based picture of the Meridian FinServe security posture across both network and application layers. The assessment identified evidence supporting a realistic, multi-stage attack scenario, with each phase supported by observed network activity and validated application 
findings.

The attack path narrative is the key output of the engagement: a  internal workstation exhibiting behaviour consistent with  C2 communications and ongoing internal reconnaissance, operating in the same environment as a financial services application carrying three critical application-layer vulnerabilities. An attacker working both angles has a network foothold and application exploitation capabilities that individually would be serious — combined, they represent a credible, end-to-end path to customer data exposure and regulatory breach.

The remediation roadmap is structured to close the most severe and immediately exploitable paths first. P1 actions — workstation isolation, TLD blocking, SQL injection remediation, and file upload hardening — should be treated as incident response activities, not scheduled development work.

Post-remediation validation testing is recommended to confirm closure of all identified paths before the environment re-enters production use with customer data.

---

<div align="center">

**Team Zero Trace — Project KAVACH — June 2026**

*This assessment was conducted as a structured exercise for educational and professional development purposes.*
*Meridian FinServe is a fictional organisation. No real systems were accessed.*

[![GitHub](https://img.shields.io/badge/GitHub-indrajitisingh-181717?logo=github)](https://github.com/indrajitisingh/kavach-cybersecurity-assesment)

</div>
