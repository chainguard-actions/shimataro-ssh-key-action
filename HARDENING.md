<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.8.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Install packages' step in every job of reusable-verify.yml directly interpolates the workflow input `${{ inputs.package_installation_command }}` as the entire `run:` command. This allows any caller of the reusable workflow to inject arbitrary shell commands. The expression is not routed through an env var — it IS the shell command.

Locations:

- `.github/workflows/reusable-verify.yml:36`
- `.github/workflows/reusable-verify.yml:52`
- `.github/workflows/reusable-verify.yml:68`
- `.github/workflows/reusable-verify.yml:84`
- `.github/workflows/reusable-verify.yml:100`
- `.github/workflows/reusable-verify.yml:116`
- `.github/workflows/reusable-verify.yml:132`
- `.github/workflows/reusable-verify.yml:148`
- `.github/workflows/reusable-verify.yml:164`
- `.github/workflows/reusable-verify.yml:180`
- `.github/workflows/reusable-verify.yml:196`
- `.github/workflows/reusable-verify.yml:212`

### script-injection (severity: high)

Sub-rule (a): In .github/actions/cache-npm/action.yml, the `${{ github.env }}` expression is interpolated directly inside `run:` shell command strings (PowerShell). Any `${{ ... }}` expression inside a `run:` block is a script-injection finding. Offending lines: `echo "NPM_CACHE_DIRECTORY=$(npm config get cache)" >> ${{ github.env }}` and `echo "NODEJS_VERSION=$(node -v)" >> ${{ github.env }}`.

Locations:

- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:14`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions by mutable tag rather than a full 40-character commit SHA, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v6` (build.yml, reusable-verify.yml), `actions/setup-node@v6` (build.yml), `actions/cache@v3` (.github/actions/cache-npm/action.yml).

Locations:

- `.github/workflows/build.yml:26`
- `.github/workflows/build.yml:28`
- `.github/workflows/reusable-verify.yml:38`
- `.github/workflows/reusable-verify.yml:54`
- `.github/workflows/reusable-verify.yml:70`
- `.github/workflows/reusable-verify.yml:86`
- `.github/workflows/reusable-verify.yml:102`
- `.github/workflows/reusable-verify.yml:118`
- `.github/workflows/reusable-verify.yml:134`
- `.github/workflows/reusable-verify.yml:150`
- `.github/workflows/reusable-verify.yml:166`
- `.github/workflows/reusable-verify.yml:182`
- `.github/workflows/reusable-verify.yml:198`
- `.github/workflows/reusable-verify.yml:214`
- `.github/actions/cache-npm/action.yml:17`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and none of the jobs within them define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Affected files: build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, verify-on-windows.yml.

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

Fixed all four findings across multiple files:

1. script-injection (reusable-verify.yml, 12 locations): Replaced `run: ${{ inputs.package_installation_command }}` with an env-var-based approach using xargs tokenization. The input is now passed via `PACKAGE_INSTALLATION_COMMAND` env var and tokenized into a bash array with `xargs printf '%s\0'` before execution, preventing shell injection while preserving multi-word command support.

2. script-injection (cache-npm/action.yml, 2 locations): Replaced `>> ${{ github.env }}` with `>> "$GITHUB_ENV"` — using the standard environment variable instead of a GitHub expression interpolated directly into the shell command.

3. unpinned-uses (15 locations): Pinned all three actions to full 40-character commit SHAs with tag comments: actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38, actions/cache@v3 → 6f8efc29b200d32929f49075959781ed54ec270c.

4. missing-permissions (8 workflow files): Added `permissions: {}` top-level block to build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, and verify-on-windows.yml.

