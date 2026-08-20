<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-just/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-just/v2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file references actions by mutable tag rather than a full 40-character commit SHA. If the tag is moved (intentionally or by a supply-chain attack), the workflow will silently execute different code. Failing references: `actions/checkout@v4`, `actions/setup-node@v4`.

Locations:

- `.github/workflows/build.yaml:11`
- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:24`
- `.github/workflows/build.yaml:38`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs (build, test-latest, test-version) define job-level `permissions:` blocks. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build.yaml:1`

### script-injection (severity: high)

Rule (a) violation: A `run:` block in the `test-version` job directly interpolates a `${{ ... }}` expression into a shell command string. The line `run: just --version | grep -E "^just v?${{ matrix.just-version }}$"` embeds `${{ matrix.just-version }}` directly in the shell command. Although `matrix.*` values are typically developer-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. The value should be passed via an `env:` variable and referenced as a quoted shell variable (e.g. `"$JUST_VERSION"`).

Locations:

- `.github/workflows/build.yaml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/build.yaml: (1) Pinned all four `uses:` references to full commit SHAs — actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (3 occurrences) and actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 (1 occurrence), with original tags preserved as comments. (2) Added `permissions: {}` at the workflow top level to enforce least privilege. (3) Moved `${{ matrix.just-version }}` in the 'Check version' step out of the `run:` shell string into an `env:` block as `JUST_VERSION`, then referenced it as `${JUST_VERSION}` in the grep command.

