# Research

This directory holds the research specification and the synthesis built on top of it.

The specification — [`research-file.md`](./research-file.md) — is kept as a **single
intact document**. It is the dated baseline for the project: findings are recorded in
[`09-findings.md`](./09-findings.md) and [`../decisions/assumption-log.md`](../decisions/assumption-log.md)
rather than by silently editing the specification, so that the gap between what was
originally assumed and what the evidence showed stays visible and diffable.

## Specification index

The five foundational topics live as sections of the specification:

| Topic | Location |
|---|---|
| Problem definition | [§1 Problem](./research-file.md#1-problem), [§2 Why this problem](./research-file.md#2-why-this-problem), [§3 Scope](./research-file.md#3-scope) |
| Research questions | [§14 Research questions](./research-file.md#14-research-questions) (RQ1–RQ10) |
| Hypotheses | [§15 Initial hypotheses](./research-file.md#15-initial-hypotheses) (HYP-01–HYP-06) |
| Evidence model | [§4 Unit of analysis](./research-file.md#4-unit-of-analysis), [§5 Hidden state](./research-file.md#5-hidden-state), [§6 Observed evidence](./research-file.md#6-observed-evidence), [§7 Evidence freezing](./research-file.md#7-evidence-freezing) |
| Decision model | [§8 Actions](./research-file.md#8-actions), [§9 Bounded autonomy](./research-file.md#9-bounded-autonomy), [§10 Hard escalation triggers](./research-file.md#10-hard-escalation-triggers), [§11 Cost model](./research-file.md#11-cost-model), [§12 Dependency type](./research-file.md#12-dependency-type-as-a-cost-modifier), [§13 Decision policy hypothesis](./research-file.md#13-decision-policy-hypothesis) |

Also in the specification: [§16 Experimental plan](./research-file.md#16-experimental-plan),
[§17 Leakage controls](./research-file.md#17-independence-and-leakage-controls),
[§18 Baselines](./research-file.md#18-baselines-to-investigate),
[§19 Failure analysis](./research-file.md#19-failure-analysis),
[§25 AI-use policy](./research-file.md#25-ai-use-policy),
[§27 Decision log schema](./research-file.md#27-decision-log-schema).

## Research documents

| File | Tier | Status |
|---|---|---|
| [`06-existing-solutions.md`](./06-existing-solutions.md) | 1 — primary technical sources | 🔄 In progress |
| [`07-incident-research.md`](./07-incident-research.md) | 2 — real incidents | 🔄 In progress |
| [`08-practitioner-research.md`](./08-practitioner-research.md) | 3 — practitioner experience | ⏳ Protocol only, no responses |
| [`09-findings.md`](./09-findings.md) | synthesis | 🔄 In progress |
| [`10-revised-design.md`](./10-revised-design.md) | conclusion | ⏳ Blocked — entry condition not met |

## How this directory relates to `evidence/`

`evidence/` holds **raw cited records** — what a source says, with a link. `research/`
holds **synthesis** — what those records mean for the hypotheses.

```text
evidence/sources.md    →  research/06-existing-solutions.md
evidence/incidents.md  →  research/07-incident-research.md
evidence/practitioner-responses.md  →  research/08-practitioner-research.md
                       ↓
              research/09-findings.md
                       ↓
              research/10-revised-design.md
```

Records are not restated in the synthesis documents; they are cited.

## Reading order

1. [`research-file.md`](./research-file.md) — what is being studied and why
2. [`06-existing-solutions.md`](./06-existing-solutions.md) — what already exists
3. [`07-incident-research.md`](./07-incident-research.md) — what has actually gone wrong
4. [`09-findings.md`](./09-findings.md) — what the evidence has changed so far
