# Implemented Fixes — Before / After Evidence

Fixes applied to the live lab during Phase 6-partial, to demonstrate closing the loop.
The full remediation plan for all 16 risks is in [`remediation-roadmap.csv`](remediation-roadmap.csv).

---

## R-01 — Public S3 bucket re-secured  ·  Critical (25)  ·  Fixed 2026-08-31

**Bucket:** `meridian-customer-reports-328064416121` · **CSF:** PR.DS-01 (+ PR.AA-05)

### Actions taken
1. Deleted the public-read bucket policy (`s3:GetObject` to `Principal:*`).
2. Re-enabled **bucket-level** Block Public Access (all four settings).
3. Re-enabled **account-level** Block Public Access (the backstop disabled in Phase 1) — confirmed "successfully updated", all four settings On.

### Evidence

| | Before | After |
|---|---|---|
| Anonymous `GET` on `meridian-population-health-report-2026Q2.csv` (no credentials) | `HTTP 200`, `text/csv`, 1135 bytes | **`HTTP 403 Forbidden`** |
| Bucket Permissions tab | Block all public access: **Off**; public-read policy present | Block all public access: **On**; "No policy to display" |
| Account-level Block Public Access | Off (disabled in Phase 1 to allow the public policy) | **On** (all four settings) |
| Screenshots | `05-s3-bucket.png`, `05-s3-object-public.png` | `06r-s3-bucket-after.png`, `06r-s3-bucket-bpa-edit.png`, `06r-s3-account-bpa-after.png` |

Both `GET` checks run from an unauthenticated workstation via `Invoke-WebRequest`.

### Residual risk
Low. Bucket is private, access is IAM-gated, and the account-level backstop now protects
future buckets too. Remaining hardening (versioning, HTTPS-only policy, server access
logging) tracked as **R-15**. Re-enabling account BPA also clears the Phase 1 change noted
in the teardown checklist.

---

## R-04 — IMDSv2 enforced on `meridian-web`  ·  High (12)  ·  Fixed 2026-08-31

**Instance:** `i-0be139d0d1ed30215` · **CSF:** PR.PS-01 (+ PR.AA-03)

### Actions taken
- EC2 → Modify instance metadata options → **IMDSv2 = Required** (IMDSv1 now rejected).

### Evidence

| | Before | After |
|---|---|---|
| `MetadataOptions.HttpTokens` (`aws ec2 describe-instances`) | `optional` | **`required`** (State: `applied`) |
| Instance Details tab | IMDSv2: **Optional** (with EC2 warning) | IMDSv2: **Required** |
| Screenshots | `03-ec2-web.png` | `06r-ec2-web-imdsv2-after.png` |

### Residual risk
Low for the metadata-theft vector — IMDSv1 SSRF path is closed. **R-02 is not yet
resolved:** the instance still carries `meridian-ec2-app-role` (AdministratorAccess), so a
host compromise via other means (e.g. SSH, still open per R-03) would still yield admin
credentials. Enforcing IMDSv2 removes the *no-vulnerability-needed* path; scoping the role
(R-02) and closing SSH (R-03) remain.

### Note on hop limit
`HttpPutResponseHopLimit` left at `2` (default). Setting it to `1` would further reduce
container-based SSRF risk on a single-host instance — minor, noted for the full plan.
