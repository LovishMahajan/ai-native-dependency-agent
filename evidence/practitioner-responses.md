# Practitioner Responses

Real, linked replies only. **Nothing may be added to this file except an actual response
from an actual person** — no paraphrase from memory, no reconstructed conversation, no
illustrative example, no AI-generated response, not even as a placeholder. See
[`../research/research-file.md` §25](../research/research-file.md#25-ai-use-policy) and
[`../discussions/README.md`](../discussions/README.md).

Responses are quoted verbatim, including original spelling and punctuation. Where a
respondent's wording carries the finding, it is quoted rather than summarised.

## Platform status

| Platform | Posted | Substantive responses |
|---|---|---|
| Reddit — r/javascript | 1 thread (4 upvotes) | **5** |
| X | attempted | 0 |
| LinkedIn | not yet posted | — |

---

## Thread T-01 — r/javascript

- **Title:** `[AskJS] How much do you actually trust the version ...` *(full title to be confirmed)*
- **URL:** https://www.reddit.com/r/javascript/comments/1vosn6a/askjs_how_much_do_you_actually_trust_the_version/
- **Thread score:** 4
- **Research question:** RQ2 (risk signals), with bearing on RQ1 (merge criteria)
- **Hidden state addressed:** `breaks_us` — **not** `unauthorised`
- **Transcribed from:** screenshot supplied by the author, 2026-08-20
- **Verified by human:** yes (author-supplied) — ⚠ permalinks to individual comments still to be added

> **Note on what this thread does and does not cover.** The question asked about version
> numbers and trust. Every response therefore concerns **regression**, not supply-chain
> compromise. This is the first Tier 3 evidence the project has on `breaks_us`, which was
> previously unresearched entirely. It says nothing about `unauthorised`.

---

### PR-01 — `Spiritual_Bee6614` (score 6, 5d)

> Semver is a polite fiction at this point honestly more of a vibe than a contract. Had a
> minor bump on a date formatting lib completely rewrite the API because the maintainer felt
> like it, changelog just said "improved performance". Took half a day to unpick.
>
> If the changelog is one line or just links to a commit list I treat it the same as a
> major, doesn't matter what the number says.

**Stated context:** consumer of third-party packages; describes a specific incident from
their own experience.

**Assumptions touched:** SemVer as a risk signal; changelog quality as a signal.

**Effect: CHALLENGES** the SemVer assumption. **SUPPORTS** changelog quality — and gives it
a concrete decision rule: a one-line changelog escalates the update to major-equivalent
handling regardless of the version number.

*(Follow-up from `abrahamguo` (score 1): "What library was it?" — unanswered at time of
transcription.)*

---

### PR-02 — `Ok_Woodpecker_9104` (score 2, 5d)

> from the publishing side the number gets decided by a human at tag time, and semver has no
> word for the thing that actually breaks you.
>
> i maintain a linter. if i tighten one existing rule so it catches a case it used to miss,
> thats a patch by the letter of semver. no api change, nothing renamed, no export moved.
> but every consumer running that rule with ci set to fail now has a red build on code they
> never touched. same outcome as your date lib story except nobody was being sloppy.
>
> so the thing i actually check is whether a release changes what the tool says about
> unchanged input. semver has no way to express that, which is why the number is weak for
> anything with rules or defaults in it. it works fine for plain libraries where the api is
> the whole surface.
>
> the filter that has held up for me on other peoples packages: does the changelog name
> behaviour or does it name code. "improved performance" and "refactor internals" are
> unreviewable, you have to read the diff. "now also flags X" or "default for Y changed from
> a to b" i can judge in ten seconds. a one line changelog is usually the maintainer telling
> you they didnt think about who it lands on.

**Stated context:** **package maintainer** — maintains a linter. The only publisher-side
perspective collected so far.

**Assumptions touched:** SemVer as a risk signal; changelog quality; the definition of
`breaks_us`; the evidence list in [§6](../research/research-file.md#6-observed-evidence).

**Effect: CHALLENGES** SemVer, with a mechanism rather than an anecdote. **SUPPORTS**
changelog quality with an operational test. **INTRODUCES A SIGNAL NOT IN THE DESIGN** — see
F-10 in [`../research/09-findings.md`](../research/09-findings.md).

This is the highest-value response collected. Three distinct contributions:

1. **Why SemVer fails, structurally.** The version number is chosen by a human at tag time
   and "has no word for the thing that actually breaks you". Correctly-applied SemVer still
   misses the failure — "nobody was being sloppy".
2. **A bounded scope condition.** SemVer "works fine for plain libraries where the api is
   the whole surface" and is weak "for anything with rules or defaults in it". This is a
   *predicate on the package*, not on the release — something the design has no notion of.
3. **An operational changelog test.** *Does the changelog name behaviour, or name code?*
   Behaviour-naming entries ("now also flags X", "default for Y changed from a to b") are
   reviewable in seconds; code-naming entries ("improved performance", "refactor internals")
   are unreviewable without reading the diff.

---

### PR-03 — `create-third-places` (score 1, 5d)

> I assume any version update is not safe to merge until a full round of testing has been
> done with it.

**Stated context:** none given.

**Assumptions touched:** the auto-merge envelope; CI as evidence.

**Effect: CHALLENGES** the premise of auto-merge. Stated as a default posture — *no* update
is safe until tested. Sets a stricter bar than [§9](../research/research-file.md#9-bounded-autonomy).

---

### PR-04 — `NeatBeluga` (score 1, 5d)

> TypeScript needs extra attention and can make this pretty interesting if you dont pay
> attention early on

*(Posted as link text; the linked destination has not been retrieved.)*

**Stated context:** none given.

**Effect: QUALIFIES.** Suggests type-level breakage is a distinct concern from runtime
breakage — relevant because [§3](../research/research-file.md#3-scope) scopes the project to
TypeScript services. A type-only break passes runtime tests and fails the build.

⚠ Needs follow-up: the linked resource should be retrieved before this is weighted.

---

### PR-05 — `brian_sword` (score 1, 4d)

> I will be honest that I don't think the version number is enough to decide whether an
> update is safe. I've had minor updates break things
>
> For automated updates, semver would be a good choice as a starting point, but I still
> check the changelog and let the tests have the final say. It would be better if you check
> how the package is maintained and whether other people are reporting issues with the new
> version.

**Stated context:** consumer; reports minor updates breaking things.

**Assumptions touched:** SemVer; changelog; CI; **release age / ecosystem signal**.

**Effect: CHALLENGES** SemVer as sufficient. **SUPPORTS** changelog and tests as evidence.
**INDEPENDENTLY SUPPORTS F-03** — "whether other people are reporting issues with the new
version" is ecosystem signal that only exists *after* publication, which is the mechanism
behind release-age cooldowns. Arrived at from the regression side, with no prompting about
supply chain.

---

## Convergence across responses

| Claim | PR-01 | PR-02 | PR-03 | PR-04 | PR-05 | Count |
|---|---|---|---|---|---|---|
| SemVer is insufficient to decide safety | ✅ | ✅ | ✅ | — | ✅ | **4/5** |
| Changelog quality is a real decision signal | ✅ | ✅ | — | — | ✅ | **3/5** |
| Tests must have the final say | — | — | ✅ | — | ✅ | 2/5 |
| Post-publication ecosystem signal matters | — | — | — | — | ✅ | 1/5 |
| Type-level breakage is distinct | — | — | — | ✅ | — | 1/5 |

Per rule 5 below, this table records **reasoning that recurred**, not a vote. Four
independent respondents converging on SemVer's weakness — including one maintainer
explaining the publishing-side mechanism — is stronger than the count suggests, because the
agreement is causal rather than merely directional.

## Coverage against research questions

| RQ | Topic | Responses |
|---|---|---|
| RQ1 | Merge criteria | 2 (PR-03, PR-05) |
| RQ2 | Risk signals | **5** |
| RQ3 | Provenance | 0 |
| RQ4 | Behavioural evidence | 0 |
| RQ5 | Human escalation | 0 |
| RQ6 | Build dependencies | 0 |
| RQ7 | Transitive closure | 0 |
| RQ8 | Waiting | 1 (PR-05, indirect) |
| RQ9 | Asymmetric loss | 0 |
| RQ10 | Historical memory | 0 |

**All supply-chain questions remain at zero.** The collected evidence is entirely about
regression.

## Record schema

```text
ID:              PR-NN
Date:            YYYY-MM-DD
Platform:        reddit | x | linkedin
Link:            <permalink to the actual comment>
Respondent:      <handle>
Stated context:  <what they said about their own experience — never inferred>
Question asked:  RQ-NN / Q-NN
Response:        <verbatim quote>
Assumption(s) touched: <IDs from ../decisions/assumption-log.md>
Effect:          SUPPORTS | CHALLENGES | QUALIFIES | INTRODUCES | NO EFFECT
Design change:   <link to ../decisions/design-changelog.md entry, or "none">
```

## Rules

1. **Link or it does not exist.** Entries without a working permalink are provisional.
2. **Do not infer expertise.** Record only stated background. PR-02 is weighted as a
   maintainer perspective because they said so, not because the answer sounded expert.
3. **Record disagreement in full.** Responses contradicting the hypotheses get more space,
   not less.
4. **Record non-responses.** A question that draws no engagement is a finding about reach.
5. **Do not aggregate into a verdict.** "4 of 5 said X" is an observation about a
   self-selected sample, not a measurement. Report reasoning, not tallies.

## Note on the X non-response

One attempt, no engagement. With no follower base, the likely explanation is distribution,
not topic. This is **not** evidence that the question is uninteresting and must not be
recorded as such.
