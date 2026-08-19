<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in action.yml. Before the shell ever sees the command, YAML template substitution inserts the raw value, enabling script injection. Offending line: `run: ${{ github.action_path }}/deploy.sh`

Locations:

- `action.yml:22`

### script-injection (severity: high)

Sub-rule (a): `${{ steps.title.outputs.result }}` is interpolated directly inside a `run:` shell command string. The step output is derived from `GITHUB_REF` (a workflow-controlled value) and is injected unsanitized into the shell command `gh pr create --title "${{ steps.title.outputs.result }}"`, enabling script injection. Offending line: `run: gh pr create --title "${{ steps.title.outputs.result }}" --body-file ./.github/release-pull-request-template.md`

Locations:

- `.github/workflows/release-pull-request.yml:18`

### github-env-injection (severity: high)

In deploy.sh (called from action.yml), the variable `SLUG` is derived from the inherited process env var `GITHUB_REPOSITORY` (a workflow-controlled value: `SLUG=${GITHUB_REPOSITORY#*/}`). It is then written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. An attacker-controlled repository name containing newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT.

Locations:

- `deploy.sh:75`

### github-env-injection (severity: high)

In release-pull-request.yml, the variable `VERSION` is derived from `GITHUB_REF` (a workflow-controlled env var: `VERSION=${BRANCH#'release/'}`). It is then written to `$GITHUB_OUTPUT` without the required sanitization step: `echo "result=Release: ${VERSION}" >> "${GITHUB_OUTPUT}"`. A branch name containing newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/release-pull-request.yml:16`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is moved: `lee-dohm/no-response@v0.5.0` (tag).

Locations:

- `.github/workflows/no-response.yml:17`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs: `actions/checkout@v3` (tag).

Locations:

- `.github/workflows/release-pull-request.yml:11`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to mutable branch names instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks: `actions/checkout@master` (branch) and `ludeeus/action-shellcheck@master` (branch).

Locations:

- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:16`

### permissions (severity: medium)

missing-permissions: The workflow has no top-level `permissions:` key and the single job `noResponse` also has no `permissions:` key. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/no-response.yml:1`

### permissions (severity: medium)

missing-permissions: The workflow has no top-level `permissions:` key and the single job `release-pull-request-automation` also has no `permissions:` key. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/release-pull-request.yml:1`

### permissions (severity: medium)

missing-permissions: The workflow has no top-level `permissions:` key and the single job `shellcheck` also has no `permissions:` key. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, permissions

**Notes:**

Fixed all 10 findings across 4 files:

1. action.yml (script-injection): Moved `${{ github.action_path }}` into env var `ACTION_PATH` and referenced it as `$ACTION_PATH/deploy.sh` in the run command.

2. deploy.sh (github-env-injection): Sanitized SLUG before writing to GITHUB_OUTPUT: `safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')` then used `$safe_slug` in the echo.

3. release-pull-request.yml (script-injection + github-env-injection + unpinned-uses + missing-permissions): Pinned actions/checkout@v3 to full SHA; added `permissions: contents: read, pull-requests: write`; sanitized VERSION before writing to GITHUB_OUTPUT; moved `${{ steps.title.outputs.result }}` into env var `PR_TITLE`.

4. no-response.yml (unpinned-uses + missing-permissions): Pinned lee-dohm/no-response@v0.5.0 to full SHA 9bb0a4b5e6a45046f00353d5de7d90fb8bd773bb; added `permissions: issues: write`.

5. test.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@master to SHA 61b9e3751b92087fd0b06925ba6dd6314e06f089 and ludeeus/action-shellcheck@master to SHA 00b27aa7cb85167568cb48a3838b75f4265f2bca; added `permissions: contents: read`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three instances of script injection in deploy.sh where INPUT_DRY_RUN and INPUT_GENERATE_ZIP were used directly as shell commands via `if $VAR; then`. Replaced all three with safe string comparisons: `if [[ "$INPUT_DRY_RUN" == "true" ]]; then` (lines 22 and 183) and `if [[ "$INPUT_GENERATE_ZIP" == "true" ]]; then` (inside the generate_zip function). This prevents attacker-controlled input values from being executed as arbitrary shell commands.

