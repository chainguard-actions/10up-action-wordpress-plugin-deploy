# Hardening Report: 10up--action-wordpress-plugin-deploy/2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` field directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string (`run: ${{ github.action_path }}/deploy.sh`). Per security best practice, GitHub context expressions must not be interpolated directly into `run:` blocks; they should be assigned to an environment variable via `env:` and referenced as `$ENV_VAR` in the shell command instead. Although `github.action_path` is typically system-controlled, this pattern is still a violation of the script-injection check.

Locations:

- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml line 21: moved `${{ github.action_path }}` out of the `run:` field and into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The shell command now uses `"$ACTION_PATH/deploy.sh"` instead of `${{ github.action_path }}/deploy.sh`.

