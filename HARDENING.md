<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-just/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-just/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v6` (a mutable tag reference, not a pinned SHA) in two steps. If the tag is moved or the repository is compromised, the action could execute arbitrary code. Replace with a full 40-character commit SHA, e.g. `actions/checkout@<sha> # v6`.

Locations:

- `.github/workflows/build.yaml:15`
- `.github/workflows/build.yaml:33`

### script-injection (severity: high)

Rule (a) violation: `${{ matrix.just-version }}` is interpolated directly inside a `run:` shell command string on line 36: `run: just --version | grep -E "^just v?${{ matrix.just-version }}$"`. The matrix value is workflow-controllable and is substituted into the shell command before the shell parses it, enabling command injection. Move the value into an `env:` variable and double-quote it in the shell script instead.

Locations:

- `.github/workflows/build.yaml:36`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and neither job (`test-latest` nor `test-version`) declares its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary. Add a top-level `permissions: {}` or minimal per-job permissions.

Locations:

- `.github/workflows/build.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/build.yaml: (1) Pinned both `actions/checkout@v6` references to full SHA `d23441a48e516b6c34aea4fa41551a30e30af803` with `# v6` comment. (2) Moved `${{ matrix.just-version }}` out of the `run:` shell string into an `env:` block as `JUST_VERSION`, referencing it as `${JUST_VERSION}` in the grep command. (3) Added `permissions: {}` at the top level of the workflow to restrict default token permissions.

