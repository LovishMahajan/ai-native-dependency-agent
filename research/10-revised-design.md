# Revised Design

> **⛔ Not written. Entry condition not met.**

## Entry condition

This document may not be written until
[`09-findings.md`](./09-findings.md) contains a finding, with cited evidence, for each of
HYP-01 through HYP-06 — including practitioner evidence, since several hypotheses cannot be
settled from documentation and incidents alone.

**Current state:**

| Hypothesis | Tier 1–2 evidence | Tier 3 evidence | Ready |
|---|---|---|---|
| HYP-01 — provenance/publisher useful but insufficient | ✅ (challenged — stronger than "insufficient") | ❌ | No |
| HYP-02 — sandbox behaviour beats static metadata | ⚠ qualified — depends on attack channel | ❌ | No |
| HYP-03 — build/CI deps need stricter bounds | ✅ supported | ❌ | No |
| HYP-04 — transitive changes alter risk materially | ✅ supported | ❌ | No |
| HYP-05 — asymmetric loss makes low probability unacceptable | ⚠ complicated by INC-02 | ❌ | No |
| HYP-06 — explicit costs change decisions | ❌ none | ❌ | No |

## Why this file exists empty

This is the file most likely to be written prematurely. The design already exists in the
author's head, the evidence so far is suggestive, and the temptation is to write the
revision now and backfill citations.

Keeping it empty and visible is a control, not an oversight. It makes premature conclusion a
visible act — a commit that changes this file — rather than a quiet drift in
[`research-file.md`](./research-file.md).

Interim design *proposals* arising from evidence are recorded in
[`../decisions/design-changelog.md`](../decisions/design-changelog.md) as proposals, with
their triggering evidence attached. They are not applied to the specification.

## What this document will have to answer

Written now, so the questions cannot be quietly dropped later:

1. **Does the project still have a reason to exist?**
   [`07-incident-research.md`](./07-incident-research.md) finds that a fixed cooldown would
   have been correct on two of three incidents using no evidence model at all. If the
   marginal value over a cooldown is small, that is the finding, and this document should
   say so.
2. **What replaces `publisher unchanged`?** The signal is present in the envelope and did
   not fire in any recorded incident.
3. **How does the policy avoid treating clean signals as evidence of safety?** In INC-02
   every condition was satisfied. This is structural, not a threshold.
4. **What is the scope of `EXTENDED_CHECK`?** Install-time sandboxing missed INC-02
   entirely. Does the extended check include published-tarball source diffing?
5. **Is release age evidence, an envelope condition, or an action?** It is currently none of
   the three, despite the strongest evidence of any signal.
6. **What is the false-positive cost of hard trigger 3?** A rule escalating every build-tool
   update may be correct and unusable at once.
7. **Should `unauthorised` ground truth change?** Advisories lag, and S-07 shows the record
   is not stable over time.

## Structure (when it is written)

1. What the evidence changed
2. Revised evidence model — signals in, signals out, with citations
3. Revised action space
4. Revised hard constraints
5. Revised autonomy envelope
6. Revised cost model and its sensitivity
7. What remains unsupported and why it is kept anyway
8. Honest assessment: agent, deterministic policy, or neither
