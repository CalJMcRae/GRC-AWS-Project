# Prowler Findings — Triage

**Scan:** `prowler aws --region us-west-1`, 2026-08-31 19:31 UTC, from AWS CloudShell
**Prowler version:** see report footer · **Account:** 328064416121 · **Region:** us-west-1
**Raw reports:** `prowler-output-328064416121-20260831193100.{csv,html}` (same folder)

## Headline

| | |
|---|---|
| Checks executed | 300 |
| PASS | 179 |
| **FAIL (unmuted)** | **116** — 3 critical, 28 high, 55 medium, 30 low |
| Manual | 5 |

> Of Prowler's 3 "critical" fails, two are real and severe (`s3_bucket_public_access` → R-01,
> `iam_aws_attached_policy_no_administrative_privileges` → R-02). The third
> (`iam_root_hardware_mfa_enabled`) is a context downgrade: root **does** have virtual MFA, so
> this is a minor hardware-MFA gap (see P-04 / R-07), not an unprotected root account.

116 raw failures is misleading. Most are CIS/AWS-best-practice checks that fail on **any
small single-account environment** (no AWS Organizations, no Config aggregator, no Network
Firewall, no Bedrock/SageMaker usage, no multi-region footprint). After removing those,
**~15 consolidated risks** remain — the 5 built deliberately plus ~10 the scan surfaced.

---

## A. Prowler confirmed all 5 deliberate misconfigurations

| Built | Prowler check(s) | Sev | Evidence from scan |
|---|---|---|---|
| **M-01** SSH open to world | `ec2_securitygroup_allow_ingress_from_internet_to_tcp_port_22`, `..._to_any_port`, `ec2_instance_port_ssh_exposed_to_internet` | high | `web-sg (sg-0c6a8765c52a2d56d) has SSH port 22 open to the Internet` |
| **M-02** S3 bucket public | `s3_bucket_public_access` **(critical)**, `s3_bucket_level_public_access_block`, `s3_account_level_public_access_blocks`, `s3_bucket_cross_account_access` | critical | `meridian-customer-reports-328064416121 has public access due to bucket policy` |
| **M-03** Admin role on EC2 | `iam_aws_attached_policy_no_administrative_privileges` **(critical)**, `iam_role_cross_service_confused_deputy_prevention` | critical | `AdministratorAccess is attached and allows '*:*'`; role `meridian-ec2-app-role` |
| **M-04** User without MFA | `iam_user_mfa_enabled_console_access`, `iam_user_hardware_mfa_enabled` | high | `User meridian-analyst-1 has Console Password enabled but MFA disabled` |
| **M-05** IMDSv1 enabled | `ec2_instance_imdsv2_enabled`, `ec2_instance_account_imdsv2_enabled` | high | `EC2 Instance i-0be139d0d1ed30215 has IMDSv2 disabled or not required` |

Independent tool corroboration of every planted weakness — the lab behaves as designed.

> Point-in-time caveat: `ec2_instance_port_ssh_exposed_to_internet` reported "...but with no
> public IP address" because `meridian-web` was **stopped** during the scan (public IP
> released). The exposure is live whenever the instance runs. Worth noting in the write-up.

---

## B. Genuine additional findings the scan surfaced (NOT planted — carry into the register)

| # | Finding | Prowler check(s) | Sev | Detail |
|---|---|---|---|---|
| P-01 | **`Callum-v2` privilege-escalation path** | `iam_inline_policy_allows_privilege_escalation` | high | Inline policies `AltPermsEC2` + `EC2Perm` grant `ec2:ModifyInstanceAttribute`, `ec2:CreateLaunchTemplateVersion`, `ec2:StartInstances/StopInstances` — enough to attach the `AdministratorAccess` role to an instance and pivot to admin. Chains with M-03. |
| P-02 | **`Callum-v2` no MFA + console password** | `iam_user_mfa_enabled_console_access`, `iam_user_hardware_mfa_enabled` | high | Same class as M-04 but on the build/admin user — higher impact. |
| P-03 | **`Callum-v2` long-lived access keys** | `iam_user_with_temporary_credentials` | high | Long-lived keys with broad service access; no rotation to short-lived creds. |
| P-04 | **Root MFA is virtual, not hardware** | `iam_root_hardware_mfa_enabled` FAIL — but `iam_root_mfa_enabled` **PASS** | critical (Prowler) → **low (context)** | Root *does* have MFA (virtual TOTP, confirmed). Gap is a phishing-resistant hardware device (CIS Level 2). |
| P-05 | **Root account actively used** | `iam_avoid_root_usage` | high | `Root user last accessed 0 days ago` — lab was operated as root; should be break-glass only. |
| P-06 | **EBS volumes unencrypted at rest** | `ec2_ebs_volume_encryption`, `ec2_ebs_default_encryption` | high | `vol-0a3126a768fc6238a`, `vol-04097aa795bc6cbd4` unencrypted; account default EBS encryption off. Matters for the DB host that would hold clinical data. |
| P-07 | **No VPC flow logs** | `vpc_flow_logs_enabled` | medium | `meridian-vpc` — no network traffic record; weakens detection/forensics. |
| P-08 | **No real-time monitoring / alerting** | `cloudtrail_cloudwatch_logging_enabled` + ~14 `cloudwatch_*` metric-filter/alarm checks | medium | CloudTrail not delivered to CloudWatch Logs; no metric filters or alarms for root use, unauthorized calls, policy/SG/NACL changes, etc. Partly a known lab trade-off (Step 7). Treat as ONE finding. |
| P-09 | **KMS CMK rotation disabled** | `kms_cmk_rotation_enabled` | high (Prowler) / low (context) | `meridian` key `2c4c6389-…` — no annual rotation. |
| P-10 | **Weak IAM account password policy** | `iam_password_policy_*` (length<14, no reuse-24, no expiry, missing char classes) | medium | Account-wide password policy is weak/default. |
| P-11 | **Stale / unmanaged IAM roles** | (from `iam_role_cross_service_confused_deputy_prevention`) | low | Leftover roles `DemoEC2`, `aws-ec2-spot-fleet-tagging-role` from prior experimentation — no owner, no confused-deputy conditions. |
| P-12 | **S3 hardening gaps (both buckets)** | `s3_bucket_secure_transport_policy`, `s3_bucket_object_versioning`, `s3_bucket_server_access_logging_enabled` | medium | No HTTPS-only policy, no versioning, no access logging. Consolidate. |

---

## C. Excluded as not-applicable (enterprise-scale controls, single-account lab)

Recorded so the exclusion is a deliberate, defensible decision — not an oversight.

| Theme | Checks | Why excluded |
|---|---|---|
| AWS Organizations | `organizations_*` (5), `organizations_scp_check_deny_regions`, `fms_policy_compliant` | Standalone account; no org to configure |
| AWS Config | `config_*` (2) | Deliberately not enabled (cost); noted as a detective gap in P-08 scope if desired |
| Unused services | `bedrock_*` (2), `sagemaker_*` (3), `networkfirewall_in_all_vpc`, `ssmincidents_*` | Services not in use |
| Backup / DR | `backup_vaults_exist`, `ec2_ebs_volume_snapshots_exists`, `ec2_ebs_volume_protected_by_backup_plan`, `s3_bucket_cross_region_replication`, `s3_bucket_lifecycle_enabled` | Out of lab scope; note as "resilience not assessed" |
| Design choices | `vpc_different_regions`, `vpc_endpoint_for_ec2_enabled`, `ec2_instance_profile_attached` (wants a role on `meridian-db` — intentionally none), `cloudtrail_s3_dataevents_*`, `cloudtrail_insights_exist` | Intentional architecture / cost decisions |
| Account contacts | `account_maintain_different_contact_details...` | Personal account admin detail |

---

## Next step

Consolidate A + B into `../risk-register.csv` as ~15 rows (R-01…R-15), score each
Likelihood × Impact per `../risk-assessment-methodology.md`, assign owner and treatment.
