# Remediation Plan — Meridian Health Analytics Cloud Lab

Companion narrative to [`remediation-roadmap.csv`](remediation-roadmap.csv) (the detailed,
per-risk register) and [`implemented-fixes.md`](implemented-fixes.md) (before/after evidence
for fixes already applied).

## Approach

- **Prioritise by risk score**, then by effort — the two Criticals first, then the
  cluster of Highs, then Mediums, with Lows folded into whichever window they fit cheaply.
- **Every recommendation ties to a NIST CSF 2.0 subcategory** (from Phase 3) so remediation
  moves a specific maturity gap, not just "a finding".
- **Sequence the identity/access work together.** 8 of 16 risks sit in PR.AA-* — one
  account-wide MFA-enforcement policy, one permissions-boundary rollout and one credential
  clean-up close most of them at once.
- **Close the loop with evidence.** Each fix is verified (config read-back or an external
  check) and screenshotted before/after.

## Progress

| | Count | Risks |
|---|---|---|
| Remediated | 2 | R-01 (Critical), R-04 (High) |
| Open — 0–30 day window | 8 | R-02, R-03, R-16, R-06, R-05, R-09, R-12, R-13 |
| Open — 30–90 day window | 6 | R-10, R-11, R-15, R-08, R-07, R-14 |

Residual exposure once the 0–30 day window is complete: both Criticals and all but one High
(R-06 depends on the permissions-boundary work) are closed.

## Phased timeline

### Window 1 — 0–30 days (contain the Criticals and the cheap high-value fixes)

| Risk | Action | Effort | Cost |
|---|---|---|---|
| R-02 | Scope `meridian-ec2-app-role` down from AdministratorAccess; permissions boundary | M | none |
| R-03 | Remove `0.0.0.0/0` SSH on `web-sg`; move to SSM Session Manager | S | none |
| R-16 | Enforce MFA on `Callum-v2` | S | none |
| R-06 | Remove the privilege-escalation inline policies from `Callum-v2` | S–M | none |
| R-05 | Register MFA for `meridian-analyst-1`; account-wide deny-without-MFA policy | S | none |
| R-09 | Enable VPC flow logs | S | ~$1–3/mo |
| R-12 | Strengthen the IAM password policy | S | none |
| R-13 | Enable KMS key rotation (quick win) | S | negligible |
| R-04 | Set the account-level IMDSv2 default (instance already done) | S | none |

### Window 2 — 30–90 days (detection, credential hygiene, encryption)

| Risk | Action | Effort | Cost |
|---|---|---|---|
| R-10 | CloudTrail → CloudWatch Logs; CIS metric-filter alarms; consider GuardDuty | M | ~$5–20/mo |
| R-11 | Retire `Callum-v2` static keys; move to short-lived / SSO credentials | M | none |
| R-15 | HTTPS-only bucket policies; versioning + access logging on the log bucket | S | negligible |
| R-08 | Account-level EBS default encryption; re-create volumes encrypted | M | negligible |
| R-07 | Hardware MFA for root; stop routine root use; alert on root sign-in | S | ~$25–50 |
| R-14 | Review and delete stale roles; start a quarterly IAM access review | S | none |

### Window 3 — 90+ days (structural)

- Complete the move to **IAM Identity Center / federation** for all human console access and
  retire static IAM users (`Callum-v2`, `meridian-analyst-1`) — finishes R-11 and R-16.
- Stand up a lightweight **detection-and-response runbook** so the R-10 alarms have an owner
  and a process (this is where the DETECT → RESPOND gap from the Phase 3 heat map is closed).
- Adopt **AWS Config** or a scheduled Prowler run in CI as a recurring control check, so new
  drift is caught without a manual scan.

## Cost summary

Recurring: roughly **$10–25 / month** at lab scale (flow-log storage, CloudWatch alarms,
optional GuardDuty). One-off: **~$25–50** for a hardware security key. Everything else is
configuration change at no additional AWS cost.

## Ownership

| Owner (Meridian fictional org) | Risks |
|---|---|
| Security team | R-02, R-05, R-06, R-09, R-10, R-11, R-12, R-13, R-14, R-15, R-16 |
| Platform team | R-03, R-04, R-08 |
| Data team | R-01, R-15 |
| Account owner | R-07 |

## Reassessment

Re-run Prowler after Window 1 and again after Window 2; update
[`../02-risk-assessment/risk-register.csv`](../02-risk-assessment/risk-register.csv) and the
Phase 3 maturity ratings. Target end state: no Critical or High open, DETECT subcategories at
maturity 2+.
