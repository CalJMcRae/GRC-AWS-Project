# 01 — AWS Lab Notes

**Company:** Meridian Health Analytics
**Environment purpose:** Small, realistic AWS footprint with *plausible* misconfigurations
to assess in Phase 2 — not hardened, not wide open.

---

## Architecture

See **[`architecture-diagram.md`](architecture-diagram.md)** — Mermaid diagram (renders on GitHub)
plus the misconfiguration legend and the hardened-element contrast. Optional: export a
polished `architecture-diagram.png` for the top of the repo README.

## Build method

- [x] Console-first, screenshotted → see [`console-setup-steps.md`](console-setup-steps.md)
- [ ] Terraform (IaC) → see [`terraform/`](terraform/) *(optional stretch goal)*

Screenshots go in [`../docs/screenshots/`](../docs/screenshots/).

## Environment facts

| | |
|---|---|
| AWS account | `328064416121` |
| Region | `us-west-1` (N. California) |
| AZs in use | `us-west-1a` (primary), `us-west-1c` (secondary, unused) |
| Build principal | `arn:aws:iam::328064416121:user/Callum-v2` |
| Cost guardrail | Budget `meridian-lab-monthly`, $5/mo, alerts to email — `00-budget.png` |

## Build log

| Date | Component | Identifier(s) | Screenshot | Notes |
|---|---|---|---|---|
| 2026-08-28 | VPC | `vpc-0637dab0dab26a5f4` (10.0.0.0/16) | `01-vpc-resource-map.png` | Built via "VPC and more" wizard, NAT gateways = None |
| 2026-08-28 | Public subnet | `meridian-subnet-public1-us-west-1a` = `subnet-0f5569d0f19eb0fe6` (10.0.0.0/20) | " | Route table `meridian-rtb-public` (`rtb-01cde09b71e11c9dc`) → `meridian-igw` |
| 2026-08-28 | Private subnet | `meridian-subnet-private1-us-west-1a` = `subnet-083d9c1a556e27767` (10.0.128.0/20) | " | Route table `meridian-rtb-private1-us-west-1a` (`rtb-0574d112658c79641`) — no `0.0.0.0/0` route (no NAT), DB host has no internet egress |
| 2026-08-28 | Internet gateway | `meridian-igw` = `igw-03d232decf13f3f45` | " | Attached to VPC |
| 2026-08-28 | Extra subnets (unused) | `...public2-us-west-1c` (`subnet-0c4c3de8f35d9da0b`, 10.0.16.0/20), `...private2-us-west-1c` (`subnet-06e449001fef70b48`, 10.0.144.0/20) | " | Created by wizard, no cost, no instances placed |
| 2026-08-28 | Security group `web-sg` | `sg-0c6a8765c52a2d56d` | `02-web-sg.png` | Inbound 80/443 from `0.0.0.0/0` (legit) **+ SSH 22 from `0.0.0.0/0` = M-01** |
| 2026-08-28 | Security group `db-sg` | `sg-0d8b174e89ca578a5` | `02-db-sg.png` | Inbound 5432 from source `web-sg` only — least-privilege contrast |
| 2026-08-28 | EC2 `meridian-web` | `i-0be139d0d1ed30215` (t3.micro, AL2023) | `03-ec2-web.png` | Public subnet `...public1-us-west-1a`, public IP `50.18.11.11`, private `10.0.13.28`, SG `web-sg`. **IMDSv2 = Optional → M-05.** No IAM role attached. httpd user-data. Stopped after build. |
| 2026-08-30 | EC2 `meridian-db` | `i-00b082701da0ffbdd` (t3.micro, AL2023) | `04-ec2-db.png` | Private subnet `subnet-083d9c1a556e27767 (...private1-us-west-1a)`, **no public IP**, private `10.0.133.145`, SG `db-sg`, IMDSv2 = Required (secure). First attempt `i-0c7e1b1762fd5af9c` landed in the public subnet — terminated and relaunched. Stopped after build. |
| 2026-08-30 | S3 bucket `meridian-customer-reports-328064416121` | us-west-1 | `05-s3-bucket.png`, `05-s3-object-public.png` | Holds `meridian-population-health-report-2026Q2.csv` (synthetic). **Bucket-level Block Public Access = Off; account-level Block Public Access = Off; public-read bucket policy (`s3:GetObject`, `Principal:*`) → M-02.** Verified anonymously readable: `GET .../meridian-population-health-report-2026Q2.csv` → HTTP 200, text/csv, 1135 bytes, no auth. |
| 2026-08-30 | IAM group `meridian-analysts` | `arn:aws:iam::328064416121:group/meridian-analysts` | `06-iam-group.png` | Policy `AmazonS3ReadOnlyAccess` (AWS managed). Good-practice contrast — scoped, permissions via group. |
| 2026-08-30 | IAM user `meridian-analyst-1` | member of `meridian-analysts` | `06-iam-user-no-mfa.png` | Console login profile (password reset required), **0 MFA devices → M-04**. No directly-attached policies. |
| 2026-08-30 | IAM role `meridian-ec2-app-role` | `arn:aws:iam::328064416121:role/meridian-ec2-app-role` | `06-iam-role.png`, `06-iam-role-attached-ec2.png` | `AdministratorAccess` attached, trust `ec2.amazonaws.com`. **Attached to `meridian-web` (`i-0be139d0d1ed30215`) → M-03**, chains with M-05. Verified via `aws iam list-attached-role-policies`. |
| 2026-08-31 | CloudTrail `meridian-trail` | `arn:aws:cloudtrail:us-west-1:328064416121:trail/meridian-trail` | `07-cloudtrail.png` | **Multi-region = Yes, logging ON, log file validation Enabled.** SSE-KMS enabled (key alias `meridian`, `key/2c4c6389-6e15-43e8-8c54-441869727451`, ~$1/mo). Logs → S3 `aws-cloudtrail-logs-328064416121-55e65cb1`. CloudWatch Logs not wired. Good-practice detective control. |

## Components

| Component | Represents (Meridian context) | Notes |
|---|---|---|
| VPC (public + private subnets) | Core network boundary | ✅ built — `vpc-0637dab0dab26a5f4`, see Build log |
| EC2 — `web` (public subnet) | Customer-facing dashboard host | ✅ `i-0be139d0d1ed30215` — carries M-05 + M-03 role |
| EC2 — `db` (private subnet) | Analytics database host | ✅ `i-00b082701da0ffbdd` — hardened contrast |
| S3 bucket — `customer-reports` | Stored population-health report exports | ✅ `meridian-customer-reports-328064416121` — M-02 (public) |
| IAM users / roles / groups | Staff + service access | ✅ group `meridian-analysts` (good), user `meridian-analyst-1` (M-04), role `meridian-ec2-app-role` (M-03) |
| Security groups | Host-level network rules | ✅ `web-sg` (M-01), `db-sg` (locked down) |
| CloudTrail | Audit logging evidence | ✅ `meridian-trail` — multi-region, validated, KMS-encrypted |
| AWS Config / GuardDuty *(optional)* | Auto-generated findings | ⏭️ Skipped by choice — Phase 2 uses Prowler (free, open-source). Avoids per-item / usage billing. |

## Deliberate misconfigurations built in

_These are "discovered" in Phase 2. Keep this list accurate as the lab evolves._

| ID | Component | Misconfiguration | Intended finding | Status |
|---|---|---|---|---|
| M-01 | Security group `web-sg` (`sg-0c6a8765c52a2d56d`) | SSH (22) open to `0.0.0.0/0` | Internet-exposed management port → brute force | ✅ built |
| M-02 | S3 `meridian-customer-reports-328064416121` | Block Public Access off (bucket **and** account level) + public-read bucket policy (`Principal:*`, `s3:GetObject`) | Anonymous internet download of customer reports → data exfiltration | ✅ built (verified public) |
| M-03 | IAM role `meridian-ec2-app-role` (trust: `ec2.amazonaws.com`) | `AdministratorAccess` (`*:*`) attached; role attached to internet-facing `meridian-web` | Least-privilege violation; **chains with M-05**: SSRF via IMDSv1 → steal admin session creds → account takeover | ✅ built |
| M-04 | IAM user `meridian-analyst-1` | Console login profile enabled, **no MFA device** | Credential compromise / phishing → unauthorised console access | ✅ built |
| M-05 | EC2 `meridian-web` (`i-0be139d0d1ed30215`) instance metadata | IMDSv1 left enabled (IMDSv2 = Optional) | SSRF → EC2 credential theft | ✅ built |

## Cost control

- [ ] Stay in free tier where possible (t2.micro / t3.micro, single-AZ)
- [ ] Stop or `terraform destroy` resources when not in use
- [x] Billing alert / budget configured — `meridian-lab-monthly`, $5/mo

## Disclosure / sensitive-data notes (for the public repo)

Screenshots in `docs/screenshots/` were reviewed before commit. What they contain and why it's acceptable:

| Item | In screenshots? | Assessment |
|---|---|---|
| AWS access keys, secret keys, session tokens, passwords | No | — |
| `meridian-key.pem`, `meridian-analyst-1_credentials.csv` | No (moved to `~/.aws/meridian-lab/`, git-ignored) | Not exposed |
| **AWS account ID `328064416121`** | Yes — nav bar, ARNs, bucket names | Low sensitivity; not a credential. **Accepted risk:** dedicated throwaway lab account, decommissioned at project end (see Teardown). Impractical to redact as it is embedded in resource names. |
| Resource IDs (`vpc-`, `sg-`, `i-`, KMS key id, role/user ARNs) | Yes | Not sensitive — useless without account credentials |
| Public IP `50.18.11.11` (was on `meridian-web`) | Yes | Ephemeral, released when instance stopped; a public IP is not a secret |
| Private IPs `10.0.x.x` | Yes | RFC1918, meaningless externally |
| Account owner name / `root` label | Yes | Portfolio is published under my name by design |
| Bucket name `meridian-customer-reports-328064416121` + its public state | Yes | Contains only synthetic data; remediated in Phase 6 |

## Teardown

_Full teardown runs at the end of the whole project (after Phase 6 before/after evidence is captured)._

Order:
1. Terminate EC2 `meridian-web` (`i-0be139d0d1ed30215`), `meridian-db` (`i-00b082701da0ffbdd`).
2. Empty then delete S3 buckets: `meridian-customer-reports-328064416121`, `aws-cloudtrail-logs-328064416121-55e65cb1`.
3. Delete CloudTrail `meridian-trail`; schedule deletion of KMS key alias `meridian`.
4. Delete IAM: user `meridian-analyst-1`, group `meridian-analysts`, role `meridian-ec2-app-role`, instance profile.
5. Delete `web-sg`, `db-sg`, then the VPC (`vpc-0637dab0dab26a5f4`) via "Delete VPC" (removes subnets, route tables, IGW).
6. Delete key pair `meridian-key`; delete local `~/.aws/meridian-lab/` secrets.
7. **Re-enable account-level S3 Block Public Access.**
8. Delete budget `meridian-lab-monthly`.
9. Consider closing the AWS account / rotating root credentials + enabling root MFA.
