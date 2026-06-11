# 🛡️ Project KAVACH — Cybersecurity Assessment
**Network Forensics · Web Application Security · Threat Modeling · Attack Path Analysis · Defense-in-Depth**

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Workstreams](https://img.shields.io/badge/Workstreams-2-orange)
![Findings](https://img.shields.io/badge/Findings-9%20Total-red)

---

## 📌 About This Project

KAVACH is a simulated end-to-end cybersecurity assessment of a fictional financial services environment — **Meridian FinServe**. It was conducted by **Team Zero Trace** as a structured, professional-grade security engagement covering everything from raw PCAP analysis and web application exploitation through to executive-level risk reporting.

The point wasn't just to list vulnerabilities. It was to understand how individual findings chain together into a realistic attack scenario, assess what that means for a business carrying live customer data, and produce deliverables that would hold up in front of both a technical team and a boardroom.

This repository is the full record of that work — findings, evidence, analysis, reports, and the methodology behind all of it.


---

## Executive Summary

The assessment identified an active compromise involving command-and-control communications from an internal workstation and multiple critical vulnerabilities within a customer-facing web application.

Correlation of network forensic evidence and application security findings demonstrated a credible attack path capable of enabling unauthorized access to sensitive customer information.

Overall Risk Rating: CRITICAL

---

## 👥 Team Zero Trace

| Name | Role |
|---|---|
| **Indrajit Singh** | Team Lead & Lead Security Analyst |
| **Chaitanya** | Evidence & Repository Coordinator |
| **M. Dubey** | Security Documentation Reviewer |
| **Pushpendra Sahu** | Reporting & Compliance Support |

---

## 🔍 Assessment Scope

### Workstream A — Network Forensics
Analysis of network capture files from a compromised internal environment. The focus was on identifying C2 communication patterns, internal reconnaissance activity, lateral movement indicators, and extracting a usable IOC set. Key tools: Wireshark, Tshark, JA3 hash analysis.

### Workstream B — Web Application Security
Security testing of a B2C financial services customer portal with live customer data. Tested for injection vulnerabilities, authentication failures, business logic flaws, and file handling weaknesses. Approach was risk-prioritised — authentication and injection first, then logic and access control.

### Supporting Analysis
- **Threat Modeling** — threat actors, entry points, key assets, business impact
- **Attack Path Analysis** — correlating network and application findings into a realistic end-to-end scenario
- **Defense-in-Depth Design** — layered control recommendations mapped directly to findings
- **Executive Reporting** — risk narrative and remediation priorities written for a mixed technical/management audience


---

## Engagement Statistics

| Metric | Value |
|----------|----------|
| Workstreams | 2 |
| Findings Identified | 9 |
| Critical Findings | 3 |
| High Findings | 5 |
| Medium Findings | 1 |
| MITRE Techniques Mapped | 5+ |
| OWASP Categories Mapped | 4 |
| Reports Produced | 8 |


---


## 🚨 Key Findings

### Workstream A — Network Forensics

| ID | Finding | Severity | MITRE Technique |
|---|---|---|---|
| F-A1 | C2 Beaconing — Compromised Internal Workstation (10.1.21.58) | 🔴 Critical | T1071 — Application Layer Protocol |
| F-A2 | SMB Reconnaissance — LSARPC/SAMR Enumeration (7,000+ frames) | 🟠 High | T1087 — Account Discovery |
| F-A3 | DNS to High-Risk TLDs (.su, .sbs, .cc, .lat) | 🟠 High | T1071.004 — DNS |
| F-A4 | AutoRun.inf Access Over SMB — Lateral Spread Indicator | 🟡 Medium | T1091 / T1570 |

### Workstream B — Web Application Security

| ID | Finding | Severity | OWASP Category |
|---|---|---|---|
| F-01 | SQL Injection — Login Endpoint (Boolean-based, stack trace leakage) | 🔴 Critical | A03 — Injection |
| F-02 | Stored XSS — Admin Dashboard via Customer Message Field | 🟠 High | A03 — Injection |
| F-03 | Auth Failure — No Rate Limiting + Username Enumeration via Password Reset | 🔴 Critical | A07 — Auth Failures |
| F-04 | Unrestricted File Upload — Web-Accessible Directory (.php/.aspx accepted) | 🔴 Critical | A04 — Insecure Design |
| F-05 | IDOR — Sequential Customer Account IDs (/account/1042, /account/1043…) | 🟠 High | A01 — Broken Access Control |

---

## ⛓️ Attack Path — How It All Connects

The most significant output of this engagement wasn't any single finding. It was the chain.

A compromised workstation with confirmed C2 activity and active internal reconnaissance, sitting in the same environment as a web application with three critical vulnerabilities and live customer financial data — that's not a list of separate problems. That's an end-to-end breach scenario.

```
┌─────────────────────────────────────────────────────────────┐
│                     ATTACK PATH SUMMARY                     │
└─────────────────────────────────────────────────────────────┘

  Phase 1 │ Initial Compromise
          │ C2 beaconing from 10.1.21.58 → whitepepper.su
          │ HTTP traffic to /api/set_agent endpoint
          ↓

  Phase 2 │ Internal Reconnaissance
          │ 4,576 LSARPC frames + 2,538 SAMR frames
          │ Account enumeration — users, groups, password policies
          ↓

  Phase 3 │ Credential Discovery & Lateral Movement
          │ AutoRun.inf access over SMB
          │ Potential propagation to adjacent hosts
          ↓

  Phase 4 │ Web Application Exploitation
          │ SQLi on login endpoint → Auth bypass
          │ IDOR on /account/ → Customer record access
          │ File upload → RCE capability
          ↓

  Phase 5 │ Customer Data Exposure
          │ Financial records, account data, PII at risk
          │ Regulatory exposure (financial services context)
```

Full narrative with evidence references: `/Attack-Path/`

---

## 🧱 Defense-in-Depth Framework

Findings were mapped to a six-layer control architecture rather than listed as a flat remediation list. Each layer addresses specific findings:

| Layer | Control | Findings Addressed |
|---|---|---|
| Layer 1 | EDR / Endpoint Detection | F-A1, F-A4 |
| Layer 2 | Network Segmentation | F-A2, F-A3 |
| Layer 3 | DNS Filtering & Threat Intel | F-A1, F-A3 |
| Layer 4 | Web Application Controls (WAF + Secure Dev) | F-01, F-02, F-04 |
| Layer 5 | MFA & Identity Controls | F-03, F-05 |
| Layer 6 | SIEM & Detection Rules | F-A1, F-A2, F-A3 |

Full control specifications: `/Defense-in-Depth/`

---

## 🛠️ Tools & Techniques

| Category | Used |
|---|---|
| Network Analysis | Wireshark, Tshark |
| Traffic Filtering | Beacon interval analysis, SMB named pipe isolation, DNS anomaly filters |
| Threat Intelligence | JA3 hash databases, MITRE ATT&CK framework |
| Web App Testing | Burp Suite, manual injection testing, business logic analysis |
| Documentation | Markdown, structured evidence logging, Git version control |
| Reporting | Technical + executive register writing, risk compounding narrative |

---

## 🗺️ MITRE ATT&CK Coverage

| Technique ID | Name | Workstream |
|---|---|---|
| T1071 | Application Layer Protocol (C2 over HTTP) | A |
| T1071.004 | DNS | A |
| T1018 | Remote System Discovery | A |
| T1087 | Account Discovery (LSARPC/SAMR) | A |
| T1091 | Replication Through Removable Media | A |
| T1570 | Lateral Tool Transfer | A |
| T1190 | Exploit Public-Facing Application | B |
| T1078 | Valid Accounts (Auth Bypass) | B |
| T1059 | Command and Scripting Interpreter (File Upload → RCE) | B |

---

## ## AI Usage Disclosure

AI-assisted tools were used to support documentation structuring, report formatting, and editorial review activities.

All security analysis, evidence validation, finding development, attack-path construction, and risk assessments were reviewed and validated by the assessment team prior to inclusion in project deliverables.

AI-generated content was treated as a drafting aid and was not accepted without analyst verification.


---


## 📖 How to Read This Repository

**Start with the attack path** (`/Attack-Path/`) — it gives you the full picture and makes the individual findings make sense in context.

**Then the technical report** (`/Reports/Technical-Report/`) — detailed findings with evidence, exploitation narratives, and remediation guidance.

**For the business risk framing** — the executive report (`/Reports/Executive-Report/`) is written for a non-technical audience and covers overall risk position and prioritised actions.

**For methodology and reflection** — `retro.md` covers what worked, what didn't, and what we'd do differently. The AI prompt log shows how the work was actually done.

---

## ⚠️ Disclaimer

This project was conducted as a structured cybersecurity assessment exercise for educational and professional development purposes. Meridian FinServe is a fictional organisation. All IP addresses, domain names, and identifiers used in this project are fictionalised. No real systems were accessed or harmed. This repository is published strictly for learning and portfolio purposes.

---

<div align="center">

**Team Zero Trace — Project KAVACH**

*Network Forensics · Web Application Security · Threat Modeling · Defense-in-Depth*

[![GitHub](https://img.shields.io/badge/GitHub-indrajitisingh-181717?logo=github)](https://github.com/indrajitisingh/kavach-cybersecurity-assesment)

</div>
