# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string. Per the script-injection check, all `github.*` expressions must be assigned to an environment variable first and then referenced as `$ENV_VAR` in the run step — not interpolated directly. An attacker who can influence `github.action_path` (or who exploits this pattern in a broader context) could inject arbitrary shell commands. The safe pattern is to assign `${{ github.action_path }}` to an `env:` variable and reference that variable in the `run:` block.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml line 22: moved `${{ github.action_path }}` from the `run:` block into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, and updated the `run:` command to reference it as `$ACTION_PATH/deploy.sh` instead of directly interpolating the GitHub Actions expression.

