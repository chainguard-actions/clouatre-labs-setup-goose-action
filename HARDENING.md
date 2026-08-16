<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.10** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell commands in action.yml. In the 'Check platform' step, ${{ runner.os }} is embedded directly in the shell script. In the 'Resolve version' step, ${{ inputs.version }} and ${{ inputs.check-latest }} are interpolated directly (the zizmor ignore comment does not fix the vulnerability). In the 'Install Goose' step, ${{ steps.resolve-version.outputs.version }} and ${{ runner.arch }} are interpolated directly. Any of these values flowing through YAML template substitution before the shell sees them can allow shell metacharacter injection.

Locations:

- `action.yml:31`
- `action.yml:32`
- `action.yml:38`
- `action.yml:40`
- `action.yml:82`
- `action.yml:92`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell commands in the workflow file. In 'Verify installation', ${{ steps.goose.outputs.goose-version }} and ${{ steps.goose.outputs.goose-path }} are embedded in echo commands. In 'Verify cache-hit output is set', ${{ steps.install.outputs.cache-hit }} is assigned inside the shell script. In 'Verify cache-hit is true on restore', ${{ steps.restore.outputs.cache-hit }} appears in an if-condition and an error message. In 'Verify latest version installed', ${{ steps.check-latest.outputs.goose-version }} is assigned inside the shell. In 'Verify all jobs passed or were skipped', ${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') }} is interpolated directly into an if-condition.

Locations:

- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:74`
- `.github/workflows/test.yml:115`
- `.github/workflows/test.yml:155`

### github-env-injection (severity: high)

In the 'Resolve version' step of action.yml, the shell variable VERSION is set from the untrusted input ${{ inputs.version }} and then written to $GITHUB_OUTPUT without the required sanitization step (printf '%s' "$VERSION" | tr -d '\n\r'). An attacker-controlled version input containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. The offending line is: `echo "version=$VERSION" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:68`

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

Fixed all script injection issues in action.yml:
1. 'Check platform' step: moved ${{ runner.os }} to env var RUNNER_OS
2. 'Resolve version' step: moved ${{ inputs.version }} to INPUT_VERSION and ${{ inputs.check-latest }} to INPUT_CHECK_LATEST in env block; removed zizmor ignore comment
3. 'Resolve version' step: sanitized VERSION before writing to GITHUB_OUTPUT using printf '%s' "$VERSION" | tr -d '\n\r', and quoted $GITHUB_OUTPUT
4. 'Install Goose' step: moved ${{ steps.resolve-version.outputs.version }} to RESOLVED_VERSION and ${{ runner.arch }} to RUNNER_ARCH in env block

The .github/workflows/test.yml findings were not fixed as that file is a test harness and per the rules, security fixes should only be applied to action.yml and supporting scripts that are part of the distributed action.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 script injection instances in hardened/action/.github/workflows/test.yml by moving ${{ ... }} expressions into step-level env: blocks and referencing them as plain shell environment variables:
1. 'Verify installation' step: moved steps.goose.outputs.goose-version and steps.goose.outputs.goose-path to GOOSE_VERSION and GOOSE_PATH env vars
2. 'Verify cache-hit output is set' step: moved steps.install.outputs.cache-hit to CACHE_HIT env var (also removed redundant inline assignment)
3. 'Verify cache-hit is true on restore' step: moved steps.restore.outputs.cache-hit to CACHE_HIT env var (both occurrences replaced)
4. 'Verify latest version installed' step: moved steps.check-latest.outputs.goose-version to INSTALLED_VERSION env var (also removed redundant inline assignment)
5. 'Verify all jobs passed or were skipped' step: moved contains(needs.*.result, ...) expression to CI_FAILED env var

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings involving `${{ github.base_ref }}` being directly interpolated into `run:` shell commands:

1. `examples/tier2-balanced-security.yml` (line 21): Added `env: BASE_REF: ${{ github.base_ref }}` to the 'Generate Diff Stats' step and changed the run command from `git diff --stat origin/${{ github.base_ref }}...HEAD` to `git diff --stat "origin/$BASE_REF"...HEAD`.

2. `examples/tier3-advanced-patterns.yml` (line 33): Added `BASE_REF: ${{ github.base_ref }}` to the existing `env:` block of the 'AI Diff Review' step and changed `git diff origin/${{ github.base_ref }}...HEAD` to `git diff "origin/$BASE_REF"...HEAD`.

In both cases the attacker-controlled value is now passed through an environment variable and double-quoted in the shell, preventing shell metacharacter injection.

