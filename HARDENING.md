# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the run: field directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Per the check rules, all `github.*` expressions interpolated directly into a `run:` block are flagged as script injection. The safe pattern is to assign the value to an environment variable first (e.g., `ACTION_PATH: ${{ github.action_path }}`) and then reference `$ACTION_PATH` in the run block.

Locations:

- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml line 23: moved `${{ github.action_path }}` out of the `run:` shell command and into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The run command now uses `$ACTION_PATH/deploy.sh` instead of `${{ github.action_path }}/deploy.sh`.

