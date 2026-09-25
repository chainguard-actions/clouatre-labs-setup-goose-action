<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.10** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Check platform' step directly interpolates ${{ runner.os }} inside a run: shell script. Any ${{ }} expression inside a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Offending lines: `if [ "${{ runner.os }}" != "Linux" ]; then` and `echo "::error::This action only supports Linux runners. Current OS: ${{ runner.os }}"`

Locations:

- `action.yml:35`

### script-injection (severity: high)

Sub-rule (a): The 'Resolve version' step directly interpolates attacker-controlled inputs ${{ inputs.version }} and ${{ inputs.check-latest }} inside a run: shell script. These values are supplied by the calling workflow and can contain shell metacharacters. Offending lines: `VERSION="${{ inputs.version }}"` and `if [ "${{ inputs.check-latest }}" = "true" ]; then`

Locations:

- `action.yml:46`

### script-injection (severity: high)

Sub-rule (a): The 'Install Goose' step directly interpolates ${{ steps.resolve-version.outputs.version }} and ${{ runner.arch }} inside a run: shell script. Both are ${{ }} expressions substituted into the shell before parsing. Offending lines: `VERSION="${{ steps.resolve-version.outputs.version }}"` and `ARCH="${{ runner.arch }}"`

Locations:

- `action.yml:96`

### github-env-injection (severity: high)

The 'Resolve version' step writes the shell variable VERSION to $GITHUB_OUTPUT without sanitization. VERSION is derived directly from the untrusted input ${{ inputs.version }} (set as `VERSION="${{ inputs.version }}"`). An attacker-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$VERSION" | tr -d '\n\r'`) is absent before the write: `echo "version=$VERSION" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:75`

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
1. 'Check platform' step: moved ${{ runner.os }} to RUNNER_OS env var, referenced as $RUNNER_OS in shell.
2. 'Resolve version' step: moved ${{ inputs.version }} to INPUT_VERSION and ${{ inputs.check-latest }} to INPUT_CHECK_LATEST env vars; removed zizmor ignore comment since injection is now properly fixed; added `safe_version=$(printf '%s' "$VERSION" | tr -d '\n\r')` sanitization before writing to GITHUB_OUTPUT.
3. 'Install Goose' step: moved ${{ steps.resolve-version.outputs.version }} to RESOLVED_VERSION and ${{ runner.arch }} to RUNNER_ARCH env vars, referenced as $RESOLVED_VERSION and $RUNNER_ARCH in shell.

