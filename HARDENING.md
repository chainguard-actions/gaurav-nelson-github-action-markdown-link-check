<!-- markdownlint-disable -->

# Hardening Report: gaurav-nelson--github-action-markdown-link-check/1.0.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gaurav-nelson--github-action-markdown-link-check/1.0.17** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in .github/workflows/push.yml are pinned to mutable branch or tag refs instead of full 40-character commit SHAs. Specifically: `actions/checkout@master` (branch ref, appears twice) and `bewuethr/shellcheck-action@v2` (tag ref). A supply-chain attacker who pushes to those refs can inject arbitrary code into the workflow.

Locations:

- `.github/workflows/push.yml:7`
- `.github/workflows/push.yml:20`
- `.github/workflows/push.yml:29`

### permissions (severity: medium)

The workflow file .github/workflows/push.yml has no top-level `permissions:` key and none of its three jobs (`markdown-link-check`, `markdown-link-check-folders`, `shellcheck`) define job-level `permissions:` blocks. This means the workflow runs with the default, overly-broad token permissions.

Locations:

- `.github/workflows/push.yml:1`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a `run:` shell command string. In the `display errors` step, the line `echo "${{ steps.markdown-link-check.outputs.MLC_OUTPUT }}"` injects the step output value directly into the shell before quoting occurs. If the output contains shell metacharacters (`;`, `|`, `$(...)`, etc.) they will be interpreted by the shell, enabling command injection.

Locations:

- `.github/workflows/push.yml:15`

### github-env-injection (severity: high)

In entrypoint.sh, the `check_errors` function writes the raw contents of `error.txt` directly to `$GITHUB_ENV` using a heredoc pattern (`echo "MLC_OUTPUT<<EOF" >> "$GITHUB_ENV"` followed by `cat error.txt >> "$GITHUB_ENV"`). The file `error.txt` contains output from markdown-link-check run against user-controlled markdown files and external URLs, and is never sanitized with `printf '%s' ... | tr -d '\n\r'` before being written. A malicious markdown file or URL response containing a newline followed by `KEY=value` could inject arbitrary environment variables into subsequent workflow steps.

Locations:

- `entrypoint.sh:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings in push.yml and entrypoint.sh:
1. unpinned-uses: Pinned actions/checkout@master → @61b9e3751b92087fd0b06925ba6dd6314e06f089 (both occurrences) and bewuethr/shellcheck-action@v2 → @80bac2daa9fcf95d648200a793d00060857e6dc4.
2. permissions: Added top-level `permissions: {}` and job-level `permissions: contents: read` for all three jobs (markdown-link-check, markdown-link-check-folders, shellcheck).
3. script-injection: Moved `${{ steps.markdown-link-check.outputs.MLC_OUTPUT }}` into the step's `env:` block as MLC_OUTPUT and referenced it as `$MLC_OUTPUT` in the shell script.
4. github-env-injection: Replaced the unsafe heredoc write of error.txt to $GITHUB_ENV with a safe write to $GITHUB_OUTPUT using a cryptographically random delimiter (openssl rand -hex 16), preventing newline-based environment variable injection from user-controlled markdown content.

