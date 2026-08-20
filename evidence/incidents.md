# Incident Records

Raw factual records only — package, versions, dates, mechanism, sources. The counterfactual
analysis ("would our policy have caught this?") lives in
[`../research/07-incident-research.md`](../research/07-incident-research.md) and is
deliberately kept out of this file, so that the facts stay separable from our reading of
them.

**All records below are `VERIFIED BY HUMAN: no`.** Source retrieval status is recorded per
source in [`sources.md`](./sources.md); several primary sources could not be fetched
directly and rest on search-index summaries.

Where reported figures differ between sources, both figures are given with attribution
rather than reconciled. Disagreement between sources is itself data.

---

## INC-01 — Nx `s1ngularity`

| Field | Value |
|---|---|
| **Packages** | `nx`, `@nx/devkit`, `@nx/js`, `@nx/workspace`, `@nx/node`, `@nx/eslint`, `@nx/key`, `@nx/enterprise-cloud` |
| **Versions** | `nx`: 20.9.0, 20.10.0, 20.11.0, 20.12.0, 21.5.0, 21.6.0, 21.7.0, 21.8.0. Others: 21.5.0 / 20.9.0; `@nx/key` 3.2.0, 5.0.7; `@nx/enterprise-cloud` 3.2.0 |
| **Date** | Began 26 August 2025, ~22:32 UTC. Advisory published 27 August 2025 |
| **Window** | Malicious versions live approximately 4 hours (S-17) |
| **Dependency type** | **build** — Nx is a build system / monorepo tool |
| **Advisory** | GHSA-cxm3-wv7p-598c / CVE-2025-10894, severity High. **Withdrawn 28 July 2026** |
| **Patched versions** | None — remediation was by unpublishing/reverting, not patching |
| **Sources** | S-07 (direct), S-16, S-17, S-18 |

### Mechanism

Initial access was not a compromised maintainer account. A PR validation workflow contained
unescaped variables permitting shell command injection through a malicious pull request
title. The workflow used `pull_request_target`, granting `GITHUB_TOKEN` read/write
repository access. The attacker used this to trigger the repository's own `publish.yml`
workflow, obtaining the npm publishing token.

The published packages carried a `postinstall` script that:

- scanned the filesystem for configuration and credential files;
- collected paths and sensitive data;
- posted encoded credentials to newly created repositories under victims' own GitHub
  accounts, named `s1ngularity-repository`;
- appended `sudo shutdown -h 0` to `.zshrc` and `.bashrc`.

Reported by S-18 to have leaked 2,349 GitHub, cloud, and AI credentials. S-17 and S-18
report the payload invoked locally installed AI CLI tools to assist reconnaissance. A second
wave using the leaked credentials was reported 28 August 2025, ~20:00 UTC.

### Facts bearing on decision-time observability

- Publisher identity: **unchanged**. Packages were published by the legitimate publishing
  workflow using a legitimately issued token.
- Install script: **present** (`postinstall`).
- Provenance: not established by available sources — **open question, needs verification**.
- SemVer: releases spanned patch, minor, and major-adjacent version lines across two
  major series concurrently — an unusual publication pattern.

---

## INC-02 — `chalk` / `debug` and 16 further packages

| Field | Value |
|---|---|
| **Packages** | 18 packages including `chalk`, `debug`, `ansi-styles`, `color` |
| **Versions** | Not fully enumerated in retrieved sources — **gap, must be filled from S-13/S-15** |
| **Date** | 8 September 2025, malicious versions appearing from ~13:15 UTC |
| **Window** | Approximately 2 hours; clean versions restored and malicious releases unpublished by ~15:30 UTC |
| **Dependency type** | **runtime** (and transitively near-universal) |
| **Reach** | Reported >2 billion downloads per week combined (S-14) |
| **Sources** | S-13 (blocked), S-14, S-15 |

### Mechanism

A maintainer account was hijacked by **phishing**. The phishing email originated from
`support@npmjs.help`, a domain registered three days earlier on 5 September 2025.

The injected payload was **not an install script**. It was client-side JavaScript that
executed in end users' browsers, silently intercepting cryptocurrency and web3 activity,
manipulating wallet interactions, and rewriting payment destinations to attacker-controlled
accounts.

S-14 reports that exposure required a fresh install (or lockfile generation) during the
roughly two-and-a-quarter-hour window with one of the affected packages in the tree,
directly or transitively.

### Facts bearing on decision-time observability

- Publisher identity: **unchanged**. The legitimate maintainer's real account published.
- Install script: **absent**. The payload required no install-time execution.
- Provenance: not established by available sources — **open question, needs verification**.
- Exposure window: ~2 hours, which is shorter than any plausible cooldown period.

---

## INC-03 — Shai-Hulud worm (and Shai-Hulud 2.0)

| Field | Value |
|---|---|
| **Packages** | Reported 500+ packages (S-Truesec) / 1,300+ package versions (S-10, S-11). Named packages include `keyv`, `cacheable`, `flat-cache`, `file-entry-cache` |
| **Versions** | Not enumerated — the affected set changed continuously as the worm propagated |
| **Date** | First wave from 15 September 2025. "Shai-Hulud 2.0" first reported early November 2025. Still active into 2026 per S-09 |
| **Dependency type** | **mixed** — spread was determined by token reach, not dependency role |
| **Reach** | Reported ~2 billion monthly downloads combined (S-10, S-11) |
| **Advisories** | CISA alert 23 September 2025 (S-08); CSA Singapore AD-2026-009 (S-09) |
| **Sources** | S-08, S-09, S-10, S-11, S-12, S-19 |

### Mechanism

Malicious versions carried a `postinstall` script that harvested secrets from CI/CD
environments, environment variables, cloud metadata endpoints, and via TruffleHog scanning,
exfiltrating them to attacker-created public GitHub repositories named `Shai-Hulud`. It also
migrated private organisational repositories to public personal repositories under
attacker-controlled accounts, with the description `Shai-Hulud Migration`.

The defining property is **self-propagation**: on encountering additional npm tokens in the
environment, the malware automatically published malicious versions of every package those
tokens could reach. The compromise therefore spread through the ecosystem without further
attacker action.

### Facts bearing on decision-time observability

- Publisher identity: **unchanged** for each individual compromised package — each was
  published using its own legitimate maintainer's stolen token.
- Install script: **present** (`postinstall`).
- Transitive exposure: high by construction. A worm propagating along token-reach lines
  places compromised packages at arbitrary depths in dependency closures.
- Time distribution: the affected set grew continuously rather than appearing at once, so
  "how old is this release" varies per package and there is no single incident window.

---

## Cross-incident observations

Recorded here as facts, not conclusions. Interpretation is in
[`../research/07-incident-research.md`](../research/07-incident-research.md).

| Property | INC-01 | INC-02 | INC-03 |
|---|---|---|---|
| Publisher identity changed | No | No | No |
| Publishing credential stolen | Yes (workflow injection) | Yes (phishing) | Yes (worm-harvested tokens) |
| Install script involved | Yes | **No** | Yes |
| Dependency type | build | runtime | mixed |
| Time-to-remediation | ~4 h | ~2 h | ongoing across months |
| Credentials targeted | Yes | No (end-user funds) | Yes |
| National advisory issued | No | No | Yes |

---

## Gaps in this file

- Exact affected version list for INC-02.
- Whether provenance attestations were present on any of the malicious releases in any of
  the three incidents. **This is the single most decision-relevant unknown**, because RQ3
  and HYP-01 turn on it.
- Older canonical incidents not yet recorded: `event-stream` (2018), `ua-parser-js` (2021),
  `node-ipc` (2022), `coa`/`rc` (2021). These matter because all three incidents recorded
  above are recent and may not represent the historical distribution.
- Any incident where the correct decision was **merge** — this file currently records only
  compromises, which is a sampling bias that will distort the counterfactual analysis if
  left uncorrected.
