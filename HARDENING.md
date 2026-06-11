<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates `${{ github.action_path }}` as a shell command string: `run: ${{ github.action_path }}/deploy.sh`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, allowing an attacker to influence the executed command. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Sub-rule (b): In deploy.sh, the variables `$INPUT_DRY_RUN` and `$INPUT_GENERATE_ZIP` — which are set from `inputs.dry-run` and `inputs.generate-zip` respectively via the action.yml `env:` block — are expanded unquoted in shell conditionals: `if $INPUT_DRY_RUN; then` and `if $INPUT_GENERATE_ZIP; then`. Unquoted expansion of workflow-controllable env vars allows shell metacharacter injection. These should be quoted: `if "$INPUT_DRY_RUN"; then`.

Locations:

- `deploy.sh:47`
- `deploy.sh:91`
- `deploy.sh:163`

### github-env-injection (severity: high)

In deploy.sh, the `SLUG` variable is derived from the inherited process env var `$GITHUB_REPOSITORY` (a workflow-controlled value: `SLUG=${GITHUB_REPOSITORY#*/}`) and then written unsanitized to `$GITHUB_OUTPUT`: `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. A calling workflow could set `GITHUB_REPOSITORY` to a value containing newlines, allowing injection of arbitrary key-value pairs into the GitHub output environment. The value must be sanitized with `printf '%s' "$SLUG" | tr -d '\n\r'` before the write.

Locations:

- `deploy.sh:52`
- `deploy.sh:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings: (1) action.yml: replaced `${{ github.action_path }}/deploy.sh` with `$GITHUB_ACTION_PATH/deploy.sh` to eliminate YAML template injection risk. (2) deploy.sh: quoted all three unquoted boolean variable expansions in shell conditionals (`if $INPUT_DRY_RUN` → `if "$INPUT_DRY_RUN"` at lines 47 and 163, `if $INPUT_GENERATE_ZIP` → `if "$INPUT_GENERATE_ZIP"` at line 91). (3) deploy.sh: sanitized the SLUG variable before writing to $GITHUB_OUTPUT by adding `safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')` and using `$safe_slug` in the zip-path output line, preventing newline injection attacks.

