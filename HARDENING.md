<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. Sub-rule (b): deploy.sh uses unquoted expansions of input-derived env vars `$INPUT_DRY_RUN` (line 22: `if $INPUT_DRY_RUN; then`) and `$INPUT_GENERATE_ZIP` (line 64: `if $INPUT_GENERATE_ZIP; then`). These env vars are set from `inputs.dry-run` and `inputs.generate-zip` respectively in action.yml's `env:` block. Unquoted expansion of workflow-controllable env vars allows shell metacharacter injection.

Locations:

- `action.yml:25`
- `deploy.sh:22`
- `deploy.sh:64`

### github-env-injection (severity: high)

In deploy.sh, the `generate_zip()` function writes `zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip` to `$GITHUB_OUTPUT` without sanitization. `SLUG` is derived from the inherited process env var `$GITHUB_REPOSITORY` (set by the calling workflow, not computed in the same run block), and `GITHUB_WORKSPACE` is also an inherited env var. Neither value is passed through `printf '%s' ... | tr -d '\n\r'` before the write, allowing a newline-injection attack that could poison subsequent `$GITHUB_OUTPUT` entries.

Locations:

- `deploy.sh:71`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed four issues across action.yml and deploy.sh:
1. action.yml: Replaced `${{ github.action_path }}/deploy.sh` with `$GITHUB_ACTION_PATH/deploy.sh` to eliminate template interpolation before shell execution.
2. deploy.sh (line 22): Changed `if $INPUT_DRY_RUN; then` to `if [[ "$INPUT_DRY_RUN" == "true" ]]; then` to prevent shell metacharacter injection from the unquoted boolean env var.
3. deploy.sh (line 64): Changed `if $INPUT_GENERATE_ZIP; then` to `if [[ "$INPUT_GENERATE_ZIP" == "true" ]]; then` for the same reason.
4. deploy.sh (line 71): Added sanitization of the zip path before writing to $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'` to prevent newline injection attacks.

