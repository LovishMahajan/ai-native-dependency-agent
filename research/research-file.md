# AI Dependency Update Decision Agent — Research File

> **Status:** Problem definition / public research
>
> **Project type:** AI-native engineering experiment
>
> **Scope:** One automated npm dependency-update PR, one production TypeScript service, one decision.

---

## 1. Problem

Automated dependency-update tools can create pull requests for new package versions, but a version bump does not tell us whether the release is safe for our particular production service.

The proposed agent evaluates one direct third-party npm dependency update using evidence available **before installation** and chooses one of four routes:

1. `INSTALL_AND_MERGE`
2. `EXTENDED_CHECK`
3. `DELAY_AND_PIN`
4. `REJECT_AND_ESCALATE`

The agent does not write the dependency change, modify upstream code, or communicate with the package maintainer. It owns the routing decision and records the reasoning in a decision log.

### Problem statement

> **The agent observes a proposed direct npm dependency update and must choose an action because the update's actual impact on our service and the integrity of the published artifact are not fully known.**

---

## 2. Why this problem

Dependency automation already solves part of the problem: detecting new versions and opening update pull requests.

The harder question is:

> **When should an automated dependency-update PR be trusted enough to merge, when should we buy more evidence, and when should we stop and involve a human?**

The problem is interesting because the cost of errors is asymmetric.

A regression can cause bounded engineering and rollback work. A compromised dependency can have a much larger blast radius, especially when the dependency executes in a build or CI environment with credentials.

This project therefore treats dependency updating as a **decision-under-uncertainty problem**, not simply as a version-bumping problem.

---

## 3. Scope

### Included

- npm packages
- TypeScript production services
- third-party dependencies
- exactly one direct dependency changed per case
- runtime, development, and build/CI dependency types
- transitive dependency changes caused by the direct bump, observed as evidence

### Excluded

- lockfile-only refreshes
- batch dependency upgrades
- multiple direct dependency changes in one case
- internal/private packages
- agent-generated code patches
- upstream maintainer communication

### Why direct dependencies only?

The action space is defined around the manifest line we control.

A transitive dependency has no direct manifest line. Controlling it through `overrides` or `resolutions` introduces a different action and a different cost structure.

However, the transitive closure is still important evidence.

A direct bump may introduce:

- no transitive changes
- additional transitive packages
- a new publisher in the closure
- an install script in the closure

Therefore:

```text
direct dependency = decision target
transitive closure = evidence
```

---

## 4. Unit of analysis

One case represents:

```text
case_id = (
  package,
  from_version,
  to_version,
  service,
  decision_timestamp
)
```

Each case contains:

```text
evidence = frozen snapshot of information observable at decision time

labels:
  breaks_us ∈ {0,1}
  unauthorised ∈ {0,1}
```

### Regression horizon

`breaks_us = 1` when a defect attributable to the dependency bump is observed within **30 days after merge**.

### Unauthorized-code label

`unauthorised = 1` when the released version is subsequently established as malicious or unauthorized through an appropriate public security record/advisory or equivalent evidence.

This definition is provisional and will be challenged during research.

---

## 5. Hidden state

The project does not collapse all risk into one hidden variable.

There are two independent hidden dimensions.

### H1 — Regression

> Does the release break our service in a way that our available validation does not detect before merge?

### H2 — Unauthorized code

> Does the published artifact contain or execute code that was not authorized by the intended dependency update?

These produce four useful underlying states:

| Regression | Unauthorized code | Interpretation |
|---|---|---|
| 0 | 0 | Safe relative to the defined horizon |
| 1 | 0 | Compatibility/regression failure |
| 0 | 1 | Supply-chain compromise |
| 1 | 1 | Compromise plus observed regression |

The model should therefore represent:

```text
H = (breaks_us, unauthorised)
```

rather than one generic `risk_score`.

---

## 6. Observed evidence

The agent only receives information that was available **before the decision**.

Potential evidence includes:

### Package/release metadata

- semantic-version bump size
- publisher/maintainer identity compared with the previous release
- package release history
- release cadence relative to the package's own history
- changelog quality
- months since the last upstream commit
- download trend
- whether the release fixes a published advisory

### Artifact and provenance signals

- package signature information
- provenance attestation
- registry-to-repository tag/commit consistency
- install-script changes
- package-file diff size
- newly introduced network calls
- newly introduced filesystem calls

### Dependency-graph signals

```text
transitive_delta ∈ {
  none,
  additions_only,
  new_publisher_in_closure,
  install_script_in_closure
}
```

### Our-service evidence

- CI result
- test coverage
- coverage of the API surface actually imported from the package
- dependency type:
  - runtime
  - dev
  - build/CI
- whether the package runs in a credentialed or privileged environment

---

## 7. Evidence freezing

Evidence must be frozen at the decision timestamp.

The experiment must not use information that became available after the decision.

For example, the following are potential sources of leakage:

- a package being yanked after the decision
- an advisory published after the decision
- a later download-count collapse
- later reports from other teams
- later maintainer activity

The rule is:

```text
features(case) =
information observable at decision_timestamp
```

not:

```text
information available when the dataset was constructed
```

This is required for a meaningful evaluation.

---

## 8. Actions

### A1 — INSTALL_AND_MERGE

Install the dependency update and merge the PR.

This is the only action allowed to have merge authority.

### A2 — EXTENDED_CHECK

Acquire additional behavioral evidence:

- sandboxed install
- install-script trace
- integration tests against the actual imported API surface
- additional dependency-graph inspection

Then re-evaluate the case.

The extended check has a real cost and is therefore treated as an information-acquisition action, not free validation.

### A3 — DELAY_AND_PIN

Do not merge now.

Pin the current version and schedule a revisit.

### A4 — REJECT_AND_ESCALATE

Block the update and route it to a human with the observed evidence and decision reasoning attached.

---

## 9. Bounded autonomy

The key design principle is:

> **Autonomy is bounded by reversibility, not confidence.**

A repository change can be reverted while the consequences of executing malicious code may not be reversible.

Therefore, automatic merge is permitted only when the worst-case consequences are considered cheaply reversible under the project's policy.

Initial proposed auto-merge envelope:

- no install-script change
- provenance verified
- publisher unchanged
- CI passes
- CI meaningfully covers the imported API surface
- posterior probability of unauthorized code is below the policy threshold
- dependency is not build/CI privileged
- no hard safety trigger

Everything outside this envelope follows one of the slower actions.

This is an initial policy hypothesis, not a validated result.

---

## 10. Hard escalation triggers

Initial hard triggers:

1. publisher identity changed
2. provenance is missing or mismatched
3. dependency has access to credentials or build/publishing infrastructure
4. evidence is materially contradictory

The public research phase will test whether experienced engineers agree that these should be hard triggers.

---

## 11. Cost model

The project uses asymmetric decision costs.

Initial author estimates:

| Event | Approximate cost |
|---|---:|
| Silent regression | ₹12,000 |
| Unauthorized/compromised release | ₹15,00,000 |
| Unnecessary escalation | Senior engineering time |
| Unnecessary delay | Migration debt / delayed security fixes |
| Extended check | Compute + engineering/CI time |

The current estimated ratio between a compromised release and a regression is approximately:

```text
₹15,00,000 / ₹12,000 ≈ 125
```

These numbers are **estimates, not measured empirical costs**.

They will be treated as model assumptions and subjected to sensitivity analysis rather than presented as observed facts.

---

## 12. Dependency type as a cost modifier

Dependency type is not used as a simple inclusion/exclusion filter.

Instead:

```text
Cost(action, hidden_state, dependency_type)
```

where:

```text
dependency_type ∈ {
  runtime,
  dev,
  build
}
```

The same hidden state can have different consequences depending on where the dependency executes.

A build/CI dependency can be especially sensitive because installation or build execution may occur near:

- CI credentials
- package-publishing credentials
- cloud credentials
- deployment credentials

This is a hypothesis to validate with practitioners and security research.

---

## 13. Decision policy hypothesis

The initial policy is expected to combine:

### Hard constraints

Certain evidence can force escalation regardless of the numerical score.

### Probabilistic belief

Estimate:

```text
P(breaks_us | evidence)

P(unauthorised | evidence)
```

### Expected decision cost

Compare the expected consequences of available actions.

### Information value

For cases where uncertainty is material, determine whether the expected value of an extended check exceeds its cost.

Conceptually:

```text
observe evidence
       ↓
check hard constraints
       ↓
estimate hidden-state beliefs
       ↓
estimate action costs
       ↓
decide whether more evidence is worth buying
       ↓
choose action
       ↓
record decision
```

---

## 14. Research questions

### RQ1 — Merge criteria

What evidence do experienced engineers consider sufficient to automatically merge a dependency update?

### RQ2 — Risk signals

Which observable signals are considered meaningful indicators of dependency regression or supply-chain risk?

### RQ3 — Provenance

How much confidence should valid package provenance provide when deciding whether to trust a dependency update?

### RQ4 — Behavioral evidence

Is behavioral evidence from a sandboxed installation more decision-useful than static package/repository metadata?

### RQ5 — Human escalation

Which dependency-update conditions should always require human review?

### RQ6 — Build dependencies

How should dependency type affect the acceptable risk of an automated merge?

### RQ7 — Transitive closure

How important are newly introduced transitive packages, publishers, or install scripts when reviewing a direct dependency update?

### RQ8 — Waiting

Does delaying an update for additional evidence provide a useful security trade-off, or merely accumulate maintenance debt?

### RQ9 — Asymmetric loss

How should an automated system behave when the probability of a compromise is low but the cost of compromise is extremely high?

### RQ10 — Historical memory

Can previous dependency-update outcomes improve future decisions without causing leakage or overfitting to individual packages or publishers?

---

## 15. Initial hypotheses

These are hypotheses, not conclusions.

### HYP-01

Provenance and publisher continuity are useful but insufficient indicators of safety.

### HYP-02

Behavioral evidence from sandboxed installation can reveal risk that static metadata misses.

### HYP-03

Build/CI dependencies deserve stricter autonomy boundaries than ordinary development dependencies.

### HYP-04

Transitive dependency changes can materially alter the risk of an otherwise small direct version bump.

### HYP-05

Asymmetric loss makes a low probability of compromise potentially unacceptable for privileged dependencies.

### HYP-06

A policy with explicit decision costs will make different decisions from a policy based only on predicted probability.

These hypotheses must be tested or revised after human discussions and experiments.

---

## 16. Experimental plan

The assignment requires 30–50 test cases, at least two agent policies, one baseline, saved predictions/actions, and analysis of incorrect decisions.

Initial plan:

### Evaluation set

30 cases sampled at approximately natural prevalence.

Purpose:

- decision cost
- calibration
- realistic operating behavior

### Diagnostic set

10–20 cases enriched for rare/high-risk states.

Purpose:

- inspect dangerous-case recall
- expose failure modes
- test hard triggers

The sets must not be pooled for a single headline metric.

At a very low compromise base rate, a small natural-prevalence evaluation set may contain zero compromised examples. That does not establish good compromise detection.

---

## 17. Independence and leakage controls

Repeated cases from the same package or publisher are not independent observations.

Record:

- random seed
- fixed case order
- number of distinct packages
- number of distinct publishers
- cases per package
- cases per publisher

If the agent contains publisher/package memory, the ordering of cases must be fixed and documented.

No post-decision evidence may enter the feature snapshot.

---

## 18. Baselines to investigate

Initial baseline candidates:

### Baseline A — Simple SemVer policy

```text
patch → merge
minor → test
major → human
```

### Baseline B — Security-first rule policy

Use explicit hard rules around provenance, publisher changes, install scripts, dependency type, and advisories.

### Agent policy

Use evidence, hidden-state beliefs, action costs, and the bounded-autonomy policy.

The final baseline definitions will be revised after researching existing dependency-update practices.

---

## 19. Failure analysis

At least five incorrect decisions should be examined.

Potential failure categories:

- false reassurance from valid provenance
- missed transitive risk
- insufficient API coverage
- anomalous publisher history
- misleading changelog
- install-script behavior missed before installation
- contradictory evidence
- over-escalation
- unnecessary delay
- historical-memory bias
- post-decision information leakage

Each failure should result in:

```text
failure_condition
→ why the policy failed
→ evidence that was missing/misweighted
→ proposed design change
→ whether the change should be tested
```

---

## 20. Public research plan

The public discussion is not marketing.

The purpose is to challenge our assumptions with engineers who have dealt with dependency updates, npm supply-chain incidents, CI security, and production reliability.

### Reddit

Target communities should be selected based on relevance and verified before posting.

Initial community categories to investigate:

- Node.js / JavaScript engineering
- npm / package-management discussions
- DevOps / CI/CD
- software security
- application security
- supply-chain security
- SRE / production engineering
- TypeScript

Do not copy the same question into every community.

Each contribution should focus on one concrete research question.

### X

Find relevant:

- Node.js engineers
- npm/package maintainers
- application-security engineers
- software supply-chain researchers
- DevSecOps engineers
- SREs
- dependency-management/tooling engineers

Follow the account because of relevant work, not follower count.

### LinkedIn

Publish one project-introduction post only after the problem statement and GitHub research file are public.

The first post should communicate:

1. the problem
2. why dependency automation is insufficient
3. the uncertainty being studied
4. the GitHub research file
5. one or two questions for experienced engineers

Do not claim that the agent works yet.

---

## 21. Initial search queries

Use these as starting points.

### Dependency automation

- `npm dependency update automation production risk`
- `Dependabot auto merge dependency updates production`
- `Renovate dependency update automerge best practices`
- `npm dependency update testing strategy`

### Supply-chain security

- `npm package supply chain attack install scripts`
- `npm package provenance trusted publishing`
- `npm dependency publisher compromise`
- `npm transitive dependency supply chain attack`
- `software supply chain dependency update risk`

### CI/build security

- `npm install scripts CI credentials security`
- `dependency update build pipeline security`
- `npm dependency CI token exfiltration`
- `software supply chain build environment credentials`

### Decision making

- `automated dependency updates human review`
- `dependency update risk scoring`
- `dependency update confidence threshold`
- `dependency update sandbox testing`

---

## 22. Sources to investigate

### npm provenance

npm documents provenance attestations as a mechanism for establishing where and how a package was built and who published it. This is directly relevant to the provenance evidence proposed by this project.

Source: npm Documentation — Generating provenance statements.

### npm trusted publishing

npm documents trusted publishing through OIDC and explains how it reduces reliance on long-lived publishing tokens. This is relevant to the publisher/provenance portion of the model.

Source: npm Documentation — Trusted publishing for npm packages.

### GitHub Dependabot

GitHub documents Dependabot security updates and version updates, establishing the existing automation layer that this project is trying to make more decision-aware.

Source: GitHub Docs — Dependabot quickstart / security updates.

### GitHub dependency review

GitHub's supply-chain documentation describes dependency review as a way to inspect dependency changes, release dates, popularity, and vulnerability information when deciding whether to accept a change.

Source: GitHub Docs — Supply chain security.

These sources are starting points, not evidence that our proposed policy is correct.

---

## 23. Questions for practitioners

These questions will be used in Reddit/X discussions.

### Question 1

What signals make you stop and investigate an automated npm dependency PR instead of merging it?

### Question 2

Would you allow an agent to auto-merge a patch dependency update into production? What conditions would you require?

### Question 3

How much trust do you place in npm provenance when reviewing a new release?

### Question 4

Would a changed publisher identity always require human review?

### Question 5

How do you evaluate newly introduced transitive dependencies?

### Question 6

Do you treat runtime, development, and build/CI dependencies differently?

### Question 7

Would you run a sandboxed `npm install` before merging a suspicious but otherwise valid update?

### Question 8

What dependency-update failure has caused the most expensive incident in your experience?

### Question 9

What evidence would change your mind from "merge" to "investigate"?

### Question 10

Where should an automated dependency agent stop and require a human?

---

## 24. What we currently do not know

This section is intentionally incomplete.

We do not yet know:

- whether engineers consider provenance strong enough to influence merge decisions materially
- whether publisher changes are usually hard escalation triggers
- how practitioners evaluate transitive dependency changes
- how often install scripts are treated as high-risk
- what realistic cost people assign to extended checks
- how teams distinguish build dependencies from runtime dependencies
- whether 30 days is a useful regression-label horizon
- whether public advisories provide sufficient ground truth for unauthorized-code cases
- what the natural prevalence of malicious npm releases is in a defensible evaluation population
- whether a probability model will outperform a simple rule-based baseline

These are research questions, not assumptions to quietly turn into facts.

---

## 25. AI-use policy

AI tools may be used to:

- discover technical terminology
- prepare search queries
- identify relevant communities
- summarize discussions
- identify assumptions and contradictions
- review the agent design
- write and repair code
- review experimental methodology

Human responsibility remains for:

- verifying sources
- reading referenced material
- conducting public discussions
- recording actual responses
- running experiments
- validating results
- deciding which AI suggestions to accept or reject

No fictional conversations, fabricated experiments, or unsupported claims will be included.

---

## 26. Research log

This file should change as evidence arrives.

| Date | Question | Source/person | Observation | Assumption affected | Design change |
|---|---|---|---|---|---|
| | | | | | |

---

## 27. Decision log schema

The eventual agent should produce an auditable decision record containing:

```text
case_id
decision_timestamp
package
from_version
to_version
dependency_type

evidence_snapshot
beliefs
hard_triggers
expected_costs

selected_action
reason

model_version
policy_version
data_version
```

This makes the agent's decisions reproducible and reviewable.

---

## 28. Current project position

### What is decided

- Direct dependency bump is the unit of decision.
- Transitive closure is evidence.
- Runtime/dev/build dependencies are included.
- There are two hidden-state dimensions.
- There are four actions.
- Only install-and-merge has merge authority.
- Autonomy is bounded by reversibility.
- Evidence must be frozen at decision time.
- Evaluation and diagnostic cases remain separate.
- Cost estimates are explicitly treated as estimates.

### What is not decided

- Exact probability model
- Exact thresholds
- Final hard triggers
- Final baseline
- Final case dataset
- Final regression ground-truth method
- Final unauthorized-code ground-truth method
- Whether the proposed evidence signals are actually predictive

Those decisions should be informed by research and experiments rather than assumed in advance.

---

## 29. Next step

The immediate task is **not to build the agent**.

The immediate task is to challenge the assumptions in this document through:

1. verified Reddit communities
2. relevant X engineers/researchers
3. primary technical documentation
4. real dependency-update and supply-chain incidents

The research file should then be updated with what humans and sources actually teach us.

Only after that should the agent policy and experiment be finalized.

---

## References

- npm Documentation — Generating provenance statements
- npm Documentation — Trusted publishing for npm packages
- GitHub Docs — Dependabot quickstart guide
- GitHub Docs — Dependabot security updates
- GitHub Docs — Supply chain security

