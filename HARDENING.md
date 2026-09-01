<!-- markdownlint-disable -->

# Hardening Report: crs-k--stale-branches/v10.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crs-k--stale-branches/v10.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `github.event.action` context expression is interpolated directly inside `run:` shell commands via `echo '${{ github.event.action }}'`. This flows through YAML template substitution before the shell processes it, enabling script injection if the value contains shell metacharacters. This pattern appears in three separate jobs (Lint, Test-Unit, Build).

Locations:

- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:38`
- `.github/workflows/ci.yml:79`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

- ci.yml: `actions/checkout@v7`, `actions/setup-node@v6` (multiple occurrences), `./` (local)
- codeql.yml: `actions/checkout@v7`, `github/codeql-action/init@v4`, `github/codeql-action/analyze@v4`
- dependabot-automerge.yml: `dependabot/fetch-metadata@v3.1.0`
- release-draft.yml: `crs-k/release-draft@v0.7.2`
- stale-branches-test.yml: `crs-k/stale-branches@v9.0.1`
- stale.yml: `actions/stale@v10`

Locations:

- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:20`
- `.github/workflows/codeql.yml:52`
- `.github/workflows/codeql.yml:60`
- `.github/workflows/codeql.yml:63`
- `.github/workflows/codeql.yml:91`
- `.github/workflows/dependabot-automerge.yml:14`
- `.github/workflows/release-draft.yml:14`
- `.github/workflows/stale-branches-test.yml:14`
- `.github/workflows/stale.yml:8`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its jobs (Lint, Test-Unit, Test-Action, Build) define job-level `permissions:` blocks. This means the workflow runs with the default token permissions, which may be broader than necessary. stale.yml similarly has no top-level or job-level `permissions:` key.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/stale.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across six workflow files:

1. script-injection (ci.yml lines 15, 38, 79): Moved `${{ github.event.action }}` out of `run:` shell strings into `env:` blocks as `GITHUB_EVENT_ACTION` in all three jobs (Lint, Test-Unit, Build). Shell now references `$GITHUB_EVENT_ACTION` safely.

2. unpinned-uses: Pinned all mutable action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (ci.yml, codeql.yml)
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 (ci.yml)
   - github/codeql-action/init@v4 → @cdf488f595d80d6e07e03d4674febd5ab45fa938 (codeql.yml)
   - github/codeql-action/analyze@v4 → @cdf488f595d80d6e07e03d4674febd5ab45fa938 (codeql.yml)
   - dependabot/fetch-metadata@v3.1.0 → @25dd0e34f4fe68f24cc83900b1fe3fe149efef98 (dependabot-automerge.yml)
   - crs-k/release-draft@v0.7.2 → @b232aaf8608c0ba7df6d7c6382fffa238230e9b1 (release-draft.yml)
   - crs-k/stale-branches@v9.0.1 → @e876b957ab08f0bad43c8cc271a22cdd7604bd09 (stale-branches-test.yml)
   - actions/stale@v10 → @1e223db275d687790206a7acac4d1a11bd6fe629 (stale.yml)

3. missing-permissions: Added top-level `permissions: {}` and job-level `permissions: { contents: read }` to ci.yml for all four jobs. Added top-level `permissions: {}` and job-level `permissions: { issues: write, pull-requests: write }` to stale.yml.

