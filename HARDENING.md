# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Per the script-injection check, any `github.*` expression interpolated directly in a `run:` block (rather than being assigned to an environment variable first and referenced as `$ENV_VAR`) is a finding. The safe pattern would be to set `DEPLOY_SCRIPT: ${{ github.action_path }}/deploy.sh` in the `env:` block and then use `run: "$DEPLOY_SCRIPT"`.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 22: moved `${{ github.action_path }}` out of the `run:` block and into the `env:` block as `DEPLOY_SCRIPT: ${{ github.action_path }}/deploy.sh`. The `run:` block now uses `"$DEPLOY_SCRIPT"` instead of directly interpolating the GitHub Actions expression.

