<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-just/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-just/v1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file references actions by mutable version tags instead of full 40-character commit SHAs. If the tag is moved or the repository is compromised, the action could execute arbitrary code. Failing references: `actions/checkout@v3` (lines 11, 33, 56) and `actions/setup-node@v3` (line 12). These must be pinned to their full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/build.yaml:11`
- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:33`
- `.github/workflows/build.yaml:56`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and none of the three jobs (`build`, `test-latest`, `test-version`) define their own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or per job.

Locations:

- `.github/workflows/build.yaml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In the `test-version` job's "Check version" step, `${{ matrix.just-version }}` is embedded directly in the shell command: `run: just --version | grep -E "^just v?${{ matrix.just-version }}$"`. Although `matrix.just-version` values are defined in the workflow itself, any `${{ ... }}` inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it — an attacker who can influence matrix values (e.g. via a fork PR or workflow_dispatch) could inject shell metacharacters. The value should be passed via an `env:` variable and referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/build.yaml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/build.yaml: (1) Pinned actions/checkout@v3 to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 at all three occurrences (lines 11, 33, 56) and actions/setup-node@v3 to SHA 3235b876344d2a9aa001b8d1453c930bba69e610 (line 12), preserving the original tag as a comment. (2) Added a top-level `permissions: contents: read` block to restrict the default token permissions. (3) Moved `${{ matrix.just-version }}` out of the `run:` shell command in the 'Check version' step into an `env:` block as `JUST_VERSION`, referencing it as `${JUST_VERSION}` in the shell script to prevent script injection.

