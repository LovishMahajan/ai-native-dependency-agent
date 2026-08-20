# Source Register

Raw, cited records of what each source establishes. Interpretation lives in
[`../research/06-existing-solutions.md`](../research/06-existing-solutions.md), not here.

## How to read this file

| Field | Meaning |
|---|---|
| **Tier** | Evidence tier per [`../methodology/research-method.md`](../methodology/research-method.md) |
| **Retrieval** | `direct` = primary source fetched and read; `index` = obtained via search index summary, primary source not fetched |
| **Verified** | Whether the author has independently read the primary source |

> **Retrieval note.** Several primary sources could not be fetched directly from the
> research environment because the network egress proxy blocks those domains
> (`cisa.gov`, `nx.dev`, `aikido.dev`, and others). Entries marked `Retrieval: index`
> rest on search-index summaries and are **weaker evidence than their tier suggests**
> until someone opens the URL. They are marked accordingly and must not be used to
> support a design decision in their current state.

---

## S-01 — npm provenance attestations

- **Tier:** 1
- **URL:** https://docs.npmjs.com/generating-provenance-statements/
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** npm provenance allows a publisher to publicly establish where a
package was built and who published it, providing cryptographic proof of build origin.
Publishing with provenance requires building on a supported cloud CI/CD provider using a
cloud-hosted runner — currently GitHub Actions and GitLab CI/CD. Provenance is unavailable
when publishing from private source repositories.

**Relevance.** Directly underpins the provenance evidence proposed in
[`research-file.md` §6](../research/research-file.md#6-observed-evidence) and RQ3.

**Limits to note.** Provenance attests to *build origin*, not to *code safety*. A package
built by the intended workflow from the intended repository can still contain malicious
code if the repository or the workflow was compromised. See INC-01 in
[`incidents.md`](./incidents.md).

---

## S-02 — npm trusted publishing (OIDC)

- **Tier:** 1
- **URL:** https://docs.npmjs.com/trusted-publishers/
- **Secondary:** https://github.blog/changelog/2025-07-31-npm-trusted-publishing-with-oidc-is-generally-available/
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** Trusted publishing creates a trust relationship between npm and a
CI/CD provider so that npm accepts publishes only from a specifically authorised workflow,
reducing reliance on long-lived publishing tokens. Generally available since 31 July 2025.
Requires npm CLI v11.5.1 or later. Supports GitHub Actions and GitLab CI/CD on
cloud-hosted runners. Packages published this way get provenance attestations
automatically, without the `--provenance` flag.

**Relevance.** Bears directly on the publisher/provenance portion of the model, and on
whether "publisher unchanged" is a meaningful safety signal (RQ3, RQ4, HYP-01).

**Limits to note.** Trusted publishing narrows the credential-theft attack surface but does
not eliminate workflow compromise — the authorised workflow itself remains a target. See
INC-01.

---

## S-03 — GitHub Dependabot

- **Tier:** 1
- **URL:** https://docs.github.com/en/code-security/tutorials/secure-your-dependencies/automate-dependabot-with-actions
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** Dependabot opens version-update and security-update pull requests.
Auto-merge is not a Dependabot decision: it requires the repository to enable auto-merge and
a separate GitHub Actions workflow, typically using `dependabot/fetch-metadata` to inspect
the update type and gate on it (commonly patch and minor only). GitHub then merges once
required checks pass.

**Relevance.** Establishes the existing automation layer this project tries to make
decision-aware, and shows that the prevailing auto-merge criterion in practice is
**SemVer bump size plus CI green** — which is precisely Baseline A in
[`research-file.md` §18](../research/research-file.md#18-baselines-to-investigate).

---

## S-04 — Renovate: minimum release age

- **Tier:** 1
- **URL:** https://docs.renovatebot.com/key-concepts/minimum-release-age/
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** Renovate's `minimumReleaseAge` refuses updates to releases newer
than a configured age. **Renovate explicitly recommends `minimumReleaseAge: "14 days"` when
automerging third-party dependencies**, with the stated rationale of giving upstream
registries time to pull malicious dependencies before Renovate merges them. When configured,
Renovate passes `--before=<date>` to npm during lockfile generation, so npm resolves only
versions that existed before the cooldown threshold — extending the protection to newly
published transitive dependencies.

**Relevance.** This is the single most important Tier 1 finding for the action space. It is
an existing, widely deployed, vendor-recommended implementation of the project's
`DELAY_AND_PIN` action, with a concrete default (14 days) and an explicit supply-chain
rationale. It bears directly on RQ8 (is waiting worth its cost?) and on HYP-04 (transitive
risk), since the `--before` mechanism addresses the transitive closure and not just the
direct bump.

---

## S-05 — Renovate: automerge

- **Tier:** 1
- **URL:** https://docs.renovatebot.com/key-concepts/automerge/
- **Secondary:** https://docs.renovatebot.com/configuration-options/, https://docs.renovatebot.com/upgrade-best-practices/
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** Renovate exposes automerge as configurable policy rather than a
single decision, combined with a dependency dashboard for human visibility.

**Relevance.** Renovate's configuration surface is effectively a hand-authored version of
the policy this project proposes to derive from evidence. Its options are a catalogue of
what practitioners have already found necessary, and should inform Baseline B.

---

## S-06 — GitHub supply chain security / dependency review

- **Tier:** 1
- **URL:** https://docs.github.com/en/code-security/supply-chain-security
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** Dependency review surfaces dependency changes, release dates,
popularity, and vulnerability information at review time.

**Relevance.** Establishes which evidence the platform already considers decision-relevant
enough to surface — a useful prior on the evidence list in
[`research-file.md` §6](../research/research-file.md#6-observed-evidence).

---

## S-07 — CVE-2025-10894 / GHSA-cxm3-wv7p-598c (Nx)

- **Tier:** 1
- **URL:** https://github.com/advisories/GHSA-cxm3-wv7p-598c
- **Retrieval:** direct
- **Verified by human:** no

**What it establishes.** Published 27 August 2025. Severity High. Affected packages and
versions: `nx` (20.9.0, 20.10.0, 20.11.0, 20.12.0, 21.5.0, 21.6.0, 21.7.0, 21.8.0),
`@nx/devkit`, `@nx/js`, `@nx/workspace`, `@nx/node` (21.5.0, 20.9.0 each), `@nx/eslint`
(21.5.0), `@nx/key` (3.2.0, 5.0.7), `@nx/enterprise-cloud` (3.2.0). No patched versions.

Mechanism: a PR validation workflow contained unescaped variables permitting command
injection via a malicious PR title, combined with `pull_request_target`, which granted
`GITHUB_TOKEN` read/write repository access. This was used to trigger the `publish.yml`
workflow. The published packages carried a `postinstall` script that scanned the filesystem
for credential files, posted encoded credentials to repositories created under victims'
accounts named `s1ngularity-repository`, and modified `.zshrc`/`.bashrc` to run a shutdown
command.

**Relevance.** Primary record for INC-01.

> **Observation worth recording separately.** This advisory was **withdrawn on 28 July
> 2026**, while the remediation details remain documented. This is directly relevant to the
> §4 proposal to define `unauthorised = 1` via public advisory: advisory records are not
> stable over time, so "is there an advisory" can return different answers depending on
> when the dataset is built. Tracked as an open issue in
> [`../methodology/evaluation-plan.md`](../methodology/evaluation-plan.md).
> **This point needs human verification before it is relied on** — confirm the withdrawal
> and its stated reason at the URL above.

---

## S-08 — CISA alert: widespread npm supply chain compromise

- **Tier:** 1
- **URL:** https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem
- **Retrieval:** index — **domain blocked by egress proxy, not fetched**
- **Verified by human:** no

**What it establishes.** A national-CERT-level advisory issued 23 September 2025 concerning
widespread compromise of the npm ecosystem.

**Relevance.** Establishes that the Shai-Hulud campaign (INC-03) reached a severity
threshold warranting government advisory, which bears on the cost side of
[`research-file.md` §11](../research/research-file.md#11-cost-model).

**⚠ Must be fetched and read before use.** The content summary above is thin because the
source could not be retrieved.

---

## S-09 — CSA Singapore advisory AD-2026-009 (Keyv / Shai-Hulud, ongoing)

- **Tier:** 1
- **URL:** https://www.csa.gov.sg/alerts-and-advisories/advisories/ad-2026-009/
- **Retrieval:** index
- **Verified by human:** no

**What it establishes.** An advisory concerning an **ongoing** npm supply chain attack
affecting `keyv` and related packages, attributed to the Shai-Hulud worm. The 2026
advisory identifier indicates the campaign remained active into 2026.

**Relevance.** Bears on base rate. [`research-file.md` §24](../research/research-file.md#24-what-we-currently-do-not-know)
lists the natural prevalence of malicious npm releases as unknown; an actively propagating
worm means prevalence is not stationary, which complicates the "approximately natural
prevalence" sampling in §16.

---

## Secondary reporting (Tier 2 corroboration only)

Not sufficient on their own to establish an incident; recorded because they carry detail the
advisories do not.

| Ref | Source | URL | Retrieval |
|---|---|---|---|
| S-10 | Wiz — Shai-Hulud analysis | https://www.wiz.io/blog/shai-hulud-npm-supply-chain-attack | index |
| S-11 | Unit 42 — Shai-Hulud worm | https://unit42.paloaltonetworks.com/npm-supply-chain-attack/ | index |
| S-12 | JFrog — newly detected compromised packages | https://jfrog.com/blog/shai-hulud-npm-supply-chain-attack-new-compromised-packages-detected/ | index |
| S-13 | Aikido — debug and chalk compromised | https://www.aikido.dev/blog/npm-debug-and-chalk-packages-compromised | index — **blocked** |
| S-14 | Wiz — debug/chalk scope and impact | https://www.wiz.io/blog/widespread-npm-supply-chain-attack-breaking-down-impact-scope-across-debug-chalk | index |
| S-15 | Semgrep — chalk, debug, color compromised | https://semgrep.dev/blog/2025/chalk-debug-and-color-on-npm-compromised-in-new-supply-chain-attack/ | index |
| S-16 | Nx — s1ngularity postmortem (maintainer) | https://nx.dev/blog/s1ngularity-postmortem | index — **blocked** |
| S-17 | StepSecurity — Nx compromise analysis | https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware | index |
| S-18 | The Hacker News — s1ngularity credential counts | https://thehackernews.com/2025/08/malicious-nx-packages-in-s1ngularity.html | index |
| S-19 | Red Hat — multiple npm supply chain attacks | https://access.redhat.com/security/supply-chain-attacks-NPM-packages | index |

S-16 is a **maintainer postmortem** and would be Tier 1 once read. It is the highest-value
unfetched source in this register.

---

## Gaps in this register

Sources that should exist here and do not yet:

- Academic or large-scale empirical research on dependency-update failure rates
- Evidence on how often SemVer major/minor/patch actually predicts breakage
- Evidence on the cost of extended checks in practice
- Any source establishing a defensible base rate for malicious npm releases
- Any source on regression-horizon conventions (the 30-day figure in §4 is unsupported)
