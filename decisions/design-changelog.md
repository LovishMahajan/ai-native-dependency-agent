# Design Changelog

Proposed and applied changes to the design, each with the evidence that caused it.

## Rules

1. **No change without evidence.** Every entry names a finding and its sources.
2. **Proposals are not applications.** A proposal records what the evidence suggests. It is
   applied only when the research phase closes and
   [`../research/10-revised-design.md`](../research/10-revised-design.md) is written.
3. **[`research-file.md`](../research/research-file.md) is not edited during the research
   phase.** It is the dated baseline. Changing it in place would destroy the record of what
   was originally assumed.

## Applied changes

**None.** The research phase is open.

## Proposed changes

Ordered by the strength of the evidence behind them.

### P-01 — Add release age as evidence and as an envelope condition

- **Evidence:** F-03 — S-04 (Renovate recommends `minimumReleaseAge: 14 days` when
  automerging), INC-01 (~4h to remediation), INC-02 (~2h), PR-05 (independent, from the
  regression side)
- **Change:** add release age to [§6](../research/research-file.md#6-observed-evidence);
  give `DELAY_AND_PIN` an evidence-based default duration; add release age to the
  [§9](../research/research-file.md#9-bounded-autonomy) envelope
- **Status:** proposed. Strongest evidence of any item here — Tier 1, 2 and 3 agree

### P-02 — Add a fixed-cooldown policy as Baseline C

- **Evidence:** F-03 — a 14-day cooldown would have been correct on INC-01 and INC-02, where
  the specified policy scores one hit and one confident miss
- **Change:** add Baseline C to [§18](../research/research-file.md#18-baselines-to-investigate)
- **Status:** proposed. **This is the baseline most likely to beat the agent policy.**
  Omitting it would make the evaluation flattering, which is the reason to include it

### P-03 — Reframe publisher-identity change as credential compromise

- **Evidence:** F-01 — publisher identity unchanged in INC-01, INC-02, INC-03
- **Change:** [§10](../research/research-file.md#10-hard-escalation-triggers) trigger 1
  detects account handover, a class absent from all recorded incidents. The observable of
  interest is credential compromise — which is not directly observable at decision time.
  Removing the trigger without a replacement leaves a gap; that gap is the finding
- **Status:** proposed, **unresolved** — no replacement signal identified

### P-04 — Separate *missing* from *mismatched* provenance

- **Evidence:** F-05 — S-01: provenance is unavailable for private source repos and
  unsupported runners, so absence is frequently benign
- **Change:** split [§10](../research/research-file.md#10-hard-escalation-triggers) trigger 2
- **Status:** proposed

### P-05 — Downgrade SemVer from primary evidence to weak prior

- **Evidence:** F-09 — Tier 3 T-01, four of five respondents, including a maintainer
  explaining that SemVer fails even when correctly applied
- **Change:** reorder [§6](../research/research-file.md#6-observed-evidence); retain
  Baseline A but report it as *what teams run and what practitioners say does not work*
- **Status:** proposed

### P-06 — Add a differential-behaviour check to `EXTENDED_CHECK`

- **Evidence:** F-10 — PR-02: *"the thing i actually check is whether a release changes what
  the tool says about unchanged input"*
- **Change:** [§8](../research/research-file.md#8-actions) `EXTENDED_CHECK` gains
  old-vs-new output diffing against unchanged inputs. Expands HYP-02 beyond its current
  security framing
- **Status:** proposed

### P-07 — Add package class as a dimension

- **Evidence:** F-09 — PR-02: SemVer "works fine for plain libraries where the api is the
  whole surface", weak "for anything with rules or defaults in it"
- **Change:** [§12](../research/research-file.md#12-dependency-type-as-a-cost-modifier)
  currently classifies only runtime/dev/build. Add rule-bearing vs plain-API as an
  orthogonal axis
- **Status:** proposed

### P-08 — Resolve the install-script `change` vs `presence` ambiguity

- **Evidence:** [`../research/07-incident-research.md`](../research/07-incident-research.md)
  INC-03 — the verdict changes from CAUGHT to PARTIAL depending on which reading applies
- **Change:** define the comparison explicitly in
  [§6](../research/research-file.md#6-observed-evidence) and
  [§9](../research/research-file.md#9-bounded-autonomy)
- **Status:** proposed. **Must be resolved before any experiment runs**, or results depend
  on an undocumented implementation choice

### P-09 — Revisit ground truth for `unauthorised`

- **Evidence:** F-07 — advisories lag by construction; S-07 withdrawn 28 July 2026
- **Change:** [§4](../research/research-file.md#4-unit-of-analysis) defines
  `unauthorised = 1` via public advisory. Both the lag and the instability of the record
  undermine it
- **Status:** proposed, **unresolved** — no better ground-truth method identified.
  ⚠ The withdrawal reason has not been verified

### P-10 — Address the clean-signals problem

- **Evidence:** F-06 — in INC-02 every envelope condition was satisfied
- **Change:** [§9](../research/research-file.md#9-bounded-autonomy) treats absence of red
  flags as positive evidence of safety. The design needs to distinguish *evidence of safety*
  from *absence of evidence*
- **Status:** proposed, **structural** — not fixable by tuning a threshold. The most
  difficult item on this list
