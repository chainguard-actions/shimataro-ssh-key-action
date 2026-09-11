<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.8.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in the composite action embed `${{ github.env }}` directly inside shell command strings. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk, as the value is substituted by the YAML template engine before the shell ever sees it. Offending lines:
- Line 11: `run: echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}`
- Line 14: `run: echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`
Fix: use the `$GITHUB_ENV` environment variable instead of the `${{ github.env }}` expression, e.g. `>> "$GITHUB_ENV"`

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:14`

### unpinned-uses (severity: high)

The composite action references `actions/cache@v3`, which uses a mutable version tag rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. Pin to a specific commit SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4.1.2`.

Locations:

- `.github/actions/cache-npm/action.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed two findings in hardened/action/.github/actions/cache-npm/action.yml: (1) script-injection: replaced `${{ github.env }}` with `"$GITHUB_ENV"` on both `run:` steps (lines 11 and 14) so the value is resolved by the runner environment rather than interpolated by the YAML template engine; (2) unpinned-uses: pinned `actions/cache@v3` to its full commit SHA `actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3`.

