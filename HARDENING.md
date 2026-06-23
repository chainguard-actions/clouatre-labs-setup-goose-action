<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **clouatre-labs--setup-goose-action/v1.0.8** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings.

1. 'Check platform' step: `${{ runner.os }}` is interpolated directly in the run: block on lines 36-37. Even though runner.os is GitHub-controlled, any ${{ }} in a run: block is a script-injection finding per the check rules.

2. 'Resolve version' step: `${{ inputs.version }}` (line 47) and `${{ inputs.check-latest }}` (line 49) are attacker-controlled inputs interpolated directly into shell commands. A calling workflow or pull request can supply arbitrary values including shell metacharacters. The `# zizmor: ignore[template-injection]` comment suppresses the linter but does not fix the vulnerability.

3. 'Install Goose' step: `${{ steps.resolve-version.outputs.version }}` (line 93) and `${{ runner.arch }}` (line 102) are interpolated directly in the run: block.

Locations:

- `action.yml:36`
- `action.yml:37`
- `action.yml:47`
- `action.yml:49`
- `action.yml:93`
- `action.yml:102`

### github-env-injection (severity: high)

The 'Resolve version' step writes VERSION to $GITHUB_OUTPUT without sanitization. VERSION is initialized from `${{ inputs.version }}` (an untrusted, caller-controlled input). Although the script validates the version format with a regex, it does not apply the required sanitization pipeline (`printf '%s' "$VERSION" | tr -d '\n\r'`) immediately before the write `echo "version=$VERSION" >> $GITHUB_OUTPUT`. A newline embedded in the input could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:79`

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

Fixed all findings in action.yml: (1) Moved ${{ runner.os }} to env block as RUNNER_OS in 'Check platform' step. (2) Moved ${{ inputs.version }} and ${{ inputs.check-latest }} to env block as INPUT_VERSION and INPUT_CHECK_LATEST in 'Resolve version' step. (3) Moved ${{ steps.resolve-version.outputs.version }} and ${{ runner.arch }} to env block as RESOLVED_VERSION and RUNNER_ARCH in 'Install Goose' step. (4) Added sanitization pipeline (printf '%s' "$VERSION" | tr -d '\n\r') before writing version to $GITHUB_OUTPUT to prevent newline injection.

