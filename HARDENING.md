# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. The `github.action_path` expression is a GitHub context value interpolated directly into the run command rather than being assigned to an environment variable first. This allows potential script injection if the value is attacker-influenced.

Locations:

- `action.yml:24`

### github-env-injection (severity: high)

In deploy.sh, the variable `SLUG` is derived from the GitHub context value `GITHUB_REPOSITORY` (via `SLUG=${GITHUB_REPOSITORY#*/}`), which is an attacker-influenced value. This variable is then written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. An attacker who can control the repository name could inject newlines to poison the GitHub output environment.

Locations:

- `deploy.sh:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. action.yml (line 24): Moved `${{ github.action_path }}` out of the `run:` shell string into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The run command now uses `"$ACTION_PATH/deploy.sh"` — a plain shell variable reference, preventing script injection. 2. deploy.sh (line 155): Added sanitization of the `SLUG` variable before writing to `$GITHUB_OUTPUT`. A new `safe_slug` variable is computed via `printf '%s' "$SLUG" | tr -d '\n\r'` to strip any embedded newlines, and `safe_slug` is used in the echo statement instead of the raw `SLUG`.

