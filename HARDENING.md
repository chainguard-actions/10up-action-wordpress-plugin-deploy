# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string (`run: ${{ github.action_path }}/deploy.sh`). Per the check rules, `github.*` expressions must be assigned to an `env:` variable first and then referenced as `$ENV_VAR` in the run step. While `github.action_path` is typically a trusted path, it is still a `github.*` context value interpolated directly into the shell command rather than being safely passed through an environment variable.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 22: moved `${{ github.action_path }}` out of the `run:` shell string and into the step's `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The run command now uses `"$ACTION_PATH/deploy.sh"` instead of `${{ github.action_path }}/deploy.sh`, ensuring the GitHub context expression is not directly interpolated into the shell command string.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in deploy.sh at line 65 (generate_zip function). The SLUG variable, derived from GITHUB_REPOSITORY (attacker-controllable), was being written directly to $GITHUB_OUTPUT without sanitization. Added a sanitization step using `safe_slug=$(printf '%s' "${SLUG}" | tr -d '\n\r')` and replaced the direct use of `${SLUG}` with `${safe_slug}` in the echo statement that writes to $GITHUB_OUTPUT. This prevents newline injection attacks where a malicious repository name could inject arbitrary key-value pairs into the GitHub output environment.

