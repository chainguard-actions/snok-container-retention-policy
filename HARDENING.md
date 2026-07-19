<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v3.0.1-rc1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **snok--container-retention-policy/v3.0.1-rc1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml uses a Docker image reference with a mutable tag instead of a SHA digest: `image: 'docker://ghcr.io/snok/container-retention-policy:v3.0.0'`. This is vulnerable to supply-chain attacks if the tag is moved. All `uses:` references in the workflow files also use mutable tags or branch names instead of pinned 40-character commit SHAs. Unpinned references in live_test.yaml: `docker/login-action@v3.0.0`, `snok/container-retention-policy@main` (×3), `actions/create-github-app-token@v1`, `docker/setup-buildx-action@v3`, `docker/login-action@v3`. Unpinned references in release.yaml: `actions/checkout@v4`, `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3` (×2), `docker/login-action@v3` (×2), `docker/metadata-action@v5` (×2), `docker/build-push-action@v6`, `actions/upload-artifact@v4`, `actions/download-artifact@v4`. Unpinned references in test.yaml: `actions/checkout@v4`, `swatinem/rust-cache@v2`, `cargo-bins/cargo-binstall@main`, `actions/cache@v4`.

Locations:

- `action.yaml:56`
- `.github/workflows/live_test.yaml:13`
- `.github/workflows/live_test.yaml:25`
- `.github/workflows/live_test.yaml:38`
- `.github/workflows/live_test.yaml:52`
- `.github/workflows/live_test.yaml:65`
- `.github/workflows/live_test.yaml:73`
- `.github/workflows/live_test.yaml:74`
- `.github/workflows/release.yaml:36`
- `.github/workflows/release.yaml:37`
- `.github/workflows/release.yaml:38`
- `.github/workflows/release.yaml:39`
- `.github/workflows/release.yaml:46`
- `.github/workflows/release.yaml:52`
- `.github/workflows/release.yaml:65`
- `.github/workflows/release.yaml:72`
- `.github/workflows/release.yaml:77`
- `.github/workflows/release.yaml:82`
- `.github/workflows/release.yaml:85`
- `.github/workflows/test.yaml:12`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:19`
- `.github/workflows/test.yaml:24`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within them defines a job-level `permissions:` key. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all`), granting unnecessarily broad access to the GITHUB_TOKEN.

Locations:

- `.github/workflows/live_test.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Multiple `run:` blocks in release.yaml directly interpolate `${{ ... }}` expressions into shell commands, violating sub-rule (a). This allows template substitution to inject shell metacharacters before the shell ever parses the command. (1) Line 35 — `platform="${{ matrix.platform }}"`: matrix context interpolated directly into shell. (2) Line 57 — `digest="${{ steps.build.outputs.digest }}"`: step output interpolated directly into shell. (3) Line 79 — `$(printf '${{ env.REGISTRY_IMAGE }}@sha256:%s ' *)`: env context interpolated directly into shell command substitution. (4) Line 86 — `docker buildx imagetools inspect ${{ env.REGISTRY_IMAGE }}:${{ steps.meta.outputs.version }}`: two expressions interpolated directly into a shell command.

Locations:

- `.github/workflows/release.yaml:35`
- `.github/workflows/release.yaml:57`
- `.github/workflows/release.yaml:79`
- `.github/workflows/release.yaml:86`

### github-env-injection (severity: high)

In release.yaml, the 'Prepare' step (line 34–36) writes an unsanitized value derived from `${{ matrix.platform }}` to `$GITHUB_ENV`. The expression is assigned to the shell variable `platform` and then written as `PLATFORM_PAIR=${platform//\//-} >> $GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence the matrix value could inject additional environment variable definitions via embedded newlines.

Locations:

- `.github/workflows/release.yaml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yaml and three workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-character commit SHAs with tag comments. Pinned the container image in action.yaml to its SHA digest while preserving the docker:// scheme and tag.

2. **permissions**: Added minimal `permissions:` blocks to all three workflow files (live_test.yaml, release.yaml, test.yaml). live_test.yaml and release.yaml need `packages: write` for GHCR operations; test.yaml only needs `contents: read`.

3. **script-injection**: Fixed all four injection points in release.yaml by moving `${{ ... }}` expressions into step `env:` blocks and referencing them as plain shell variables.

4. **github-env-injection**: Fixed the Prepare step in release.yaml by moving `${{ matrix.platform }}` to an env var and sanitizing with `printf '%s' "$PLATFORM" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

