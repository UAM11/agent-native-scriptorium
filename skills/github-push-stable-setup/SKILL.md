---
name: github-push-stable-setup
description: Diagnoses and stabilizes GitHub push failures on Windows by moving unreliable HTTPS Git transport to SSH over 443 and validating end-to-end connectivity. Use when git push to github.com fails with "Connection was reset", "Couldn't connect to server", timeout errors, or when dry-run works but the real push fails. Do not use for clear permission errors, missing repository access, or non-GitHub remotes.
metadata:
  author: UAM11
  version: 1.0.0
  category: git-networking
  tags: [github, git, ssh, windows, networking]
---

# GitHub Push Stable Setup

## What this skill does

This skill turns an unstable GitHub push path into a stable one on Windows systems.

Its default strategy is:

1. confirm the failure is transport-related rather than permission-related
2. move GitHub traffic away from unreliable HTTPS push
3. route GitHub Git traffic through SSH over port 443
4. validate with both read-only and real push operations

## When to use it

Use this skill when the target remote is GitHub and symptoms look like transport instability, for example:

- `fatal: unable to access 'https://github.com/...': Recv failure: Connection was reset`
- `Failed to connect to github.com port 443`
- `git push --dry-run` works but `git push` fails
- `git ls-remote` works but real upload fails

Do not use it as the primary path for:

- `403`, `404`, or explicit repository permission errors
- wrong remote URLs
- missing GitHub SSH keys
- non-GitHub remotes

## Workflow

### Step 1: Diagnose the failure class

First, confirm the issue is really about transport reliability.

Use `references/diagnosis-checklist.md` to separate:

- connectivity problems
- auth or permission problems
- HTTPS-upload-only failures

If the symptoms point to permission or repository access problems, stop and solve that first.

### Step 2: Apply the stable transport pattern

Use `references/windows-github-ssh-over-443.md` for the exact setup pattern.

Preferred approach:

- use SSH for GitHub remotes
- route SSH through `ssh.github.com:443`
- prefer Windows built-in `OpenSSH` if the bundled Git SSH implementation is unstable
- if needed, use `core.sshCommand` or a wrapper script to enforce the route

### Step 3: Validate progressively

Validate in this order:

1. `git ls-remote` against the GitHub remote
2. read-only remote access over the new path
3. real `git push`

Do not treat `git push --dry-run` as sufficient proof. The real upload path must succeed.

## Success criteria

The skill has succeeded when:

- GitHub remote access works over the new SSH path
- `git ls-remote` returns commit hashes
- a real `git push` succeeds without HTTPS reset errors

## Common failure modes

### SSH auth still fails

Likely cause:

- no valid GitHub SSH key is available

Action:

- stop treating this as a transport-only issue
- fix SSH authentication first

### The skill triggers but the problem is actually permissions

Likely cause:

- remote exists, but user lacks repo access

Action:

- verify the repository, account, and permission model before changing transport

### GitHub is not the remote

Action:

- do not apply the GitHub-specific SSH-over-443 pattern blindly

## Example triggers

- "git push keeps failing to GitHub on Windows"
- "github.com port 443 connection reset during push"
- "dry-run works but real push fails"
- "please stabilize GitHub push networking"

## Bundled references

- `references/diagnosis-checklist.md`
- `references/windows-github-ssh-over-443.md`

## Related archive note

- Human-readable playbook: `playbooks/github/github-push-stable-setup.md`
