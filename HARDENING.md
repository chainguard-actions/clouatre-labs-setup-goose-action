<!-- markdownlint-disable -->

# Hardening Report: clouatre-labs--setup-goose-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **clouatre-labs--setup-goose-action/v1.0.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, violating rule (a). This allows the shell to parse the raw expression value before any quoting can protect it.

- Line 35 (Check platform step): `if [ "${{ runner.os }}" != "Linux" ]; then`
- Line 36 (Check platform step): `echo "::error::...Current OS: ${{ runner.os }}"`
- Line 48 (Resolve version step): `VERSION="${{ inputs.version }}"`  — inputs.version is attacker-controlled
- Line 50 (Resolve version step): `if [ "${{ inputs.check-latest }}" = "true" ]; then`  — inputs.check-latest is attacker-controlled
- Line 88 (Install Goose step): `VERSION="${{ steps.resolve-version.outputs.version }}"`
- Line 100 (Install Goose step): `ARCH="${{ runner.arch }}"`

All of these must be moved to `env:` blocks and referenced as `"$ENV_VAR"` in the shell script instead.

Locations:

- `action.yml:35`
- `action.yml:36`
- `action.yml:48`
- `action.yml:50`
- `action.yml:88`
- `action.yml:100`

### github-env-injection (severity: high)

In the 'Resolve version' step, the shell variable `VERSION` is set directly from the untrusted input `${{ inputs.version }}` (line 48). This value is then written to `$GITHUB_OUTPUT` on line 79 via `echo "version=$VERSION" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VERSION" | tr -d '\n\r'`). An attacker-controlled `inputs.version` containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs or poisoning the environment of downstream steps.

Locations:

- `action.yml:48`
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

Fixed all script-injection and github-env-injection findings in action.yml:

1. 'Check platform' step: moved `${{ runner.os }}` to env block as RUNNER_OS, replaced inline expressions in run: with $RUNNER_OS.

2. 'Resolve version' step: moved `${{ inputs.version }}` to env block as INPUT_VERSION and `${{ inputs.check-latest }}` as INPUT_CHECK_LATEST; replaced inline expressions in run: with $INPUT_VERSION and $INPUT_CHECK_LATEST. Also sanitized VERSION before writing to GITHUB_OUTPUT using `printf '%s' "$VERSION" | tr -d '\n\r'`.

3. 'Install Goose' step: moved `${{ steps.resolve-version.outputs.version }}` to env block as RESOLVED_VERSION and `${{ runner.arch }}` as RUNNER_ARCH; replaced inline expressions in run: with $RESOLVED_VERSION and $RUNNER_ARCH.

Expressions in `with:` blocks (cache key) and `outputs:` value fields are not run: shell strings and were left unchanged.

