<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in action.yml. The line `run: ${{ github.action_path }}/deploy.sh` causes YAML template substitution to inject the value before the shell ever sees it, enabling script injection if the value is attacker-influenced.

Locations:

- `action.yml:20`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ steps.title.outputs.result }}` is interpolated directly inside a `run:` shell command string. The line `run: gh pr create --title "${{ steps.title.outputs.result }}" ...` injects a workflow-controllable `steps.*.outputs.*` value directly into the shell command, enabling script injection.

Locations:

- `.github/workflows/release-pull-request.yml:16`

### github-env-injection (severity: high)

In deploy.sh (called by action.yml), the variable `$SLUG` is derived from the inherited environment variable `$GITHUB_REPOSITORY` (set by the calling workflow) and written unsanitized to `$GITHUB_OUTPUT`: `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into the output file.

Locations:

- `deploy.sh:76`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved:
- no-response.yml: `lee-dohm/no-response@v0.5.0` (tag)
- release-pull-request.yml: `actions/checkout@v3` (tag)
- test.yml: `actions/checkout@master` (branch), `ludeeus/action-shellcheck@master` (branch)

Locations:

- `.github/workflows/no-response.yml:17`
- `.github/workflows/release-pull-request.yml:9`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job in any of these files defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files are affected: no-response.yml, release-pull-request.yml, and test.yml.

Locations:

- `.github/workflows/no-response.yml:1`
- `.github/workflows/release-pull-request.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings: (1) action.yml script-injection: moved `${{ github.action_path }}` to env var ACTION_PATH; (2) release-pull-request.yml script-injection: moved `${{ steps.title.outputs.result }}` to env var PR_TITLE; (3) deploy.sh github-env-injection: sanitized SLUG and GITHUB_WORKSPACE with tr -d '\n\r' before writing to GITHUB_OUTPUT; (4) unpinned-uses: pinned lee-dohm/no-response@v0.5.0 to SHA 9bb0a4b5e6a45046f00353d5de7d90fb8bd773bb, actions/checkout@v3 to a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/checkout@master to 61b9e3751b92087fd0b06925ba6dd6314e06f089, ludeeus/action-shellcheck@master to 00b27aa7cb85167568cb48a3838b75f4265f2bca; (5) missing-permissions: added `permissions: issues: write` to no-response.yml, `permissions: contents: read, pull-requests: write` to release-pull-request.yml, and `permissions: contents: read` to test.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 3 unquoted shell variable expansions in deploy.sh: changed `if $INPUT_DRY_RUN;` (lines 22 and 196) and `if $INPUT_GENERATE_ZIP;` (line 64) to use double-quoted forms `if "$INPUT_DRY_RUN";` and `if "$INPUT_GENERATE_ZIP";` to prevent command substitution/word-splitting injection. Fixed GITHUB_OUTPUT injection in release-pull-request.yml by sanitizing the VERSION variable with `printf '%s' "$VERSION" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in the 'Generate title' step of .github/workflows/release-pull-request.yml. Changed `echo $BRANCH` to `echo "$BRANCH"` to prevent shell metacharacter interpretation from the workflow-controllable GITHUB_REF value.

