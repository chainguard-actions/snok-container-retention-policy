<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **snok--container-retention-policy/v2.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `actions/setup-python@v4`, which is pinned to a mutable version tag (`@v4`) rather than an immutable 40-character commit SHA. This means the action could silently change under a supply-chain attack. It should be pinned to a full SHA, e.g. `actions/setup-python@<40-char-sha> # v4`.

Locations:

- `action.yml:65`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Run Container Retention Policy' step directly interpolates a GitHub Actions expression inside the shell command string: `python ${{ github.action_path }}/main.py`. Any `${{ ... }}` expression embedded directly in a `run:` script undergoes YAML template substitution before the shell parses the string, meaning shell metacharacters in the value would be interpreted by the shell. This should be replaced with the equivalent environment variable `$GITHUB_ACTION_PATH` (which is already set by the runner), e.g. `python "$GITHUB_ACTION_PATH/main.py"`.

Locations:

- `action.yml:79`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned `actions/setup-python@v4` to full commit SHA `7f4fc3e22c37d6ff65e88745f38bd3157c663f7c` with `# v4` comment for readability. 2. Replaced `python ${{ github.action_path }}/main.py` with `python "$GITHUB_ACTION_PATH/main.py"`, using the runner-provided `$GITHUB_ACTION_PATH` environment variable instead of the inline GitHub Actions expression to prevent shell metacharacter injection.

