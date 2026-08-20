# Evidence Matrix

One row per proposed signal. A signal earns a place in the design by evidence, not by
seeming useful — see [`../methodology/research-method.md`](../methodology/research-method.md).

## Legend

- **Tier** — highest evidence tier currently supporting the row (1–4; 4 = author assumption only)
- **Strength** — `strong` / `moderate` / `weak` / `none`
- **Verified** — whether a human has read the cited primary sources
- **Decision impact** — what the evidence says the signal should do in the policy, *so far*

| Signal | Why we think it matters | Evidence | Tier | Strength | Hypothesis | Decision impact | Verified |
|---|---|---|---|---|---|---|---|
| **SemVer bump size** | Major/minor/patch may correlate with breakage | Dependabot gates on update type (S-03) — establishes it is *used*. **Tier 3: PR-01, PR-02, PR-03, PR-05 all reject it as sufficient.** PR-02 (maintainer) gives the mechanism: the number is chosen by a human at tag time and "has no word for the thing that actually breaks you" | 1 + **3** | **strong, negative** | — | **Contradicted.** Weak predictor of `breaks_us`. Directly undermines Baseline A. See F-09 | **yes** |
| **Provenance attestation** | Establishes where and how a package was built and by whom | S-01, S-02 | 1 | moderate (for what it attests) / **none** (for safety) | HYP-01 | Attests build origin only. Cannot distinguish a legitimate build of malicious code. Whether any INC package carried provenance is **unknown** — see [`incidents.md`](./incidents.md) gaps | no |
| **Publisher identity change** | Possible account takeover | INC-01, INC-02, INC-03 — identity unchanged in **all three** | 2 | strong, **negative** | HYP-01 | **Challenges** the §10 hard trigger. Detects handover, misses token theft and phishing — the mechanism in every recorded incident | no |
| **Publishing-credential compromise** *(new row — the incidents force this distinction)* | Token theft and phishing produce legitimate-looking publishes | INC-01 (workflow injection), INC-02 (phishing), INC-03 (worm-harvested tokens) | 2 | strong | HYP-01 | The actual attack class. Not directly observable at decision time — this is the gap the design must confront | no |
| **Install-script presence/change** | Executes during installation, near credentials | INC-01, INC-03 present; **INC-02 absent** | 2 | moderate | HYP-02 | Necessary but **not sufficient**. §9's "no install-script change" would have auto-merged INC-02 | no |
| **Transitive publisher / closure change** | Expands supply-chain surface | INC-03 propagates along token reach to arbitrary depth; Renovate's `--before` cooldown explicitly covers transitive resolution (S-04) | 1 + 2 | moderate | HYP-04 | Supported. Direct-only evaluation is insufficient; vendor tooling already treats the closure as in scope | no |
| **Release age / cooldown** | Time lets the ecosystem detect and pull malicious releases | S-04 — Renovate **recommends 14 days** when automerging. INC-01 ~4 h, INC-02 ~2 h remediation. **Tier 3: PR-05 independently arrives at it from the regression side** — check "whether other people are reporting issues with the new version" | 1 + 2 + **3** | **strong** | HYP-05, RQ8 | Strongly supported across all three tiers, and currently **absent from §6 and §9**. A cooldown would have caught INC-01 and INC-02 on time alone | partial |
| **CI result** | Tests actual application behaviour | Tier 3: PR-03 ("not safe to merge until a full round of testing"), PR-05 ("let the tests have the final say") | 3 | moderate | — | Supported for `breaks_us`. **Useless for `unauthorised`** — none of INC-01–03 would have failed CI | yes |
| **API-surface coverage** | Determines how meaningful a CI pass is | — | 4 | none | — | Assumed. Needs Tier 3 evidence | no |
| **Dependency type (runtime/dev/build)** | Changes blast radius | INC-01 (build tool, harvested CI credentials); INC-03 (harvested CI/CD env, cloud metadata) | 2 | moderate–strong | HYP-03 | Supported. Build/CI proximity to credentials is the realised harm in two of three incidents | no |
| **Changelog quality** | Poor changelogs may signal risk | **Tier 3: PR-01, PR-02, PR-05.** PR-02 gives an operational test — does the changelog name *behaviour* or name *code*? PR-01 escalates one-line changelogs to major-equivalent handling | **3** | **strong** | — | **Supported, and now implementable.** Promoted from assumption to evidenced signal. Still note a compromised release can carry a plausible changelog | yes |
| **Behaviour change on unchanged input** *(new — introduced by PR-02)* | A release can alter what a tool reports about code the consumer never touched, with no API change at all | Tier 3: PR-02, a linter maintainer — tightening one rule is "a patch by the letter of semver" yet turns consumer CI red | **3** | moderate | — | **Not in [§6](../research/research-file.md#6-observed-evidence) at all.** A `breaks_us` mechanism the evidence model cannot currently represent. See F-10 | yes |
| **Package class (rules/defaults vs plain API)** *(new — introduced by PR-02)* | SemVer "works fine for plain libraries where the api is the whole surface" but is weak "for anything with rules or defaults in it" | Tier 3: PR-02 | **3** | moderate | — | A predicate on the *package*, not the release. The design has no such notion; [§12](../research/research-file.md#12-dependency-type-as-a-cost-modifier) classifies only by runtime/dev/build | yes |
| **Type-level compatibility** *(new — introduced by PR-04)* | A type-only break passes runtime tests and fails the build | Tier 3: PR-04 (brief; linked resource not retrieved) | 3 | weak | — | Relevant because [§3](../research/research-file.md#3-scope) scopes to TypeScript services. Needs follow-up | partial |
| **Release cadence anomaly** | Off-pattern releases may signal compromise | INC-01 published across two major series concurrently — anomalous, but observed post hoc | 2 | weak | — | Promising but unquantified. Needs a base rate of *benign* anomalous cadence before it is usable | no |
| **Download trend** | Popularity as a proxy for scrutiny | — | 4 | none | — | Assumed. **Leakage risk** — post-incident collapse is not decision-time information. See [`../methodology/leakage-controls.md`](../methodology/leakage-controls.md) | no |
| **Published advisory exists** | Direct evidence of known compromise | S-07 advisory **withdrawn 28 July 2026** | 1 | moderate, **with caveat** | — | Lagging by construction, and the record is not stable over time. Weak as ground truth, weaker still as a decision-time signal | no |

## What this matrix currently shows

Two of the five conditions in the §9 auto-merge envelope — `publisher unchanged` and
`no install-script change` — have **Tier 2 evidence against them**, and the one signal with
strong Tier 1 *and* Tier 2 support (release age / cooldown) does not appear in the
specification at all.

That is a real result, and it is the reason
[`../research/10-revised-design.md`](../research/10-revised-design.md) exists but is not yet
written: the revision should follow the full evidence set, not the first three incidents.

## What Tier 3 changed

The first practitioner evidence (thread T-01, five respondents) landed on the **regression**
half of the hidden state, which had no evidence at all before it. It moved three rows:

- **SemVer** fell from "unresolved" to **contradicted** — four independent respondents, one
  of them a maintainer explaining the publishing-side mechanism.
- **Changelog quality** rose from Tier 4 assumption to **evidenced signal with an
  operational test** — the single most actionable thing collected so far.
- **Release age** gained independent Tier 3 support from a respondent who reached it from
  the regression side without being asked about supply chain.

It also introduced three signals the design did not have.

## What is missing

- **Tier 3 on supply chain is still zero.** Every collected response concerns `breaks_us`.
  RQ3–RQ7 and RQ9–RQ10 have no practitioner evidence at all.
- **No benign base rates.** Every row above is informed by compromises. Without knowing how
  often these signals fire on *safe* updates, none of them has a usable false-positive rate,
  and a signal that fires on everything is worthless regardless of its recall.
- **Asymmetry between the two hidden states.** Tier 2 evidence covers only `unauthorised`;
  Tier 3 evidence covers only `breaks_us`. Neither state has been examined from both
  directions, and no evidence source yet speaks to both.
