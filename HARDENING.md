<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.8** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Check platform' step directly interpolates `${{ runner.os }}` inside the `run:` shell script on lines 39 and 40. Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell sees it, making it a script-injection risk regardless of whether the context appears GitHub-controlled.

Locations:

- `action.yml:39`
- `action.yml:40`

### script-injection (severity: high)

Sub-rule (a): The 'Resolve version' step directly interpolates `${{ inputs.version }}` (line 50) and `${{ inputs.check-latest }}` (line 52) inside the `run:` shell script. Both are attacker-controlled inputs that flow through YAML template substitution before the shell executes them, enabling command injection. The `# zizmor: ignore[template-injection]` comment is a linter suppression annotation, not a security fix.

Locations:

- `action.yml:50`
- `action.yml:52`

### script-injection (severity: high)

Sub-rule (a): The 'Install Goose' step directly interpolates `${{ steps.resolve-version.outputs.version }}` (line 92) and `${{ runner.arch }}` (line 102) inside the `run:` shell script. These expressions are substituted by the YAML template engine before the shell executes the script, bypassing shell quoting and enabling injection.

Locations:

- `action.yml:92`
- `action.yml:102`

### github-env-injection (severity: high)

The 'Resolve version' step sets `VERSION="${{ inputs.version }}"` (line 50) — an attacker-controlled input — and then writes it to `$GITHUB_OUTPUT` on line 73 via `echo "version=$VERSION" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VERSION" | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:50`
- `action.yml:73`

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
2. 'Resolve version' step: moved `${{ inputs.version }}` and `${{ inputs.check-latest }}` to env block as INPUT_VERSION and INPUT_CHECK_LATEST; sanitized VERSION with `printf '%s' "$VERSION" | tr -d '\n\r'` before writing to GITHUB_OUTPUT; removed the now-unnecessary zizmor suppression comment.
3. 'Install Goose' step: moved `${{ steps.resolve-version.outputs.version }}` and `${{ runner.arch }}` to env block as RESOLVED_VERSION and RUNNER_ARCH.

