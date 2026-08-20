# Research Method

## Evidence hierarchy

Evidence is ranked into four tiers. A claim's tier is recorded wherever it is used, and a
lower-tier claim never silently overrides a higher-tier one.

### Tier 1 — Primary technical sources

Highest weight. Vendor and standards documentation, official incident reports, maintainer
postmortems, CVE/GHSA records, national CERT advisories, peer-reviewed or otherwise
substantive security research.

Examples: npm documentation, GitHub documentation, Renovate documentation, GHSA entries,
CISA advisories.

### Tier 2 — Real incidents

Documented events with an identifiable package, version, date, and mechanism. Analysed as
counterfactuals rather than as anecdotes:

```text
incident
  → what happened
  → what signal existed before the release
  → could an automated system have observed it at decision time?
  → would the proposed four-action policy have acted differently?
```

The final question is the one that matters. An incident that the policy would have merged
is more informative than one it would have caught.

Secondary reporting (vendor security blogs) is acceptable for Tier 2 corroboration, but a
Tier 1 record — an advisory or a maintainer postmortem — is required before an incident is
treated as established.

### Tier 3 — Practitioner experience

Reddit, X, and LinkedIn responses from engineers with relevant production experience.
Weighted below documentation and incidents, but uniquely able to answer questions that no
document answers: what people actually do, what they actually trust, and what has actually
cost them.

**A response only counts if it is real and linked.** See
[`../discussions/README.md`](../discussions/README.md).

### Tier 4 — Author assumptions

The project's own estimates and design choices. These remain **labelled as assumptions**
until Tier 1–3 evidence supports them. Current Tier 4 items include the ₹15,00,000
compromise cost, the ₹12,000 regression cost, the 30-day regression horizon, the four hard
escalation triggers, and the auto-merge envelope conditions.

Tracked in [`../decisions/assumption-log.md`](../decisions/assumption-log.md).

## The rule that makes this research rather than design

> A signal enters the design because evidence supports it, not because it seems useful.

Concretely:

```text
❌  "Provenance would be a useful signal, therefore the agent should check provenance."

✅  Evidence (Tier 1 npm docs: what provenance actually attests)
    + Evidence (Tier 2 incidents: whether provenance was present in real compromises)
    + Evidence (Tier 3: whether engineers actually trust it)
    → finding
    → design decision, recorded in decisions/design-changelog.md
```

Every row of [`../evidence/evidence-matrix.md`](../evidence/evidence-matrix.md) is subject
to this rule.

## Assumption lifecycle

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

An assumption may not move to `SUPPORTED` or `CHALLENGED` without a citation. Movement to
`DESIGN DECISION` requires an entry in
[`../decisions/design-changelog.md`](../decisions/design-changelog.md) naming the evidence
that caused it.

## Verification status

Every external claim in this repository carries a verification flag:

| Flag | Meaning |
|---|---|
| `VERIFIED BY HUMAN: no` | Gathered and cited, but the author has not yet read the primary source end-to-end |
| `VERIFIED BY HUMAN: yes (YYYY-MM-DD)` | The author has read the primary source and confirms the claim |

Unverified claims are usable for shaping research questions. They are **not** usable as
support for a design decision. Per
[`../research/research-file.md` §25](../research/research-file.md#25-ai-use-policy),
verifying sources is a human responsibility.

## What this method deliberately does not do

- It does not weight sources by how well they agree with the project's hypotheses.
- It does not treat consensus as evidence. Six practitioners agreeing that provenance is
  trustworthy is a Tier 3 observation about belief, not a Tier 1 fact about provenance.
- It does not permit a finding to be recorded before its evidence is recorded.
