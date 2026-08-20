<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The reusable workflow directly interpolates the workflow_call input `inputs.package_installation_command` inside a `run:` shell command: `run: ${{ inputs.package_installation_command }}`. Any caller of this reusable workflow can supply arbitrary shell commands as the input value, leading to full remote code execution on the runner. This pattern appears in every job in the file (12 occurrences). The value must be passed via an `env:` variable and the shell command must be restructured so that the expression is never directly embedded in the `run:` string.

Locations:

- `.github/workflows/reusable-verify.yml:38`
- `.github/workflows/reusable-verify.yml:51`
- `.github/workflows/reusable-verify.yml:68`
- `.github/workflows/reusable-verify.yml:81`
- `.github/workflows/reusable-verify.yml:98`
- `.github/workflows/reusable-verify.yml:111`
- `.github/workflows/reusable-verify.yml:128`
- `.github/workflows/reusable-verify.yml:141`
- `.github/workflows/reusable-verify.yml:158`
- `.github/workflows/reusable-verify.yml:171`
- `.github/workflows/reusable-verify.yml:188`
- `.github/workflows/reusable-verify.yml:201`

### unpinned-uses (severity: high)

Multiple workflow files reference third-party actions using mutable version tags instead of full 40-character commit SHA digests. This exposes the workflow to supply-chain attacks if the tag is moved or the upstream repository is compromised. Affected references: `actions/checkout@v3` (build.yml line 27, reusable-verify.yml — 12 occurrences) and `actions/setup-node@v3` (build.yml line 29). All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/build.yml:27`
- `.github/workflows/build.yml:29`
- `.github/workflows/reusable-verify.yml:40`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `read-all` or `write-all` depending on repository settings). All workflow files should declare minimal required permissions explicitly, e.g. `permissions: read-all` or scoped permissions such as `contents: read`.

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

Fixed all 3 findings across 8 workflow files:

1. script-injection (12 occurrences in reusable-verify.yml): Replaced all `run: ${{ inputs.package_installation_command }}` patterns with a safe env-variable + xargs tokenization approach. The input is placed in PACKAGE_INSTALLATION_COMMAND env var, then tokenized via `xargs printf '%s\0'` and executed as an array, preventing shell injection while correctly handling multi-word commands like `apk add openssh-client git`.

2. unpinned-uses: Pinned actions/checkout@v3 to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 and actions/setup-node@v3 to SHA 3235b876344d2a9aa001b8d1453c930bba69e610, with original tags preserved as comments.

3. missing-permissions: Added `permissions: contents: read` at top-level and job-level to all 8 workflow files (build.yml, reusable-verify.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml, verify-on-macos.yml, verify-on-ubuntu.yml, verify-on-windows.yml).

