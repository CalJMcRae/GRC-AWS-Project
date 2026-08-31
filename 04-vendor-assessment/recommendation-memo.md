# Vendor Risk Recommendation — SlotHive, Inc.

**To:** Meridian Health Analytics — Procurement & Leadership
**From:** GRC (Callum McRae)
**Date:** 2026-09
**Re:** Security assessment of SlotHive for the partner-clinic booking widget

---

## Recommendation

**Approve with conditions.** SlotHive is a **Medium-risk** vendor (weighted score
2.60 / 4.00). Its security programme is appropriate for a growth-stage SaaS — current SOC 2
Type II, enforced MFA, tested backups and pen testing — but there are specific gaps around
**data handling, contractual breach terms, and supply-chain oversight** that must be closed
in the contract before Meridian sends any data.

Do **not** sign on SlotHive's standard terms.

---

## Why not a straight approval

Three issues carry real business risk for a healthtech company:

1. **SlotHive uses customer data for its own analytics.** Even "de-identified," Meridian
   does not want partner-clinic scheduling patterns feeding a third party's product
   analytics. This is a contractual fix, not a technical one.
2. **SlotHive will not sign a BAA.** It argues it never handles PHI. That position is
   debatable — appointment type and clinic, linked over time, can be health-related
   information. If Meridian's counsel disagrees with SlotHive, Meridian is exposed.
3. **The breach-notification commitment is vague** ("without undue delay"). Meridian cannot
   meet its own downstream obligations to clinics without a firm number.

None of these is disqualifying. All are fixable in the contract.

## Why not a rejection

The fundamentals are sound: independent SOC 2 Type II with an unqualified opinion, US-only
hosting, encryption at rest and in transit, MFA with no local production accounts, quarterly
restore tests, annual third-party penetration testing with remediation. The one past
incident (a 2025 cloud misconfiguration exposing internal logs, no customer data) was
disclosed openly and followed by concrete guardrail improvements — a positive signal about
culture. The data in scope is limited to workforce contact details and appointment
metadata, not patient records.

---

## Conditions of approval (contractual)

| # | Condition | Addresses |
|---|---|---|
| C1 | Execute a Data Processing Agreement with HIPAA-aligned safeguards. Obtain SlotHive's written legal position on Business Associate status; if Meridian's counsel concludes a BAA is required, a BAA is a precondition to go-live. | BAA gap (B5) |
| C2 | Prohibit any secondary use of Meridian data, including aggregated or de-identified analytics, without separate written consent. | Secondary use (B4) |
| C3 | Contractual breach notification **within 72 hours** of SlotHive confirming a security incident affecting Meridian data, with a defined minimum content set. | Breach SLA (E2) |
| C4 | Right to audit (or to receive responses to an annual security questionnaire), plus annual delivery of the current SOC 2 report and subprocessor list. | Ongoing assurance (A1, F2) |
| C5 | Advance notice of new subprocessors with a **right to terminate without penalty** if Meridian reasonably objects. | Subprocessor rights (F3) |
| C6 | Data minimisation: the integration transmits only staff work-contact fields and appointment date / time / type code. No free-text location, no patient identifiers, enforced by field-level allow-list on Meridian's side. | Scope creep, isolation impact (B2) |
| C7 | Contractual cap on backup retention (<= 90 days post-termination) and written confirmation of deletion. | Retention tail (B3) |

## Meridian-side compensating controls

| # | Control | Addresses |
|---|---|---|
| M1 | Scope the SlotHive API credential to the booking endpoints only; rotate every 90 days; store in Meridian's secrets manager. | Credential compromise |
| M2 | Monitor the integration — request volume, error rate, off-hours activity — and alert on anomalies. | Detect misuse / breach early |
| M3 | Quarterly manual check that departed Meridian staff have no SlotHive access (no SCIM available). | Deprovisioning gap (D4) |
| M4 | Record the data flow in Meridian's Register of Processing Activities and review at each reassessment. | Privacy governance |
| M5 | Do not enable SMS/email appointment reminders (Twilio/SendGrid subprocessors) unless separately assessed. | Supply-chain scope |

## Residual risk after conditions

**Low–Medium.** Remaining exposure Meridian knowingly accepts:

- Logical (not physical) tenant isolation — mitigated by SlotHive's SOC 2 controls and by C6 data minimisation.
- No customer-managed encryption key — standard at this vendor tier.
- Subprocessor security is monitored only annually on Meridian's side (via C4).
- DAST and formal threat modelling absent from SlotHive's SDLC — pen testing partially compensates.

## Review cadence

- **Reassess annually**, or sooner on any of: a security incident affecting Meridian data;
  a material new subprocessor; a change of control / acquisition of SlotHive; loss or
  qualification of the SOC 2 report.
- Owner: GRC, with Procurement.
