<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): A ${{ }} expression is interpolated directly inside a `run:` block in action.yml. The line `run: ${{ github.action_path }}/deploy.sh` injects the `github.action_path` context value directly into the shell command string before the shell ever sees it. Even though `github.action_path` is typically GitHub-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value flows through YAML template substitution before shell quoting applies.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansions of env vars sourced from `inputs.*`. In deploy.sh, `INPUT_DRY_RUN` and `INPUT_GENERATE_ZIP` are set from `${{ inputs.dry-run }}` and `${{ inputs.generate-zip }}` respectively (via action.yml env: block), then used unquoted in shell as `if $INPUT_DRY_RUN; then` and `if $INPUT_GENERATE_ZIP; then`. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the input values to be interpreted by the shell, enabling command injection.

Locations:

- `deploy.sh:23`
- `deploy.sh:136`
- `deploy.sh:141`

### github-env-injection (severity: high)

deploy.sh writes `zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip` to `$GITHUB_OUTPUT` without sanitization. `SLUG` is derived from the inherited process env var `$GITHUB_REPOSITORY` (set by the calling workflow, and thus workflow-controlled/untrusted). A calling workflow could set `GITHUB_REPOSITORY` to a value containing newlines, allowing injection of additional key=value pairs into `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$SLUG" | tr -d '\n\r'`) is absent before the write.

Locations:

- `deploy.sh:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Three fixes applied across two files:

1. action.yml: Moved `${{ github.action_path }}` from the `run:` block into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, then referenced it as `"$ACTION_PATH/deploy.sh"` in the shell command, eliminating direct YAML template injection into the shell string.

2. deploy.sh (lines 23, 136, 141 - script-injection): Added sanitization block at the top of the script that normalizes INPUT_DRY_RUN and INPUT_GENERATE_ZIP to literal `true` or `false` shell builtins using `[[ "$VAR" == "true" ]]` comparisons. This prevents shell metacharacters in input values from being interpreted when the variables are used in `if $VAR; then` constructs.

3. deploy.sh (line 148 - github-env-injection): Added `safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')` before writing to GITHUB_OUTPUT, and used `$safe_slug` in the echo statement to strip newlines from the SLUG value (derived from the workflow-controlled GITHUB_REPOSITORY env var), preventing injection of additional key=value pairs into GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 22: changed `run: "$ACTION_PATH/deploy.sh"` to `run: '"$ACTION_PATH/deploy.sh"'`. The YAML single-quoted string ensures the shell receives the command with proper shell-level double quotes around the $ACTION_PATH variable expansion, preventing shell metacharacter interpretation.

