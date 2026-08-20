# Dependency Update Decision Agent — Research

Researching whether software dependency updates can be safely automated as
**decision-making under uncertainty** rather than version bumping.

> **🟡 Research phase. No agent has been implemented yet.**
>
> This repository contains a research specification, primary-source research, incident
> analysis, and an evidence trail. It contains no agent, no model, and no experimental
> results. Nothing described here should be read as a working system.

---

## Problem

Dependabot and Renovate can tell us that a newer version of a dependency exists, and can
open a pull request for it. They do not answer the question a reviewer actually has:

> **Should this particular update, of this particular package, be trusted enough to merge
> into this particular service?**

A version bump is not evidence of safety. This project investigates whether that decision
can be made from observable evidence, explicit and asymmetric costs, and bounded autonomy —
and whether an AI system is even the right tool for it.

## Decision space

The system under study observes one proposed direct npm dependency update and selects one
of four actions:

| Action | Meaning |
|---|---|
| `INSTALL_AND_MERGE` | Install and merge. The only action with merge authority. |
| `EXTENDED_CHECK` | Buy more evidence (sandboxed install, install-script trace, API-surface tests), then re-decide. |
| `DELAY_AND_PIN` | Do not merge now. Pin the current version and revisit. |
| `REJECT_AND_ESCALATE` | Block and route to a human with evidence attached. |

Risk is not collapsed into a single score. Two independent hidden dimensions are modelled:

```text
H = (breaks_us, unauthorised)
```

— a regression and a supply-chain compromise are different failures with different costs,
and an update can be either, neither, or both.

## Research questions

1. What evidence do engineers consider sufficient to merge automatically?
2. Which observable signals actually predict dependency risk?
3. How much confidence should valid provenance provide?
4. Which conditions should always require human review?
5. Does behavioural evidence outperform static metadata?
6. How should build/CI dependencies differ from runtime dependencies?
7. How important are transitive dependency changes?
8. When is waiting worth its cost?
9. How should a system behave when compromise is unlikely but very expensive?
10. Can historical outcomes improve decisions without leakage or overfitting?

Full statements: [`research/research-file.md` §14](./research/research-file.md#14-research-questions).

## Repository map

| Directory | Contents |
|---|---|
| [`research/`](./research/) | The research specification and its synthesis — findings, incident analysis, existing solutions |
| [`evidence/`](./evidence/) | Raw cited records: sources, incidents, practitioner responses, the evidence matrix |
| [`decisions/`](./decisions/) | Assumption log with lifecycle states, design changelog, rejected ideas |
| [`methodology/`](./methodology/) | Evidence hierarchy, evaluation plan, leakage controls |
| [`discussions/`](./discussions/) | Public research protocol and logs for Reddit, X, LinkedIn |
| [`docs/`](./docs/) | Proposed architecture sketch — **not implemented** |

Start here: **[`research/README.md`](./research/README.md)**.

## Status

| Phase | Status |
|---|---|
| Problem definition | ✅ |
| Research questions | ✅ |
| Initial hypotheses | ✅ |
| Existing-solution research | 🔄 In progress |
| Incident research | 🔄 In progress |
| Practitioner research | ⏳ Not started |
| Evidence synthesis | 🔄 In progress |
| Revised decision policy | ⏳ Blocked on research |
| Experimental dataset | ⏳ Not started |
| Agent implementation | ⏳ Not started |

## A note on outcomes

This research is permitted to conclude that the agent should not be built. Legitimate
outcomes include:

- a simple deterministic policy performs as well as or better than an AI agent;
- AI reasoning is useful only for evidence synthesis, while hard security constraints
  remain deterministic;
- the problem is not sufficiently observable for safe autonomous decisions at all.

The design is structured to make those outcomes visible rather than to argue past them.

## Evidence and verification

All external claims carry a source link and a verification flag. Claims marked
`VERIFIED BY HUMAN: no` have been gathered but **not yet independently checked by the
author against the primary source**. No practitioner response appears anywhere in this
repository unless it is a real, linked reply from a real person.

See [`methodology/research-method.md`](./methodology/research-method.md) for the evidence
hierarchy and [`research/research-file.md` §25](./research/research-file.md#25-ai-use-policy)
for the AI-use policy.
