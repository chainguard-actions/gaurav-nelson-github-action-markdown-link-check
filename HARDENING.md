<!-- markdownlint-disable -->

# Hardening Report: gaurav-nelson--github-action-markdown-link-check/1.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gaurav-nelson--github-action-markdown-link-check/1.0.13** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in .github/workflows/push.yml are pinned to mutable refs instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks:
- `actions/checkout@master` (branch ref)
- `bewuethr/shellcheck-action@v2` (tag ref)
These should be pinned to their full SHA digests, e.g. `actions/checkout@<40-char-sha> # master`.

Locations:

- `.github/workflows/push.yml:6`
- `.github/workflows/push.yml:13`
- `.github/workflows/push.yml:22`

### missing-permissions (severity: medium)

.github/workflows/push.yml has no top-level `permissions:` key and none of its three jobs (markdown-link-check, markdown-link-check-folders, shellcheck) define job-level `permissions:` blocks. This means the workflow runs with the default, overly broad token permissions. A minimal permissions block (e.g. `permissions: read-all` or specific scopes) should be added.

Locations:

- `.github/workflows/push.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed .github/workflows/push.yml: (1) Pinned actions/checkout@master to @61b9e3751b92087fd0b06925ba6dd6314e06f089 # master (applied to both jobs using it), and pinned bewuethr/shellcheck-action@v2 to @80bac2daa9fcf95d648200a793d00060857e6dc4 # v2. (2) Added top-level `permissions: contents: read` block to restrict the workflow token to the minimum required permissions.

