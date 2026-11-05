<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): a GitHub expression is placed directly inside a run: line. Offending line: `run: ${{ github.action_path }}/deploy.sh`. Any ${{ }} inside a run: line is a finding, whatever context it reads from. Fix: use the env var form `run: "$GITHUB_ACTION_PATH/deploy.sh"`.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Sub-rule (b): an unquoted env var that holds input data is run as a shell command. In deploy.sh, `if $INPUT_DRY_RUN; then` (lines 23 and near the end) and `if $INPUT_GENERATE_ZIP; then` (line 70) expand inputs.dry-run and inputs.generate-zip with no quotes and run the result as a command. A caller-supplied value such as `true; malicious-cmd` runs arbitrary commands. Fix: compare the quoted value instead, e.g. `if [[ "$INPUT_DRY_RUN" == "true" ]]; then`.

Locations:

- `deploy.sh:23`
- `deploy.sh:70`

### github-env-injection (severity: high)

deploy.sh is called from the composite run: step. It writes `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"` with no sanitization. SLUG is inherited from the calling workflow's environment, and if that is empty it comes from GITHUB_REPOSITORY. A newline in SLUG would add extra output entries. Fix: strip newlines first, e.g. `safe=$(printf '%s' "${GITHUB_WORKSPACE}/${SLUG}.zip" | tr -d '\n\r')`, then write `zip-path=$safe`.

Locations:

- `deploy.sh:78`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2ca03ca..4fa9389 100644
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
+      run: '"$GITHUB_ACTION_PATH/deploy.sh"'
       shell: bash

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed deploy.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/deploy.sh b/deploy.sh
index a843ba7..cd55fe3 100755
--- a/deploy.sh
+++ b/deploy.sh
@@ -20,7 +20,7 @@ if [[ -z "$SVN_PASSWORD" ]]; then
 	exit 1
 fi
 
-if $INPUT_DRY_RUN; then
+if [[ "$INPUT_DRY_RUN" == "true" ]]; then
 	echo "ℹ︎ Dry run: No files will be committed to Subversion."
 fi
 
@@ -67,7 +67,7 @@ svn update --set-depth infinity trunk
 svn update --set-depth immediates tags
 
 generate_zip() {
-  if $INPUT_GENERATE_ZIP; then
+  if [[ "$INPUT_GENERATE_ZIP" == "true" ]]; then
     echo "Generating zip file..."
 
     # use a symbolic link so the directory in the zip matches the slug
@@ -75,7 +75,9 @@ generate_zip() {
     zip -r "${GITHUB_WORKSPACE}/${SLUG}.zip" "$SLUG"
     unlink "${SVN_DIR}/${SLUG}"
 
-    echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"
+    zip_path="${GITHUB_WORKSPACE}/${SLUG}.zip"
+    safe_zip_path=$(printf '%s' "$zip_path" | tr -d '\n\r')
+    echo "zip-path=${safe_zip_path}" >> "${GITHUB_OUTPUT}"
     echo "✓ Zip file generated!"
   fi
 }
@@ -188,7 +190,7 @@ svn update
 
 svn status
 
-if $INPUT_DRY_RUN; then
+if [[ "$INPUT_DRY_RUN" == "true" ]]; then
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

action.yml: run now calls '"$GITHUB_ACTION_PATH/deploy.sh"' instead of inlining ${{ github.action_path }} (shell is bash). deploy.sh: the three spots that ran an input as a command ('if $INPUT_DRY_RUN', twice, and 'if $INPUT_GENERATE_ZIP') now compare the quoted value to "true". deploy.sh: newlines are stripped from zip-path before it is written to GITHUB_OUTPUT. This is the same as the previous harden.

