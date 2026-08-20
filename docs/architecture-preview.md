# Architecture Preview

> **⚠ NOTHING HERE IS IMPLEMENTED.**
>
> This is a sketch of a *proposed* design, recorded so the research has something concrete
> to argue against. There is no code in this repository. Several components below rest on
> assumptions that the research has already challenged — see
> [`../research/09-findings.md`](../research/09-findings.md). This document will change, and
> may be discarded entirely.

## Proposed flow

```text
        proposed dependency update
                    │
                    ▼
        ┌───────────────────────┐
        │  Evidence collectors  │   frozen at decision_timestamp
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   Hard constraints    │──── fires ──▶  REJECT_AND_ESCALATE
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  Belief estimation    │   P(breaks_us | e), P(unauthorised | e)
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  Expected cost, by    │   Cost(action, H, dependency_type)
        │  action               │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  Information value    │   is more evidence worth its cost?
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   Action selection    │
        └───────────┬───────────┘
                    │
        ┌───────────┼───────────┬───────────────┐
        ▼           ▼           ▼               ▼
  INSTALL_AND   EXTENDED    DELAY_AND    REJECT_AND
    _MERGE       _CHECK       _PIN        ESCALATE
                    │
                    └──▶ re-enter with new evidence
                    │
                    ▼
        ┌───────────────────────┐
        │    Decision log       │   §27 schema — auditable, reproducible
        └───────────────────────┘
```

## Components, and what the research says about each

| Component | Status |
|---|---|
| **Evidence collectors** | Signal list is [§6](../research/research-file.md#6-observed-evidence). **Known incomplete** — release age is missing despite the strongest evidence of any signal (F-03), and no signal captures behaviour change on unchanged input (F-10) |
| **Hard constraints** | [§10](../research/research-file.md#10-hard-escalation-triggers). Three of four triggers challenged (F-01, F-05). Only credential/build-infra access survives, with an unmeasured false-positive rate (F-04) |
| **Belief estimation** | No model chosen. Structurally vulnerable to F-06 — when every signal is clean, a correctly-derived low posterior still produces the wrong action |
| **Expected cost** | Driven by [§11](../research/research-file.md#11-cost-model). Every input is an unevidenced author estimate (A-16, A-17, A-21) |
| **Information value** | Requires a cost for `EXTENDED_CHECK`, which nobody has measured (P3) |
| **Action selection** | Four actions, [§8](../research/research-file.md#8-actions). `EXTENDED_CHECK`'s scope is under-specified for both hidden states (F-02, F-10) |
| **Decision log** | [§27](../research/research-file.md#27-decision-log-schema). The least contested part of the design |

## The uncomfortable part

The evidence so far suggests this architecture may be more machinery than the problem
warrants. A fixed cooldown — no collectors, no beliefs, no costs — would have been correct
on two of the three recorded incidents where this design scores one hit and one confident
miss (F-03).

That comparison is the point of Baseline C (P-02), and if it holds, the honest output of
this project is a short paper explaining why the simple thing wins, not this diagram.
