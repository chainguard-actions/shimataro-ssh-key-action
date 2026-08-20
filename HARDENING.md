<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.7.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in the cache-npm composite action directly interpolate `${{ github.env }}` inside the shell command string. Although `github.env` is a path to a file rather than attacker-controlled data, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules. Line 11: `run: echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}`. Line 15: `run: echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`. The correct pattern is to use the `$GITHUB_ENV` environment variable instead of the `${{ github.env }}` expression.

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:15`

### unpinned-uses (severity: high)

Multiple workflow files and the cache-npm composite action reference external actions using mutable version tags instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: build.yml — `actions/checkout@v4` (line 25), `actions/setup-node@v4` (line 28); reusable-verify.yml — `actions/checkout@v3` (lines 32, 52, 72, 92, 112, 132, 152, 172, 192, 212); cache-npm/action.yml — `actions/cache@v3` (line 18).

Locations:

- `.github/workflows/build.yml:25`
- `.github/workflows/build.yml:28`
- `.github/workflows/reusable-verify.yml:32`
- `.github/workflows/reusable-verify.yml:52`
- `.github/workflows/reusable-verify.yml:72`
- `.github/workflows/reusable-verify.yml:92`
- `.github/workflows/reusable-verify.yml:112`
- `.github/workflows/reusable-verify.yml:132`
- `.github/workflows/reusable-verify.yml:152`
- `.github/workflows/reusable-verify.yml:172`
- `.github/workflows/reusable-verify.yml:192`
- `.github/workflows/reusable-verify.yml:212`
- `.github/actions/cache-npm/action.yml:18`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all` for some repositories), violating the principle of least privilege. All eight workflow files are affected: build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, verify-on-windows.yml.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/reusable-verify.yml:1`
- `.github/workflows/verify-on-container-alpine.yml:1`
- `.github/workflows/verify-on-container-centos.yml:1`
- `.github/workflows/verify-on-container-ubuntu.yml:1`
- `.github/workflows/verify-on-macos.yml:1`
- `.github/workflows/verify-on-ubuntu.yml:1`
- `.github/workflows/verify-on-windows.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed script-injection in cache-npm/action.yml by replacing ${{ github.env }} with $GITHUB_ENV in two run: steps. Pinned all unpinned action references to full commit SHAs: actions/checkout@v4 (11d5960a), actions/setup-node@v4 (49933ea5), actions/checkout@v3 (a37ce912, 12 occurrences in reusable-verify.yml), actions/cache@v3 (6f8efc29). Added 'permissions: {}' top-level block to all 8 workflow files to enforce least-privilege.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 12 occurrences of script injection in .github/workflows/reusable-verify.yml. Each 'Install packages' step previously used `run: ${{ inputs.package_installation_command }}` which allowed arbitrary shell command injection. Fixed by moving the expression into an `env:` block (PACKAGE_INSTALLATION_COMMAND) and using xargs-based NUL-delimited tokenization into a bash array before executing, which safely handles quoted arguments without allowing shell injection.

