<!-- markdownlint-disable -->

# Hardening Report: gacts--install-dnscontrol/v1.3.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-dnscontrol/v1.3.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.actor }}` is interpolated directly inside a `run:` block in release.yml, not via an `env:` variable. This allows an attacker-controlled value to be injected into the shell command string before the shell ever sees it. Offending lines:
  - `git config --local user.name "${{ github.actor }}"`
  - `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
These should be moved to an `env:` block and referenced as `"$GIT_ACTOR"` in the shell.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if the referenced tag is moved.

In `.github/workflows/release.yml`:
  - `uses: actions/checkout@v7` (line 15)
  - `uses: gacts/github-slug@v1` (line 16)

In `.github/workflows/tests.yml`:
  - `uses: actions/checkout@v7` (lines 23, 30, 43, 58, 68)
  - `uses: gacts/gitleaks@v1` (line 25)
  - `uses: actions/setup-node@v7` (lines 31, 44)
  - `uses: actions/upload-artifact@v7` (line 47)
  - `uses: actions/download-artifact@v8` (line 62)
  - `uses: stefanzweifel/git-auto-commit-action@v7` (line 63)

All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:58`
- `.github/workflows/tests.yml:62`
- `.github/workflows/tests.yml:63`
- `.github/workflows/tests.yml:68`

### missing-permissions (severity: medium)

Two workflow files lack adequate permissions declarations:

1. `.github/workflows/release.yml`: Has no top-level `permissions:` key and its only job (`update-git-tag`) also has no job-level `permissions:` block. This means the job runs with the default (potentially broad) token permissions.

2. `.github/workflows/tests.yml`: Has no top-level `permissions:` key, and three of its four jobs (`gitleaks`, `eslint`, `dist-built`, `run-this-action`) have no job-level `permissions:` block. Only `commit-and-push-fresh-dist` has explicit job-level permissions. A top-level `permissions:` block (e.g. `permissions: read-all` or specific minimal scopes) should be added, with job-level overrides where elevated access is needed.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **script-injection** (release.yml lines 20-21): Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline `run:` shell string into the step's `env:` block as `GIT_ACTOR` and `GITHUB_TOKEN`. Shell script now references them as `$GIT_ACTOR` and `$GITHUB_TOKEN`.

2. **unpinned-uses** (release.yml and tests.yml): Pinned all 7 unique action references to full 40-character commit SHAs resolved via lookup_action_sha, preserving original tags as inline comments.

3. **missing-permissions** (both files): Added `permissions: {}` at the top level of both workflow files. Added job-level `permissions: contents: write` to `update-git-tag` (needs to push tags), and `permissions: contents: read` to `gitleaks`, `eslint`, `dist-built`, and `run-this-action` jobs in tests.yml.

