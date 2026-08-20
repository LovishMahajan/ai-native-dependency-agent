# Incident Research

**Tier 2 counterfactual analysis.** Facts and sources live in
[`../evidence/incidents.md`](../evidence/incidents.md); this document asks one question of
each of them:

> **Standing at `decision_timestamp`, with no knowledge of what happened afterwards, what
> would the four-action policy in [§8](./research-file.md#8-actions)–[§10](./research-file.md#10-hard-escalation-triggers)
> have chosen?**

## Method and its limits

Each incident is run through the [§9](./research-file.md#9-bounded-autonomy) auto-merge
envelope and the [§10](./research-file.md#10-hard-escalation-triggers) hard triggers as
currently written. Verdicts are `CAUGHT`, `PARTIAL`, or `MISSED`.

Three honesty constraints, per
[`../methodology/leakage-controls.md`](../methodology/leakage-controls.md):

1. **No hindsight inputs.** That a package is now known-malicious is the answer, not a
   signal. Only what a decision-time observer could have seen counts.
2. **Reconstructed, not observed.** All three incidents were remediated, so the registry no
   longer shows what an observer would have seen. Evidence snapshots are reconstructed from
   advisories and reporting and are flagged as such.
3. **Selection bias is acknowledged, not corrected.** All three cases are compromises. A
   policy is easy to make look good, or bad, on a set containing no correct merges. These
   verdicts say something about **recall on compromise and nothing about false-positive
   rate**. See "What this analysis cannot tell us" below.

**All verdicts below are `VERIFIED BY HUMAN: no` and rest partly on sources that could not
be fetched directly.**

---

## INC-01 — Nx `s1ngularity` → **CAUGHT**

### Reconstructed decision-time evidence

| Signal | Value at decision time | Observable? |
|---|---|---|
| SemVer bump | Patch/minor within two major lines, published concurrently | Yes |
| Publisher identity | **Unchanged** — legitimate workflow, legitimate token | Yes |
| Provenance | Unknown — see gaps | Yes, if present |
| Install script | `postinstall` **present** | Yes |
| Dependency type | **build** — Nx is the build system | Yes |
| Credential proximity | High — runs in CI with repository and cloud credentials | Yes |
| CI result | Would pass; payload does not break builds | Yes |
| Release cadence | **Anomalous** — eight `nx` versions across two major series in one window | Yes |

### Policy trace

```text
§10 hard triggers
  1. publisher identity changed ............... NO   (does not fire)
  2. provenance missing/mismatched ............ UNKNOWN
  3. credential / build-infra access .......... YES  ← FIRES
  4. materially contradictory evidence ........ arguably YES (cadence anomaly)

→ REJECT_AND_ESCALATE
```

**Verdict: CAUGHT** — by hard trigger 3, before any probabilistic reasoning.

### What actually did the work

Not the compromise detection. The policy caught this because **Nx is a build tool**, and
[§10](./research-file.md#10-hard-escalation-triggers) escalates anything with build-infra
access unconditionally. The same policy would escalate every benign Nx update too.

That is a meaningful result for HYP-03 — the realised harm was exactly the harm HYP-03
predicts, credential theft from a privileged build context — but it is a result about a
**blanket rule**, not about evidence-based decision-making. The evidence signals in
[§6](./research-file.md#6-observed-evidence) contributed nothing; trigger 3 fired on
dependency type alone.

### Assumption impact

- **HYP-03 supported.** Build/CI dependencies deserve stricter boundaries. INC-01 and INC-03
  both realised credential theft from privileged contexts.
- **§10 trigger 1 shown weak.** The publisher never changed. Initial access was command
  injection in a PR-validation workflow using `pull_request_target`.
- **§10 trigger 2 shown insufficient.** Whether provenance was present is unknown, but the
  attacker used the project's *real* `publish.yml`. Valid provenance was available to them
  by construction — see [`06-existing-solutions.md`](./06-existing-solutions.md).
- **Cadence anomaly promising.** Eight versions across two major lines in one window is
  visibly abnormal. Unusable until a base rate for *benign* cadence anomalies exists.

---

## INC-02 — `chalk` / `debug` → **MISSED**

This is the most important case in the file.

### Reconstructed decision-time evidence

| Signal | Value at decision time | Observable? |
|---|---|---|
| SemVer bump | Patch/minor | Yes |
| Publisher identity | **Unchanged** — the real maintainer's real account | Yes |
| Provenance | Unknown — see gaps | Yes, if present |
| Install script | **Absent.** Payload was browser-side, not install-time | Yes |
| Dependency type | **runtime** — not build-privileged | Yes |
| Credential proximity | Low, by the policy's own model | Yes |
| CI result | Would pass — payload targets end-user wallets, not the build | Yes |
| API-surface coverage | High — `chalk`/`debug` have small, heavily exercised APIs | Yes |
| Release age | **~0 hours** | Yes |

### Policy trace

```text
§10 hard triggers
  1. publisher identity changed ............... NO
  2. provenance missing/mismatched ............ UNKNOWN (assume present → no)
  3. credential / build-infra access .......... NO   (runtime dependency)
  4. materially contradictory evidence ........ NO   (all signals agree — wrongly)

§9 auto-merge envelope
  no install-script change .................... ✅ SATISFIED
  provenance verified ......................... ✅ (assumed)
  publisher unchanged ......................... ✅ SATISFIED
  CI passes ................................... ✅ SATISFIED
  CI covers imported API surface .............. ✅ SATISFIED
  P(unauthorised) below threshold ............. ✅ every signal is clean
  not build/CI privileged ..................... ✅ SATISFIED
  no hard trigger ............................. ✅ SATISFIED

→ INSTALL_AND_MERGE
```

**Verdict: MISSED.** The policy would have auto-merged a compromised release into
production, and would have done so *confidently* — the envelope is satisfied on every
condition, not marginally.

### Why it failed

The design's two security anchors both point at the wrong thing:

- **`publisher unchanged`** anchors on *identity*, but the attack compromised *credentials*.
  A phished maintainer's real account publishing malicious code produces an unchanged
  publisher identity. The signal is not merely weak here; it actively contributes to a
  confident wrong answer, because a clean signal *lowers* the posterior.
- **`no install-script change`** anchors on *install-time execution*, but the payload
  executed in end users' browsers. Install-script analysis — and, importantly, the sandboxed
  install of `EXTENDED_CHECK` — inspects a channel the attack never used.

The deeper problem: the envelope treats clean signals as positive evidence of safety. When
an attacker publishes through legitimate infrastructure, *every* signal is clean. The
policy's confidence is highest exactly when the attacker is most competent.

### Would `EXTENDED_CHECK` have helped?

Probably not, as specified. [§8](./research-file.md#8-actions) lists sandboxed install,
install-script trace, integration tests against the imported API surface, and dependency
graph inspection. The payload ran only in a browser, on end-user wallet interactions. It
would not fire during install, during a Node integration test, or in a graph diff. Catching
it needs source-diff review of the published tarball — which
[§6](./research-file.md#6-observed-evidence) gestures at ("package-file diff size", "newly
introduced network calls") but [§8](./research-file.md#8-actions) does not include as an
extended-check capability.

This qualifies **HYP-02**: behavioural evidence beats static metadata only where the
behaviour is observable in the sandbox. Sandbox scope is itself a design variable.

### What would have caught it

**Release age.** Malicious versions were live for roughly two hours before maintainers
reverted and unpublished. *Any* cooldown — Renovate's recommended 14 days, or 24 hours, or
6 hours — would have been correct, using no evidence model at all.

### Assumption impact

- **HYP-01 challenged, hard.** Publisher continuity is not merely "useful but insufficient";
  in this case it is misleading.
- **§9 envelope challenged.** Two conditions are unsound as safety evidence.
- **§10 triggers challenged.** None fire. The hard-constraint layer offers no protection here.
- **RQ8 elevated.** Waiting is no longer a maintenance-debt trade-off question. It is the
  only mechanism in the study that would have prevented this.
- **RQ9 sharpened.** The asymmetric-loss machinery never engages, because the posterior is
  low and *correctly derived* from clean evidence. Cost asymmetry cannot rescue a decision
  when the evidence itself is uninformative.

---

## INC-03 — Shai-Hulud → **PARTIAL**

### Reconstructed decision-time evidence

| Signal | Value at decision time | Observable? |
|---|---|---|
| Publisher identity | **Unchanged** per package — each stolen token was legitimate | Yes |
| Install script | `postinstall` **present** | Yes |
| Install script *change* | Yes for packages with no prior install script (e.g. `keyv`) | Yes |
| Dependency type | **Mixed** — spread followed token reach, not dependency role | Yes |
| Transitive exposure | High — worm reaches arbitrary closure depth | Partially |
| Release age | Hours to days, varying per package | Yes |

### Policy trace

```text
§10 hard triggers
  1. publisher identity changed ............... NO
  2. provenance missing/mismatched ............ UNKNOWN
  3. credential / build-infra access .......... DEPENDS on the package
  4. materially contradictory evidence ........ NO

§9 auto-merge envelope
  no install-script change .................... ❌ FAILS where postinstall is newly added
                                                ⚠ AMBIGUOUS where one already existed

→ EXTENDED_CHECK (not auto-merge) for most packages
```

**Verdict: PARTIAL.** For a package like `keyv` gaining a `postinstall` it never had, the
envelope fails and the case routes to `EXTENDED_CHECK` — and here a sandboxed install
*would* observe the payload, since it executes at install time and reaches for credentials
and network. This is the case HYP-02 was written for.

But the routing depends on a distinction the specification does not make.

### The `change` versus `presence` ambiguity

[§9](./research-file.md#9-bounded-autonomy) says **"no install-script change"**.
[§6](./research-file.md#6-observed-evidence) says **"install-script changes"**. Neither
defines the comparison. For a package that already ships a `postinstall`, does a modified
script count as a "change" — and is it detected by presence, by hash, or by semantic diff?

This is not pedantry: it decides the verdict. Under *presence*, every affected package
routes to `EXTENDED_CHECK` and INC-03 is `CAUGHT`. Under *change*, packages with a
pre-existing install script may pass. The specification is silent, so the honest verdict is
`PARTIAL`.

**This must be resolved before any experiment runs**, or the result will depend on an
undocumented implementation choice.

### Assumption impact

- **HYP-02 supported here.** Install-time behaviour is exactly what a sandbox observes.
  Note this cuts the opposite way from INC-02 — HYP-02's validity is conditional on the
  attack channel, not general.
- **HYP-04 supported.** Self-propagation along token reach places compromised packages at
  arbitrary depth. Direct-only evaluation is insufficient, and Renovate's `--before`
  cooldown already treats the closure as in scope.
- **§9 wording defect identified.** See above.
- **Base-rate assumption challenged.** An actively propagating worm still live in 2026
  (S-09) means prevalence is **not stationary**, undermining "approximately natural
  prevalence" sampling in [§16](./research-file.md#16-experimental-plan).

---

## Cross-incident synthesis

| | INC-01 Nx | INC-02 chalk/debug | INC-03 Shai-Hulud |
|---|---|---|---|
| **Verdict** | CAUGHT | **MISSED** | PARTIAL |
| What decided it | Hard trigger 3 (build dep) | Nothing fired | Install-script condition |
| Publisher changed | No | No | No |
| Install script | Present | **Absent** | Present |
| Would 14-day cooldown catch it? | **Yes** (~4 h live) | **Yes** (~2 h live) | Partially |
| Would Baseline A catch it? | No | No | No |

### Three findings

**1. Publisher-identity change did not occur in any recorded incident.** Every case used
legitimate publishing credentials obtained through workflow injection, phishing, or worm
harvesting. As a hard trigger it detects account handover — a class not represented here at
all — while missing the class that is. It should be reframed as evidence of *credential
compromise*, which is the thing that actually matters and is not directly observable.

**2. A fixed cooldown outperforms the proposed policy on these cases.** It would have been
correct on INC-01 and INC-02, where the specified policy scores one hit and one confident
miss — using no evidence model, no beliefs, no cost model, and no AI. This is a direct
challenge to the project's premise and belongs in the evaluation as **Baseline C**. Omitting
it would make the comparison flattering.

**3. Clean signals are not evidence of safety.** In INC-02 every envelope condition was
satisfied. A policy that treats absence-of-red-flags as positive evidence will be most
confident against the most competent attacker. This is a structural problem with
[§9](./research-file.md#9-bounded-autonomy), not a threshold to tune.

## What this analysis cannot tell us

Stated plainly, because these verdicts are easy to over-read:

- **Nothing about false positives.** All three cases are compromises. Hard trigger 3 catching
  INC-01 says nothing about how many benign build-tool updates it would also escalate — and
  a rule that escalates every Nx update may be operationally unusable.
- **Nothing about regression** (`breaks_us`). All Tier 2 evidence so far concerns
  `unauthorised`. Half the hidden state is unresearched.
- **Nothing about the historical distribution.** Three incidents, all from a 13-month window,
  all high-profile. `event-stream` (2018), `ua-parser-js` (2021) and `node-ipc` (2022) have
  different mechanisms and may produce different verdicts.
- **Nothing settled about provenance.** Whether any malicious release carried a provenance
  attestation is unknown, and RQ3 turns on it.

## Next steps

1. Establish whether provenance was present on any malicious release in INC-01–03.
2. Add INC-04+ for the older canonical incidents.
3. Add benign cases — updates where `INSTALL_AND_MERGE` was correct — or the counterfactual
   remains one-sided.
4. Resolve the install-script `change`/`presence` ambiguity in
   [§9](./research-file.md#9-bounded-autonomy).
5. Take finding 2 to practitioners: *does a cooldown make the rest of this unnecessary?*
