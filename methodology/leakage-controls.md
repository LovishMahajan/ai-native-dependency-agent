# Leakage Controls

> **Pointer document.** Authoritative definitions live in
> [`research-file.md` §7](../research/research-file.md#7-evidence-freezing) and
> [`§17`](../research/research-file.md#17-independence-and-leakage-controls).

Leakage is the failure mode most likely to make this project produce an impressive and
meaningless result. A policy evaluated on information that did not exist at decision time
will appear to detect compromises it could not actually have detected.

## The core rule

```text
features(case) = information observable at decision_timestamp
```

not

```text
information available when the dataset was constructed
```

## Known leakage vectors

| Vector | Why it leaks |
|---|---|
| Package yanked/unpublished after the decision | Absence from the registry is post-decision information |
| Advisory published after the decision | GHSA/CVE records are a lagging indicator by construction |
| Download-count collapse | Reflects the ecosystem reacting to the incident |
| Reports from other teams | Post-decision community signal |
| Later maintainer activity | Includes remediation commits and revert releases |
| Current registry state of any kind | The registry reflects *now*, not the decision moment |

## Vectors specific to this project's incident research

The Tier 2 counterfactual analysis in
[`../research/07-incident-research.md`](../research/07-incident-research.md) is especially
exposed, because incidents are by definition studied after they were resolved. Two rules
apply there:

1. **Hindsight labelling is not permitted in the counterfactual.** When asking "would the
   policy have caught this", only signals that existed *before the malicious version was
   pulled* may be used. That a package is now known-malicious is the answer, not an input.
2. **Reverted versions must be reconstructed, not observed.** Several incidents were
   remediated within hours. The registry no longer shows what a decision-time observer
   would have seen, so the evidence snapshot must be reconstructed from advisories and
   postmortems — and the reconstruction flagged as such.

## Independence

Repeated cases from the same package or publisher are not independent observations.
Per §17, record:

- random seed
- fixed case order
- number of distinct packages
- number of distinct publishers
- cases per package
- cases per publisher

If the agent carries publisher or package memory across cases (RQ10), case ordering becomes
part of the experimental design and must be fixed and documented before any run.

## Audit

Before any result is reported, each field in the evidence snapshot must be justified
against a single question:

> Could an observer standing at `decision_timestamp`, with no knowledge of what happened
> afterwards, have obtained this value?

If the answer is no, or unclear, the field is excluded and the exclusion recorded.
