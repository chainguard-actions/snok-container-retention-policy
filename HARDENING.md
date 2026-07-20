<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **snok--container-retention-policy/v3.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable tag/branch refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks.

action.yaml: `image: 'docker://ghcr.io/snok/container-retention-policy:v3.0.1'` — tag ref, not a SHA digest.

live_test.yaml: `docker/login-action@v3.0.0`, `snok/container-retention-policy@main`, `actions/create-github-app-token@v1`, `snok/container-retention-policy@v3.0.1`, `snok/container-retention-policy@main`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`.

release.yaml: `actions/checkout@v4`, `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`, `docker/build-push-action@v6`, `actions/upload-artifact@v4`, `actions/download-artifact@v4`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`, `docker/metadata-action@v5`.

test.yaml: `actions/checkout@v4`, `swatinem/rust-cache@v2`, `cargo-bins/cargo-binstall@main`, `actions/cache@v4`.

Locations:

- `action.yaml:56`
- `.github/workflows/live_test.yaml:14`
- `.github/workflows/live_test.yaml:26`
- `.github/workflows/live_test.yaml:37`
- `.github/workflows/live_test.yaml:44`
- `.github/workflows/live_test.yaml:55`
- `.github/workflows/live_test.yaml:68`
- `.github/workflows/live_test.yaml:69`
- `.github/workflows/release.yaml:37`
- `.github/workflows/release.yaml:38`
- `.github/workflows/release.yaml:39`
- `.github/workflows/release.yaml:40`
- `.github/workflows/release.yaml:46`
- `.github/workflows/release.yaml:52`
- `.github/workflows/release.yaml:63`
- `.github/workflows/release.yaml:72`
- `.github/workflows/release.yaml:78`
- `.github/workflows/release.yaml:80`
- `.github/workflows/release.yaml:87`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:20`
- `.github/workflows/test.yaml:23`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within them defines job-level permissions either. This means workflows run with the default (overly broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/live_test.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Several `run:` blocks in release.yaml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker-controlled value to be parsed by the shell before quoting can protect it.

1. (line 34) `platform="${{ matrix.platform }}"` — matrix context interpolated directly into shell.
2. (line 57) `digest="${{ steps.build.outputs.digest }}"` — step output interpolated directly into shell.
3. (line 83) `$(printf '${{ env.REGISTRY_IMAGE }}@sha256:%s ' *)` — env context interpolated inside a command substitution in a run block.
4. (line 86) `run: docker buildx imagetools inspect ${{ env.REGISTRY_IMAGE }}:${{ steps.meta.outputs.version }}` — two env/step-output expressions interpolated directly into a single-line run command.

Locations:

- `.github/workflows/release.yaml:34`
- `.github/workflows/release.yaml:57`
- `.github/workflows/release.yaml:83`
- `.github/workflows/release.yaml:86`

### github-env-injection (severity: high)

In release.yaml, the 'Prepare' step writes a value derived from `${{ matrix.platform }}` to `$GITHUB_ENV` without sanitization. The expression is first assigned to the shell variable `platform` (itself a script-injection risk), and then `${platform//\//-}` is echoed into `$GITHUB_ENV`. Because `matrix.platform` is workflow-controlled and no `printf '%s' ... | tr -d '\n\r'` sanitization step is applied before the write, a newline injected into the matrix value could poison the environment file and set arbitrary environment variables for subsequent steps.

Offending lines:
  `platform="${{ matrix.platform }}"`  (line 34)
  `echo "PLATFORM_PAIR=${platform//\//-}" >> $GITHUB_ENV`  (line 35)

Locations:

- `.github/workflows/release.yaml:34`
- `.github/workflows/release.yaml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all action refs to full SHA hashes in all workflow files. Pinned the container image in action.yaml to its sha256 digest (preserving docker:// scheme and tag inline).

2. **missing-permissions**: Added top-level permissions blocks: release.yaml and live_test.yaml get `contents: read, packages: write`; test.yaml gets `contents: read`.

3. **script-injection**: In release.yaml, moved all ${{ }} expressions from run: blocks into step-level env: blocks and referenced them as plain shell variables.

4. **github-env-injection**: In release.yaml's Prepare step, moved matrix.platform into env.MATRIX_PLATFORM and sanitized with `printf '%s' "$MATRIX_PLATFORM" | tr -d '\n\r'` before writing to $GITHUB_ENV.

