# AWS Console Setup Steps

Step-by-step build log for the Meridian lab, done in the AWS console.
Screenshot each step into [`../docs/screenshots/`](../docs/screenshots/) and reference the
filename in the "Evidence" column.

> If you build with Terraform instead, record that in [`terraform/`](terraform/) and leave
> this file as the manual fallback / narrative walkthrough.

| # | Step | Settings of note | Evidence |
|---|------|------------------|----------|
| 1 | Create VPC (`meridian-vpc`, 10.0.0.0/16) | | |
| 2 | Public subnet (10.0.1.0/24) + IGW + route | | |
| 3 | Private subnet (10.0.2.0/24) + NAT (or none, for cost) | | |
| 4 | Security group `web-sg` | **SSH 22 from 0.0.0.0/0 (deliberate, M-01)**; HTTP 80 open | |
| 5 | Security group `db-sg` | 5432 from `web-sg` only | |
| 6 | EC2 `web` in public subnet | t3.micro, Amazon Linux 2023 | |
| 7 | EC2 `db` in private subnet | t3.micro | |
| 8 | S3 bucket `meridian-customer-reports` | **public access block / policy per M-02** | |
| 9 | IAM group `analysts` + user with console access, **no MFA (M-04)** | | |
| 10 | IAM role with **`*:*` policy (M-03)** | | |
| 11 | Enable CloudTrail (all regions) → log bucket | | |
| 12 | *(optional)* Enable AWS Config / GuardDuty | | |
| 13 | Set a budget / billing alert | | |

## Notes

_Anything that didn't go to plan, region choices, quota issues, etc._
