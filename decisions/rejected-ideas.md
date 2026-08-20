# Rejected Ideas

Approaches considered and set aside, with the reason. Kept so that they are not
reconsidered without new evidence, and so the reasoning stays checkable.

## Format

```text
### R-NN — <idea>
Considered because:  <why it looked right>
Rejected because:    <what ruled it out>
Evidence:            <citation, or "reasoning only">
Would reopen if:     <what new evidence would change this>
```

---

### R-01 — Collapse risk into a single `risk_score`

**Considered because:** one number is simpler to model, threshold, and report.

**Rejected because:** a regression and a supply-chain compromise are different failures with
different costs, different evidence, and different correct actions. Averaging them produces
a number that cannot drive either decision — a 0.4 from "probably breaks the build" and a
0.4 from "possibly malicious" call for opposite responses.

**Evidence:** [§5](../research/research-file.md#5-hidden-state). Reinforced by the split now
visible in the evidence itself: Tier 2 evidence bears only on `unauthorised`, Tier 3 only on
`breaks_us`.

**Would reopen if:** evidence showed the two states are so strongly correlated in practice
that separating them adds no decision value.

---

### R-02 — Treat transitive dependencies as decision targets

**Considered because:** transitive packages carry real risk, and INC-03 propagates through
the closure.

**Rejected because:** a transitive dependency has no manifest line we control. Acting on it
requires `overrides`/`resolutions` — a different action with a different cost structure and
different failure modes. It is a different research problem, not a bigger version of this one.

**Evidence:** [§3](../research/research-file.md#3-scope).

**Note:** the closure remains *evidence*. Renovate's `--before` already extends cooldown to
transitive resolution (S-04), so the closure is actionable in practice — but through the
direct manifest line, which is consistent with this rejection.

**Would reopen if:** the evaluation showed most realised risk enters through the closure in a
way direct-line evidence cannot represent.

---

### R-03 — Let an LLM output a merge/no-merge decision directly

**Considered because:** it is the obvious shape for an "AI agent", and the simplest to build.

**Rejected because:** it makes the decision unauditable and unreproducible, gives no way to
enforce hard safety constraints, and provides no mechanism for asymmetric cost. It also
makes the research question unanswerable — if the model outputs an action directly, there is
no way to tell whether evidence, cost, or the prompt produced it.

**Evidence:** reasoning only, but consistent with the structure of
[§13](../research/research-file.md#13-decision-policy-hypothesis), which separates hard
constraints, beliefs, costs, and information value precisely so each can be evaluated.

**Would reopen if:** a direct policy measurably outperformed the structured one *and* the
decision log remained auditable.

---

### R-04 — Use post-decision information because it is easier to collect

**Considered because:** advisories, yanks, and download collapses are the clearest signals
available, and building a dataset without them is much harder.

**Rejected because:** it invalidates the entire evaluation. A policy scored on information
that did not exist at decision time will appear to detect compromises it could not have
detected.

**Evidence:** [§7](../research/research-file.md#7-evidence-freezing),
[§17](../research/research-file.md#17-independence-and-leakage-controls),
[`../methodology/leakage-controls.md`](../methodology/leakage-controls.md).

**Would reopen if:** never, for evaluation. Post-decision information is legitimate for
*labelling* — with the caveats in F-07 — but never as a feature.

---

### R-05 — Build the agent first and research afterwards

**Considered because:** it demonstrates progress faster and produces something to show.

**Rejected because:** the research has already invalidated significant parts of the design.
Two of the five conditions in the auto-merge envelope have Tier 2 evidence against them
(F-01, F-02), the strongest available signal is absent from the specification entirely
(F-03), and the primary evidence signal is rejected by practitioners (F-09). An
implementation built before this evidence arrived would have encoded all four mistakes.

**Evidence:** [`../research/07-incident-research.md`](../research/07-incident-research.md),
[`../research/09-findings.md`](../research/09-findings.md).

**Would reopen if:** the research phase closes.
