# Cybersecurity GRC Portfolio Project

> **Status:** Phase 2 complete (asset inventory, Prowler scan, 16-risk register) — Phase 3 (NIST CSF 2.0 mapping) next.

An end-to-end GRC engagement built around a small, real AWS environment. It walks a
finding from cloud misconfiguration → risk register → NIST CSF 2.0 control mapping →
remediation → verified fix, and adds two independent GRC workstreams (a mock vendor
security assessment and a SOC 2 report review).

## Scenario — the fictional company

**Meridian Health Analytics** is an early-stage healthtech startup (~25 staff) that
ingests de-identified clinical data from partner clinics and returns population-health
dashboards. It runs entirely on AWS, holds data its customers consider sensitive, and
is starting to face security questionnaires from prospects — so it needs a defensible
risk picture, a control framework, and a remediation plan.

Every artifact in this repo is written as if produced for Meridian, to give the work a
real business context rather than a checklist feel.

## Skills demonstrated

- Cloud risk assessment on a live AWS environment (simplified NIST SP 800-30)
- Risk register construction and qualitative risk scoring
- Translating raw risks into NIST CSF 2.0 (Govern / Identify / Protect / Detect / Respond / Recover)
- Third-party / vendor risk management (SIG-Lite-style questionnaire + scorecard)
- SOC 2 Type II report literacy (scope, TSC, exceptions, customer impact)
- Remediation planning and prioritisation, with verified before/after fixes
- Infrastructure as Code with Terraform *(if used)*

## Repo navigation

| Folder | Contents | Phase |
|---|---|---|
| [`01-aws-lab/`](01-aws-lab/) | AWS lab build: architecture diagram, Terraform / console steps, lab notes, deliberate misconfigurations | 1 |
| [`02-risk-assessment/`](02-risk-assessment/) | Asset inventory, risk register, risk assessment methodology | 2 |
| [`03-nist-csf-mapping/`](03-nist-csf-mapping/) | Each risk mapped to a CSF 2.0 subcategory with current vs. target maturity | 3 |
| [`04-vendor-assessment/`](04-vendor-assessment/) | Vendor security questionnaire, completed responses, risk scorecard, recommendation memo | 4 |
| [`05-soc2-review/`](05-soc2-review/) | Structured review notes on a sample SOC 2 Type II report | 5 |
| [`06-remediation-plan/`](06-remediation-plan/) | Remediation recommendations, prioritised roadmap, evidence of implemented fixes | 6 |
| [`docs/`](docs/) | Screenshots and supporting images referenced across the repo | all |

## Key findings summary

16 risks identified (2 critical, 4 high, 8 medium, 2 low) — 5 built into the lab by design,
11 surfaced by an independent Prowler scan. Full register:
[`02-risk-assessment/risk-register.csv`](02-risk-assessment/risk-register.csv). Top risks:

| # | Risk | Asset | Score | CSF subcategory | Treatment |
|---|------|-------|-------|-----------------|-----------|
| R-01 | Public S3 bucket → customer report exfiltration | Customer reports (S3) | 25 · Critical | _Phase 3_ | Mitigate |
| R-02 | AdministratorAccess role on internet-facing web host (chain) | meridian-ec2-app-role / meridian-web | 20 · Critical | _Phase 3_ | Mitigate |
| R-03 | SSH open to the internet | web-sg | 12 · High | _Phase 3_ | Mitigate |
| R-04 | IMDSv1 enabled (metadata credential theft) | meridian-web | 12 · High | _Phase 3_ | Mitigate |
| R-16 | Build/admin IAM user without MFA | Callum-v2 | 12 · High | _Phase 3_ | Mitigate |
| R-06 | Privilege-escalation path to administrator | Callum-v2 → admin role | 10 · High | _Phase 3_ | Mitigate |

## Tools & frameworks referenced

NIST CSF 2.0 · NIST SP 800-30 · AWS (VPC, EC2, S3, IAM, CloudTrail, Config, GuardDuty) ·
Terraform · Prowler / ScoutSuite *(if used)* · SIG Lite · SOC 2 (AICPA TSC)

## Spreadsheet artifacts

Deliverables the roadmap lists as `.xlsx` are kept here as **`.csv`** so they diff
cleanly in version control. Column headers are pre-filled; open in Excel / Sheets and
export to `.xlsx` for the final portfolio if preferred.

## Safety notes

- No real personal or clinical data — all data is synthetic.
- AWS resources are destroyed / stopped when not in active use to avoid charges.
- Deliberate misconfigurations are documented in [`01-aws-lab/lab-notes.md`](01-aws-lab/lab-notes.md).
