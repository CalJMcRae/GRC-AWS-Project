# Risk Assessment Methodology

**Company:** Meridian Health Analytics
**Approach:** Simplified NIST SP 800-30 Rev.1 (qualitative)

## Why this approach

NIST SP 800-30 is a widely recognised risk assessment process and maps cleanly onto
NIST CSF (Phase 3). A qualitative Likelihood × Impact model is proportionate for an
early-stage company with a small, well-understood environment and no historical loss
data to support quantitative modelling.

## Process

1. **Asset inventory** — enumerate every component in the Phase 1 lab: compute, storage,
   identity, logging, and the data types each holds. Assign an owner and a data
   classification.
2. **Threat identification** — for each asset, identify realistic threat events and
   threat sources (external attacker, malicious insider, accidental misuse).
3. **Vulnerability identification** — tie each threat to a specific weakness. Sources:
   the deliberate misconfigurations in `../01-aws-lab/lab-notes.md`, plus findings from
   an open-source scan (Prowler or ScoutSuite) run against the live lab.
4. **Likelihood** — rate 1–5 (see scale). Consider exposure, attacker effort, and
   whether a detective control exists.
5. **Impact** — rate 1–5 (see scale). Consider confidentiality of data at risk,
   availability of the service, and regulatory / contractual exposure.
6. **Risk score** — Likelihood × Impact (1–25), banded Low / Medium / High / Critical.
7. **Risk register** — record all of the above plus risk owner and status in
   `risk-register.csv`.

## Data classification scale

| Class | Definition | Meridian examples |
|---|---|---|
| Confidential | Serious harm to customers or Meridian if disclosed | De-identified clinical datasets, customer report exports, IAM credentials |
| Internal | Limited harm; not for public release | Architecture docs, CloudTrail logs |
| Public | No harm if disclosed | Marketing site content |

## Likelihood scale

| Score | Label | Guidance |
|---|---|---|
| 1 | Rare | Requires unlikely conditions; strong controls in place |
| 2 | Unlikely | Possible but not expected |
| 3 | Possible | Could occur; partial controls |
| 4 | Likely | Expected in normal operations; weak/no controls |
| 5 | Almost certain | Actively exposed; trivial to exploit |

## Impact scale

| Score | Label | Guidance |
|---|---|---|
| 1 | Negligible | No meaningful business effect |
| 2 | Minor | Short disruption; no data exposure |
| 3 | Moderate | Internal data exposure or notable disruption |
| 4 | Major | Confidential data exposure; customer notification likely |
| 5 | Severe | Large-scale confidential data loss; regulatory action, contract loss |

## Risk banding

| Score range | Band |
|---|---|
| 1–4 | Low |
| 5–9 | Medium |
| 10–15 | High |
| 16–25 | Critical |

## Assumptions & limitations

- Scores are analyst judgement on a lab environment; no production telemetry.
- Threat likelihood assumes an internet-reachable environment with no external WAF/SIEM.
- Out of scope: physical security, corporate endpoints, SaaS not in the lab.
