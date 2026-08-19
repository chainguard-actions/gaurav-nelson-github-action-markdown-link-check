<!-- markdownlint-disable -->

# Hardening Report: gaurav-nelson--github-action-markdown-link-check/1.0.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gaurav-nelson--github-action-markdown-link-check/1.0.16** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/push.yml references actions using mutable branch/tag refs instead of pinned 40-character SHA digests. Specifically: `actions/checkout@master` (branch ref, appears in two jobs) and `bewuethr/shellcheck-action@v2` (tag ref). These can be silently updated by the upstream repository, enabling supply-chain attacks.

Locations:

- `.github/workflows/push.yml:7`
- `.github/workflows/push.yml:14`
- `.github/workflows/push.yml:29`

### missing-permissions (severity: medium)

The workflow file .github/workflows/push.yml has no top-level `permissions:` key, and none of its three jobs (markdown-link-check, markdown-link-check-folders, shellcheck) define job-level `permissions:` blocks. This means the workflow runs with the default (potentially broad) GitHub token permissions.

Locations:

- `.github/workflows/push.yml:1`

### script-injection (severity: high)

Rule (a) violation: The 'display errors' step directly interpolates `${{ steps.markdown-link-check.outputs.MLC_OUTPUT }}` inside a `run:` shell command string. The `steps.*.outputs.*` context is workflow-controllable data — if the step output contains shell metacharacters or newlines, this can lead to command injection. The offending line is: `echo "${{ steps.markdown-link-check.outputs.MLC_OUTPUT }}"`

Locations:

- `.github/workflows/push.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/push.yml: (1) Pinned actions/checkout@master to SHA 61b9e3751b92087fd0b06925ba6dd6314e06f089 (both occurrences) and bewuethr/shellcheck-action@v2 to SHA 80bac2daa9fcf95d648200a793d00060857e6dc4, preserving original refs as comments. (2) Added top-level `permissions: {}` block to restrict default token permissions. (3) Moved `${{ steps.markdown-link-check.outputs.MLC_OUTPUT }}` from the run: shell string into an env: block as MLC_OUTPUT, then referenced it as $MLC_OUTPUT in the shell script to prevent script injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the $GITHUB_ENV injection vulnerability in entrypoint.sh's check_errors() function (line 88). The vulnerable heredoc-style write (`echo "MLC_OUTPUT<<EOF" >> "$GITHUB_ENV"` / `cat error.txt >> "$GITHUB_ENV"` / `echo "EOF" >> "$GITHUB_ENV"`) was replaced with a sanitized single-line write: `safe_output=$(printf '%s' "$(cat error.txt)" | tr -d '\n\r')` followed by `echo "MLC_OUTPUT=${safe_output}" >> "$GITHUB_ENV"`. This strips all newlines and carriage returns from the user-controlled error.txt content before writing to $GITHUB_ENV, preventing attackers from injecting additional environment variable assignments via crafted markdown files containing "EOF" or newline sequences.

