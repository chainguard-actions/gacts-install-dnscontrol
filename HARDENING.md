<!-- markdownlint-disable -->

# Hardening Report: gacts--install-dnscontrol/v1.3.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-dnscontrol/v1.3.4** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in release.yml use mutable tag refs instead of pinned 40-character SHA commits, making the workflow vulnerable to supply-chain attacks if the referenced action tag is moved or compromised. Unpinned refs: `actions/checkout@v6` (line 15), `gacts/github-slug@v1` (line 16).

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### unpinned-uses (severity: high)

Multiple `uses:` references in tests.yml use mutable tag refs instead of pinned 40-character SHA commits. Unpinned refs: `actions/checkout@v6` (lines 24, 29, 38, 53, 61), `gacts/gitleaks@v1` (line 25), `actions/setup-node@v6` (lines 30, 39), `actions/upload-artifact@v7` (line 42), `actions/download-artifact@v8` (line 54), `stefanzweifel/git-auto-commit-action@v7` (line 55).

Locations:

- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:39`
- `.github/workflows/tests.yml:42`
- `.github/workflows/tests.yml:54`
- `.github/workflows/tests.yml:55`
- `.github/workflows/tests.yml:61`

### script-injection (severity: high)

Sub-rule (a): In release.yml, the `run:` block at the third step directly interpolates `${{ github.actor }}` into shell commands. This allows an attacker who controls the actor name to inject arbitrary shell commands. Offending lines: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`. These expressions should be moved to an `env:` block and the env var double-quoted in the shell.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and the `update-git-tag` job has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository token permissions, which may be overly broad.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level `permissions:` key, and the jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` have no job-level `permissions:` key. Only the `commit-and-push-fresh-dist` job defines permissions. Without explicit permissions on all jobs, those jobs inherit default (potentially broad) token permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings in release.yml and tests.yml:

1. **unpinned-uses**: Pinned all action references to full 40-character commit SHAs with tag comments for readability. Actions pinned: actions/checkout@v6, gacts/github-slug@v1, gacts/gitleaks@v1, actions/setup-node@v6, actions/upload-artifact@v7, actions/download-artifact@v8, stefanzweifel/git-auto-commit-action@v7.

2. **script-injection**: In release.yml, moved `${{ github.actor }}` from the `run:` shell block into the step's `env:` block as `GIT_ACTOR`, then referenced it as `$GIT_ACTOR` in the shell script to prevent shell injection.

3. **missing-permissions**: Added `permissions: {}` at the top level of both release.yml and tests.yml. Added job-level `permissions: contents: write` to the `update-git-tag` job in release.yml (needed for git push). Added `permissions: {}` to all jobs in tests.yml that don't require token access (gitleaks, eslint, dist-built, run-this-action). The commit-and-push-fresh-dist job already had appropriate permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/release.yml line 27: moved `${{ secrets.GITHUB_TOKEN }}` from the `run:` shell command string into the step's `env:` block as `GH_TOKEN: "${{ secrets.GITHUB_TOKEN }}"`, and updated the shell script to reference it as `$GH_TOKEN` instead of the direct expression interpolation.

