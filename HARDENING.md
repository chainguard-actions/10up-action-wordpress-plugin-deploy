# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the run: block directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Although `github.action_path` is generally considered low-risk (it resolves to the action's own directory), it is still a `github.*` context value interpolated directly into a run: block rather than being assigned to an environment variable first. The safe pattern is to assign it to an env var (e.g., `ACTION_PATH: ${{ github.action_path }}`) and then reference `$ACTION_PATH/deploy.sh` in the run: block.

Locations:

- `action.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 24: moved `${{ github.action_path }}` out of the `run:` block and into the step's `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The `run:` block now uses the plain shell variable `$ACTION_PATH/deploy.sh` instead of directly interpolating the GitHub Actions expression.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in deploy.sh at line 78. In the `generate_zip` function, the SLUG variable (derived from GITHUB_REPOSITORY, an attacker-influenced value) was written directly to $GITHUB_OUTPUT without sanitization. Added a sanitization step: `safe_slug=$(printf '%s' "${SLUG}" | tr -d '\n\r')` and replaced the raw `${SLUG}` reference in the echo statement with `${safe_slug}` to prevent newline injection attacks.

