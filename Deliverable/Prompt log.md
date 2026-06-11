# AI Prompt Log
## Cybersecurity Assessment Project — Team Zero Trace | CONFIDENTIAL

**Prepared:** June 2026 | **4 Members** | **40 Prompt Interactions**

---

> **Purpose of this log**
> This document records how AI tools were used by the four members of Team Zero Trace during the cybersecurity assessment engagement. Each entry captures the objective the member had in mind, the prompt they actually typed, and what came back. The intent is to make clear that every AI interaction was human-directed and human-validated — AI was used as a drafting and thinking aid, not as a replacement for professional judgement. All findings, conclusions, and recommendations in the final deliverables represent the team's own assessments.

---

## Member Responsibilities

| Member | Role | Core Responsibilities |
|---|---|---|
| **Indrajit Singh** | Team Lead & Lead Security Analyst | Project leadership, network forensics, web application security testing, threat modeling, attack-path analysis, risk assessment, executive reporting, remediation planning, final deliverable review |
| **Chaitanya** | Evidence & Repository Coordinator | Evidence management, screenshot verification, repository organisation, version-control coordination, artifact tracking, documentation support, submission readiness review |
| **M. Dubey** | Security Documentation Reviewer | Reviewing technical reports, validating documentation quality, checking consistency of findings, reviewing remediation recommendations, quality assurance, final report preparation support |
| **Pushpendra Sahu** | Reporting & Compliance Support | Report formatting, compliance review, deliverable verification, risk-rating consistency checks, executive-report support, quality assurance, final submission review |

---

---

## Indrajit Singh
### Team Lead & Lead Security Analyst


---

**Contribution Summary**

Indrajit led the engagement end-to-end — from initial scoping through to final deliverable sign-off. On the technical side, he drove both the network forensics workstream and the web application security testing, handling everything from PCAP triage and C2 analysis to injection testing and business logic review. Beyond the hands-on assessment work, he owned the threat model, the attack-path narrative, and the overall risk position presented to the client. Executive reporting, remediation prioritisation, and the final review of all outgoing documents sat with him. His prompts reflect the dual nature of the role — part analyst working through live evidence, part lead trying to make sure the whole thing hangs together as a coherent deliverable.

---

| # | Objective | Prompt | Outcome |
|---|---|---|---|
| 01 | Get a structured starting point before diving into the PCAP | *okay so i've got the capture file and i'm about to start going through it. before i do that help me put together a checklist of things i should actually be looking for — C2 indicators, lateral movement, credential theft stuff. keep it practical i don't need textbook definitions* | Generated a triage checklist covering beaconing intervals, DNS anomalies, SMB named pipe patterns, and LSARPC/SAMR activity — used as the opening framework for the network forensics workstream. |
| 02 | Figure out whether unusual DNS traffic is C2 or something benign | *i'm seeing a bunch of DNS queries going out to some weird TLDs — .su, .sbs, .cc and a couple others — from one internal machine. the timing is regular but not crazy in volume. is this C2 beaconing or could it be something legit? what would i need to see to confirm it either way* | Confirmed the pattern as consistent with C2 domain rotation. Provided secondary indicators to validate including JA3 fingerprints and TLS certificate anomalies — used to build the initial C2 finding. |
| 03 | Write up the C2 communication chain for a technical audience who hasn't seen the PCAP | *need to write this up for the report. workstation is 10.1.21.58, it's talking to a domain that resolves to an external IP, traffic is HTTP to a specific endpoint. how do i frame this so a technical reader who hasn't looked at the capture can follow it* | Produced a clear technical narrative covering the communication chain, IOCs, and endpoint significance — used as the base for the Phase 1 attack path section. |
| 04 | Explain what the LSARPC and SAMR activity actually means to a non-technical stakeholder | *i've got a lot of LSARPC and SAMR frames from the compromised machine going to an internal server. i know it's account enumeration but i need to be able to explain to someone non-technical exactly what information the attacker gets from this. help me frame that* | Drafted a plain-language explanation of what LSARPC and SAMR expose — user accounts, group memberships, password policies — and how that feeds into the next exploitation phase. Used in Phase 3 of the attack narrative. |
| 05 | Scope the web app test before touching a live financial services environment | *about to start the web app assessment on a live B2C financial portal — there's real customer data in there. help me think through the testing sequence given the risk profile, and flag anything i should specifically avoid doing on a live system* | Produced a risk-prioritised testing sequence starting with authentication, then injection points, then business logic — with explicit safety notes around live transaction endpoints. Used as the Workstream B engagement plan. |
| 06 | Work out the right severity for the SQL injection finding without actually exploiting it | *found what looks like a SQL injection on the login form — error messages are leaking stack traces, i'm getting boolean-based responses. i haven't pushed it on a live system. how do i document this as confirmed without pulling data, and what exploitation path do i describe* | Provided a documentation framework for confirmed-but-not-exploited SQLi, including the evidence threshold for Critical classification and a written exploitation narrative — filed as the lead finding in the web app workstream. |
| 07 | Validate the IOC list before sending it to the client | *before i hand the IOC list to the client i want to sanity check it. i've got the domain, the IP, and the C2 endpoint path. anything else i should derive from what i've described? and what format makes sense for the client — STIX, CSV, just a table* | Suggested additional derived IOCs including the full URL pattern and TLS fingerprint, and recommended a simple structured table for client delivery given the audience. Final IOC list formatted accordingly. |
| 08 | Write a threat model that's actually grounded in the engagement findings rather than generic | *i need to build the threat model for this environment based on everything we found. want it structured as assets, threat actors, entry points, key threats, business impact. help me populate from the actual findings rather than writing something boilerplate* | Produced a fully populated threat model pulling directly from both workstreams — used as the source document for the threat model deliverable and the visual threat model diagram. |
| 09 | Prepare talking points for the client debrief with the CTO | *we've got a call tomorrow, their CTO is on it and they want to know what the attacker was actually doing and whether they got to what they were after. give me five or six talking points i can use to structure the conversation — not scripted, just things i can work off* | Generated six plain-language talking points covering attacker timeline, confirmed vs inferred activity, and immediate containment advice — used to structure the debrief call. |
| 10 | Review the full attack path document before it goes out | *can you read through this attack path section and tell me if the logic holds — does the evidence actually support the conclusions or am i making any leaps a client could push back on. i want every claim tied to something observable* | Identified two claims needing stronger evidential grounding and one section where language overstated certainty — qualifier language suggested and incorporated before final submission. |

---


