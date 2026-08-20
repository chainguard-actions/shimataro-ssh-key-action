<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.6.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The entire `run:` value in 12 job steps of reusable-verify.yml is the expression `${{ inputs.package_installation_command }}`, which is a workflow `inputs.*` value interpolated directly into the shell command string. A caller can supply arbitrary shell commands (e.g. `rm -rf /` or exfiltration commands) as this input and they will be executed verbatim on the runner. Additionally, .github/actions/cache-npm/action.yml uses `${{ github.env }}` directly inside two `run:` blocks (lines 11 and 14), which is a `github.*` expression interpolated into the shell command string.

Locations:

- `.github/workflows/reusable-verify.yml:39`
- `.github/workflows/reusable-verify.yml:55`
- `.github/workflows/reusable-verify.yml:93`
- `.github/workflows/reusable-verify.yml:109`
- `.github/workflows/reusable-verify.yml:147`
- `.github/workflows/reusable-verify.yml:163`
- `.github/workflows/reusable-verify.yml:201`
- `.github/workflows/reusable-verify.yml:218`
- `.github/workflows/reusable-verify.yml:235`
- `.github/workflows/reusable-verify.yml:252`
- `.github/workflows/reusable-verify.yml:269`
- `.github/workflows/reusable-verify.yml:286`
- `.github/actions/cache-npm/action.yml:11`
- `.github/actions/cache-npm/action.yml:14`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if the upstream action tag is moved or compromised. Failing references: `actions/checkout@v3` (build.yml and reusable-verify.yml), `actions/setup-node@v3` (build.yml), `actions/cache@v3` (.github/actions/cache-npm/action.yml).

Locations:

- `.github/workflows/build.yml:27`
- `.github/workflows/build.yml:29`
- `.github/workflows/reusable-verify.yml:41`
- `.github/actions/cache-npm/action.yml:18`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job within any of these files defines a `permissions:` block either. Without explicit permissions, workflows run with the default token permissions (which may be `write-all` depending on repository settings), granting unnecessary access to repository resources. All 8 workflow files are affected: build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, verify-on-windows.yml.

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

Fixed all three findings:

1. script-injection: (a) In reusable-verify.yml, all 12 `run: ${{ inputs.package_installation_command }}` steps were replaced with a safe pattern using an env: block + xargs-based array tokenization to safely execute the multi-word command. (b) In cache-npm/action.yml, both `${{ github.env }}` references in run: blocks were replaced with the safe `$GITHUB_ENV` environment variable.

2. unpinned-uses: Pinned actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 (build.yml + reusable-verify.yml), actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610 (build.yml), actions/cache@v3 → @6f8efc29b200d32929f49075959781ed54ec270c (cache-npm/action.yml). All retain # v3 comments for readability.

3. missing-permissions: Added `permissions: {}` top-level block to all 8 workflow files: build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, verify-on-windows.yml.

