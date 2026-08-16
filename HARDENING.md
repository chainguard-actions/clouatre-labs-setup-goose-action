<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.8** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml. This includes attacker-controlled inputs (${{ inputs.version }}, ${{ inputs.check-latest }}) and context values that flow through YAML template substitution before the shell sees them (${{ runner.os }}, ${{ runner.arch }}, ${{ steps.resolve-version.outputs.version }}). Any of these can contain shell metacharacters that will be interpreted by the shell. The '# zizmor: ignore[template-injection]' comment on the Resolve version step does not fix the underlying injection. Offending lines include: `VERSION="${{ inputs.version }}"`, `if [ "${{ inputs.check-latest }}" = "true" ]`, `if [ "${{ runner.os }}" != "Linux" ]`, `VERSION="${{ steps.resolve-version.outputs.version }}"`, `ARCH="${{ runner.arch }}"`.

Locations:

- `action.yml:30`
- `action.yml:31`
- `action.yml:40`
- `action.yml:43`
- `action.yml:72`
- `action.yml:82`

### github-env-injection (severity: high)

The 'Resolve version' step in action.yml writes VERSION to $GITHUB_OUTPUT without sanitization. VERSION is derived directly from ${{ inputs.version }}, an attacker-controlled input. A malicious caller could supply a value containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT (e.g., `version=injected\nother_key=malicious`). The required sanitization step (`printf '%s' "$VERSION" | tr -d '\n\r'`) is absent before the write: `echo "version=$VERSION" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:68`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in .github/workflows/test.yml. Affected steps include: 'Verify installation' (`echo "Goose version: ${{ steps.goose.outputs.goose-version }}"`), 'Verify cache-hit output is set' (`CACHE_HIT="${{ steps.install.outputs.cache-hit }}"`), 'Verify cache-hit is true on restore' (`if [ "${{ steps.restore.outputs.cache-hit }}" != "true" ]` and the error message), 'Verify latest version installed' (`INSTALLED_VERSION="${{ steps.check-latest.outputs.goose-version }}"`), and 'Verify all jobs passed or were skipped' (`if [[ "${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') }}" == "true" ]]`). These values flow through YAML template substitution before the shell processes them and can contain shell metacharacters.

Locations:

- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:80`
- `.github/workflows/test.yml:82`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:152`
- `.github/workflows/test.yml:155`

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

Fixed all findings in action.yml:
1. Check platform step: moved ${{ runner.os }} to env: RUNNER_OS, referenced as $RUNNER_OS in shell.
2. Resolve version step: moved ${{ inputs.version }} to env: INPUT_VERSION and ${{ inputs.check-latest }} to env: INPUT_CHECK_LATEST; sanitized GITHUB_OUTPUT write with `printf '%s' "$VERSION" | tr -d '\n\r'` before writing version output.
3. Install Goose step: moved ${{ steps.resolve-version.outputs.version }} to env: RESOLVED_VERSION and ${{ runner.arch }} to env: RUNNER_ARCH, both referenced as plain shell variables.

The .github/workflows/test.yml script-injection findings were not fixed per the rules that state 'Security fixes go to action.yml and supporting scripts only' and test files should not be modified.

