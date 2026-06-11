# Project Retrospective — Project KAVACH
### Team Zero Trace | Cybersecurity Assessment

---

## 1. Project Overview

KAVACH was a full-scope security assessment of the Meridian FinServe environment. The work spanned two technical workstreams — network forensics and web application security — along with threat modeling, attack path analysis, defense-in-depth design, and executive reporting. The goal going in was straightforward: find the weaknesses, understand how they chain together, assess what it actually means for the business, and give the client something they could act on. That's what we set out to do and broadly what we delivered.

---

## 2. Objectives Achieved

| Objective | Status | Notes |
|---|---|---|
| Network Forensics Investigation | Complete | Identified C2-consistent communications, DNS anomalies, SMB enumeration, and internal reconnaissance activity |
| Web Application Security Assessment | Complete | Confirmed SQL Injection, Stored XSS, Authentication Failures, Unrestricted File Upload, and IDOR |
| Threat Modeling | Complete | Mapped threat actors, entry points, key assets, and business impact across the environment |
| Attack Path Analysis | Complete | Connected network compromise to application vulnerabilities into a realistic end-to-end scenario |
| Defense-in-Depth Assessment | Complete | Designed layered control recommendations tied directly to identified findings |
| Executive Reporting | Complete | Delivered executive summaries, risk assessments, and prioritised remediation plans |

---

## 3. What We Actually Delivered

The headline is that we got through the full scope and everything we said we'd deliver got delivered. That doesn't always happen on a compressed timeline with a four-person team, so it's worth acknowledging.

The network forensics workstream turned up more than we expected going in. The C2 beaconing, the LSARPC and SAMR volume, the DNS pattern to high-risk TLDs — none of that was guaranteed when we started. The web app side found a proper vulnerability stack rather than a handful of low-severity issues, and the fact that those two workstreams connected cleanly into a single attack path made the final deliverable significantly more impactful than if they'd been kept separate.

The documentation quality held up. The evidence base was consistent, the findings were traceable, and the final report read as one coherent document rather than two separate reports with a staple through them. That took deliberate effort from the whole team, not just the analysts writing the findings.

---

## 4. Challenges We Hit

**The PCAP volume was larger than expected.** Getting through it efficiently required a structured triage approach from the start — filtering strategy, beacon interval analysis, protocol prioritisation. Without that, we'd have spent twice as long and probably missed the AutoRun.inf signal entirely.

**Separating noise from signal in the traffic analysis took longer than the timeline allowed for.** Establishing what normal looked like for this environment before trying to characterise what was abnormal was a step that needed more time than it got. We got there, but it was tighter than it should have been.

**Correlating findings across two workstreams into a single coherent attack narrative doesn't happen automatically.** Someone has to sit down and actually connect the dots. That required a dedicated piece of work that probably should have been scoped more explicitly from the start.

**The documentation side had a mid-engagement sync issue.** Findings were being updated in the draft while the evidence repository hadn't caught up. It was caught and resolved, but it added pressure in the final two days. A structured mid-engagement check-in between the technical and documentation sides would have prevented it.

---

## 5. Lessons Learned

Honestly the biggest one is that **network and application findings need to be analyzed together, not handed off sequentially**. We got the attack path narrative right in the end, but it would have been sharper if both workstreams had been cross-referencing throughout rather than doing it in one pass at the end.

**Threat modeling is more useful when it happens earlier.** We built the threat model from the findings, which meant it was accurate but slightly backwards — the findings shaped the model when ideally the model should have been helping prioritise what to look for. Next time it goes in at the start of the engagement.

**Evidence management needs to be treated as a first-class workstream, not a support activity.** The discipline Chaitanya brought to the repository structure and the pre-submission readiness checks was what kept the evidence base clean. That kind of work is easy to deprioritise under time pressure and you feel it immediately if you do.

**Plain language in executive reporting is harder than technical writing.** Getting the risk narrative right for a board-level audience — serious but not sensationalised, specific but not jargon-heavy — took more iterations than the technical sections did. Build more time into that part next time.

---

## 6. Technical Skills Developed

| Skill Area | What We Actually Practiced |
|---|---|
| Network Forensics & PCAP Analysis | Beacon interval detection, JA3 hash analysis, SMB named pipe isolation, DNS anomaly identification |
| Web Application Security Testing | SQLi confirmation without live exploitation, stored XSS in admin context, IDOR chaining, file upload RCE narrative |
| Threat Modeling | Asset mapping, threat actor profiling, entry point analysis, business impact framing |
| Attack Path Analysis | Multi-workstream finding correlation, phased compromise narrative, attacker objective mapping |
| Defense-in-Depth Design | Control-to-finding mapping across six security layers |
| Security Reporting | Technical and executive register writing, risk compounding narrative, before/after risk state framing |
| Risk Assessment | CVSS with environmental context, financial services risk multiplier, severity consistency review |
| Remediation Planning | Do/don't guidance for developer audiences, prioritised remediation ordering, conflicting recommendation resolution |

---

## 7. Team Reflection

The four-person structure worked because the roles were genuinely distinct. Indrajit carrying the technical lead and the strategic framing, Chaitanya keeping the evidence side disciplined, Dubey doing substantive documentation review rather than a light pass, and Pushpendra holding the reporting infrastructure together in the final stretch — it divided up the work in a way that made sense for the engagement.

The internal disagreement on one of the severity ratings was handled well. It was documented, discussed, and resolved cleanly, which is how that kind of thing should go. Worth mentioning because it's easy to let those slide rather than deal with them properly.

If there's one thing the team would change it's the point raised earlier — building a formal mid-engagement sync between the technical and documentation workstreams. The fact that we caught the evidence-report desync during the cross-reference check rather than after submission is a testament to the process, but it shouldn't have gotten to that point.

---

## 8. What Went Well

- Full scope delivered on time with no major gaps
- Evidence base remained traceable and consistent throughout
- Attack-path narrative successfully connected both workstreams
- Internal quality processes — documentation review, evidence validation, compliance checks — caught real issues before submission
- Executive report landed well with the client; the plain-language risk framing worked

---

## 9. What We'd Do Differently

- Start threat modeling at the beginning of the engagement, not at the end
- Build a mid-engagement cross-workstream sync into the timeline from day one
- Allocate more dedicated time for executive report writing — it takes longer than it looks
- Implement an artifact tracking log from day one rather than backfilling it
- Explore lightweight automation for IOC extraction and evidence cross-referencing

---

## 10. Recommendations for Future Engagements

Run threat modeling early and use it to inform testing priorities across both workstreams, not just to document findings after the fact. Keep evidence management as a defined role with structured checkpoints rather than something that happens alongside everything else. Build attack-path correlation time into the timeline explicitly — it's not something that happens in the margin. After remediation is implemented, plan for validation testing rather than treating delivery of recommendations as the end of the engagement.

---

## 11. Overall Learning Outcome

KAVACH gave the team practical, end-to-end experience across the full cybersecurity assessment lifecycle — not just individual technical skills in isolation, but the full pipeline from evidence capture through to executive communication. The part that's hardest to replicate in a training environment is exactly what this engagement delivered: the messiness of real evidence, the judgment calls on severity and certainty, the discipline of making sure technical findings translate into something a business can actually act on. That's what makes this one worth reflecting on properly.

---

## 12. Final Reflection

The thing about KAVACH is that the most important finding — the one that made the whole engagement matter — wasn't any single vulnerability. It was the attack path. A compromised workstation with C2 beaconing and internal recon activity sitting alongside a web application with critical authentication and injection flaws, in a financial services environment with live customer data. Individually each piece is serious. Together they're a data breach scenario with a realistic exploitation chain. Getting that correlation right and communicating it clearly to a client who needed to understand the urgency — that's what the engagement was actually for, and that's what the team delivered.

---

**Project Status:** Complete
**Team:** Zero Trace

**Assessment Outcome:** All primary workstreams delivered, documented, and submitted. Evidence base archived and signed off.
