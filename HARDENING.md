# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string (`run: ${{ github.action_path }}/deploy.sh`). Per security best practice, all `github.*` context values should be passed via an `env:` variable and referenced as `$ENV_VAR` in the shell command, rather than being interpolated directly into the `run:` string. Although `github.action_path` is an internal context value, direct interpolation of any `github.*` expression into a `run:` block is a script-injection risk.

Locations:

- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 24: moved `${{ github.action_path }}` from the `run:` block into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, and updated the run command to `"$ACTION_PATH/deploy.sh"` so the shell references the plain environment variable instead of directly interpolating the GitHub Actions expression.

