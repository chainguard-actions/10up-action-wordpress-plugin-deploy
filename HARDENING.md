<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 9 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a `run:` shell command string. The line `run: ${{ github.action_path }}/deploy.sh` embeds `github.action_path` directly into the shell command before the shell ever sees it. While `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules. It should be replaced with the equivalent env var `$GITHUB_ACTION_PATH`.

Locations:

- `action.yml:22`

### script-injection (severity: high)

Sub-rule (a): A `${{ steps.title.outputs.result }}` expression is directly interpolated inside a `run:` shell command string: `gh pr create --title "${{ steps.title.outputs.result }}"`. The `steps.*.outputs.*` context is workflow-controllable and flows through YAML template substitution before the shell quotes it, enabling command injection. The value should be passed via an `env:` variable and double-quoted in the shell.

Locations:

- `.github/workflows/release-pull-request.yml:19`

### github-env-injection (severity: high)

In deploy.sh, the line `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"` writes the value of `$SLUG` to `$GITHUB_OUTPUT` without sanitization. `$SLUG` is derived from the inherited env var `$GITHUB_REPOSITORY` (set by the calling workflow) via `SLUG=${GITHUB_REPOSITORY#*/}`. Because `$GITHUB_REPOSITORY` is a workflow-controlled value, a newline embedded in it could inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$SLUG" | tr -d '\n\r'`) is missing before the write.

Locations:

- `deploy.sh:163`

### unpinned-uses (severity: high)

Two `uses:` references in test.yml use mutable branch refs instead of pinned 40-character commit SHAs: `actions/checkout@master` and `ludeeus/action-shellcheck@master`. These can be silently updated to include malicious code at any time.

Locations:

- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:16`

### unpinned-uses (severity: high)

The `uses:` reference `lee-dohm/no-response@v0.5.0` in no-response.yml uses a mutable tag ref instead of a pinned 40-character commit SHA. Tags can be force-pushed to point to different commits, enabling supply-chain attacks.

Locations:

- `.github/workflows/no-response.yml:20`

### unpinned-uses (severity: high)

The `uses:` reference `actions/checkout@v3` in release-pull-request.yml uses a mutable tag ref instead of a pinned 40-character commit SHA. Tags can be force-pushed to point to different commits, enabling supply-chain attacks.

Locations:

- `.github/workflows/release-pull-request.yml:11`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and the `shellcheck` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

The workflow file no-response.yml has no top-level `permissions:` key and the `noResponse` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/no-response.yml:1`

### missing-permissions (severity: medium)

The workflow file release-pull-request.yml has no top-level `permissions:` key and the `release-pull-request-automation` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/release-pull-request.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 9 findings across 5 files:

1. action.yml (script-injection): Replaced `${{ github.action_path }}/deploy.sh` with `$GITHUB_ACTION_PATH/deploy.sh` to avoid template interpolation in run: blocks.

2. deploy.sh (github-env-injection): Added `SAFE_SLUG=$(printf '%s' "$SLUG" | tr -d '\n\r')` before writing to GITHUB_OUTPUT to strip newlines that could inject additional key=value pairs.

3. release-pull-request.yml (script-injection): Moved `${{ steps.title.outputs.result }}` out of the `run:` shell string into an `env:` block as `PR_TITLE`, then referenced it as `"$PR_TITLE"` in the shell command.

4. test.yml (unpinned-uses + missing-permissions): Pinned `actions/checkout@master` → SHA `61b9e375...` and `ludeeus/action-shellcheck@master` → SHA `00b27aa7...`. Added `permissions: {}` at top level.

5. no-response.yml (unpinned-uses + missing-permissions): Pinned `lee-dohm/no-response@v0.5.0` → SHA `9bb0a4b5...`. Added `permissions: {}` at top level.

6. release-pull-request.yml (unpinned-uses + missing-permissions): Pinned `actions/checkout@v3` → SHA `a37ce912...`. Added `permissions: {}` at top level and job-level `contents: read` + `pull-requests: write` for the job that creates PRs.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Generate title' step of .github/workflows/release-pull-request.yml, added sanitization of the VERSION variable before writing to GITHUB_OUTPUT. Added `safe=$(printf '%s' "$VERSION" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${VERSION}`. This prevents newline injection attacks via crafted branch names.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three instances of script injection in deploy.sh where $INPUT_DRY_RUN (lines ~22 and ~164) and $INPUT_GENERATE_ZIP (line ~170) were used directly as shell commands via `if $VAR; then`. Replaced all three with safe string comparisons: `if [[ "$INPUT_DRY_RUN" == 'true' ]]; then` and `if [[ "$INPUT_GENERATE_ZIP" == 'true' ]]; then`. This prevents an attacker from supplying a value like `true; malicious_command` to achieve arbitrary command execution.

