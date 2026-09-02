# Decision Log — Cloud Risk Assessment & Remediation Lab

A phase-by-phase account of what was decided at each step of this engagement and how the
conclusion was reached. It is the companion to the deliverables in the numbered folders —
where those show *what* was produced, this explains *why*.

**Project:** GRC-AWS-Project · **Author:** Callum McRae · **Scenario:** Meridian Health Analytics (simulated)

Every decision below was held to the same test, which is also the standard the portfolio
roadmap sets: **state the assumptions, make the trade-off explicit (what was chosen and what
was given up), and acknowledge the residual risk.**

---

## Overview

The engagement runs a single finding from cloud misconfiguration → risk register → NIST CSF
2.0 control mapping → prioritised remediation → verified fix, and adds two independent
workstreams — a mock third-party vendor assessment and a SOC 2 Type II report review.

**Why a fictional company.** "Meridian Health Analytics" — an early-stage healthtech startup
(~25 staff) that ingests de-identified clinical data from partner clinics and returns
population-health dashboards — is the narrative wrapper. It was chosen so every artifact
shares one consistent business context: the data-sensitivity story (confidential customer
data, contractual pressure, incoming security questionnaires) stays coherent across the risk
assessment, the vendor assessment and the SOC 2 review, and scoring impact against a real
scenario is defensible rather than abstract.

---

## Phase 0 — Repository structure

**Decision.** Numbered phase folders (`01-aws-lab` … `06-remediation-plan`); spreadsheet
deliverables kept as `.csv`, not `.xlsx`; a fictional-company narrative wrapper.

**How the conclusion was reached.**

- **Numbered folders** mirror the engagement sequence, so a reviewer can follow the reasoning
  path in order rather than reconstructing it.
- **CSV over XLSX** because plain-text diffs cleanly in version control and renders directly
  on GitHub — a reviewer never has to download a binary to read the risk register. The
  trade-off (no formulas, no formatting) is immaterial for a register that is read, not
  computed. Noted in the README so the choice is visible.
- The **narrative wrapper** was fixed before any technical work so the scenario constrains
  every later decision consistently.

---

## Phase 1 — AWS lab

**Decision.** Build in the AWS console, not Terraform; region `us-west-1`; no NAT gateway;
five specific deliberate misconfigurations; a deliberately hardened database tier as a
contrast.

**How the conclusion was reached.**

- **Console vs Terraform.** Terraform is a genuine résumé differentiator, but the GRC value
  of this project sits in Phases 2–6, and an Infrastructure-as-Code detour risked stalling
  before the actual risk work. The console build also produces the screenshot evidence the
  portfolio needs. Terraform is recorded as an optional later rebuild — the console build is
  the reference.
- **No NAT gateway.** A NAT gateway is roughly $32/month and not free-tier. The private
  subnet simply has no outbound internet, which is correct for a lab database host. Cost
  discipline (no NAT, no Elastic IPs, instances stopped between sessions, a $5 budget alert)
  is part of the engagement, not an afterthought.
- **The five misconfigurations** were chosen to be the most common and most teachable cloud
  findings *and* to chain together:

  | ID | Weakness | Why this one |
  |---|---|---|
  | M-01 | SSH open to `0.0.0.0/0` | The canonical "management port exposed" finding |
  | M-02 | Public S3 bucket | The canonical data-exposure finding; verifiable end to end |
  | M-03 | `AdministratorAccess` on an EC2 instance role | Least-privilege violation with maximum blast radius |
  | M-04 | Console IAM user with no MFA | The canonical identity finding |
  | M-05 | IMDSv1 left enabled | Turns a web-app SSRF into instance-credential theft |

  M-03 and M-05 were **deliberately co-located on the internet-facing host** so the
  SSRF → credential-theft → account-takeover chain is real infrastructure, not a hypothetical
  in the write-up.
- **The hardened DB tier** (private subnet, no public IP, IMDSv2 required, security group
  scoped to the web tier's security group rather than a CIDR) exists so the assessment has a
  "done right" baseline. Findings then read as deviations from a standard the same person
  clearly knows how to meet, not just a list of bad things.
- **The subnet error.** `meridian-db` was first launched into the public subnet. It was
  caught by reading the resource IDs back against the intended design, not by eyeballing the
  console. An instance cannot change subnet, so it was terminated and relaunched. This is
  left in the build log on purpose — catching and correcting your own mistake is part of
  defensible work.

---

## Phase 2 — Risk assessment

**Decision.** Simplified NIST SP 800-30 qualitative Likelihood × Impact scoring; run Prowler
against the account; triage 116 raw scanner failures down to 16 risks; consolidate related
findings; specific, individually-reasoned scores.

**How the conclusion was reached.**

- **Qualitative, not quantitative.** There is no loss history, the environment is small and
  well understood, and the company is early-stage — a qualitative model is proportionate. The
  1–5 Likelihood and Impact scales and the Low/Medium/High/Critical banding are written out
  in the methodology document so the reasoning is inspectable, not implied.
- **Prowler** was run from AWS CloudShell (no local Python was installed; CloudShell is
  already authenticated and disposable). Independent tool output is the strongest available
  corroboration for a self-built lab.
- **The 116 → 16 triage is the core judgment.** About 100 of the failures are
  enterprise-scale checks that fail on *any* single-account environment — no AWS
  Organizations, no Config aggregator, no Network Firewall, services not in use. Each
  exclusion category is recorded with its reason in `findings-triage.md`, so the exclusions
  are a documented, defensible decision rather than a silent omission. What survived: the 5
  seeded misconfigurations (all independently confirmed by Prowler) plus 11 genuinely present
  issues — notably a **privilege-escalation path on the pre-existing `Callum-v2` user** that
  was surfaced by the scan, not planted.
- **Consolidation.** Related Prowler checks were collapsed to one root-cause risk — for
  example, ~14 separate CloudWatch metric-filter checks became a single "no monitoring or
  alerting" risk. The register reads as risks, not scanner rows.
- **Scoring rationale — worked examples:**
  - **R-01 (public S3) = 5 × 5 = 25.** Likelihood 5: verified anonymously readable, and open
    buckets are found by internet scanners within hours. Impact 5: confidential customer data
    with contractual and breach-notification exposure.
  - **R-02 (admin role on the web host) = 4 × 5 = 20.** M-03, M-01 and M-05 were merged into
    one *chain* risk. Likelihood was set to 4 rather than 3 on this reasoning: with SSH open,
    an attacker does not need an application vulnerability to reach the metadata service and
    take the admin credentials. The register notes this is a close call a stricter assessor
    might split into separate rows.
  - **R-03 (SSH open) = 3 × 4 = 12 — High, not Critical.** Likelihood was held at 3 because
    Amazon Linux 2023 disables password authentication by default; key-only auth makes pure
    brute force unlikely to succeed even though the port is exposed. Raising Likelihood to 4
    would make it Critical — that alternative is stated rather than hidden.
  - **R-07 (root account) — a conclusion that was revised.** It was first scored 10 / High
    as "no MFA and actively used." When the account owner confirmed root *does* have MFA, the
    scan data was rechecked: `iam_root_mfa_enabled` **PASS**, only
    `iam_root_hardware_mfa_enabled` **FAIL**. It was rescored to 5 / Medium, and the finding
    reframed to what is actually true — "virtual rather than phishing-resistant hardware MFA,
    and root used for routine work" — not "unprotected root." Left visible as an example of
    updating a conclusion when a fact changes.
  - **R-05 was split into two rows.** One row covering both `meridian-analyst-1` (S3
    read-only) and `Callum-v2` (broad access plus the escalation path) understated the risk,
    because the two identities have very different blast radius. Split into R-05 (Medium) and
    R-16 (High).

---

## Phase 3 — NIST CSF 2.0 mapping

**Decision.** Map each risk to a single best-fit CSF 2.0 subcategory; rate current-vs-target
maturity on a 0–4 scale; do **not** score the GOVERN, RESPOND or RECOVER functions.

**How the conclusion was reached.**

- **CSF 2.0** rather than 1.1 because it is current and adds the GOVERN function.
- **One best-fit subcategory per risk**, with any secondary mapping noted in a column. Mapping
  every risk to several subcategories would blur the heat map; the point of the exercise is
  to show *where the risk concentrates* in the framework.
- **Debatable calls are flagged in the artifact** for a reviewer to challenge — for example
  R-04 (IMDSv1) mapped to PR.PS-01 (secure configuration) rather than PR.AA-03
  (authentication); R-09 (flow logs) to DE.CM-01 (detection) rather than PR.PS-04 (log
  generation); R-10 to DE.AE-02 (nothing analyses the events) rather than DE.CM-09.
- **Maturity scale** is defined explicitly (0 = not implemented … 4 = optimised). Target is 3
  for most controls, 4 only where the blast radius justifies it (root, admin identities), and
  2 where that is proportionate to lab scale (flow logs).
- **GOVERN / RESPOND / RECOVER not scored.** A single-account technical lab has no policy
  function, incident-response team or recovery process to assess. Recording this as a
  deliberate scope boundary — with a note that for Meridian in production these would be the
  next assessment's scope — is more honest than inventing findings to fill the columns.
- **The conclusion the mapping produced:** 14 of 16 risks are PROTECT-function gaps,
  concentrated in identity/access (PR.AA — 8 risks) and data protection (PR.DS — 4). DETECT
  is near-absent. That directs remediation at PR.AA-05 and PR.DS-01 first (which clears both
  Criticals) and flags that the environment could barely detect an incident in progress.

---

## Phase 4 — Mock vendor security assessment

**Decision.** A condensed SIG-Lite-style questionnaire across 8 domains; a simulated vendor,
"SlotHive," with a deliberately mixed security profile; a weighted scorecard; verdict
**approve with conditions**.

**How the conclusion was reached.**

- **SIG Lite** as the structure because it is the recognised real-world standard, and
  condensing it to the domains that matter for this engagement demonstrates judgment.
- **The vendor profile deliberately mixes** solid controls (current SOC 2 Type II, MFA via
  SSO, tested backups, annual third-party penetration testing) with specific, realistic gaps
  (secondary use of customer data for analytics, refusal to sign a BAA, a vague breach-
  notification commitment, logical-only tenant isolation, no ongoing subprocessor
  monitoring). A uniformly good or uniformly bad vendor produces no decision worth
  documenting.
- **Weighting** reflects what actually drives risk for this data and this buyer: Data
  Handling & Privacy 20% (highest — healthtech, PII), Governance / Access Control / Incident
  Response 15% each, Encryption / Subprocessors / Application Security 10%, Business
  Continuity 5%. Weighted total **2.60 / 4.00**, which falls in the Medium band (Low ≥ 3.0,
  Medium 2.0–2.9, High < 2.0).
- **Verdict logic.** Not a straight approval, because three issues carry real business risk —
  secondary data use, the BAA refusal, and the vague breach terms — all of which are
  contractually fixable. Not a rejection, because the security fundamentals are sound and the
  data in scope is limited to workforce contact details and appointment metadata, not patient
  records. "Approve with conditions" is the defensible middle: 7 contractual conditions (DPA
  with a legal ruling on BAA applicability, prohibit secondary use, 72-hour breach SLA, right
  to audit, subprocessor termination right, data minimisation, backup-retention cap) plus 5
  Meridian-side compensating controls, with residual risk stated and an annual reassessment
  trigger.

---

## Phase 5 — SOC 2 Type II report review

**Decision.** Review an **illustrative** SOC 2 Type II report (structured per the AICPA
illustrative report), framed as SlotHive's, and cross-check it against the Phase 4
questionnaire.

**How the conclusion was reached.**

- **Why illustrative.** Every publicly available sample report was a locked or encrypted PDF.
  The roadmap explicitly permits a template. More to the point: real SOC 2 reports are shared
  under NDA, so working from the illustrative report is the actual constraint a GRC analyst
  operates within — which is stated plainly in a source note rather than glossed over.
- **Framing it as SlotHive's report** gives portfolio continuity and, more usefully, enables
  the cross-check below.
- **The review works through the parts a customer relying on the report actually depends on:**
  the Type II wording, the unqualified opinion, the examination period, the Trust Services
  Criteria in scope (Privacy is *out* of scope despite the vendor holding PII — a gap), the
  carved-out subservice organisations and the CSOCs the reader must obtain elsewhere (AWS via
  AWS Artifact), the Complementary User Entity Controls the customer must satisfy, the sample
  controls tested with two exceptions and what those mean, and the ~3-month gap between the
  period end and reliance that needs a bridge letter.
- **The cross-check is the point of doing both phases.** The questionnaire answer said access
  reviews were *annual*; the illustrative report shows *quarterly* (with one completed late).
  A discrepancy between what a vendor claims and what its auditor tested is exactly what a
  third-party-risk analyst is there to chase.
- **Conclusion:** the report supports the approve-with-conditions decision — the core
  security, availability and confidentiality controls operate effectively, and the gaps
  (Privacy scope, the AWS carve-out, the late review, the currency gap) are all addressable
  through the bridge letter and the Phase 4 conditions.

---

## Phase 6 — Remediation

**Decision.** Prioritise by risk score, then by effort; tie every recommendation to its CSF
subcategory; phase the work into 0–30 / 30–90 / 90+ day windows; implement two fixes now
(R-01 and R-04) and verify them.

**How the conclusion was reached.**

- **Prioritise by score, then effort**, so the two Criticals lead and the cheap high-value
  fixes cluster in the first window rather than being spread thin.
- **Every recommendation references a CSF subcategory**, so remediation is framed as moving a
  specific maturity gap, not just "closing a finding."
- **Sequence the identity work together.** 8 of 16 risks are PR.AA; one account-wide
  MFA-enforcement policy, one permissions-boundary rollout and one credential clean-up close
  most of them at once.
- **Why R-01 and R-04 first.** R-01 is the top Critical and the bucket was live-public, so
  closing it was also basic hygiene before the repo circulated. R-04 (enforce IMDSv2) removes
  the *no-vulnerability-needed* path in the R-02 chain — high value for a single setting
  change.
- **Verification, not assertion.** R-01 was confirmed by an unauthenticated HTTP request to
  the object returning `403` where it had returned `200`. R-04 was confirmed by
  `describe-instances` reporting `HttpTokens = required`, state `applied`. Config read-back
  plus an external check, screenshotted before and after.
- **Residual risk is stated per fix.** Enforcing IMDSv2 closes the metadata path, but R-02
  and R-03 remain open, so a host compromise by other means would still expose the role. The
  fix is progress, not closure of the chain.

---

## Phase 7 — Presentation

**Decision.** Keep the README current through every phase rather than writing it last; embed
the architecture diagram as Mermaid; write the résumé bullets into the repo.

**How the conclusion was reached.** A README updated per phase never has a stale-status
problem. Mermaid renders natively on GitHub, needs no binary asset kept in sync, and stays
next to the text it describes. The diagram deliberately shows the environment *as assessed* —
with its misconfigurations — because that is the state the assessment was performed against;
remediation status is tracked separately so the diagram and the findings never contradict
each other.

---

## The through-line — how conclusions were reached

Across every phase the same method applied:

- **Assumptions stated, trade-offs made explicit, residual risk acknowledged** — in the
  artifact itself, not just in the author's head.
- **Independent corroboration wherever possible** — Prowler for the risk assessment, the SOC
  2 cross-check for the vendor's claims, external HTTP checks for the fixes.
- **Judgment calls surfaced, not hidden** — the debatable scores and framework mappings are
  flagged *in the deliverables* for a reviewer to push back on.
- **Willingness to revise** — R-07 was rescored when a fact changed; R-05 was split when one
  row understated the risk; the database subnet error was caught and corrected rather than
  quietly fixed.

---

## Limitations

Stated plainly, because acknowledging them is part of the standard the project sets for
itself.

- **It is a solo lab against a small environment.** It cannot show organisational friction,
  competing priorities, or the work of getting sign-off from people who do not speak
  security — which is a large part of real GRC.
- **The vendor questionnaire responses and the SOC 2 report are self-authored.** They
  demonstrate knowledge of the structure and of what to look for; they are not the same as
  assessing a real vendor giving evasive answers.
- **The Prowler scan is point-in-time.** It reported the web host's SSH exposure as "no
  public IP" because the instance was stopped during the scan; the exposure is live whenever
  the instance runs.
- **AI assistance was used.** Every decision recorded here is one the author can defend in an
  interview — and that conversation is ultimately what the portfolio is tested by.
