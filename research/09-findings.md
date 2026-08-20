# Findings

Synthesis across evidence tiers. Each finding names its evidence, its confidence, and its
design impact.

**Confidence is capped at Medium wherever Tier 3 (practitioner) evidence is absent.** The
first practitioner evidence (thread T-01, five respondents) covers **regression only**, so
F-01 through F-08 — all of which concern supply-chain compromise — remain capped. F-09 and
F-10 carry Tier 3 support.

| # | Finding | Confidence |
|---|---|---|
| F-01 | Publisher-identity continuity is a misleading safety signal | Medium |
| F-02 | Absence of install scripts does not bound blast radius | Medium |
| F-03 | Release age is the strongest single signal found, and is absent from the design | Medium |
| F-04 | Build/CI dependency privilege is the realised harm | Medium |
| F-05 | Provenance attests build origin, not code safety | Medium |
| F-06 | Clean signals are treated as evidence of safety — a structural defect | Medium |
| F-07 | Advisories are unstable as ground truth | Low |
| F-08 | The gap versus existing tools is narrower than claimed | Medium |
| F-09 | SemVer is rejected as a safety signal by practitioners, including on the publishing side | Medium-High |
| F-10 | The evidence model cannot represent the regression mechanism practitioners actually described | Medium |

---

## F-01 — Publisher-identity continuity is a misleading safety signal

**Original assumption.** [§10](./research-file.md#10-hard-escalation-triggers) makes
"publisher identity changed" a hard escalation trigger; [§9](./research-file.md#9-bounded-autonomy)
makes "publisher unchanged" a condition for auto-merge. HYP-01 calls publisher continuity
"useful but insufficient".

**Evidence.** In INC-01, INC-02, and INC-03 the publisher identity was **unchanged**.
Initial access was workflow command injection (S-07), maintainer phishing (S-13–S-15), and
worm-harvested tokens (S-08–S-12) respectively. Every attack published through legitimate
credentials.

**Practitioner evidence.** `PENDING — Tier 3`. Directly addressed by Q4.

**Conclusion.** **Contradicted, more strongly than HYP-01 states.** The signal is not merely
insufficient. Because a clean value *lowers* the posterior in the §9 envelope, it actively
pushes toward merge in exactly the cases that matter. The trigger detects account handover —
a class absent from all recorded incidents — while missing credential compromise, the class
present in all of them.

**Design impact.** The observable of interest is *credential compromise*, not *identity
change*, and it is not directly observable at decision time. Proposed, not applied — see
[`../decisions/design-changelog.md`](../decisions/design-changelog.md).

**Confidence: Medium.** Three incidents, one recent window, no practitioner input, and no
data on how often benign publisher changes occur.

---

## F-02 — Absence of install scripts does not bound blast radius

**Original assumption.** [§9](./research-file.md#9-bounded-autonomy) lists "no
install-script change" as an auto-merge condition; HYP-02 expects sandboxed install to
reveal risk static metadata misses.

**Evidence.** INC-02 carried **no install script**. The payload executed in end users'
browsers, intercepting wallet interactions. It would not appear in an install trace, a Node
integration test, or a dependency-graph diff. INC-01 and INC-03 did use `postinstall`, so
the signal is real — just not sufficient.

**Practitioner evidence.** `PENDING — Tier 3`. Addressed by Q7 and P2.

**Conclusion.** **Qualified.** Install-script analysis is necessary and not sufficient.
HYP-02 holds where the attack channel is install-time and fails where it is not — its
validity is conditional on channel, not general. Sandbox *scope* is therefore a design
variable, not an implementation detail.

**Design impact.** `EXTENDED_CHECK` as specified in [§8](./research-file.md#8-actions) would
not have caught INC-02. Catching it requires source-diff review of the published tarball,
which [§6](./research-file.md#6-observed-evidence) gestures at but [§8](./research-file.md#8-actions)
does not provide.

**Confidence: Medium.**

---

## F-03 — Release age is the strongest single signal found, and the design omits it

**Original assumption.** Release age appears nowhere in
[§6](./research-file.md#6-observed-evidence) or [§9](./research-file.md#9-bounded-autonomy).
`DELAY_AND_PIN` exists as an action but with no duration and no rationale beyond "revisit".
RQ8 frames waiting as a possible security/debt trade-off — an open question.

**Evidence.**
- Tier 1: Renovate implements `minimumReleaseAge` and **explicitly recommends 14 days when
  automerging third-party dependencies**, with the rationale that registries need time to
  pull malicious packages (S-04). When set, `--before=<date>` extends the cooldown to
  transitive resolution.
- Tier 1: GitHub dependency review surfaces release date as decision-relevant (S-06).
- Tier 2: INC-01 remediated in ~4 hours; INC-02 in ~2 hours. **Any** cooldown would have
  been correct on both.

**Practitioner evidence.** **PR-05 (T-01) independently supports it** — asked about version
numbers, with no prompt about supply chain, they recommended checking "whether other people
are reporting issues with the new version". That is post-publication ecosystem signal: the
mechanism a cooldown exploits, arrived at from the regression side. One respondent, so this
is corroboration rather than confirmation, and P1 remains the priority question.

**Conclusion.** **Supported, and it challenges the project's premise.** A fixed 14-day
cooldown would have scored two-for-two on the incidents where the proposed policy scores one
hit and one confident miss — with no evidence model, no belief estimation, no cost model,
and no AI.

**Design impact.** Three changes proposed, none applied:
1. Add release age to [§6](./research-file.md#6-observed-evidence) as evidence.
2. Give `DELAY_AND_PIN` an evidence-based default duration.
3. **Add a fixed-cooldown policy as Baseline C** in [§18](./research-file.md#18-baselines-to-investigate).
   Omitting it would make the evaluation flattering, and this is the baseline most likely to
   beat the agent.

**Confidence: Medium.** The mechanism is well-evidenced; the *cost* of waiting — delayed
security patches, migration debt — is entirely unmeasured, and RQ8's trade-off remains
genuinely open.

---

## F-04 — Build/CI dependency privilege is where the harm was realised

**Original assumption.** HYP-03; [§12](./research-file.md#12-dependency-type-as-a-cost-modifier)
treats dependency type as a cost modifier; [§10](./research-file.md#10-hard-escalation-triggers)
trigger 3 escalates anything with credential or build-infra access.

**Evidence.** INC-01 compromised a build system and harvested credentials from CI
(2,349 GitHub, cloud, and AI credentials reported, S-18). INC-03 harvested from CI/CD
environments, environment variables, and cloud metadata endpoints. In
[`07-incident-research.md`](./07-incident-research.md), trigger 3 is the *only* reason INC-01
is caught.

**Practitioner evidence.** `PENDING — Tier 3`. Q6 and P4.

**Conclusion.** **Supported.** The realised harm matches HYP-03's prediction.

**Design impact.** Retain trigger 3 — with a caveat that is not optional. The trigger fires
on *dependency type*, not on evidence of compromise, so it escalates every benign build-tool
update too. **No evidence exists about that false-positive rate**, and a rule that sends
every Nx update to a human may be correct and operationally unusable at the same time. That
question (P4) must be answered before this is called a success.

**Confidence: Medium.**

---

## F-05 — Provenance attests build origin, not code safety

**Original assumption.** RQ3 asks how much confidence provenance should provide.
[§10](./research-file.md#10-hard-escalation-triggers) trigger 2 escalates when provenance is
"missing or mismatched".

**Evidence.** Tier 1 (S-01, S-02): provenance establishes *where and how* a package was
built and *who* published it. Tier 2: in INC-01 the attacker induced the project's real
`publish.yml` to publish. Provenance generated under those conditions would be
cryptographically valid and semantically true.

Two structural constraints (S-01): provenance is unavailable when publishing from private
source repositories, and requires a cloud-hosted runner on a supported provider.

**Practitioner evidence.** `PENDING — Tier 3`. Q3.

**Conclusion.** **Qualified.** Provenance answers a narrower question than the design leans
on. It cannot distinguish a legitimate build of malicious code.

**Design impact.** Trigger 2 conflates two different observations. *Missing* provenance is
common, structural, and often benign; *mismatched* provenance is genuinely anomalous. They
should be separated.

**Open and important:** whether any malicious release in INC-01–03 carried a provenance
attestation is **unknown**, and RQ3 turns on it. See
[`../evidence/incidents.md`](../evidence/incidents.md) gaps.

**Confidence: Medium** for the boundary; **Low** for anything about provenance in practice.

---

## F-06 — Clean signals are treated as evidence of safety

**Original assumption.** [§9](./research-file.md#9-bounded-autonomy) permits auto-merge when
a list of conditions is satisfied. [§13](./research-file.md#13-decision-policy-hypothesis)
estimates `P(unauthorised | evidence)`.

**Evidence.** In INC-02 **every** envelope condition was satisfied: no install script,
publisher unchanged, CI green, good API coverage, runtime dependency, no hard trigger. The
policy would have merged confidently — not marginally.

**Practitioner evidence.** `PENDING — Tier 3`. P2.

**Conclusion.** **Structural defect, not a threshold problem.** When an attacker publishes
through legitimate infrastructure, every observable signal is clean by construction. A
policy that treats absence of red flags as positive evidence is *most* confident against the
*most* competent attacker. Tuning the threshold does not help: the posterior is low and
correctly derived from uninformative evidence.

This also complicates **HYP-05**. Asymmetric loss cannot rescue the decision, because the
cost machinery only engages when the probability estimate is elevated — and here it is not.

**Design impact.** The design needs a way to distinguish *evidence of safety* from *absence
of evidence*. Release age (F-03) is one such mechanism: it does not ask whether signals are
clean, it waits for the ecosystem to generate signal that did not exist at publish time.

**Confidence: Medium.**

---

## F-07 — Advisories are unstable as ground truth

**Original assumption.** [§4](./research-file.md#4-unit-of-analysis) defines
`unauthorised = 1` via public security advisory.

**Evidence.** GHSA-cxm3-wv7p-598c / CVE-2025-10894 (INC-01) was published 27 August 2025 and
**withdrawn 28 July 2026** (S-07, retrieved directly). Separately, advisories are published
after discovery and therefore after any decision.

**Conclusion.** **Challenged**, on two counts: advisories lag by construction, and the
record is not stable over time — a dataset built on "does an advisory exist" can return
different labels depending on when it is built. This is also a leakage vector.

**Design impact.** Ground-truth methodology for `unauthorised` needs revisiting. Recorded as
an open question in [`../methodology/evaluation-plan.md`](../methodology/evaluation-plan.md).

**Confidence: Low.** The withdrawal is a single observation and **its reason has not been
established**. It must be verified before being relied on — withdrawal may reflect a
tracking-process change rather than anything about the incident.

---

## F-08 — The gap versus existing tools is narrower than claimed

**Original assumption.** [§2](./research-file.md#2-why-this-problem) frames existing
automation as detecting versions and opening PRs, leaving the decision open.

**Evidence.** Per [`06-existing-solutions.md`](./06-existing-solutions.md), existing tools
already provide: SemVer+CI merge gating (S-03), cooldown including transitive resolution
(S-04), configurable per-package automerge policy (S-05), build-origin attestation (S-01),
credential-exposure reduction (S-02), change metadata at review time (S-06), advisory
matching.

**Conclusion.** **Qualified.** Most of the *evidence* this project proposes to gather is
already gathered by something. What is genuinely absent: acquiring behavioural evidence
before deciding, weighting by explicit asymmetric cost, deciding whether more evidence is
worth buying, and varying autonomy by reversibility.

**Design impact.** Baselines must be built from what tools actually do, not from
simplifications. Baseline B should be built from Renovate's real configuration surface — and
if it is, the honest question becomes whether the agent policy differs meaningfully from a
well-configured Renovate.

**Confidence: Medium.**

---

## What these findings do not establish

- **Nothing about false-positive rates.** Every Tier 2 case is a compromise. No finding here
  says how often any signal fires on a safe update — and without that, no signal has a
  usable operating point.
- **The two hidden states have disjoint evidence.** Tier 2 covers `unauthorised` only;
  Tier 3 covers `breaks_us` only. No source yet speaks to both, so nothing is known about
  whether a signal useful for one is useful for the other — and the design assumes a single
  evidence set serves both.
- **No practitioner evidence on supply chain.** F-01 through F-08 remain untested against
  any practising engineer, which is why they stay at Medium.
- **One community, one thread, five self-selected respondents** for all Tier 3 claims.
- **Nothing about HYP-06.** No evidence has been gathered on whether explicit costs change
  decisions relative to probability alone.
- **A narrow historical window.** Three incidents from ~13 months, all high-profile. Older
  incidents with different mechanisms may point elsewhere.

---

## F-09 — SemVer is rejected as a safety signal by practitioners, including on the publishing side

**Original assumption.** [§6](./research-file.md#6-observed-evidence) lists "semantic-version
bump size" first among package metadata evidence.
[§18](./research-file.md#18-baselines-to-investigate) makes SemVer the whole of Baseline A
(patch → merge, minor → test, major → human), and S-03 shows this is what teams actually run.

**Evidence.** Tier 3, thread T-01 — four of five respondents reject SemVer as sufficient,
independently:

- PR-01: *"Semver is a polite fiction at this point honestly more of a vibe than a
  contract"* — a minor bump rewrote a date library's API; changelog said "improved
  performance"; half a day lost.
- PR-02, **a linter maintainer, from the publishing side**: *"the number gets decided by a
  human at tag time, and semver has no word for the thing that actually breaks you."*
- PR-03: assumes no update is safe until fully tested.
- PR-05: *"I don't think the version number is enough... I've had minor updates break
  things."*

**Conclusion.** **Contradicted.** PR-02 makes this stronger than an anecdote pile: SemVer
fails *even when correctly applied*. Tightening one linter rule is a patch by the letter of
the spec — no API change, nothing renamed, no export moved — yet every consumer with CI set
to fail gets a red build on code they never touched. *"nobody was being sloppy."*

PR-02 also supplies a **scope condition** the design lacks: SemVer "works fine for plain
libraries where the api is the whole surface" and is weak "for anything with rules or
defaults in it". Predictive power depends on *what kind of package it is* — a property of
the package, not the release. [§12](./research-file.md#12-dependency-type-as-a-cost-modifier)
classifies only by runtime/dev/build and cannot express this.

**Design impact.** Three proposals, none applied:
1. Downgrade SemVer bump size from primary evidence to weak prior.
2. Add package class (rule-bearing vs plain-API) as evidence.
3. Keep Baseline A — but report it as *what teams actually run and what practitioners say
   does not work*, which is a more interesting baseline than a strawman.

**Confidence: Medium-High.** Five respondents, self-selected, single community, single
thread — but the convergence is causal rather than directional, and includes the publisher
side, which is the perspective most likely to defend SemVer and did not.

---

## F-10 — The evidence model cannot represent the regression mechanism practitioners described

**Original assumption.** [§6](./research-file.md#6-observed-evidence) enumerates the evidence
available before a decision. [§4](./research-file.md#4-unit-of-analysis) defines
`breaks_us = 1` as a defect attributable to the bump observed within 30 days.

**Evidence.** PR-02 describes the failure mode precisely: *"the thing i actually check is
whether a release changes what the tool says about unchanged input."* A linter, formatter,
compiler, or any rule-bearing package can alter its output on code the consumer never
modified. No API changed. No export moved. Consumer CI goes red.

PR-01's date-library case and PR-04's note on TypeScript type-level breakage are variants:
breakage without an API-surface change that a runtime test would catch.

**Conclusion.** **The evidence model has a gap.** Every signal in
[§6](./research-file.md#6-observed-evidence) is about the *package* — its metadata, its
provenance, its graph — or about our CI passing. None captures *the delta in what the package
says about our unchanged code*. "Coverage of the API surface actually imported" comes
closest and still misses it: the API surface is unchanged in PR-02's scenario. That is the
whole point.

**Design impact.** Two proposals:
1. Add a differential-behaviour signal — run the old and new versions against the same
   unchanged inputs and diff the outputs. This is a form of behavioural evidence, so it
   belongs in `EXTENDED_CHECK` ([§8](./research-file.md#8-actions)) and expands HYP-02 beyond
   the security framing it currently has.
2. Note that this also strengthens F-02's conclusion from the opposite direction:
   `EXTENDED_CHECK`'s scope is under-specified for *both* hidden states.

**Confidence: Medium.** One respondent, but a maintainer describing their own package's
behaviour, and PR-01/PR-04 are consistent variants.

---

## The finding that matters most

**F-03 combined with F-06.** The design's security anchors rest on signals that are clean
during a competent compromise, while the one mechanism that would have worked on two of
three incidents — waiting — is absent from the specification and already shipped by a tool
most teams could turn on today.

If that survives practitioner scrutiny and a wider incident set, the honest conclusion may
be the one [`../README.md`](../README.md) already permits: a simple deterministic policy is
sufficient, and the agent is not justified. That outcome would be a result, not a failure.

**And a convergence worth noting.** F-03 (waiting, from supply-chain evidence) and F-09/F-10
(SemVer's failure and the behaviour-delta gap, from regression evidence) were reached from
opposite directions and point the same way: **the signals available at publish time are
weaker than the design assumes, for both hidden states.** What helps in both cases is
evidence that does not exist yet at decision time — the ecosystem's reaction, or the
package's observed behaviour against our own unchanged code. That is an argument for
`DELAY_AND_PIN` and `EXTENDED_CHECK` carrying more weight than
[§9](./research-file.md#9-bounded-autonomy)'s envelope currently gives them, and for
`INSTALL_AND_MERGE` being rarer than the design anticipates.
