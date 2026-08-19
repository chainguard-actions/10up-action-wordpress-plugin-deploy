<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.1.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` field in action.yml directly interpolates a GitHub Actions expression `${{ github.action_path }}` inside the shell command string: `run: ${{ github.action_path }}/deploy.sh`. Any `${{ ... }}` expression interpolated directly into a run: block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead: `run: "$GITHUB_ACTION_PATH/deploy.sh"`

Locations:

- `action.yml:19`

### script-injection (severity: high)

Sub-rule (b): `inputs.generate-zip` is routed through the env var `INPUT_GENERATE_ZIP` (set via `env: INPUT_GENERATE_ZIP: ${{ inputs.generate-zip }}`), but in deploy.sh it is expanded completely unquoted as `if $INPUT_GENERATE_ZIP; then`. An unquoted shell expansion of a workflow-controllable value allows shell metacharacter injection (e.g. a value like `true; malicious-command`). The expansion must be double-quoted: `if "$INPUT_GENERATE_ZIP"; then`.

Locations:

- `action.yml:17`
- `deploy.sh:119`

### github-env-injection (severity: high)

deploy.sh writes the inherited env var `SLUG` (derived from `GITHUB_REPOSITORY`, which is set by the calling workflow) to the legacy `::set-output` special command without sanitization: `echo "::set-output name=zip-path::${GITHUB_WORKSPACE}/${SLUG}.zip"`. A newline embedded in `SLUG` or `GITHUB_WORKSPACE` could inject additional output commands. The value must be sanitized with `printf '%s' "$SLUG" | tr -d '\n\r'` before being written. The same applies to the `GITHUB_WORKSPACE` component.

Locations:

- `deploy.sh:120`

### unpinned-uses (severity: high)

Both workflow files reference actions by mutable tag or branch name instead of a pinned 40-character commit SHA, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised.

- `.github/workflows/no-response.yml`: `uses: lee-dohm/no-response@v0.5.0` (tag)
- `.github/workflows/test.yml`: `uses: actions/checkout@master` (branch)
- `.github/workflows/test.yml`: `uses: ludeeus/action-shellcheck@master` (branch)

All three should be pinned to their full 40-character commit SHA.

Locations:

- `.github/workflows/no-response.yml:17`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:16`

### missing-permissions (severity: medium)

`.github/workflows/no-response.yml` has no top-level `permissions:` key and the `noResponse` job also has no job-level `permissions:` key. Without explicit permissions the workflow inherits the repository's default token permissions, which may be broader than necessary. A minimal permissions block (e.g. `permissions: issues: write`) should be added.

Locations:

- `.github/workflows/no-response.yml:1`

### missing-permissions (severity: medium)

`.github/workflows/test.yml` has no top-level `permissions:` key and the `shellcheck` job also has no job-level `permissions:` key. Without explicit permissions the workflow inherits the repository's default token permissions. A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings: (1) action.yml: replaced `${{ github.action_path }}/deploy.sh` with `"$GITHUB_ACTION_PATH/deploy.sh"` to eliminate template injection; (2) deploy.sh line 119: quoted `$INPUT_GENERATE_ZIP` as `"$INPUT_GENERATE_ZIP"` to prevent shell metacharacter injection; (3) deploy.sh line 120: sanitized GITHUB_WORKSPACE and SLUG with `printf '%s' | tr -d '\n\r'` before writing to `::set-output`; (4) pinned lee-dohm/no-response@v0.5.0 to SHA 9bb0a4b5e6a45046f00353d5de7d90fb8bd773bb, actions/checkout@master to SHA 61b9e3751b92087fd0b06925ba6dd6314e06f089, and ludeeus/action-shellcheck@master to SHA 00b27aa7cb85167568cb48a3838b75f4265f2bca; (5) added `permissions: issues: write` to no-response.yml; (6) added `permissions: contents: read` to test.yml.

