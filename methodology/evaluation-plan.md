# Evaluation Plan

> **Pointer document.** The authoritative experimental design lives in the specification.
> This file records the decisions that shape evaluation and links to them, rather than
> duplicating them — there is one source of truth.

## Authoritative sources

| Topic | Location |
|---|---|
| Experimental plan, set sizes, split rationale | [`research-file.md` §16](../research/research-file.md#16-experimental-plan) |
| Independence and leakage controls | [`research-file.md` §17](../research/research-file.md#17-independence-and-leakage-controls) |
| Baseline definitions | [`research-file.md` §18](../research/research-file.md#18-baselines-to-investigate) |
| Failure analysis protocol | [`research-file.md` §19](../research/research-file.md#19-failure-analysis) |
| Decision log schema | [`research-file.md` §27](../research/research-file.md#27-decision-log-schema) |

## Shape of the evaluation

Two sets, never pooled into a single headline metric:

| Set | Size | Sampling | Purpose |
|---|---|---|---|
| Evaluation | 30 | Approximately natural prevalence | Decision cost, calibration, realistic operating behaviour |
| Diagnostic | 10–20 | Enriched for rare high-risk states | Dangerous-case recall, failure modes, hard-trigger behaviour |

The reason for the split is stated in §16 and is worth repeating because it is the most
common way this kind of evaluation is misreported: **at a very low compromise base rate, a
small natural-prevalence set may contain zero compromised examples.** A policy that scores
perfectly on such a set has demonstrated nothing about compromise detection.

## Policies compared

Per §18 — subject to revision once
[`../research/06-existing-solutions.md`](../research/06-existing-solutions.md) is complete,
since baselines should reflect what practitioners actually run:

- **Baseline A** — simple SemVer policy (patch → merge, minor → test, major → human)
- **Baseline B** — security-first rule policy (provenance, publisher, install scripts,
  dependency type, advisories)
- **Agent policy** — evidence, hidden-state beliefs, action costs, bounded autonomy

## Open evaluation questions

These are unresolved and must not be assumed away:

1. **Ground truth for `unauthorised`.** §4 defines it via public advisory. Advisories are
   published *after* the decision, are incomplete, and are themselves a lagging indicator.
   Whether this is defensible ground truth is an open question — see
   [`research-file.md` §24](../research/research-file.md#24-what-we-currently-do-not-know).
2. **Ground truth for `breaks_us`.** The 30-day horizon is a Tier 4 assumption.
3. **Base rate.** The natural prevalence of malicious npm releases in a defensible
   evaluation population is not known. Without it, "approximately natural prevalence"
   cannot be operationalised.
4. **Cost sensitivity.** The ≈125 ratio between compromise and regression cost drives
   every expected-cost comparison. Results must be reported with sensitivity analysis
   across a range of ratios, not at a single point estimate.
5. **Whether the comparison is fair.** If the agent policy has access to evidence the
   baselines do not, the comparison measures evidence access, not policy quality.

## Reporting requirements

Any published result must state:

- both sets separately, never pooled;
- the random seed, case order, and the counts required by §17;
- the cost ratio used, plus sensitivity across alternatives;
- at least five incorrect decisions analysed per §19;
- which evidence was available to which policy.

A result that cannot be reported this way is not reportable.
