<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.8.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in the composite action directly interpolate `${{ github.env }}` inside the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, allowing a malicious value to break out of the intended command. Line 11: `run: echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}`. Line 14: `run: echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`. These should use the `$GITHUB_ENV` environment variable instead of the `${{ github.env }}` expression.

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:14`

### unpinned-uses (severity: high)

The composite action references `actions/cache@v3`, which is a mutable tag reference rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, making this a supply-chain risk. It should be pinned to a full SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4`.

Locations:

- `.github/actions/cache-npm/action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed two findings in hardened/action/.github/actions/cache-npm/action.yml: (1) Replaced `${{ github.env }}` with `$GITHUB_ENV` on lines 11 and 14 to eliminate script-injection risk — the Actions template engine interpolates `${{ ... }}` expressions before the shell sees them, so using the environment variable directly is safe; (2) Pinned `actions/cache@v3` to full commit SHA `actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3` to eliminate supply-chain risk from a mutable tag.

