<!-- markdownlint-disable -->

# Hardening Report: snok--container-retention-policy/v2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **snok--container-retention-policy/v2.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Run Container Retention Policy' step in action.yml directly interpolates `${{ github.action_path }}` inside a `run:` shell command string: `python ${{ github.action_path }}/main.py`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the expression is substituted by the Actions template engine before the shell ever sees the string, allowing the value to break out of the intended command structure.

Locations:

- `action.yml:68`

### unpinned-uses (severity: high)

The composite action uses `actions/setup-python@v4`, which is pinned to a mutable version tag (`@v4`) rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, making this a supply-chain risk. It should be replaced with a full SHA pin, e.g. `actions/setup-python@<40-char-sha> # v4`.

Locations:

- `action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned actions/setup-python from mutable tag @v4 to immutable SHA @7f4fc3e22c37d6ff65e88745f38bd3157c663f7c # v4. 2. Fixed script-injection in the 'Run Container Retention Policy' step: moved ${{ github.action_path }} out of the run: shell string into the env: block as ACTION_PATH, then referenced it as "$ACTION_PATH/main.py" in the shell script.

