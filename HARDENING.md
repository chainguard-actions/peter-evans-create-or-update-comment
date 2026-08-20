<!-- markdownlint-disable -->

# Hardening Report: peter-evans--create-or-update-comment/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--create-or-update-comment/v5.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tag-based refs instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if a tag is moved or a dependency is compromised. Unpinned refs found include: `actions/checkout@v5`, `actions/setup-node@v5`, `actions/upload-artifact@v4`, `actions/download-artifact@v5`, `peter-evans/create-pull-request@v7`, `peter-evans/enable-pull-request-automerge@v3`, `peter-evans/slash-command-dispatch@v4`, `chuhlomin/render-template@v1.10`, `peter-evans/create-or-update-comment@v4`.

Locations:

- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:34`
- `.github/workflows/ci.yml:38`
- `.github/workflows/ci.yml:88`
- `.github/workflows/automerge-dependabot.yml:7`
- `.github/workflows/slash-command-dispatch.yml:8`
- `.github/workflows/test-command.yml:19`
- `.github/workflows/test-command.yml:56`
- `.github/workflows/test-command.yml:72`
- `.github/workflows/test-v3.yml:5`
- `.github/workflows/test-v3.yml:10`
- `.github/workflows/test-v3.yml:22`
- `.github/workflows/test-v3.yml:33`
- `.github/workflows/test-v3.yml:43`
- `.github/workflows/test-v3.yml:50`
- `.github/workflows/test-v3.yml:57`
- `.github/workflows/update-major-version.yml:18`

### script-injection (severity: high)

Multiple `run:` blocks interpolate GitHub Actions expressions (`${{ ... }}`) directly into shell commands, enabling script injection. Sub-rule (a) violations:

- ci.yml: `if [[ "${{ github.event_name }}" == "pull_request" ]]` and `echo "issue-number=${{ github.event.number }}" >> $GITHUB_OUTPUT` — github context values interpolated directly into shell.
- test-command.yml: `repository=${{ github.event.client_payload.slash_command.repository }}`, `repository=${{ github.repository }}`, `branch=${{ github.event.client_payload.slash_command.branch }}` — attacker-controllable payload values interpolated directly into shell.
- update-major-version.yml: `git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` and `git push origin ${{ github.event.inputs.main_version }} --force` — user-supplied workflow_dispatch inputs interpolated directly into shell commands without quoting or sanitization.

Locations:

- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:44`
- `.github/workflows/test-command.yml:10`
- `.github/workflows/test-command.yml:11`
- `.github/workflows/test-command.yml:13`
- `.github/workflows/update-major-version.yml:24`
- `.github/workflows/update-major-version.yml:26`

### github-env-injection (severity: high)

Untrusted values are written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

- ci.yml: `echo "issue-number=${{ github.event.number }}" >> $GITHUB_OUTPUT` writes a raw github context value directly to GITHUB_OUTPUT. An attacker controlling the PR number field could inject newlines to poison the output.
- test-command.yml: Variables `$repository` and `$branch` are set directly from `${{ github.event.client_payload.slash_command.repository }}` and `${{ github.event.client_payload.slash_command.branch }}` (attacker-controlled via repository_dispatch payload), then written to GITHUB_OUTPUT via `echo "repository=$repository" >> $GITHUB_OUTPUT` and `echo "branch=$branch" >> $GITHUB_OUTPUT` without sanitization.

Locations:

- `.github/workflows/ci.yml:44`
- `.github/workflows/test-command.yml:14`
- `.github/workflows/test-command.yml:16`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` key and no job-level `permissions:` keys. Without explicit permissions, workflows run with the default token permissions (which may be broad, e.g., `write` on all scopes depending on repository settings). Each of these files should declare minimal required permissions.

- automerge-dependabot.yml: no permissions declared
- slash-command-dispatch.yml: no permissions declared
- test-command.yml: no permissions declared
- test-v3.yml: no permissions declared
- update-major-version.yml: no permissions declared

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/test-command.yml:1`
- `.github/workflows/test-v3.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 6 workflow files:

**unpinned-uses**: Pinned all 9 action references to full SHA digests:
- actions/checkout@v5 → @fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09
- actions/setup-node@v5 → @a0853c24544627f65ddf259abe73b1d18a591444
- actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- actions/download-artifact@v5 → @634f93cb2916e3fdff6788551b99b062d0335ce0
- peter-evans/create-pull-request@v7 → @22a9089034f40e5a961c8808d113e2c98fb63676
- peter-evans/enable-pull-request-automerge@v3 → @a660677d5469627102a1c1e11409dd063606628d
- peter-evans/slash-command-dispatch@v4 → @13bc09769d122a64f75aa5037256f6f2d78be8c4
- chuhlomin/render-template@v1.10 → @807354a04d9300c9c2ac177c0aa41556c92b3f75
- peter-evans/create-or-update-comment@v4 → @71345be0265236311c031f5c7866368bd1eff043

**script-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks in ci.yml (github.event_name, github.event.number), test-command.yml (client_payload values, github.repository), and update-major-version.yml (workflow_dispatch inputs).

**github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT in ci.yml and test-command.yml.

**missing-permissions**: Added minimal permissions blocks to automerge-dependabot.yml (pull-requests: write, contents: write), slash-command-dispatch.yml (issues: read, pull-requests: read), test-command.yml (issues: write, pull-requests: write, contents: read), test-v3.yml (issues: write, pull-requests: write, contents: read), and update-major-version.yml (contents: write).

