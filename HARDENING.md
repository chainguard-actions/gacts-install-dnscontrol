<!-- markdownlint-disable -->

# Hardening Report: gacts--install-dnscontrol/v1.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **gacts--install-dnscontrol/v1.3.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of ${{ github.actor }} and ${{ secrets.GITHUB_TOKEN }} inside a run: block in release.yml. The expressions are substituted by the Actions template engine before the shell sees the string, allowing an attacker who controls the actor name to inject arbitrary shell commands. Offending lines:
  Line 21: `git config --local user.name "${{ github.actor }}"`
  Line 22: `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
These values should be passed via env: variables and referenced as $ACTOR / $TOKEN in the shell script.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files use mutable tag or version refs instead of immutable 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

In .github/workflows/release.yml:
  - Line 15: actions/checkout@v6
  - Line 16: gacts/github-slug@v1

In .github/workflows/tests.yml:
  - Line 23: actions/checkout@v6
  - Line 25: gacts/gitleaks@v1
  - Line 30: actions/checkout@v6
  - Line 31: actions/setup-node@v6
  - Line 41: actions/checkout@v6
  - Line 42: actions/setup-node@v6
  - Line 45: actions/upload-artifact@v7
  - Line 54: actions/checkout@v6
  - Line 55: actions/download-artifact@v8
  - Line 56: stefanzweifel/git-auto-commit-action@v7
  - Line 63: actions/checkout@v6

All should be pinned to full SHA digests, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:42`
- `.github/workflows/tests.yml:45`
- `.github/workflows/tests.yml:54`
- `.github/workflows/tests.yml:55`
- `.github/workflows/tests.yml:56`
- `.github/workflows/tests.yml:63`

### missing-permissions (severity: medium)

release.yml has no top-level permissions: key and no job-level permissions: key on any job. This means the workflow runs with the default (broad) token permissions. Every job should declare minimal explicit permissions.

tests.yml has a job-level permissions: block only on the commit-and-push-fresh-dist job; the gitleaks, eslint, dist-built, and run-this-action jobs all lack a permissions: key, so they also run with default broad permissions.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in release.yml and tests.yml:

1. script-injection: Moved ${{ github.actor }} and ${{ secrets.GITHUB_TOKEN }} from inline run: expressions into the step's env: block as ACTOR and TOKEN variables. Shell script now references $ACTOR and $TOKEN.

2. unpinned-uses: Pinned all 7 unique action references to full 40-character SHA digests with original tag preserved as comment: actions/checkout@df4cb1c069e1874edd31b4311f1884172cec0e10 # v6, gacts/github-slug@83cd3d95888be8777f0d9c810c1266da1c5c3ef1 # v1, gacts/gitleaks@c9a0338361dc45a01aa7ebaaa5330179f3c62873 # v1, actions/setup-node@249970729cb0ef3589644e2896645e5dc5ba9c38 # v6, actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7, actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8, stefanzweifel/git-auto-commit-action@4a55954c782fc1ea30b9056cd3e7a2b40ca8887d # v7.

3. missing-permissions: Added explicit permissions blocks to all jobs. release.yml update-git-tag gets contents: write (needed to push tags). tests.yml gitleaks, eslint, dist-built, and run-this-action jobs get contents: read. The commit-and-push-fresh-dist job already had contents: write and pull-requests: write which were preserved.

