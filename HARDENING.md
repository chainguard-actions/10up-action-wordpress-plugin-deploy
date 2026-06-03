# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` block directly interpolates the GitHub Actions expression `${{ github.action_path }}` into the shell command string. The PASS criterion requires that github.* context values be assigned to environment variables first and then referenced as $ENV_VAR in the run step. Instead, the run command is: `run: ${{ github.action_path }}/deploy.sh`, which embeds the expression directly in the shell command.

Locations:

- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yml line 25: moved `${{ github.action_path }}` out of the `run:` shell string and into the step's `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The `run:` command now references it as the plain environment variable `$ACTION_PATH/deploy.sh`.

