# 01 — AWS Lab Notes

**Company:** Meridian Health Analytics
**Environment purpose:** Small, realistic AWS footprint with *plausible* misconfigurations
to assess in Phase 2 — not hardened, not wide open.

---

## Architecture

![Architecture diagram](architecture-diagram.png)

_Export a diagram (draw.io / Lucidchart) to `architecture-diagram.png` in this folder._

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
| 2026-08-28 | Public subnet | `meridian-subnet-public1-us-west-1a` (10.0.1.0/24) | " | Route table `meridian-rtb-public` → `meridian-igw` |
| 2026-08-28 | Private subnet | `meridian-subnet-private1-us-west-1a` (10.0.2.0/24) | " | No `0.0.0.0/0` route (no NAT) — DB host has no internet egress |
| 2026-08-28 | Internet gateway | `meridian-igw` | " | Attached to VPC |
| 2026-08-28 | Extra subnets (unused) | `...public2-us-west-1c`, `...private2-us-west-1c` | " | Created by wizard, no cost, no instances placed |
| 2026-08-28 | Security group `web-sg` | `sg-0c6a8765c52a2d56d` | `02-web-sg.png` | Inbound 80/443 from `0.0.0.0/0` (legit) **+ SSH 22 from `0.0.0.0/0` = M-01** |
| 2026-08-28 | Security group `db-sg` | `sg-0d8b174e89ca578a5` | `02-db-sg.png` | Inbound 5432 from source `web-sg` only — least-privilege contrast |
| 2026-08-28 | EC2 `meridian-web` | `i-0be139d0d1ed30215` (t3.micro, AL2023) | `03-ec2-web.png` | Public subnet, public IP `50.18.11.11`, private `10.0.13.28`, SG `web-sg`. **IMDSv2 = Optional → M-05.** No IAM role attached. httpd user-data. Stopped after build. |

## Components

| Component | Represents (Meridian context) | Notes |
|---|---|---|
| VPC (public + private subnets) | Core network boundary | ✅ built — `vpc-0637dab0dab26a5f4`, see Build log |
| EC2 — `web` (public subnet) | Customer-facing dashboard host | |
| EC2 — `db` (private subnet) | Analytics database host | |
| S3 bucket — `customer-reports` | Stored population-health report exports | |
| IAM users / roles / groups | Staff + service access | Mix of good and bad practice |
| Security groups | Host-level network rules | One intentionally permissive |
| CloudTrail | Audit logging evidence | Enable in all regions |
| AWS Config / GuardDuty *(optional)* | Auto-generated findings | |

## Deliberate misconfigurations built in

_These are "discovered" in Phase 2. Keep this list accurate as the lab evolves._

| ID | Component | Misconfiguration | Intended finding | Status |
|---|---|---|---|---|
| M-01 | Security group `web-sg` (`sg-0c6a8765c52a2d56d`) | SSH (22) open to `0.0.0.0/0` | Internet-exposed management port → brute force | ✅ built |
| M-02 | S3 `customer-reports` | _e.g. public access block off / overly broad policy_ | Potential data exfiltration | pending (Step 5) |
| M-03 | IAM role | Overly permissive (`*:*`) policy attached | Least-privilege violation | pending (Step 6) |
| M-04 | IAM user | Console user without MFA | Credential compromise risk | pending (Step 6) |
| M-05 | EC2 `meridian-web` (`i-0be139d0d1ed30215`) instance metadata | IMDSv1 left enabled (IMDSv2 = Optional) | SSRF → EC2 credential theft | ✅ built |

## Cost control

- [ ] Stay in free tier where possible (t2.micro / t3.micro, single-AZ)
- [ ] Stop or `terraform destroy` resources when not in use
- [x] Billing alert / budget configured — `meridian-lab-monthly`, $5/mo

## Teardown

_Record how to cleanly remove everything (destroy order, buckets to empty first, etc.)._
