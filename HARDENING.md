<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-just/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-just/v3** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v5` — a mutable version tag rather than a pinned 40-character commit SHA. This means the action could be silently replaced with a different (potentially malicious) version. Both jobs in the workflow reference this unpinned action. Fix: pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v5`.

Locations:

- `.github/workflows/build.yaml:14`
- `.github/workflows/build.yaml:30`

### script-injection (severity: high)

Sub-rule (a) violation: The `run:` step in the `test-version` job directly interpolates `${{ matrix.just-version }}` inside a shell command string: `run: just --version | grep -E "^just v?${{ matrix.just-version }}$"`. GitHub Actions substitutes the expression before the shell sees it, so a crafted matrix value could inject arbitrary shell commands. Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `env: { JUST_VERSION: "${{ matrix.just-version }}" }` then `run: just --version | grep -E "^just v?${JUST_VERSION}$"`.

Locations:

- `.github/workflows/build.yaml:35`

### missing-permissions (severity: medium)

The workflow file `build.yaml` has no top-level `permissions:` key, and neither the `test-latest` nor the `test-version` job defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository default (typically `write-all` for private repos or broad read access for public repos), violating the principle of least privilege. Fix: add a top-level `permissions: {}` or restrict to the minimum required scopes (e.g. `contents: read`).

Locations:

- `.github/workflows/build.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/build.yaml: (1) Pinned both actions/checkout@v5 references to full SHA fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 # v5. (2) Moved ${{ matrix.just-version }} out of the run: shell string into an env: block as JUST_VERSION, referenced as ${JUST_VERSION} in the grep command. (3) Added top-level 'permissions: {}' to enforce least privilege.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Check version' step in .github/workflows/build.yaml by converting the run command from a YAML double-quoted flow scalar to a YAML block scalar (|). In the original flow scalar, the inner double quotes were YAML-escaped (\"...\"), so the shell received the grep pattern without shell quoting, leaving ${JUST_VERSION} unquoted. With the block scalar, the double quotes are literal shell double quotes, so ${JUST_VERSION} is now properly double-quoted in the shell command, preventing word-splitting and glob expansion.

