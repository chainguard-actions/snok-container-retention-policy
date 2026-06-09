# Hardening Report: snok--container-retention-policy/v2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **snok--container-retention-policy/v2.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses `actions/setup-python@v4`, which is a mutable tag reference rather than a pinned 40-character commit SHA. This means the action could silently change if the tag is moved, enabling supply-chain attacks.

Locations:

- `action.yml:63`

### script-injection (severity: high)

The 'Run Container Retention Policy' step in action.yml directly interpolates `${{ github.action_path }}` inside the `run:` shell command string (`python ${{ github.action_path }}/main.py ...`). GitHub Actions expressions must be assigned to an environment variable first and then referenced as `$ENV_VAR` in the shell command, rather than being interpolated directly into the run block.

Locations:

- `action.yml:70`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned `actions/setup-python@v4` to its full commit SHA `7f4fc3e22c37d6ff65e88745f38bd3157c663f7c` with the tag preserved as a comment. 2. Moved `${{ github.action_path }}` from the `run:` shell string into the step's `env:` block as `ACTION_PATH`, and updated the shell command to reference `"$ACTION_PATH/main.py"` instead.

