# Hardening Report: 10up--action-wordpress-plugin-deploy/2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string (line 20: `run: ${{ github.action_path }}/deploy.sh`). Although `github.action_path` is set by GitHub's infrastructure and is not directly user-supplied, it is still a `github.*` context value interpolated inline into a `run:` block rather than being passed through an environment variable. The safe pattern is to assign it to an env var first (e.g., `ACTION_PATH: ${{ github.action_path }}`) and then reference `$ACTION_PATH/deploy.sh` in the run block.

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 20: moved `${{ github.action_path }}` out of the `run:` block into the step's `env:` block as `ACTION_PATH: ${{ github.action_path }}`, then updated the run command to reference `$ACTION_PATH/deploy.sh` instead of the inline expression.

