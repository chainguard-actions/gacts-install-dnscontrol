<!-- markdownlint-disable -->

# Hardening Report: gacts--install-dnscontrol/v1.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--install-dnscontrol/v1.3.3** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The release workflow uses action references pinned to mutable tags instead of full 40-character commit SHAs. If the tag is moved (intentionally or by a supply-chain attacker), the workflow will silently execute different code. Failing references: `actions/checkout@v6` (line 15) and `gacts/github-slug@v1` (line 16). These must be replaced with their full SHA digests, e.g. `actions/checkout@<40-hex-sha> # v6`.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### script-injection (severity: high)

The release workflow interpolates GitHub Actions expressions directly inside `run:` shell command strings (sub-rule a), allowing template substitution to inject arbitrary shell metacharacters before the shell parses the command. Offending lines: (1) Line 21: `git config --local user.name "${{ github.actor }}"` — github.actor is injected directly into the shell command; a crafted actor name with shell metacharacters would be executed. (2) Line 22: `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"` — both github.actor and secrets.GITHUB_TOKEN are interpolated directly into the shell string. Fix: move these values into env: variables and reference them as "$VAR" in the shell script.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### missing-permissions (severity: medium)

The release workflow has no top-level `permissions:` key and the single job `update-git-tag` also has no `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary. A minimal explicit permissions block (e.g. `contents: write`) should be added at the job level.

Locations:

- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/release.yml: (1) Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and gacts/github-slug@v1 to SHA 83cd3d95888be8777f0d9c810c1266da1c5c3ef1, preserving original tags as comments. (2) Moved github.actor and secrets.GITHUB_TOKEN out of the run: shell string into the step's env: block as GIT_ACTOR and GITHUB_TOKEN, referencing them as plain shell variables. (3) Added a top-level permissions block with contents: write (minimum needed for pushing tags).

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed hardened/action/.github/workflows/tests.yml: (1) Pinned all 10 unpinned `uses:` references to full 40-character commit SHAs with original tags preserved as comments: actions/checkout@v6→d23441a48e516b6c34aea4fa41551a30e30af803, gacts/gitleaks@v1→c9a0338361dc45a01aa7ebaaa5330179f3c62873, actions/setup-node@v6→249970729cb0ef3589644e2896645e5dc5ba9c38, actions/upload-artifact@v7→043fb46d1a93c77aae656e7c1c64a875d1fc6a0a, actions/download-artifact@v8→3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c, stefanzweifel/git-auto-commit-action@v7→4a55954c782fc1ea30b9056cd3e7a2b40ca8887d. (2) Added top-level `permissions: {}` to deny all permissions by default, and added `permissions: { contents: read }` to the gitleaks, eslint, dist-built, and run-this-action jobs. The commit-and-push-fresh-dist job's existing permissions (contents: write, pull-requests: write) were preserved.

