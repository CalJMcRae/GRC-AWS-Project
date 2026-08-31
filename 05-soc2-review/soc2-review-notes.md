# SOC 2 Type II Report Review — SlotHive, Inc.

**Reviewer:** Callum McRae (GRC), for Meridian Health Analytics procurement
**Review date:** 2026-09
**Report reviewed:** SlotHive, Inc. — SOC 2 Type II, received under NDA during the Phase 4
vendor assessment ([`../04-vendor-assessment/`](../04-vendor-assessment/)).

> **Source note.** SlotHive is a simulated vendor, so this is an *illustrative* SOC 2 Type II
> report. Its structure, section order, opinion wording, control/test format, CUECs and
> exception language follow the **AICPA Illustrative SOC 2® Type 2 Report (SSAE No. 21)** and
> are consistent with publicly published sample reports (e.g. Secureframe's illustrative
> example, Sprinto/AICPA templates). Real SOC 2 reports are almost never fully public — they
> are shared under NDA — so practising against the illustrative report is the norm and is the
> constraint a GRC analyst actually works within.

---

## 1. Report identification

| Attribute | Value | Why it matters |
|---|---|---|
| Report type | **SOC 2 Type II** | Type I opines only that controls are *suitably designed* at a point in time. **Type II** also tests whether they *operated effectively* over a period — far more assurance for a customer. |
| Establishing wording | "…and the *operating effectiveness* of those controls to provide reasonable assurance…**throughout the period** 1 July 2025 to 30 June 2026" | The phrases "operating effectiveness" and "throughout the period" are what make it Type II rather than Type I. |
| Service auditor | Illustrative: a licensed CPA firm ("the service auditor") | Must be an independent CPA firm; check it is a real, reputable firm and not a low-cost paper mill. |
| Opinion | **Unqualified** ("…presented fairly, in all material respects… operated effectively throughout the period…") | Best outcome. A **qualified** opinion flags specific control failures; **adverse** = pervasive failure; **disclaimer** = auditor couldn't obtain sufficient evidence. |
| Examination period | **2025-07-01 to 2026-06-30** (12 months) | Full 12 months is standard. A short first-year period (e.g. 3–6 months) gives less assurance. |
| Report date | ~2026-08 (≈2 months after period end) | Normal lag. Combined with review in 2026-09, the report is ~3 months stale — see §6. |

---

## 2. Scope

### Trust Services Criteria in scope

| TSC | In scope? | Comment |
|---|---|---|
| Security (Common Criteria) | **Yes** | Mandatory in every SOC 2. |
| Availability | **Yes** | Appropriate — SlotHive is uptime-sensitive (booking widget). |
| Confidentiality | **Yes** | Appropriate — customer data is contractually confidential. |
| Processing Integrity | No | Defensible for a scheduling tool; note if booking accuracy ever becomes contractual. |
| **Privacy** | **No** | **Gap.** SlotHive processes PII (clinic-staff contact data). Privacy TSC out of scope means the report gives Meridian no assurance over notice, choice, retention, or disposal of personal data. Matches questionnaire finding B5 / A1. |

### System description (summary)

Cloud appointment-scheduling platform: multi-tenant web application and REST API, hosted on
AWS (us-east-1, us-west-2), PostgreSQL with row-level `tenant_id` isolation, supporting
services for email/SMS reminders, observability and billing. Description covers
infrastructure, software, people, procedures and data, plus a "system incidents during the
period" subsection.

- **System incident disclosed in the description:** one, 2025-11 — a misconfigured internal
  logging store exposed application logs to another internal environment; no customer data
  involved; remediated in 48h; IaC guardrails and config scanning added.
  *Cross-check:* consistent with questionnaire answer E3. Positive that it appears in the
  report at all.

### Subservice organizations — **carve-out method**

AWS (hosting) and the email/SMS providers are **carved out** — their controls are *not*
covered by this report. The report lists **Complementary Subservice Organization Controls
(CSOCs)** it assumes AWS performs (physical security, environmental controls, hardware
decommissioning, hypervisor isolation).

- **Action for Meridian:** obtain and review AWS's own SOC 2 / SOC 1 (via AWS Artifact) to
  cover the carved-out controls. The email/SMS subprocessors need their own assurance if
  reminders are ever enabled (ties to compensating control M5 in the Phase 4 memo).

---

## 3. Complementary User Entity Controls (CUECs)

Controls the report **assumes Meridian performs** — if Meridian doesn't, the auditor's
conclusions don't hold for Meridian.

| # | Illustrative CUEC | Can Meridian meet it? |
|---|---|---|
| 1 | User entities are responsible for provisioning/deprovisioning their own users and for periodic access reviews within the application. | Yes — but no SCIM (questionnaire D4), so this is the **manual quarterly check** already captured as compensating control M3. |
| 2 | User entities are responsible for configuring SSO and enforcing MFA for their administrators. | Yes — Meridian enforces MFA on its own IdP. |
| 3 | User entities are responsible for the accuracy and lawful basis of data they submit to the platform. | Yes — enforced by the field-level allow-list (memo condition C6). |
| 4 | User entities are responsible for protecting API credentials issued to them and rotating them. | Yes — captured as compensating control M1 (scope + 90-day rotation + secrets manager). |
| 5 | User entities are responsible for reviewing SlotHive security communications and acting on them. | Yes — assign an owner in GRC. |
| 6 | User entities are responsible for notifying SlotHive promptly of suspected unauthorized access. | Yes — add to the integration runbook. |

**Conclusion:** Meridian can satisfy all six, but three of them (1, 4, 6) only *because* of
compensating controls already identified in Phase 4. The SOC 2 review confirms those
controls are non-optional.

---

## 4. Sample of controls tested (Section 4 of the report)

Illustrative extract — format: TSC ref · control · test procedure · result.

| Ref | Control activity | Test performed | Result |
|---|---|---|---|
| CC1.1 | Code of conduct and security policies are acknowledged by personnel at hire and annually. | Inspected acknowledgement records for a sample of 45 of 210 personnel. | **Exception:** "1 of 45 new hires did not acknowledge the policies; 2 of 45 acknowledged them more than 30 days after their start date." |
| CC6.1 | Logical access to production requires SSO with MFA; no local accounts. | Inspected IdP configuration and attempted access without MFA for a sample of systems. | No exceptions. |
| CC6.2 | Access is provisioned via approved request and reviewed quarterly. | Inspected 3 of 4 quarterly access reviews and a sample of 25 provisioning tickets. | **Exception:** "For 1 of 4 quarters, the access review was completed 22 days after the scheduled date." |
| CC7.2 | Security events are logged to a SIEM and alerts are triaged within defined SLAs. | Inspected SIEM configuration and a sample of 25 alerts for triage evidence and timeliness. | No exceptions. |
| A1.2 | Backups run daily, are encrypted, and restoration is tested quarterly. | Inspected backup job logs and 4 quarterly restore-test records. | No exceptions. |
| C1.2 | Data is encrypted at rest (AES-256) and in transit (TLS 1.2+). | Inspected KMS configuration and TLS scan results. | No exceptions. |

---

## 5. Exceptions — what they mean for a customer relying on this report

| Exception | Severity for Meridian | Interpretation |
|---|---|---|
| 3 of 45 personnel acknowledgement failures/delays (CC1.1) | **Low** | Minor HR/onboarding hygiene issue, not a data-exposure control. Management response in Section 5 states the onboarding checklist now blocks system access until acknowledgement is recorded. Acceptable. |
| 1 of 4 access reviews completed 22 days late (CC6.2) | **Low–Moderate** | The review *was* performed, just late once. Because Meridian relies on quarterly access reviews (CUEC 1) and SlotHive has no SCIM, timeliness here matters. Ask SlotHive for the current-quarter review evidence and confirmation the calendar reminder/owner is fixed. |

No exceptions touched encryption, authentication, logging, backup or availability. There are
**no qualifications** to the opinion — the auditor judged the exceptions immaterial to
operating effectiveness overall.

---

## 6. Report currency (bridge letter)

Period ended **2026-06-30**; this review is **2026-09**. There is a ~3-month gap between the
period end and Meridian's reliance.

- **Action:** request a **bridge letter** (gap letter) from SlotHive management covering
  2026-07-01 to the go-live date, stating no material changes to the control environment and
  no security incidents. Standard practice; SlotHive should provide it on request.

---

## 7. Procurement assessment — what I would flag or ask

1. **Privacy TSC not in scope.** SlotHive holds PII but the report gives zero assurance over
   privacy controls. Ask for the Privacy TSC in the next examination, or accept via
   contractual DPA terms (Phase 4 condition C1).
2. **Bridge letter** for the July–go-live gap (§6).
3. **CC6.2 late access review** — request current evidence and the remediation detail (§5).
4. **Carved-out subservice orgs** — obtain AWS's SOC 2 via AWS Artifact; do not rely on this
   report for physical/environmental/hypervisor controls (§2).
5. **CUEC ownership** — assign a named Meridian owner for CUECs 1, 4, 5, 6 before go-live;
   these are the compensating controls from Phase 4, now confirmed mandatory.
6. **Confirm the service auditor** is a reputable licensed CPA firm and check for any prior
   restatements.
7. **Scope of the system description** — confirm the API and multi-tenant isolation controls
   Meridian depends on are explicitly inside the described system boundary, not adjacent to it.

## 8. Does the SOC 2 corroborate the Phase 4 questionnaire?

| Questionnaire claim | SOC 2 evidence | Verdict |
|---|---|---|
| Current SOC 2 Type II, unqualified (A1) | Confirmed — 12-month period, unqualified opinion | ✔ Corroborated |
| MFA enforced, no local prod accounts (D1) | CC6.1 tested, no exceptions | ✔ Corroborated |
| Quarterly access reviews (D2 said *annual*) | CC6.2 shows **quarterly** reviews (with one late) | ⚠ Report is *better* than the questionnaire answer — clarify which is current |
| Backups encrypted, restore-tested quarterly (G2) | A1.2 tested, no exceptions | ✔ Corroborated |
| Encryption at rest/in transit (C1, C2) | C1.2 tested, no exceptions | ✔ Corroborated |
| 2025 logging misconfiguration incident (E3) | Disclosed in the system description | ✔ Corroborated — and disclosed voluntarily |
| Privacy of PII | Privacy TSC **not in scope** | ✖ No assurance — residual risk |

**Bottom line.** The report supports an *approve-with-conditions* decision. It confirms the
core security, availability and confidentiality controls operate effectively, the two
exceptions are minor and have management responses, and the main gaps (Privacy TSC, carve-out
of AWS, a late access review, report currency) are all addressable through the bridge letter
and the contractual conditions already defined in the Phase 4 recommendation memo.
