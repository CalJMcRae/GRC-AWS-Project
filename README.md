# Cybersecurity GRC Portfolio Project

> **Status:** Phase 0 complete (repo skeleton) — Phase 1 (AWS lab) in progress.

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

_Populated at the end of Phase 2 / Phase 6._

| # | Risk | Asset | Score | CSF subcategory | Treatment |
|---|------|-------|-------|-----------------|-----------|
| _tbd_ | | | | | |

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
