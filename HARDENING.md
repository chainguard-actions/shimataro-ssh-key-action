<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.6.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.env }}` is interpolated directly inside `run:` shell command strings. Even though `github.env` resolves to a file path rather than attacker-controlled data, any `${{ ... }}` expression embedded directly in a `run:` block is a script-injection violation — the YAML template substitution occurs before the shell ever sees the command. Line 11: `run: echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}`. Line 15: `run: echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`. These should be replaced with the environment variable `$GITHUB_ENV` instead (e.g., `>> $GITHUB_ENV`).

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:15`

### unpinned-uses (severity: high)

The composite action references `actions/cache@v3`, which is a mutable tag reference rather than a pinned full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. It should be pinned to a specific commit SHA, e.g. `actions/cache@88522ab9f39a2ea568f7027eddc7d8d8bc9d59c8 # v3`.

Locations:

- `.github/actions/cache-npm/action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed two findings in hardened/action/.github/actions/cache-npm/action.yml: (1) Replaced both occurrences of `${{ github.env }}` in run: steps with `$GITHUB_ENV` (the runner-provided environment variable), eliminating script-injection via YAML template substitution. (2) Pinned `actions/cache@v3` to its full commit SHA `actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3` to prevent supply-chain attacks via mutable tag references.

