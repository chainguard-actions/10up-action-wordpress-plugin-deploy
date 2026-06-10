<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In action.yml, the `run:` field directly interpolates `${{ github.action_path }}` — a GitHub Actions expression — inside the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value.

Sub-rule (b): In deploy.sh, the env vars `$INPUT_DRY_RUN` (sourced from `inputs.dry-run`) and `$INPUT_GENERATE_ZIP` (sourced from `inputs.generate-zip`) are expanded unquoted in shell `if` conditions: `if $INPUT_DRY_RUN;` and `if $INPUT_GENERATE_ZIP;`. Unquoted expansion of workflow-controllable env vars allows shell metacharacter injection.

Locations:

- `action.yml:22`
- `deploy.sh:23`
- `deploy.sh:72`

### github-env-injection (severity: high)

In deploy.sh, the variable `SLUG` is derived from the inherited env var `GITHUB_REPOSITORY` (set by the calling workflow and therefore workflow-controlled/untrusted). It is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. A newline embedded in `SLUG` could inject arbitrary key-value pairs into the GitHub output context.

Locations:

- `deploy.sh:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 4 issues across action.yml and deploy.sh:
1. action.yml: Replaced `${{ github.action_path }}/deploy.sh` with `$GITHUB_ACTION_PATH/deploy.sh` to eliminate template interpolation in run: field.
2. deploy.sh (line 23): Changed `if $INPUT_DRY_RUN;` to `if [[ "$INPUT_DRY_RUN" == "true" ]];` to prevent shell metacharacter injection.
3. deploy.sh (generate_zip function): Changed `if $INPUT_GENERATE_ZIP;` to `if [[ "$INPUT_GENERATE_ZIP" == "true" ]];` for the same reason.
4. deploy.sh (generate_zip function): Added `safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')` and used `safe_slug` when writing to $GITHUB_OUTPUT to prevent newline injection from the workflow-controlled GITHUB_REPOSITORY value.

