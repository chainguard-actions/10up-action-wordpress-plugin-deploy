# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates `${{ github.action_path }}` into the shell command string (`run: ${{ github.action_path }}/deploy.sh`). A `github.*` context expression is interpolated directly in a `run:` block instead of being assigned to an environment variable first and referenced as `$ENV_VAR`. The safe pattern would be to set `ACTION_PATH: ${{ github.action_path }}` in the `env:` block and then use `run: "$ACTION_PATH/deploy.sh"` in the shell command.

Locations:

- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 24: moved `${{ github.action_path }}` from the `run:` block into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, and updated the shell command to `run: "$ACTION_PATH/deploy.sh"`. This follows the safe pattern of never interpolating GitHub context expressions directly into shell command strings.

