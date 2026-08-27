# 01 — AWS Lab Notes

**Company:** Meridian Health Analytics
**Environment purpose:** Small, realistic AWS footprint with *plausible* misconfigurations
to assess in Phase 2 — not hardened, not wide open.

---

## Architecture

![Architecture diagram](architecture-diagram.png)

_Export a diagram (draw.io / Lucidchart) to `architecture-diagram.png` in this folder._

## Build method

- [ ] Console-first, screenshotted → see [`console-setup-steps.md`](console-setup-steps.md)
- [ ] Terraform (IaC) → see [`terraform/`](terraform/)

Screenshots go in [`../docs/screenshots/`](../docs/screenshots/).

## Components

| Component | Represents (Meridian context) | Notes |
|---|---|---|
| VPC (public + private subnets) | Core network boundary | |
| EC2 — `web` (public subnet) | Customer-facing dashboard host | |
| EC2 — `db` (private subnet) | Analytics database host | |
| S3 bucket — `customer-reports` | Stored population-health report exports | |
| IAM users / roles / groups | Staff + service access | Mix of good and bad practice |
| Security groups | Host-level network rules | One intentionally permissive |
| CloudTrail | Audit logging evidence | Enable in all regions |
| AWS Config / GuardDuty *(optional)* | Auto-generated findings | |

## Deliberate misconfigurations built in

_These are "discovered" in Phase 2. Keep this list accurate as the lab evolves._

| ID | Component | Misconfiguration | Intended finding |
|---|---|---|---|
| M-01 | Security group (`web`) | SSH (22) open to `0.0.0.0/0` | Internet-exposed management port → brute force |
| M-02 | S3 `customer-reports` | _e.g. public access block off / overly broad policy_ | Potential data exfiltration |
| M-03 | IAM role | Overly permissive (`*:*`) policy attached | Least-privilege violation |
| M-04 | IAM user | Console user without MFA | Credential compromise risk |
| M-05 | _tbd_ | | |

## Cost control

- [ ] Stay in free tier where possible (t2.micro / t3.micro, single-AZ)
- [ ] Stop or `terraform destroy` resources when not in use
- [ ] Billing alert / budget configured

## Teardown

_Record how to cleanly remove everything (destroy order, buckets to empty first, etc.)._
