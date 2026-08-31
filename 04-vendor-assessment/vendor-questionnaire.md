# Vendor Security Assessment — SlotHive

**Assessing organisation:** Meridian Health Analytics
**Vendor:** SlotHive, Inc. — cloud appointment-scheduling SaaS
**Assessment date:** 2026-09 · **Assessor:** Callum McRae (GRC)
**Questionnaire basis:** condensed SIG-Lite structure, 8 domains
**Outcome:** Medium-risk vendor — **Approve with conditions** (see [`recommendation-memo.md`](recommendation-memo.md))

---

## 1. Engagement scope

Meridian is evaluating SlotHive to power the appointment-booking widget inside its
partner-clinic portal. If adopted, Meridian would transmit to SlotHive via API:

| Data element | Type | Sensitivity |
|---|---|---|
| Clinic staff name, work email, work phone | PII (workforce) | Internal |
| Clinic location code | Reference | Internal |
| Appointment date / time | Operational | Internal |
| Appointment type code (e.g. `FOLLOWUP`, `SCREENING`) | Operational, quasi-identifying in combination | Internal → Confidential in aggregate |

**Not in scope / never transmitted:** patient names, dates of birth, contact details,
clinical notes, diagnoses, the population-health datasets (asset A-01).

**Integration model:** server-to-server REST API, OAuth client credentials, Meridian-issued
API key scoped to the booking endpoints. No end-user SSO into SlotHive by Meridian staff.

---

## 2. Questionnaire and vendor responses

Response key: **Y** / **Partial** / **N** / **N/A**. "Assessor note" is Meridian's view.

### A. Governance & Compliance

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| A1 | Current SOC 2 Type II report? | **Y** | Security, Availability, Confidentiality TSC. 12-month period ended 2026-06-30 (3 months before assessment). Unqualified opinion. | Solid. **Privacy TSC not in scope** despite handling PII — gap. Report is recent enough. |
| A2 | ISO/IEC 27001 certified? | **N** | "On the roadmap for 2027." | Acceptable for a Series A vendor given SOC 2 coverage. |
| A3 | Named security owner? | **Y** | Head of Security (reports to CTO). No dedicated CISO. | Reasonable for company size. |
| A4 | Security policies reviewed at least annually? | **Y** | Last review 2026-02. | OK. |
| A5 | Cyber liability insurance? | **Y** | $2M aggregate, $1M per-claim. | Adequate for the data volume in scope; confirm Meridian is not sub-limited. |

### B. Data Handling & Privacy

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| B1 | Where is data stored / processed? | **Y** | AWS `us-east-1` and `us-west-2`. US only. No offshore processing or support. | Good — no cross-border transfer issues. |
| B2 | Tenant data segregation? | **Partial** | Logical segregation — shared database, row-level `tenant_id` isolation. Not physically separate schemas/instances. | Common for SaaS at this stage; raises the impact of an isolation bug. Acceptable **with** the SOC 2 controls evidence. |
| B3 | Retention and secure deletion? | **Partial** | Production data purged 30 days after contract termination. **Backups retained 90 days** before rotation. | 90-day backup tail should be disclosed to clinics; acceptable if contractually capped and documented. |
| B4 | Any secondary use of customer data? | **Partial** | "Aggregated, de-identified usage analytics for product improvement." | **Concern.** Meridian does not want its data — even de-identified — in SlotHive's analytics. Must be contractually prohibited (condition). |
| B5 | Will you sign a BAA / DPA? | **Partial** | Signs a DPA. **Declines to sign a BAA** — "we do not handle PHI, so we are not a Business Associate." | **Key finding.** SlotHive's self-classification is arguable: appointment metadata tied to a clinic can be PHI-adjacent. Meridian's counsel should rule; failing a BAA, bind SlotHive to equivalent obligations by contract. |

### C. Encryption

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| C1 | Encryption at rest? | **Y** | AES-256, AWS KMS, vendor-managed keys. | Meets baseline. |
| C2 | Encryption in transit? | **Y** | TLS 1.2 minimum, enforced; HSTS; modern cipher suites. | Meets baseline. |
| C3 | Who controls the keys? | **Partial** | Vendor-managed KMS only. No customer-managed key (CMK / BYOK) option. | Expected at this tier; note as residual risk, not a blocker. |

### D. Access Control

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| D1 | MFA enforced for all employee production access? | **Y** | Enforced via Okta SSO; no direct local accounts on production. | Good. |
| D2 | Least privilege + periodic access review? | **Partial** | RBAC in place. Access reviews performed **annually**, joiner/mover/leaver process is partly manual. | Annual review is light for production access to customer data; quarterly preferred. |
| D3 | Privileged / admin access logged and monitored? | **Y** | CloudTrail + SIEM, alerting on privileged actions, 12-month log retention. | Good. |
| D4 | SSO / SCIM for customer administrators? | **Partial** | SAML SSO available. **No SCIM** — customer-side user deprovisioning is manual or via API. | Relevant only if Meridian staff get SlotHive logins; current integration model avoids this. Note for future. |

### E. Incident Response

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| E1 | Documented and tested IR plan? | **Y** | Plan documented; tabletop exercise annually; last run 2026-04. | OK. |
| E2 | Breach notification SLA to customers? | **Partial** | "Without undue delay." No committed hour figure. | **Gap.** Meridian needs a contractual maximum (<=72h from confirmation) — condition. |
| E3 | Security incident in the last 24 months? | **Y** | One (2025-11): a misconfigured internal logging bucket exposed application logs to another internal environment. No customer data involved. Remediated in 48h; root-cause published internally; added IaC guardrails and a config-scanning check. | Disclosed openly, which is a positive signal. The incident type (a cloud misconfiguration) is a yellow flag — probe their config-scanning maturity in a follow-up call. |

### F. Subprocessors & Supply Chain

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| F1 | Public subprocessor list? | **Y** | On the trust page: AWS (hosting), SendGrid (email), Twilio (SMS reminders), Datadog (observability), Stripe (billing). | Good transparency. Twilio/SendGrid are relevant — appointment reminders would carry staff contact + appointment time. |
| F2 | How are subprocessors assessed? | **Partial** | SOC 2 reviewed at onboarding. No continuous / annual re-assessment. | Weak — should be at least annual. |
| F3 | Notice before adding a subprocessor? | **Partial** | 30 days' notice by email + trust-page update. Customers "may raise concerns." No explicit right to terminate on objection. | **Gap.** Meridian should negotiate a termination right if it objects to a new subprocessor. |

### G. Business Continuity & Resilience

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| G1 | RTO / RPO targets? | **Y** | RTO 4 hours, RPO 1 hour. Multi-AZ; cross-region standby. | Appropriate for a scheduling tool. |
| G2 | Backups encrypted and restore-tested? | **Y** | Encrypted; restore tests quarterly with documented results. | Good. |
| G3 | BC/DR plan tested at least annually? | **Y** | Full failover test 2026-03. | Good. |

### H. Application & Infrastructure Security

| # | Question | Response | Vendor detail | Assessor note |
|---|---|---|---|---|
| H1 | Annual third-party penetration test? | **Y** | Annual; last test 2026-05 by a recognised firm. Highs/criticals remediated and retested. | Good. |
| H2 | Can you share a summary / attestation? | **Y** | Executive summary + attestation letter under NDA. Full report not shared. | Acceptable — summary + attestation is standard practice. |
| H3 | Secure SDLC? | **Partial** | Mandatory peer review, SAST in CI, dependency scanning (Dependabot). **No DAST**; no formal threat modelling. | Reasonable; note the DAST/threat-model gap. |
| H4 | Vulnerability disclosure / bug bounty? | **Partial** | Public VDP (security.txt + disclosure policy). No paid bug bounty. | Acceptable for company size. |

---

## 3. Domain assessment summary

| Domain | Score (0–4) | Basis |
|---|---|---|
| A. Governance & Compliance | 3 | Current SOC 2 Type II + insurance + named owner; no ISO 27001, no Privacy TSC |
| B. Data Handling & Privacy | 2 | Logical-only isolation, secondary use of data, BAA refused, 90-day backup tail |
| C. Encryption | 3 | Strong at rest and in transit; no customer-managed key option |
| D. Access Control | 3 | MFA/SSO + privileged logging solid; annual (not quarterly) reviews, no SCIM |
| E. Incident Response | 2 | Plan tested, but no committed breach SLA and a 2025 cloud-misconfig incident |
| F. Subprocessors & Supply Chain | 2 | Public list is good; no ongoing monitoring, weak objection/termination rights |
| G. Business Continuity & Resilience | 4 | RTO 4h / RPO 1h, restore-tested quarterly, annual failover test |
| H. Application & Infrastructure Security | 3 | Annual pen test + attestation, decent SDLC; no DAST or bug bounty |

Weighted total and rating: see [`vendor-risk-scorecard.csv`](vendor-risk-scorecard.csv) →
**2.60 / 4.00 = Medium-risk vendor**.

Recommendation and conditions: [`recommendation-memo.md`](recommendation-memo.md).
