# Vendor Security Questionnaire

**Assessing organisation:** Meridian Health Analytics
**Vendor under review:** _"BrightSlot"_ — a cloud-based appointment scheduling SaaS
Meridian is considering embedding into its partner-clinic portal.
**Data the vendor would touch:** clinic staff contact details, appointment metadata
(no clinical content).
**Questionnaire basis:** condensed, SIG-Lite-style. Vendor responses are captured in
this file; scoring is in `vendor-risk-scorecard.csv`; the recommendation memo is at the
bottom.

Response key: **Y** / **Partial** / **N** / **N/A**, plus evidence or commentary.

---

## A. Governance & Compliance

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| A1 | Do you hold a current SOC 2 Type II report? | | |
| A2 | Do you hold ISO/IEC 27001 certification? | | |
| A3 | Is there a named security owner (CISO / equivalent)? | | |
| A4 | Are security policies reviewed at least annually? | | |
| A5 | Do you carry cyber liability insurance? | | |

## B. Data Handling & Privacy

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| B1 | Where (regions) is our data stored and processed? | | |
| B2 | Is customer data logically segregated between tenants? | | |
| B3 | What is your data retention and secure deletion process? | | |
| B4 | Will our data be used for any secondary purpose (e.g. model training)? | | |
| B5 | Do you sign a BAA / DPA as required? | | |

## C. Encryption

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| C1 | Is data encrypted at rest (algorithm / key management)? | | |
| C2 | Is data encrypted in transit (TLS version, enforced)? | | |
| C3 | Who controls encryption keys — you, us, or a KMS? | | |

## D. Access Control

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| D1 | Is MFA enforced for all employee access to production? | | |
| D2 | Is access granted on least-privilege and reviewed periodically? | | |
| D3 | How is privileged / admin access logged and monitored? | | |
| D4 | Is SSO / SCIM available for our administrators? | | |

## E. Incident Response

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| E1 | Do you have a documented, tested incident response plan? | | |
| E2 | What is your breach notification SLA to customers? | | |
| E3 | Have you had a security incident in the last 24 months? | | |

## F. Subprocessors & Supply Chain

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| F1 | Do you maintain a public list of subprocessors? | | |
| F2 | How are subprocessors security-assessed and monitored? | | |
| F3 | Are customers notified before new subprocessors are added? | | |

## G. Business Continuity & Resilience

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| G1 | What are your RTO and RPO targets? | | |
| G2 | Are backups encrypted and restore-tested? | | |
| G3 | Is the BC/DR plan tested at least annually? | | |

## H. Application & Infrastructure Security

| # | Question | Response | Evidence / Notes |
|---|----------|----------|------------------|
| H1 | Do you perform annual third-party penetration testing? | | |
| H2 | Can you share a summary / attestation of the latest test? | | |
| H3 | Is there a secure SDLC with code review and dependency scanning? | | |
| H4 | Do you run a vulnerability disclosure or bug bounty program? | | |

---

## Recommendation memo

**Overall vendor risk rating:** _Low / Medium / High_ (from `vendor-risk-scorecard.csv`)

**Decision:** _Approve / Approve with conditions / Reject_

**Rationale:**
_2–4 sentences._

**Required conditions / compensating controls (if approved with conditions):**
- _e.g. contractual breach-notification SLA of 72h_
- _e.g. annual review of SOC 2 report_
- _e.g. restrict integration to non-clinical data fields only_

**Reassessment trigger / cadence:** _e.g. annually, or on any material security incident._
