<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **snok--container-retention-policy/v3.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tags or branch names instead of full 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references found:
- live_test.yaml: `snok/container-retention-policy@v3.0.0`, `snok/container-retention-policy@v3.0.1`, `docker/login-action@v4`, `actions/checkout@v6`, `swatinem/rust-cache@v2`, `docker/setup-qemu-action@v4`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`
- live_test.yaml: `snok/container-retention-policy@main`
- release.yaml: `actions/checkout@v6`, `docker/setup-qemu-action@v4`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`, `docker/build-push-action@v7`, `actions/upload-artifact@v7`, `actions/download-artifact@v7`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`
- test.yaml: `actions/checkout@v6`, `swatinem/rust-cache@v2`, `cargo-bins/cargo-binstall@main`, `actions/cache@v5`
- action.yaml: Docker image `docker://ghcr.io/snok/container-retention-policy:v3.1.0` uses a mutable tag instead of a SHA digest.

Locations:

- `.github/workflows/live_test.yaml:12`
- `.github/workflows/live_test.yaml:28`
- `.github/workflows/live_test.yaml:40`
- `.github/workflows/live_test.yaml:41`
- `.github/workflows/live_test.yaml:42`
- `.github/workflows/live_test.yaml:55`
- `.github/workflows/live_test.yaml:72`
- `.github/workflows/live_test.yaml:73`
- `.github/workflows/live_test.yaml:74`
- `.github/workflows/release.yaml:36`
- `.github/workflows/release.yaml:37`
- `.github/workflows/release.yaml:38`
- `.github/workflows/release.yaml:39`
- `.github/workflows/release.yaml:47`
- `.github/workflows/release.yaml:57`
- `.github/workflows/release.yaml:67`
- `.github/workflows/release.yaml:68`
- `.github/workflows/release.yaml:69`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:21`
- `.github/workflows/test.yaml:24`
- `action.yaml:56`

### permissions (severity: medium)

None of the workflow files define a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three workflow files are affected: live_test.yaml, release.yaml, and test.yaml.

Locations:

- `.github/workflows/live_test.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, allowing template substitution before the shell parses the string. This enables script injection if any of these values contain shell metacharacters.

(a) release.yaml 'Prepare' step: `platform="${{ matrix.platform }}"` — matrix.platform is interpolated directly into the shell script.

(a) release.yaml 'Export digest' step: `digest="${{ steps.build.outputs.digest }}"` — step output interpolated directly into shell.

(a) release.yaml 'Create manifest list and push' step: `${{ env.REGISTRY_IMAGE }}` and `${{ inputs.version-tag }}` interpolated directly into shell commands.

(a) release.yaml 'Inspect image' step: `docker buildx imagetools inspect ${{ env.REGISTRY_IMAGE }}:${{ inputs.version-tag }}` — both expressions interpolated directly.

(a) live_test.yaml 'Delete test-3-* images' step: `--token="${{ secrets.GITHUB_TOKEN }}"` — expression interpolated directly into shell.

Locations:

- `.github/workflows/release.yaml:32`
- `.github/workflows/release.yaml:53`
- `.github/workflows/release.yaml:72`
- `.github/workflows/release.yaml:76`
- `.github/workflows/live_test.yaml:50`

### github-env-injection (severity: high)

In release.yaml, the 'Prepare' step writes a value derived from `${{ matrix.platform }}` to `$GITHUB_ENV` without sanitization. The expression is first assigned to the shell variable `platform`, then written as `PLATFORM_PAIR=${platform//\//-} >> $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing a newline-injection attack that could set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/release.yaml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yaml and the three workflow files:

1. **unpinned-uses**: Pinned all `uses:` references to full SHA hashes with tag comments. Pinned the Docker image in action.yaml to `docker://ghcr.io/snok/container-retention-policy:v3.1.0@sha256:884037f146d6ef574b61fbcc59bef490f887322e6b9dc2eefcf364cb4e9b4112` (preserving docker:// scheme and tag).

2. **permissions**: Added `permissions: {}` at the top level of all three workflow files. Added `permissions: { packages: write }` at the job level for the `build` and `merge` jobs in release.yaml which need to push container images to GHCR.

3. **script-injection**: Moved all `${{ ... }}` expressions from `run:` blocks into `env:` blocks and referenced them as plain shell variables in the scripts.

4. **github-env-injection**: In release.yaml's 'Prepare' step, the platform value is now sanitized with `printf '%s' "$PLATFORM" | tr -d '\n\r'` before being written to `$GITHUB_ENV`, preventing newline injection.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Create manifest list and push' step of .github/workflows/release.yaml. The original code used `$(printf "$REGISTRY_IMAGE@sha256:%s " *)` where $REGISTRY_IMAGE was unquoted inside the command substitution, allowing shell metacharacters to be interpreted. Replaced with a bash array approach: a `for` loop builds `image_refs` with each entry as `"$REGISTRY_IMAGE@sha256:$digest_file"` (properly double-quoted), then passes `"${image_refs[@]}"` to docker buildx imagetools create. This keeps REGISTRY_IMAGE fully quoted throughout and preserves the original functionality.

