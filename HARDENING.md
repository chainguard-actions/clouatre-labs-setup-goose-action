<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.6** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, violating sub-rule (a). This includes attacker-controlled inputs (`${{ inputs.version }}` at line 44, `${{ inputs.check-latest }}` at line 46), step outputs (`${{ steps.resolve-version.outputs.version }}` at line 76), and runner context values (`${{ runner.os }}` at lines 34–35, `${{ runner.arch }}` at line 89). Any expression interpolated directly into a shell `run:` block allows an attacker to inject arbitrary shell commands. For example, `VERSION="${{ inputs.version }}"` lets a caller supply a value containing shell metacharacters that are evaluated before the shell ever sees the script.

Locations:

- `action.yml:34`
- `action.yml:35`
- `action.yml:44`
- `action.yml:46`
- `action.yml:76`
- `action.yml:89`

### github-env-injection (severity: high)

In the 'Resolve version' step, the shell variable `VERSION` is set directly from the untrusted input `${{ inputs.version }}` (line 44) and then written to `$GITHUB_OUTPUT` without sanitization: `echo "version=$VERSION" >> $GITHUB_OUTPUT`. An attacker-controlled value containing newlines could inject arbitrary key=value pairs into the GitHub output environment. The required sanitization step (`printf '%s' "$VERSION" | tr -d '\n\r'`) is absent before the write. Similarly, in the 'Install Goose' step, `VERSION` is set from `${{ steps.resolve-version.outputs.version }}` (a workflow-controllable step output) and used unsanitized throughout the script.

Locations:

- `action.yml:44`
- `action.yml:82`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Resolve version"; move to env: map

Locations:

- `action.yml:50`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.check-latest }}" appears directly in run: block of step "Resolve version"; move to env: map

Locations:

- `action.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in action.yml:

1. 'Check platform' step: moved `${{ runner.os }}` to env block as RUNNER_OS.
2. 'Resolve version' step: moved `${{ inputs.version }}` → INPUT_VERSION and `${{ inputs.check-latest }}` → INPUT_CHECK_LATEST into env block; added newline sanitization (`printf '%s' "$VERSION" | tr -d '\n\r'`) before writing to $GITHUB_OUTPUT.
3. 'Install Goose' step: moved `${{ steps.resolve-version.outputs.version }}` → RESOLVED_VERSION and `${{ runner.arch }}` → RUNNER_ARCH into env block.
4. Also quoted $GITHUB_OUTPUT and $GITHUB_PATH references throughout for correctness.

The actions/cache `with:` key/restore-keys fields are not shell run blocks and were left unchanged.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 6 script injection instances in hardened/action/.github/workflows/test.yml:
1. Lines 34-35 (Verify installation step): Moved `steps.goose.outputs.goose-version` and `steps.goose.outputs.goose-path` into env vars GOOSE_VERSION and GOOSE_PATH.
2. Line 55 (Check cache-hit output exists step): Moved `steps.first-run.outputs.cache-hit` into env var CACHE_HIT; removed the redundant shell-level assignment.
3. Lines 68-69 (Verify cache-hit is true on second run step): Moved `steps.second-run.outputs.cache-hit` into env var CACHE_HIT; updated both the condition and error message to use $CACHE_HIT.
4. Line 100 (Verify latest version installed step): Moved `steps.check-latest.outputs.goose-version` into env var INSTALLED_VERSION; removed the redundant shell-level assignment.
All ${{ }} expressions are now in env: blocks and referenced as plain $VAR_NAME in shell scripts, eliminating the shell injection risk.

