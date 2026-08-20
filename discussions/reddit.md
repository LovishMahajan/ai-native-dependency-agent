# Reddit

## Posting log

### T-01 — r/javascript — ✅ posted, 5 responses

| Field | Value |
|---|---|
| **Community** | r/javascript |
| **Format** | `[AskJS]` |
| **Title** | `[AskJS] How much do you actually trust the version ...` *(full title to be confirmed)* |
| **URL** | https://www.reddit.com/r/javascript/comments/1vosn6a/askjs_how_much_do_you_actually_trust_the_version/ |
| **Score** | 4 |
| **Research question** | RQ2 — risk signals |
| **Responses** | 5 substantive (PR-01 … PR-05) |
| **Outcome** | Moved 3 evidence-matrix rows; produced findings F-09, F-10 |

**What it produced.** Four of five respondents rejected SemVer as sufficient to judge
safety. One respondent — a linter maintainer — explained the publishing-side mechanism and
introduced a failure mode the evidence model cannot represent. Changelog quality was
promoted from an author assumption to an evidenced signal with an operational test.

See [`../research/09-findings.md`](../research/09-findings.md) F-09 and F-10.

**What it did not produce.** Any evidence on supply-chain compromise. The question was about
version numbers, so every answer was about regression.

**Follow-ups owed:**
- Add permalinks to individual comments in [`../evidence/practitioner-responses.md`](../evidence/practitioner-responses.md)
- Confirm the full thread title
- PR-04's linked resource has not been retrieved
- `abrahamguo` asked PR-01 which library it was — worth following

---

## Planned threads

Not yet posted. Each targets a question with **zero** Tier 3 coverage.

### T-02 — provenance (RQ3)

- **Candidate communities:** r/netsec, r/AskNetsec, r/devsecops
- **Question:** How much does a valid npm provenance attestation actually change your review?
- **Framing:** provenance attests build origin, not code safety. In the Nx compromise the
  attacker used the project's real publish workflow — an attestation would have been valid.
  Does that change how much weight it deserves?

### T-03 — build vs runtime dependencies (RQ6, and the false-positive question P4)

- **Candidate communities:** r/devops, r/sre, r/devsecops
- **Question:** Would you accept a policy that escalates *every* build-tool update to a
  human? Two of the three big 2025 npm incidents harvested CI credentials.

### T-04 — the cooldown question (RQ8, priority P1)

- **Candidate communities:** r/node, r/javascript, r/devops
- **Question:** Renovate recommends waiting 14 days before automerging. If you wait two
  weeks, how much does anything else you check actually add?
- **Note:** this is the question most likely to invalidate the project's premise, which is
  why it should be asked plainly and early.

### T-05 — transitive closure (RQ7)

- **Candidate communities:** r/node, r/javascript
- **Question:** A direct bump adds four new transitive packages and a new publisher in the
  closure. Do you look? What would make you stop?

## Community verification

Before posting, confirm for each community: self-promotion rules, whether `[AskJS]`-style
tagging is required, minimum account age/karma, and whether links to a personal repository
are permitted. **Not yet done for T-02 through T-05.**
