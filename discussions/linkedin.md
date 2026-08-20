# LinkedIn

## Status: not yet posted

Per [`../research/research-file.md` §20](../research/research-file.md#20-public-research-plan),
the introduction post goes out only after the problem statement and research files are
public. **That condition is now met** — this repository is the artifact to link.

## What the post must do

1. State the problem
2. Say why existing dependency automation is insufficient
3. Name the uncertainty being studied
4. Link the repository
5. Ask one or two real questions
6. **Not claim the agent works, exists, or is being built yet**

## What the post must not do

- No "excited to announce". Nothing has been achieved yet.
- No implying a working system. There is no code in the repository, deliberately.
- No overstating the Reddit thread. Five responses in one community is a starting point,
  not a study.
- No claiming novelty that [`../research/06-existing-solutions.md`](../research/06-existing-solutions.md)
  contradicts. Renovate already does more of this than the framing suggests.

---

## Draft — recommended

Leads with the finding rather than the project, because the finding is the part a stranger
has reason to read.

```text
In September 2025, someone phished a maintainer and published malicious versions of
chalk and debug — packages with over 2 billion downloads a week.

I've been designing a decision policy for dependency updates, so I ran that incident
through it as a test. The result was not what I hoped.

Every safety condition in my policy passed:

  publisher unchanged        ✅  the real maintainer's real account published it
  no install script          ✅  the payload ran in end users' browsers instead
  CI green                   ✅  it doesn't break anything, it steals crypto
  not a build dependency     ✅  chalk is about as ordinary as a dependency gets
  no hard escalation trigger ✅

My policy would have merged it. Confidently — not marginally.

That's the problem with dependency automation nobody has solved. Dependabot and
Renovate can tell you a new version exists. They can't tell you whether this
particular release should be trusted enough to merge into your service. And when
an attacker publishes through legitimate infrastructure, every signal you'd check
comes back clean. Your confidence is highest exactly when the attacker is best.

So I'm researching the question properly before building anything:

Can dependency-update decisions be made safely from observable evidence — or is the
problem simply not observable enough at decision time?

Three things the research has already turned up that I didn't expect:

1. In all three major npm compromises of 2025 — Nx, chalk/debug, Shai-Hulud — the
   publisher identity never changed. Every one used stolen or phished credentials.
   "Publisher unchanged" is a signal I was treating as reassuring.

2. Renovate's boring `minimumReleaseAge: 14 days` would have caught two of those
   three on elapsed time alone. No model, no scoring, no AI. Both were pulled within
   hours. A tool most teams could enable today beats the policy I designed.

3. I asked r/javascript how much they trust version numbers. Four of five said not at
   all — including a linter maintainer who explained why from the publishing side:
   tighten one rule so it catches a case it used to miss, and that's a patch by the
   letter of semver, while every consumer with CI set to fail gets a red build on code
   they never touched. "Nobody was being sloppy."

There is no agent. There is no code. That's deliberate — the repository is a research
phase, and it's public so the reasoning can be checked and argued with:

github.com/LovishMahajan/ai-native-dependency-agent

Two questions I'd genuinely like answered by people who've run this in production:

→ If you wait 14 days before merging a dependency update, how much does anything else
  you check actually add?

→ In the chalk/debug case, every automated signal was clean. What would you have
  checked that would have caught it?

I'm equally prepared for the answer to be that a simple deterministic policy beats
anything cleverer. That would be a result too.
```

**Length:** ~430 words. Long for LinkedIn but earns it — the table and the three findings
are the substance. Cut the numbered findings to two if it needs shortening; keep 1 and 2.

**Formatting notes:**
- LinkedIn strips markdown. The checkmark block posts fine as plain text with emoji.
- Put the repo link in the **first comment** instead of the body if reach matters —
  LinkedIn suppresses posts with external links. Then the body ends at "...can be checked
  and argued with:" and the comment carries the URL.
- No hashtag block. If any: `#softwaresupplychain #npm #devsecops`.

---

## Draft — short alternative

If the long version feels like too much for a first post:

```text
I designed a policy for deciding when a dependency update is safe to auto-merge.

Then I tested it against the September 2025 chalk/debug compromise — 2 billion
weekly downloads, malicious code published by a phished maintainer.

My policy would have merged it. Publisher unchanged (the real account published it).
No install script (the payload ran in browsers). CI green (it steals crypto, it
doesn't break builds). Not a build dependency. Nothing to escalate.

Every check passed. That's the part worth sitting with: when an attacker publishes
through legitimate infrastructure, every signal comes back clean.

Meanwhile Renovate's unglamorous `minimumReleaseAge: 14 days` would have caught it,
because the malicious versions were live for about two hours.

I'm researching this before building anything. No agent, no code — just the problem,
the evidence, and the reasoning, in public:

github.com/LovishMahajan/ai-native-dependency-agent

If you wait two weeks before merging, how much does anything else you check add?
```

**Length:** ~180 words.

---

## Posting log

| Date | Version | Link | Reactions | Substantive responses |
|---|---|---|---|---|
| *(to be filled after posting)* | — | — | — | — |

Any substantive reply gets transcribed into
[`../evidence/practitioner-responses.md`](../evidence/practitioner-responses.md) under the
same rules as every other platform: verbatim, linked, and never reconstructed.

**Expect a low substantive-response rate.** LinkedIn rewards agreement, and agreement is
not evidence. A reply saying "great post" is not a Tier 3 observation. The two questions at
the end exist to give the small number of people with real experience something specific to
answer.
