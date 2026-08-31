# NIST CSF 2.0 — Risk Heat Map & Maturity Gaps

Source: [`risk-to-csf-mapping.csv`](risk-to-csf-mapping.csv) · 16 risks from
[`../02-risk-assessment/risk-register.csv`](../02-risk-assessment/risk-register.csv).

## Maturity scale

| 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| Not implemented | Partial / ad hoc | Defined but inconsistent | Managed & consistent | Optimised / automated |

Target maturity assumes a small healthtech startup — "managed and consistent" (3) for most
controls, "optimised" (4) only where the blast radius justifies it (root, admin identities).

## Heat map — CSF Subcategory × risk severity

Cells list the Risk IDs that fall in each Subcategory / severity band.

| CSF Subcategory | Critical (16–25) | High (10–15) | Medium (5–9) | Low (1–4) |
|---|---|---|---|---|
| **PR.DS-01** Data-at-rest protected | **R-01** | | R-08 | R-13 |
| **PR.AA-05** Least privilege & SoD | **R-02** | R-06 | | R-14 |
| **PR.IR-01** Networks protected from unauthorized access | | R-03 | | |
| **PR.PS-01** Secure configuration management | | R-04 | | |
| **PR.AA-03** Users/services authenticated (MFA) | | R-16 | R-05 | |
| **PR.AA-01** Identities & credentials managed | | | R-07, R-11, R-12 | |
| **PR.DS-02** Data-in-transit protected | | | R-15 | |
| **DE.CM-01** Networks monitored | | | R-09 | |
| **DE.AE-02** Adverse events analysed | | | R-10 | |

## By CSF Function

| Function | Risks | Critical | High | Medium | Low |
|---|---|---|---|---|---|
| **PROTECT** | 14 | 2 | 4 | 6 | 2 |
| **DETECT** | 2 | 0 | 0 | 2 | 0 |
| GOVERN | 0 | — | — | — | — |
| IDENTIFY | 0 | — | — | — | — |
| RESPOND | 0 | — | — | — | — |
| RECOVER | 0 | — | — | — | — |

## Maturity gaps by Subcategory

| Subcategory | Risks | Current (range) | Target | Largest gap |
|---|---|---|---|---|
| PR.AA-05 Least privilege | R-02, R-06, R-14 | 0–1 | 3 | 3 (R-02, R-06) |
| PR.DS-01 Data-at-rest | R-01, R-08, R-13 | 0–2 | 3 | 3 (R-01) |
| PR.AA-03 Authentication / MFA | R-05, R-16 | 1 | 3–4 | 3 (R-16) |
| PR.AA-01 Credential management | R-07, R-11, R-12 | 1–3 | 3–4 | 2 (R-11, R-12) |
| PR.IR-01 Network access | R-03 | 1 | 3 | 2 |
| PR.PS-01 Secure config | R-04 | 1 | 3 | 2 |
| PR.DS-02 Data-in-transit | R-15 | 1 | 3 | 2 |
| DE.CM-01 Network monitoring | R-09 | 0 | 2 | 2 |
| DE.AE-02 Event analysis | R-10 | 1 | 3 | 2 |

## What the mapping shows

- **This is a PROTECT problem.** 14 of 16 risks are preventive-control failures, and they
  cluster in two places: **identity & access** (PR.AA-* carries 8 risks) and **data
  protection** (PR.DS-* carries 4). Fixing PR.AA-05 and PR.DS-01 first clears both
  Criticals.
- **Detection is almost absent.** Only R-09 and R-10 map to DETECT, and both are at
  maturity 0–1. The environment records API activity (CloudTrail) but nothing watches it and
  there is no network telemetry — an intrusion would likely go unnoticed.
- **GOVERN / RESPOND / RECOVER were not assessed.** A single-account technical lab has no
  policy, oversight, incident-response or recovery function to evaluate. Their absence is
  acknowledged, not scored — for Meridian in production these would be the next assessment
  scope (risk management strategy, IR runbooks, backup/restore testing).
- **Two quick wins** sit at gap 1: R-13 (enable KMS key rotation) and R-07 (hardware MFA +
  break-glass discipline for root).
