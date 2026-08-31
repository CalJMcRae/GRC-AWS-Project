# AWS Console Setup Steps

The Meridian lab was built **manually in the AWS console** (not Terraform).

The authoritative, resource-by-resource build record — identifiers, settings, evidence
screenshots, and the deliberate misconfigurations — lives in the **Build log** table in
[`lab-notes.md`](lab-notes.md). This file just captures the order it was done in and any
narrative worth keeping.

## Order of build

| # | Step | Result | Evidence |
|---|------|--------|----------|
| 0 | Budget guardrail (`meridian-lab-monthly`, $5/mo) | done | `00-budget.png` |
| 1 | VPC + subnets + IGW + route tables ("VPC and more" wizard, no NAT) | done | `01-vpc-resource-map.png` |
| 2 | Security groups `web-sg` (M-01) and `db-sg` | done | `02-web-sg.png`, `02-db-sg.png` |
| 3 | EC2 `meridian-web` — public subnet, IMDSv1 (M-05) | done | `03-ec2-web.png` |
| 4 | EC2 `meridian-db` — private subnet, IMDSv2 required | done (relaunched once — first attempt hit the public subnet) | `04-ec2-db.png` |
| 5 | S3 `meridian-customer-reports` — public (M-02) | done | `05-s3-bucket.png`, `05-s3-object-public.png` |
| 6 | IAM — group `meridian-analysts`, user `meridian-analyst-1` (M-04), role `meridian-ec2-app-role` (M-03), role attached to `meridian-web` | done | `06-iam-*.png` |
| 7 | CloudTrail `meridian-trail` — multi-region, validated, KMS | done | `07-cloudtrail.png` |
| 8 | GuardDuty / AWS Config | **skipped by choice** (Prowler covers scanning in Phase 2, free) | — |

## Notes

- **Region:** us-west-1 (N. California). Second AZ in this account is `us-west-1c`, not `1b`.
- **Cost:** no NAT gateway, no Elastic IPs; both EC2 instances kept **stopped** between
  sessions. Idle spend ≈ $2–3/mo (2× 8 GB EBS + KMS key), under the $5 budget.
- **Subnet mistake:** `meridian-db` was first launched into the public subnet; instances
  can't change subnet, so it was terminated (`i-0c7e1b1762fd5af9c`) and relaunched as
  `i-00b082701da0ffbdd`.
- **S3 account-level Block Public Access** had to be turned off (as root) for the M-02
  bucket policy to save — remember to re-enable at teardown.
- **`Callum-v2`** (CLI user) has narrow permissions — fine for the build, but Phase 2
  Prowler runs will need broader read access or a dedicated role.
