# Meridian Health Analytics — Lab Architecture

GitHub renders the Mermaid diagram below automatically. For the portfolio README you can
also export a polished PNG (draw.io / Lucidchart) to `architecture-diagram.png` in this
folder — the Mermaid version is the source of truth for *what* to draw.

```mermaid
flowchart TB
    users(["Internet / Users"])

    subgraph AWS["AWS Account 328064416121 - us-west-1"]
        CT["CloudTrail meridian-trail<br/>multi-region, log-file validation, SSE-KMS"]
        KMS[("KMS key<br/>alias: meridian")]
        S3R[("S3 meridian-customer-reports<br/>M-02: PUBLIC read")]
        S3L[("S3 cloudtrail-logs")]

        subgraph VPC["VPC meridian-vpc - 10.0.0.0/16"]
            IGW{{"Internet Gateway"}}

            subgraph PUB["Public subnet - 10.0.0.0/20 - us-west-1a"]
                WSG["web-sg<br/>M-01: SSH 22 from 0.0.0.0/0<br/>HTTP 80 / HTTPS 443 from 0.0.0.0/0"]
                WEB["EC2 meridian-web - t3.micro<br/>public IP - httpd<br/>M-05: IMDSv1 enabled<br/>M-03: admin role attached"]
            end

            subgraph PRV["Private subnet - 10.0.128.0/20 - us-west-1a"]
                DSG["db-sg<br/>5432 from web-sg only (least privilege)"]
                DB["EC2 meridian-db - t3.micro<br/>no public IP - no internet egress<br/>IMDSv2 required (hardened)"]
            end
        end

        subgraph IAM["IAM"]
            ROLE["role meridian-ec2-app-role<br/>M-03: AdministratorAccess"]
            USER["user meridian-analyst-1<br/>M-04: console access, NO MFA"]
            GRP["group meridian-analysts<br/>AmazonS3ReadOnlyAccess (scoped)"]
        end
    end

    users -->|"HTTP / HTTPS / SSH"| IGW
    IGW --> WEB
    WSG --- WEB
    WEB -->|"5432"| DB
    DSG --- DB
    WEB -.->|"assumes (instance profile)"| ROLE
    USER --> GRP
    CT --> S3L
    CT --- KMS
    WEB -.->|"app reads exports"| S3R
    users -.->|"anonymous GET (M-02)"| S3R
```

## Legend — deliberate misconfigurations

| ID | Where | Weakness | Intended finding |
|---|---|---|---|
| M-01 | `web-sg` | SSH 22 open to `0.0.0.0/0` | Internet-exposed management port |
| M-02 | `meridian-customer-reports` S3 | Public Block off (bucket + account) + public-read policy | Anonymous data exfiltration |
| M-03 | `meridian-ec2-app-role` on `meridian-web` | `AdministratorAccess` on an internet-facing host | Least-privilege violation; blast radius |
| M-04 | `meridian-analyst-1` | Console user, no MFA | Credential compromise |
| M-05 | `meridian-web` | IMDSv1 still enabled | SSRF → EC2 credential theft (chains with M-03) |

## Hardened / good-practice elements (the contrast)

- `db-sg` allows `5432` only from `web-sg` (not a CIDR)
- `meridian-db` in a private subnet, no public IP, no `0.0.0.0/0` route
- `meridian-db` enforces IMDSv2
- `meridian-analysts` group carries a scoped read-only policy; `meridian-analyst-1` gets access via the group, no inline policies
- CloudTrail: multi-region, log-file validation on, KMS-encrypted
