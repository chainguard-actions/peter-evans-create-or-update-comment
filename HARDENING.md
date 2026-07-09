<!-- markdownlint-disable -->

# Hardening Report: peter-evans--create-or-update-comment/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--create-or-update-comment/v4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references in workflow files use mutable version tags (e.g. @v3, @v4, @v5, @v1.9) instead of pinned 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Affected references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, actions/download-artifact@v4, peter-evans/enable-pull-request-automerge@v3, peter-evans/slash-command-dispatch@v3, chuhlomin/render-template@v1.9, peter-evans/create-or-update-comment@v3, peter-evans/create-pull-request@v5.

Locations:

- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/ci.yml:25`
- `.github/workflows/slash-command-dispatch.yml:10`
- `.github/workflows/test-command.yml:43`
- `.github/workflows/test-v3.yml:7`
- `.github/workflows/update-major-version.yml:21`

### script-injection (severity: high)

GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings, violating rule (a). This allows an attacker to inject arbitrary shell commands via the interpolated values before the shell ever sees them.

- ci.yml line 45: `if [[ "${{ github.event_name }}" == "pull_request" ]]` — ${{ github.event_name }} interpolated directly in shell.
- ci.yml line 46: `echo "issue-number=${{ github.event.number }}" >> $GITHUB_OUTPUT` — ${{ github.event.number }} interpolated directly in shell.
- test-command.yml line 13: `repository=${{ github.event.client_payload.slash_command.repository }}` — attacker-controlled payload interpolated directly.
- test-command.yml line 14: `repository=${{ github.repository }}` — interpolated directly.
- test-command.yml line 16: `branch=${{ github.event.client_payload.slash_command.branch }}` — attacker-controlled payload interpolated directly.
- update-major-version.yml line 30: `git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` — workflow_dispatch inputs interpolated directly.
- update-major-version.yml line 32: `git push origin ${{ github.event.inputs.main_version }} --force` — workflow_dispatch input interpolated directly.

Locations:

- `.github/workflows/ci.yml:45`
- `.github/workflows/ci.yml:46`
- `.github/workflows/test-command.yml:13`
- `.github/workflows/test-command.yml:14`
- `.github/workflows/test-command.yml:16`
- `.github/workflows/update-major-version.yml:30`
- `.github/workflows/update-major-version.yml:32`

### github-env-injection (severity: high)

Untrusted github context values are written to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

- ci.yml line 46: `echo "issue-number=${{ github.event.number }}" >> $GITHUB_OUTPUT` directly writes the github.event.number expression to GITHUB_OUTPUT with no newline sanitization.
- test-command.yml line 15: `echo "repository=$repository" >> $GITHUB_OUTPUT` writes $repository, which was assigned from `${{ github.event.client_payload.slash_command.repository }}` on line 13, to GITHUB_OUTPUT without sanitization.
- test-command.yml line 18: `echo "branch=$branch" >> $GITHUB_OUTPUT` writes $branch, which was assigned from `${{ github.event.client_payload.slash_command.branch }}` on line 16, to GITHUB_OUTPUT without sanitization. An attacker could inject newlines to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `.github/workflows/ci.yml:46`
- `.github/workflows/test-command.yml:15`
- `.github/workflows/test-command.yml:18`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: key and no job-level permissions: blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for all scopes), violating the principle of least privilege.

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

Fixed all 6 workflow files:

1. automerge-dependabot.yml: Pinned peter-evans/enable-pull-request-automerge@v3 to SHA a660677d5469627102a1c1e11409dd063606628d. Added permissions: pull-requests: write, contents: write.

2. ci.yml: Pinned actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, actions/download-artifact@v4, peter-evans/create-pull-request@v5 to their respective SHAs. Fixed script-injection and github-env-injection in the vars step by moving github.event_name and github.event.number into env block and using printf/tr sanitization before writing to GITHUB_OUTPUT.

3. slash-command-dispatch.yml: Pinned peter-evans/slash-command-dispatch@v3 to SHA f996d7b7aae9059759ac55e978cff76d91853301. Added permissions: contents: read.

4. test-command.yml: Pinned actions/checkout@v4, chuhlomin/render-template@v1.9, peter-evans/create-or-update-comment@v3 to their respective SHAs. Fixed script-injection and github-env-injection in the vars step by moving all payload expressions into env block and using printf/tr sanitization before writing to GITHUB_OUTPUT. Added permissions: contents: read, issues: write.

5. test-v3.yml: Pinned actions/checkout@v4, peter-evans/create-or-update-comment@v3 (x5), chuhlomin/render-template@v1.9 to their respective SHAs. Added permissions: issues: write, pull-requests: write.

6. update-major-version.yml: Pinned actions/checkout@v4 to SHA. Fixed script-injection in git tag and git push steps by moving github.event.inputs.main_version and github.event.inputs.target into env blocks. Added permissions: contents: write.

