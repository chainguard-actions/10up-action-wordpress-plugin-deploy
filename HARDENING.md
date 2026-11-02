<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the composite step's run: block puts an expression directly into the shell command. Offending line: `run: ${{ github.action_path }}/deploy.sh`. Use the env var instead, quoted: `run: "$GITHUB_ACTION_PATH/deploy.sh"`.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Rule (b), in the deploy.sh script that the run: block calls: workflow inputs are expanded unquoted and run as commands. Line 23 is `if $INPUT_DRY_RUN; then` (from inputs.dry-run) and line 69 is `if $INPUT_GENERATE_ZIP; then` (from inputs.generate-zip). Whatever the input holds is executed as a shell command, so a value like `true; malicious_cmd` runs arbitrary code. Compare the quoted value to a fixed string instead, e.g. `if [[ "$INPUT_DRY_RUN" == "true" ]]`.

Locations:

- `deploy.sh:23`
- `deploy.sh:69`

### github-env-injection (severity: high)

deploy.sh, run by the composite step, writes to $GITHUB_OUTPUT using SLUG, which comes from the calling workflow's env (it falls back to GITHUB_REPOSITORY only when unset), and it is not sanitized first. Offending line: `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. A newline in SLUG could inject extra outputs. Strip newlines just before the write: `safe=$(printf '%s' "${GITHUB_WORKSPACE}/${SLUG}.zip" | tr -d '\n\r')`.

Locations:

- `deploy.sh:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

action.yml: the run step now calls "$GITHUB_ACTION_PATH/deploy.sh" (quoted) instead of the ${{ github.action_path }} expression. deploy.sh: the two `if $INPUT_DRY_RUN` checks and the one `if $INPUT_GENERATE_ZIP` check no longer run the input as a command; they compare it to the string "true" instead ("false" and the default behave as before). The zip-path value written to GITHUB_OUTPUT now has newlines removed before the write.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

deploy.sh generate_zip(): the zip path (which includes SLUG) now has \n and \r removed with printf '%s' | tr -d '\n\r' before it is written to $GITHUB_OUTPUT. The zip file itself is still created at the same path, so nothing else changes.

