# Assumption Log

Every assumption in the design, with its current lifecycle state and the evidence that put
it there.

## Lifecycle

```text
                    ASSUMED
                       │
                       ▼
                  RESEARCHING
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
        SUPPORTED             CHALLENGED
            │                     │
            │                     ▼
            │                  REVISED
            │                     │
            └──────────┬──────────┘
                       ▼
               DESIGN DECISION
```

An assumption may not move out of `ASSUMED` without a citation. Movement to
`DESIGN DECISION` requires an entry in [`design-changelog.md`](./design-changelog.md).

**No assumption below has reached `DESIGN DECISION`.** Nothing has been changed in
[`../research/research-file.md`](../research/research-file.md) — the specification stands as
the dated baseline, and revisions are proposals until the research phase closes.

## Register

| ID | Assumption | Source | State | Evidence | Finding |
|---|---|---|---|---|---|
| A-01 | Publisher identity change is a hard escalation trigger | §10.1 | **CHALLENGED** | INC-01, INC-02, INC-03 — identity unchanged in all three | F-01 |
| A-02 | "Publisher unchanged" supports auto-merge | §9 | **CHALLENGED** | Same. A clean value lowers the posterior and pushes toward merge | F-01 |
| A-03 | Missing or mismatched provenance is a hard trigger | §10.2 | **CHALLENGED** | S-01: provenance is structurally unavailable for private repos and non-supported runners, so absence is often benign. *Missing* ≠ *mismatched* | F-05 |
| A-04 | Provenance meaningfully indicates safety | §9, HYP-01 | **CHALLENGED** | S-01, S-02: attests build origin, not code safety. INC-01 used the real publish workflow | F-05 |
| A-05 | "No install-script change" bounds blast radius | §9 | **CHALLENGED** | INC-02 had no install script; payload ran in end-user browsers | F-02 |
| A-06 | Sandboxed install reveals what static metadata misses | HYP-02 | **QUALIFIED** | Holds for INC-03 (install-time), fails for INC-02 (browser-side). Conditional on attack channel | F-02 |
| A-07 | Build/CI dependencies need stricter autonomy bounds | HYP-03, §12 | **SUPPORTED** | INC-01 (build tool, 2,349 credentials per S-18), INC-03 (CI/CD env, cloud metadata) | F-04 |
| A-08 | Credential/build-infra access is a hard trigger | §10.3 | **SUPPORTED, with an open cost** | Only reason INC-01 is caught. But fires on dependency *type*, not evidence — false-positive rate unknown | F-04 |
| A-09 | Transitive changes materially alter risk | HYP-04 | **SUPPORTED** | INC-03 propagates to arbitrary closure depth; Renovate `--before` already treats the closure as in scope (S-04) | — |
| A-10 | SemVer bump size is meaningful evidence | §6, §18 | **CHALLENGED** | Tier 3 T-01: 4 of 5 reject it. PR-02 (maintainer) shows it fails *even when correctly applied* | F-09 |
| A-11 | Changelog quality is a useful signal | §6 | **SUPPORTED** | Tier 3: PR-01, PR-02, PR-05. PR-02 supplies an operational test — does it name behaviour or name code? | F-09 |
| A-12 | CI result is decision-useful evidence | §6, §9 | **SUPPORTED for `breaks_us`, REJECTED for `unauthorised`** | Tier 3: PR-03, PR-05. But none of INC-01–03 would have failed CI | — |
| A-13 | API-surface coverage makes a CI pass meaningful | §6, §9 | **CHALLENGED** | PR-02's failure mode leaves the API surface unchanged — coverage of it cannot detect the break | F-10 |
| A-14 | The §6 evidence list is adequate for `breaks_us` | §6 | **CHALLENGED** | No signal captures the delta in what a package reports about *our unchanged code* | F-10 |
| A-15 | Waiting is a maintenance-debt trade-off (RQ8, open) | §8.A3, RQ8 | **SUPPORTED as a security mechanism** | S-04 (Renovate recommends 14 days), INC-01 ~4h, INC-02 ~2h, PR-05 independently | F-03 |
| A-16 | Compromise costs ≈125× a regression (₹15,00,000 / ₹12,000) | §11 | **ASSUMED** | No evidence gathered. Author estimate | — |
| A-17 | 30 days is a useful regression horizon | §4 | **ASSUMED** | No evidence gathered | — |
| A-18 | Public advisories are sufficient ground truth for `unauthorised` | §4 | **CHALLENGED** | Advisories lag by construction. S-07 was **withdrawn 28 July 2026** — the record is not stable over time | F-07 |
| A-19 | Malicious-release prevalence is stable enough to sample at "natural prevalence" | §16 | **CHALLENGED** | INC-03 still propagating into 2026 (S-09). Prevalence is not stationary | — |
| A-20 | Existing tools leave the merge decision entirely open | §2 | **QUALIFIED** | S-03–S-06: cooldowns, automerge policy, attestation, change metadata already exist | F-08 |
| A-21 | Extended checks have a cost worth modelling | §8.A2, §11 | **ASSUMED** | No evidence on what an extended check actually costs a team. Question P3 | — |
| A-22 | Dependency type (runtime/dev/build) is the right package classification | §12 | **QUALIFIED** | PR-02 introduces an orthogonal axis: rule-bearing vs plain-API packages | F-09 |
| A-23 | Explicit costs change decisions vs probability alone | HYP-06 | **ASSUMED** | No evidence gathered at any tier | — |
| A-24 | Autonomy should be bounded by reversibility | §9 | **ASSUMED** | Untested. The principle is sound but no evidence yet shows it produces better decisions | — |

## Summary

| State | Count |
|---|---|
| ASSUMED — no evidence yet | 6 |
| CHALLENGED | 9 |
| QUALIFIED | 4 |
| SUPPORTED | 5 |
| DESIGN DECISION | **0** |

## What the shape of this table says

Nine assumptions challenged and zero design decisions taken is the intended state at this
point: evidence has arrived faster than it can be responsibly acted on, and acting early is
what [`../research/10-revised-design.md`](../research/10-revised-design.md) exists to
prevent.

Two clusters are worth naming.

**The security anchors are the weak part.** A-01 through A-05 — every signal
[§9](../research/research-file.md#9-bounded-autonomy) and
[§10](../research/research-file.md#10-hard-escalation-triggers) lean on for supply-chain
safety — are challenged. A-07/A-08 (dependency privilege) is the only security assumption
still standing, and it survives as a blanket rule with an unmeasured false-positive rate.

**The unevidenced assumptions are the expensive ones.** A-16 (the cost ratio), A-17 (the
horizon), A-21 (extended-check cost) and A-23 (whether costs change decisions) remain at
`ASSUMED` with no evidence at any tier — and they are precisely the inputs that drive every
expected-cost comparison in [§13](../research/research-file.md#13-decision-policy-hypothesis).
The most quantitative part of the design rests on the least evidenced part of it.
