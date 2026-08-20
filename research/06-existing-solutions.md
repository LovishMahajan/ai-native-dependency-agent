# Existing Solutions

**Tier 1 analysis.** Records and links live in [`../evidence/sources.md`](../evidence/sources.md);
this document is the reading of them.

The question asked of each tool is not "what features does it have" but:

> **What decision does it actually make, on what evidence, and what does it leave to a human?**

That framing matters because this project's contribution — if it has one — is in the gap
these tools leave, and it is easy to overstate that gap by describing existing tools as
dumber than they are.

---

## Dependabot (S-03)

**Decision made:** that a newer version exists, and — for security updates — that a known
advisory affects the current one. It opens a PR.

**Decision not made:** whether to merge. Auto-merge is not a Dependabot capability. It
requires the repository to enable auto-merge and a separate GitHub Actions workflow, which
conventionally uses `dependabot/fetch-metadata` to read the update type and gate on it,
commonly permitting patch and minor only. GitHub merges once required checks pass.

**What this means for us.** The prevailing production auto-merge rule is:

```text
SemVer bump size ∈ {patch, minor}  AND  CI green  →  merge
```

This is exactly **Baseline A** in [§18](./research-file.md#18-baselines-to-investigate).
Baseline A is therefore not a strawman invented for comparison — it is what a large number
of teams actually run, which makes it the right thing to beat and raises the bar for
claiming improvement.

It is also worth stating plainly what that rule assumes: that version-number semantics
chosen by the *publisher* are a safety signal about the *publish*. In a compromise, the
attacker chooses the version number.

---

## Renovate (S-04, S-05)

Renovate is the most important prior art for this project, and more of the proposed design
already exists in it than [§18](./research-file.md#18-baselines-to-investigate) implies.

### `minimumReleaseAge` — an existing implementation of `DELAY_AND_PIN`

Renovate refuses updates newer than a configured age, and **explicitly recommends
`minimumReleaseAge: "14 days"` when automerging third-party dependencies**, with the stated
rationale of giving registries time to pull malicious packages before merge.

Three things follow.

**First, the project's `DELAY_AND_PIN` action is not novel.** It is deployed, documented,
and vendor-recommended with a concrete default. The research question is not *whether*
waiting helps but *how much, at what maintenance cost, and whether the duration should be
fixed or evidence-dependent* — a sharper version of RQ8.

**Second, the cooldown covers the transitive closure.** When configured, Renovate passes
`--before=<date>` to npm during lockfile generation, so npm resolves only versions that
existed before the threshold. Newly published transitive dependencies are excluded too.
This is directly relevant to HYP-04 and to the §3 decision to treat the transitive closure
as evidence: existing tooling already acts on the closure, not merely observes it.

**Third — the finding with the most force — a 14-day cooldown would have caught INC-01 and
INC-02 on elapsed time alone**, with no evidence model, no provenance check, no belief
estimation, and no AI. Both were remediated within hours (~4 h and ~2 h respectively). A
policy that simply waits would have been correct on both, and would have needed to know
nothing at all.

This is the strongest challenge the research has produced so far, and it is a challenge to
the project's premise rather than to a detail of it: **a trivially simple deterministic
policy may dominate on the cases that matter most.** [§9](./research-file.md#9-bounded-autonomy)
does not currently include release age in the auto-merge envelope, and
[§6](./research-file.md#6-observed-evidence) does not list it as evidence.

### Automerge configuration

Renovate exposes automerge as configurable policy — per-package, per-update-type, with a
dependency dashboard for human visibility. Its configuration surface is effectively a
hand-authored version of the policy this project proposes to derive from evidence, and is
therefore a catalogue of the distinctions practitioners have already found necessary.
**Baseline B should be built from Renovate's actual options rather than invented**, or the
comparison risks beating a policy nobody runs.

---

## npm provenance and trusted publishing (S-01, S-02)

**Decision made:** none. These are attestation mechanisms, not decision tools. They let a
consumer verify *where and how* a package was built and that a publish came from a
specifically authorised workflow.

**What they establish — and the boundary that matters for RQ3:**

```text
provenance attests:  this artifact was built by workflow W from repository R
provenance does not attest:  the code in repository R is safe
                             workflow W was not itself compromised
```

INC-01 sits precisely on that boundary. The attacker did not forge a build; they induced the
project's *real* `publish.yml` workflow to publish, having obtained the token through a
command injection in a PR-validation workflow. A provenance attestation generated in that
circumstance would be **cryptographically valid and semantically true** — the package really
was built by the authorised workflow from the authorised repository.

Trusted publishing narrows the credential-theft surface by removing long-lived tokens, which
is a real and substantial improvement against INC-02-style phishing. It does not address
workflow compromise.

Two practical constraints bound how much any policy can lean on provenance: it is
unavailable when publishing from private source repositories, and it requires a
cloud-hosted runner on a supported provider (GitHub Actions, GitLab CI/CD). Absence of
provenance is therefore not evidence of wrongdoing — a point [§10](./research-file.md#10-hard-escalation-triggers)
should reckon with, since it currently makes "provenance is missing or mismatched" a hard
escalation trigger. *Mismatched* and *missing* are very different observations.

---

## GitHub dependency review (S-06)

**Decision made:** none. It surfaces dependency changes, release dates, popularity, and
vulnerability information at review time and leaves the judgement to the reviewer.

**Useful as a prior.** The evidence the platform chose to surface is a signal about what is
considered decision-relevant by people who have studied this problem at scale. Notably it
includes **release date** — converging with Renovate's cooldown on the same conclusion:
recency is decision-relevant.

---

## `npm audit`

**Decision made:** whether known advisories match the installed tree.

**Structural limitation, and it is the important one.** Advisories are published after
discovery, which is after publication and therefore after the decision. For a
zero-day-style supply chain compromise, `npm audit` is silent at exactly the moment a
decision is being made. All three recorded incidents were invisible to it during their
exposure windows.

This compounds a problem in [§4](./research-file.md#4-unit-of-analysis), which proposes to
define `unauthorised = 1` via public advisory. The same lag that makes advisories weak as a
*signal* makes them weak as *ground truth*, and S-07's withdrawal on 28 July 2026 shows the
record is not even stable over time. See
[`../methodology/evaluation-plan.md`](../methodology/evaluation-plan.md).

---

## Where the gap actually is

Honest summary of what existing tools already do:

| Capability | Exists today | Where |
|---|---|---|
| Detect new versions, open PRs | ✅ | Dependabot, Renovate |
| Gate merge on SemVer + CI | ✅ | Dependabot + Actions |
| Delay merge pending ecosystem scrutiny | ✅ | Renovate `minimumReleaseAge` |
| Extend the delay to transitive resolution | ✅ | Renovate `--before` |
| Attest build origin | ✅ | npm provenance |
| Reduce credential exposure | ✅ | npm trusted publishing |
| Surface change metadata for review | ✅ | GitHub dependency review |
| Match known advisories | ✅ | `npm audit`, Dependabot security updates |
| **Acquire behavioural evidence before deciding** | ❌ | — (HYP-02) |
| **Weight evidence by explicit asymmetric cost** | ❌ | — (HYP-06) |
| **Decide whether more evidence is worth buying** | ❌ | — (§13 information value) |
| **Vary autonomy by reversibility of the worst case** | ❌ | — (§9) |

The genuine gap is narrower than [§2](./research-file.md#2-why-this-problem) suggests. Most
of the *evidence* the project proposes to gather is already gathered by something; what is
missing is **acting on it as a cost-weighted decision under uncertainty, and buying more
evidence when uncertainty is material**.

That is a defensible contribution. But it must be defended against the cooldown result
above: if fixed 14-day waiting captures most of the available benefit, the marginal value of
everything else in this design is small, and the honest conclusion may be the one
[`README.md`](../README.md) already anticipates — that a simple deterministic policy wins.

---

## Open questions this raises

1. Should release age be added to [§6](./research-file.md#6-observed-evidence) as evidence
   and to [§9](./research-file.md#9-bounded-autonomy) as an envelope condition? *(The
   evidence says yes; recorded as a proposed change, not yet made.)*
2. Should a fixed-cooldown policy be added as **Baseline C**? Given that it would have been
   correct on two of three incidents, omitting it would make the evaluation flattering.
3. Should [§10](./research-file.md#10-hard-escalation-triggers) distinguish *missing*
   provenance (common, structural, often benign) from *mismatched* provenance (anomalous)?
4. If Baseline B is built from Renovate's real options, is the "agent policy" meaningfully
   different from a well-configured Renovate?

## Not yet researched

- `pnpm` / `yarn` equivalents and their differing install-script defaults
- Socket, Snyk, Sonatype and similar commercial supply-chain tools, several of which claim
  behavioural analysis and are therefore the closest prior art to HYP-02
- npm's own registry-side malware detection and unpublish policy
- Sigstore / SLSA provenance beyond npm's implementation
