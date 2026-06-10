<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In action.yml, the `run:` block directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. The value should be passed via an env var and referenced as `"$ACTION_PATH"/deploy.sh` instead.

Locations:

- `action.yml:21`

### script-injection (severity: high)

Sub-rule (b): In deploy.sh, the env var `INPUT_GENERATE_ZIP` (sourced from `${{ inputs.generate-zip }}` set in action.yml) is used in an unquoted shell expansion: `if $INPUT_GENERATE_ZIP; then`. An attacker-controlled input value containing shell metacharacters (e.g. `; malicious-command #`) could cause command injection. The variable should be double-quoted: `if "$INPUT_GENERATE_ZIP"; then`.

Locations:

- `deploy.sh:161`
- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection issues: (a) In action.yml, moved `${{ github.action_path }}` from the `run:` block into an `ACTION_PATH` env var and updated the run command to `"$ACTION_PATH/deploy.sh"`. (b) In deploy.sh line 161, added double quotes around `$INPUT_GENERATE_ZIP` in the `if` condition: `if "$INPUT_GENERATE_ZIP"; then`.

