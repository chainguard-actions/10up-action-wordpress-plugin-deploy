# Hardening Report: 10up--action-wordpress-plugin-deploy/2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **10up--action-wordpress-plugin-deploy/2.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Per security best practice, github.* expressions must not be interpolated directly into run: blocks; instead, the value should be assigned to an environment variable (e.g., `ACTION_PATH: ${{ github.action_path }}`) and referenced as `$ACTION_PATH` in the shell command.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 22: moved `${{ github.action_path }}` out of the `run:` block and into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The shell command now uses `$ACTION_PATH/deploy.sh` instead of directly interpolating the GitHub Actions expression.

