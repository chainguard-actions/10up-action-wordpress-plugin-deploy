<!-- markdownlint-disable -->

# Hardening Report: 10up--action-wordpress-plugin-deploy/2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **10up--action-wordpress-plugin-deploy/2.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): a GitHub Actions expression is placed directly in the run: command of the composite step 'deploy': `run: ${{ github.action_path }}/deploy.sh`. The value is substituted into the script text before the shell reads it. Fix: put it in an env var or use the built-in $GITHUB_ACTION_PATH, and quote it: `run: "$GITHUB_ACTION_PATH/deploy.sh"`.

Locations:

- `action.yml:25`

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In deploy.sh (called from action.yml), the variable `SLUG` is derived from the inherited process env var `GITHUB_REPOSITORY` (a workflow-controlled value: `SLUG=${GITHUB_REPOSITORY#*/}`). It is then written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`): `echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"`. An attacker-controlled repository name containing newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT.

Locations:

- `deploy.sh:75`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2ca03ca..ec99038 100644
--- a/action.yml
+++ b/action.yml
@@ -18,9 +18,27 @@ outputs:
 runs:
   using: 'composite'
   steps:
     - id: deploy
       env:
         INPUT_GENERATE_ZIP: ${{ inputs.generate-zip }}
         INPUT_DRY_RUN: ${{ inputs.dry-run }}
-      run: ${{ github.action_path }}/deploy.sh
+        ACTION_PATH: ${{ github.action_path }}
+      run: "$ACTION_PATH/deploy.sh"
       shell: bash

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed deploy.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/deploy.sh b/deploy.sh
index 50af5f6..7934528 100755
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
@@ -75,7 +75,8 @@ generate_zip() {
     zip -r "${GITHUB_WORKSPACE}/${SLUG}.zip" "$SLUG"
     unlink "${SVN_DIR}/${SLUG}"
 
-    echo "zip-path=${GITHUB_WORKSPACE}/${SLUG}.zip" >> "${GITHUB_OUTPUT}"
+    safe_slug=$(printf '%s' "$SLUG" | tr -d '\n\r')
+    echo "zip-path=${GITHUB_WORKSPACE}/${safe_slug}.zip" >> "${GITHUB_OUTPUT}"
     echo "✓ Zip file generated!"
   fi
 }
@@ -190,7 +191,7 @@ svn update
 
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

action.yml: the deploy step now runs "$GITHUB_ACTION_PATH/deploy.sh" (quoted) instead of inlining ${{ github.action_path }}. deploy.sh: the zip-path value has newlines stripped (printf | tr -d '\n\r') before it is written to GITHUB_OUTPUT. The `if $INPUT_DRY_RUN` / `if $INPUT_GENERATE_ZIP` checks, which ran the input value as a command, now compare it to "true" with [[ ... == "true" ]].

