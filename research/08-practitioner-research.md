# Practitioner Research

**Tier 3 protocol.** Responses are recorded in
[`../evidence/practitioner-responses.md`](../evidence/practitioner-responses.md); posting
logs live in [`../discussions/`](../discussions/). This document is the method.

## Purpose

Not validation. The purpose is to find out where
[`research-file.md`](./research-file.md) is **wrong**, using engineers who have run
dependency updates in production and dealt with the consequences.

Documentation says what tools do. Incidents say what went wrong. Neither says what
practitioners actually trust, what they actually do when a PR looks fine, or what an
extended check actually costs a team. Those are Tier 3 questions and nothing else answers
them.

## Success criterion

> **Assumptions changed, not comments received.**

A thread with 200 upvotes and no design impact is a failure. A single reply that invalidates
a hard trigger is a success.

**Stop condition** (from the original discussion protocol): if six or more substantive
contributions produce no design change, the questions are wrong — not the respondents.
Reconsider what is being asked before collecting more.

## Current status

| Platform | Posted | Substantive responses |
|---|---|---|
| Reddit — r/javascript | 1 thread (score 4) | **5** |
| X | attempted | 0 |
| LinkedIn | not yet | — |

Thread T-01 produced five responses and moved three rows of
[`../evidence/evidence-matrix.md`](../evidence/evidence-matrix.md), yielding findings F-09
and F-10. Transcripts: [`../evidence/practitioner-responses.md`](../evidence/practitioner-responses.md).

**But the coverage is lopsided.** T-01 asked about version numbers, so every response
concerns `breaks_us`. RQ3–RQ7 and RQ9–RQ10 — provenance, escalation, build dependencies,
transitive closure, asymmetric loss — still have **zero** practitioner evidence. The next
posts must target supply-chain questions specifically.

### What T-01 demonstrated about method

The thread worked because it asked a narrow, concrete question that practitioners had direct
experience of, and did not mention the project. The most valuable response (PR-02) came from
a **maintainer** giving the publishing-side view — a perspective the question did not ask for
and the research had not sought. Worth seeking deliberately: package maintainers see the
decision from the other end, and PR-02 undermined a project assumption more effectively than
any consumer response did.

## Question routing

One research question per contribution. Never the same text in two communities.

| Question ([§23](./research-file.md#23-questions-for-practitioners)) | RQ | Best audience |
|---|---|---|
| Q1 — what makes you stop and investigate | RQ2 | Node/JS engineering |
| Q2 — would you auto-merge a patch to production | RQ1 | DevOps, SRE |
| Q3 — how much do you trust provenance | RQ3 | AppSec, supply-chain security |
| Q4 — is a changed publisher always human-review | RQ4 | AppSec |
| Q5 — how do you evaluate new transitive deps | RQ7 | Node/JS, package management |
| Q6 — do you treat runtime/dev/build differently | RQ6 | DevSecOps, CI/CD |
| Q7 — would you sandbox-install a suspicious update | RQ4 | Security research |
| Q8 — most expensive dependency failure you've had | RQ9 (cost model) | SRE, production engineering |
| Q9 — what changes your mind from merge to investigate | RQ1 | Node/JS engineering |
| Q10 — where should an agent stop and require a human | RQ5 | AppSec, SRE |

## Questions the research has since made more urgent

These arose from Tier 1–2 work and are now higher priority than parts of the original list.
See [`09-findings.md`](./09-findings.md).

**P1 — the cooldown question (highest priority).**
> Renovate recommends `minimumReleaseAge: 14 days` before automerging. If you wait two
> weeks, how much does anything else you check actually add?

This directly tests whether the project has a reason to exist. It should be asked plainly,
and a "nothing, honestly" answer must be recorded as prominently as any other.

**P2 — the clean-signals question.**
> In the September 2025 `chalk`/`debug` compromise, the real maintainer's real account
> published, and there was no install script. Every automated signal was clean. What would
> you have checked that would have caught it?

**P3 — the cost question.**
> What does an extended check — sandboxed install, install-script trace, integration tests
> against the imported API — actually cost your team in wall-clock and attention?

[§11](./research-file.md#11-cost-model) has no evidence behind its numbers at all.

**P4 — the false-positive question.**
> Would you accept a policy that escalates *every* build-tool update to a human?

[`07-incident-research.md`](./07-incident-research.md) shows hard trigger 3 catching INC-01,
but says nothing about how often it would fire on benign updates.

## Conduct rules

1. **Ask, do not pitch.** The project is context, not the subject. Do not lead with the agent.
2. **State it is unbuilt.** Every post says no agent exists yet.
3. **Do not defend the design in-thread.** A reply that challenges an assumption is the
   deliverable. Record it; argue with it in the repository, not at the respondent.
4. **Follow community rules.** Verify each subreddit's self-promotion policy before posting.
5. **Never fabricate.** See [`../discussions/README.md`](../discussions/README.md).

## Known bias in what will be collected

Recorded now, before the data arrives, so it cannot be rationalised later:

- **Self-selection.** People who reply to dependency-security threads care unusually much
  about dependency security. Their risk tolerance is not representative.
- **Recency.** The 2025 npm incidents were widely publicised. Answers will be anchored to
  them, and so will over-weight supply-chain compromise relative to ordinary regression —
  which is the more common failure.
- **Stated versus revealed preference.** What engineers say they require before merging and
  what their repositories actually auto-merge are different measurements. Tier 3 gives the
  first only.
- **Platform skew.** LinkedIn responses will skew toward people with an audience incentive;
  Reddit toward anonymity and candour.

None of these invalidate Tier 3 evidence. They bound what it can be used to claim.
