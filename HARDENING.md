<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-just/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-just/v4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v6` (a mutable tag, not a pinned 40-character commit SHA) in two steps. If the tag is moved or compromised, the action will silently execute different code. Pin to a full SHA instead, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/build.yaml:13`
- `.github/workflows/build.yaml:27`

### permissions (severity: medium)

missing-permissions: The workflow file has no top-level `permissions:` key and neither job (`test-latest`, `test-version`) defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Add a top-level `permissions: {}` block and grant only the minimum required scopes.

Locations:

- `.github/workflows/build.yaml:1`

### script-injection (severity: high)

Sub-rule (a): The `Check version` step in job `test-version` directly interpolates `${{ matrix.just-version }}` inside a `run:` shell command: `run: just --version | grep -E "^just v?${{ matrix.just-version }}$"`. The matrix value is substituted by the GitHub Actions template engine before the shell sees it, allowing an attacker who controls the matrix input to inject arbitrary shell metacharacters. Move the value to an `env:` variable and reference it as a quoted shell variable instead: `env: { JUST_VERSION: "${{ matrix.just-version }}" }` and `run: just --version | grep -E "^just v?${JUST_VERSION}$"`.

Locations:

- `.github/workflows/build.yaml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yaml: (1) Pinned both `actions/checkout@v6` references to full SHA `d23441a48e516b6c34aea4fa41551a30e30af803 # v6`; (2) Added `permissions: {}` at the top level to restrict default token permissions; (3) Moved `${{ matrix.just-version }}` from the `run:` shell string into an `env:` block as `JUST_VERSION` and referenced it as `${JUST_VERSION}` in the grep command to prevent script injection.

