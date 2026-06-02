# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. The `github.*` context value should be assigned to an environment variable first and then referenced as `$ENV_VAR` in the run step, rather than being interpolated directly into the shell command.

Locations:

- `action.yml:24`

### github-env-injection (severity: high)

In deploy.sh (invoked from action.yml's run: step), the `SLUG` variable is derived from `GITHUB_REPOSITORY` (a `github.*` context value: `SLUG=${GITHUB_REPOSITORY#*/}`). This value is then written unsanitized to `$GITHUB_OUTPUT` via: `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. No sanitization step (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing a repository name containing newlines to inject arbitrary entries into the GitHub output environment file.

Locations:

- `deploy.sh:161`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. action.yml (line 24): Moved `${{ github.action_path }}` out of the `run:` shell string into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, then changed the run command to `"$ACTION_PATH/deploy.sh"` to prevent script injection. 2. deploy.sh (line 161): Added `safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')` before the `$GITHUB_OUTPUT` write, and used `${safe_slug}` instead of `${SLUG}` in the echo statement to prevent newline-based environment injection attacks.

