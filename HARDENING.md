<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **snok--container-retention-policy/v3.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yaml use mutable tag/branch refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks.

.github/workflows/live_test.yaml:
  - uses: snok/container-retention-policy@main (lines 13, 31, 42)
  - uses: actions/create-github-app-token@v1 (line 27)
  - uses: docker/setup-buildx-action@v3 (line 57)
  - uses: docker/login-action@v3 (line 58)

.github/workflows/release.yaml:
  - uses: actions/checkout@v4
  - uses: docker/setup-qemu-action@v3
  - uses: docker/setup-buildx-action@v3
  - uses: docker/login-action@v3
  - uses: docker/metadata-action@v5
  - uses: docker/build-push-action@v6
  - uses: actions/upload-artifact@v4
  - uses: actions/download-artifact@v4

.github/workflows/test.yaml:
  - uses: actions/checkout@v4
  - uses: swatinem/rust-cache@v2
  - uses: cargo-bins/cargo-binstall@main
  - uses: actions/cache@v4

action.yaml:
  - image: 'docker://ghcr.io/snok/container-retention-policy:v3.0.0' — uses a mutable version tag instead of a SHA digest (e.g. @sha256:<digest>).

Locations:

- `.github/workflows/live_test.yaml:13`
- `.github/workflows/live_test.yaml:27`
- `.github/workflows/live_test.yaml:31`
- `.github/workflows/live_test.yaml:42`
- `.github/workflows/live_test.yaml:57`
- `.github/workflows/live_test.yaml:58`
- `.github/workflows/release.yaml:38`
- `.github/workflows/release.yaml:39`
- `.github/workflows/release.yaml:40`
- `.github/workflows/release.yaml:41`
- `.github/workflows/release.yaml:48`
- `.github/workflows/release.yaml:55`
- `.github/workflows/release.yaml:72`
- `.github/workflows/release.yaml:78`
- `.github/workflows/release.yaml:85`
- `.github/workflows/release.yaml:86`
- `.github/workflows/release.yaml:93`
- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:21`
- `.github/workflows/test.yaml:27`
- `action.yaml:57`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. This means workflows run with the default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/live_test.yaml:1`
- `.github/workflows/release.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

release.yaml contains multiple `run:` blocks that directly interpolate `${{ }}` expressions into shell command strings (sub-rule a). This allows expression values to be interpreted as shell code before the shell ever sees them.

1. 'Prepare' step (line 35): `platform="${{ matrix.platform }}"` — matrix context interpolated directly into shell.
2. 'Export digest' step (line 63): `digest="${{ steps.build.outputs.digest }}"` — step output interpolated directly into shell.
3. 'Create manifest list and push' step (line 100): `$(printf '${{ env.REGISTRY_IMAGE }}@sha256:%s ' *)` — env context interpolated inside a command substitution.
4. 'Inspect image' step (line 103): `docker buildx imagetools inspect ${{ env.REGISTRY_IMAGE }}:${{ steps.meta.outputs.version }}` — two env/step-output expressions interpolated directly into a shell command without quoting.

Locations:

- `.github/workflows/release.yaml:35`
- `.github/workflows/release.yaml:63`
- `.github/workflows/release.yaml:100`
- `.github/workflows/release.yaml:103`

### github-env-injection (severity: high)

In release.yaml, the 'Prepare' step writes a value derived from `${{ matrix.platform }}` (a workflow-controllable context) to `$GITHUB_ENV` without sanitization. The expression is first assigned to the shell variable `$platform`, then written via `echo "PLATFORM_PAIR=${platform//\//-}" >> $GITHUB_ENV`. Routing through a shell variable does not sanitize newline characters; an attacker-controlled matrix value containing a newline could inject arbitrary environment variables. The required sanitization step (`printf '%s' "$platform" | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/release.yaml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across four files:

1. **unpinned-uses** (all workflow files + action.yaml):
   - Pinned all action `uses:` references to full 40-char SHA hashes with tag comments
   - Pinned container image in action.yaml to sha256 digest while preserving docker:// scheme and v3.0.0 tag

2. **permissions** (all three workflow files):
   - Added `permissions: {}` at top level of live_test.yaml, release.yaml, and test.yaml
   - Added minimal job-level permissions: `packages: write` for jobs pushing to GHCR, `contents: read` for test job

3. **script-injection** (release.yaml):
   - Moved `${{ matrix.platform }}` to `PLATFORM` env var in Prepare step
   - Moved `${{ steps.build.outputs.digest }}` to `BUILD_DIGEST` env var in Export digest step
   - Moved `${{ env.REGISTRY_IMAGE }}` to `REGISTRY_IMAGE` env var in Create manifest list step
   - Moved `${{ env.REGISTRY_IMAGE }}` and `${{ steps.meta.outputs.version }}` to env vars in Inspect image step

4. **github-env-injection** (release.yaml Prepare step):
   - Added sanitization via `printf '%s' "$PLATFORM" | tr -d '\n\r'` before writing to GITHUB_ENV

