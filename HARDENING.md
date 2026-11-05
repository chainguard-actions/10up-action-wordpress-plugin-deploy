<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): a GitHub Actions expression is placed directly inside a run: command string. Offending line: `run: ${{ github.action_path }}/deploy.sh`. Fix: use the runner env var instead, for example `run: "$GITHUB_ACTION_PATH/deploy.sh"`.

Locations:

- `action.yml:25`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. Two locations write workflow-controlled (untrusted) values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

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

- `deploy.sh:83`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2ca03ca..0117167 100644
--- a/action.yml
+++ b/action.yml
@@ -18,9 +18,26 @@ outputs:
 runs:
   using: 'composite'
   steps:
     - id: deploy
       env:
         INPUT_GENERATE_ZIP: ${{ inputs.generate-zip }}
         INPUT_DRY_RUN: ${{ inputs.dry-run }}
-      run: ${{ github.action_path }}/deploy.sh
+      run: "$GITHUB_ACTION_PATH/deploy.sh"
       shell: bash

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed deploy.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/deploy.sh b/deploy.sh
index 22051da..92d5cad 100755
--- a/deploy.sh
+++ b/deploy.sh
@@ -46,7 +46,7 @@ if [[ -z "$SVN_PASSWORD" ]]; then
 	exit 1
 fi
 
-if $INPUT_DRY_RUN; then
+if [[ "$INPUT_DRY_RUN" == 'true' ]]; then
 	echo "ℹ︎ Dry run: No files will be committed to Subversion."
 fi
 
@@ -93,7 +93,7 @@ svn update --set-depth infinity trunk
 svn update --set-depth immediates tags
 
 generate_zip() {
-  if $INPUT_GENERATE_ZIP; then
+  if [[ "$INPUT_GENERATE_ZIP" == 'true' ]]; then
     echo "Generating zip file..."
 
     # use a symbolic link so the directory in the zip matches the slug
@@ -101,7 +101,8 @@ generate_zip() {
     zip -r "${GITHUB_WORKSPACE}/${SLUG}.zip" "$SLUG"
     unlink "${SVN_DIR}/${SLUG}"
 
-    echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"
+    safe_zip_path=$(printf '%s' "${GITHUB_WORKSPACE}/${SLUG}.zip" | tr -d '\n\r')
+    echo "zip-path=${safe_zip_path}" >> "${GITHUB_OUTPUT}"
     echo "✓ Zip file generated!"
   fi
 }
@@ -216,7 +217,7 @@ svn update
 
 svn status
 
-if $INPUT_DRY_RUN; then
+if [[ "$INPUT_DRY_RUN" == 'true' ]]; then
   echo "➤ Dry run: Files not committed."
 else
   echo "➤ Committing files..."

```

Locations:

- `deploy.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

action.yml: the step now runs "$GITHUB_ACTION_PATH/deploy.sh" (quoted in bash) instead of putting ${{ github.action_path }} straight into the run line. deploy.sh: the zip-path value is cleaned with printf | tr -d '\n\r' before it's written to GITHUB_OUTPUT. I also re-applied the earlier fix: the script no longer runs the input as a command ('if $INPUT_DRY_RUN' / 'if $INPUT_GENERATE_ZIP') and compares it to 'true' instead ([[ "$X" == 'true' ]]). .github/workflows/release-pull-request.yml isn't in the hardened action directory, so there was nothing to change there.

