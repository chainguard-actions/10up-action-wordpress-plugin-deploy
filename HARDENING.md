# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside the shell command string (`run: ${{ github.action_path }}/deploy.sh`). Any `github.*` expression interpolated directly in a `run:` block is a script injection risk. The safe pattern is to assign the value to an environment variable and reference it as `$ENV_VAR` in the shell command.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml line 22: moved `${{ github.action_path }}` out of the `run:` shell string and into the step's `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The shell command now uses `"$ACTION_PATH/deploy.sh"` instead of `${{ github.action_path }}/deploy.sh`.

