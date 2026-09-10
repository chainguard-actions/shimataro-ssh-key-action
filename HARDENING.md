<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.7.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions `${{ github.env }}` are interpolated directly inside `run:` shell command strings in the composite action. Any `${{ ... }}` expression in a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Line 11: `run: echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}`. Line 14: `run: echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`. These should use the `$GITHUB_ENV` environment variable instead of the `${{ github.env }}` expression.

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:14`

### unpinned-uses (severity: high)

The composite action references `actions/cache@v3`, which uses a mutable version tag rather than a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a different (potentially malicious) commit. It should be pinned to a full SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4`.

Locations:

- `.github/actions/cache-npm/action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed two findings in hardened/action/.github/actions/cache-npm/action.yml: (1) Replaced `${{ github.env }}` expressions in both PowerShell `run:` steps with `$Env:GITHUB_ENV` — the proper PowerShell syntax for the GITHUB_ENV environment variable — eliminating the script-injection risk. (2) Pinned `actions/cache@v3` to its full commit SHA `actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3` to prevent supply-chain attacks via mutable tag references.

