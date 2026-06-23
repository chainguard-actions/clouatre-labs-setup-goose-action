<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **clouatre-labs--setup-goose-action/v1.0.10** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings. In the 'Check platform' step, ${{ runner.os }} is interpolated directly at line 37. In the 'Resolve version' step, the attacker-controlled ${{ inputs.version }} is interpolated at line 48 and ${{ inputs.check-latest }} at line 50. In the 'Install Goose' step, ${{ steps.resolve-version.outputs.version }} is interpolated at line 98 and ${{ runner.arch }} at line 108. YAML template substitution occurs before the shell sees the value, enabling command injection for any of these expressions.

Locations:

- `action.yml:37`
- `action.yml:48`
- `action.yml:50`
- `action.yml:98`
- `action.yml:108`

### github-env-injection (severity: high)

The 'Resolve version' step writes VERSION to $GITHUB_OUTPUT without sanitization. VERSION is assigned directly from ${{ inputs.version }} (an attacker-controlled input) at line 48, then written via 'echo "version=$VERSION" >> $GITHUB_OUTPUT' at line 79. No 'printf "%s" ... | tr -d "\n\r"' sanitization is applied before the write, allowing newline injection into the GitHub output file.

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

Fixed all findings in action.yml: (1) Moved ${{ runner.os }} in 'Check platform' step to env: block as RUNNER_OS. (2) Moved ${{ inputs.version }} and ${{ inputs.check-latest }} in 'Resolve version' step to env: block as INPUT_VERSION and INPUT_CHECK_LATEST. (3) Added newline sanitization (printf '%s' | tr -d '\n\r') before writing VERSION to $GITHUB_OUTPUT. (4) Moved ${{ steps.resolve-version.outputs.version }} and ${{ runner.arch }} in 'Install Goose' step to env: block as RESOLVED_VERSION and RUNNER_ARCH. All shell scripts now reference plain environment variables instead of ${{ }} expressions.

