# Project KAVACH — Web Application Security Assessment · Workstream B

> **Risk Rating: 🔴 HIGH** · Assessor: indrajitisingh · Date: 2026-06-09 · Status: Complete

---

## What We Tested

The Meridian FinServe customer-facing web application — covering authentication, authorization, input validation, session handling, file uploads, and data access controls. Testing combined manual techniques with vulnerability validation against the simulation environment.

---

## What We Found

Five vulnerabilities. Two are critical. All are exploitable.

| ID | Finding | Severity |
|---|---|---|
| F-01 | SQL Injection | 🔴 Critical |
| F-02 | Cross-Site Scripting (XSS) | 🟠 High |
| F-03 | Authentication Failure | 🟠 High |
| F-04 | Unrestricted File Upload | 🔴 Critical |
| F-05 | Insecure Direct Object Reference (IDOR) | 🟠 High |


| Finding | OWASP Top 10 Category |
|----------|----------------------|
| SQL Injection | A03:2021 – Injection |
| Cross-Site Scripting | A03:2021 – Injection |
| Authentication Failure | A07:2021 – Identification & Authentication Failures |
| Unrestricted File Upload | A05:2021 – Security Misconfiguration |
| IDOR | A01:2021 – Broken Access Control |


**F-01 — SQL Injection:** Input wasn't sanitized before hitting the database. An attacker can read customer records, bypass authentication, or dump the entire DB.

**F-04 — Unrestricted File Upload:** The app accepted files without validation. That's a straight path to remote code execution and full server compromise.

**F-05 — IDOR:** Authorization checks were missing on object references. One user could pull another user's data just by changing an ID in the request.

**F-02 — XSS:** User content rendered without encoding. Session hijacking, credential theft, account takeover.

**F-03 — Auth Failure:** Weak authentication controls. Easier account compromise, privilege abuse.

---

## The Real Risk: It's a Chain

These don't exist in isolation. SQLi gets initial access. File upload gets code execution. IDOR gets the data. Together these findings create a viable attack chain that could enable unauthorized access, sensitive data exposure, privilage escalation, and full application compromise.

This also connects directly to **Workstream A**: the network investigation found active reconnaissance and C2 communications on the same environment. An attacker working both angles has network footholds *and* application vulnerabilities to exploit.

---

## What To Do

| Priority | Action |
|---|---|
| 🔴 Immediate | Fix SQL Injection — parameterized queries, no exceptions |
| 🔴 Immediate | Lock down file uploads — strict type validation and allowlists |
| 🔴 Immediate | Add server-side authorization checks to every object access |
| 🟠 High | Output encoding + input validation to close XSS |
| 🟠 High | Strengthen auth and session management |
| 🟡 Medium | Centralized app logging + secure code review programme |

---

## Bottom Line

Five vulnerabilities, two critical, all exploitable, and they chain together cleanly. Combined with the network findings from Workstream A, the Meridian FinServe environment has realistic end-to-end attack paths that need to be closed.

---
*Project KAVACH · Meridian FinServe Simulated Environment · Workstream B*
