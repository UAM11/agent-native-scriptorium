# GitHub Push Stable Setup

- Created: 2026-03-23
- Updated: 2026-03-23
- Type: playbook
- Status: verified
- Tags: github, git, ssh, networking, windows
- Source: verified from a real debugging session on Windows

## Context

On this machine, `git push` to GitHub over `HTTPS` was intermittently failing even though GitHub itself was reachable and authentication was valid.

Typical error:

```text
fatal: unable to access 'https://github.com/...': Recv failure: Connection was reset
```

## Diagnosis

The issue was not a bad remote URL or missing repository permissions.

What was verified:

- `github.com:443` was reachable
- `git ls-remote` worked
- `git push --dry-run` worked
- real `git push` over `HTTPS` failed during the upload phase with connection resets

This strongly suggested an unstable `HTTPS` Git transport path rather than a GitHub account problem.

## Distilled Fix

Route GitHub traffic through `SSH over 443`, and use Windows built-in `OpenSSH` instead of Git for Windows' bundled SSH.

Why this helped:

- `443` is usually easier to pass through restricted or unstable networks than standard SSH port `22`
- `SSH over 443` avoids the broken `HTTPS` push path
- Windows built-in `OpenSSH` proved more stable in this environment

## Implemented Global Setup

### Global Git config

Written to `C:\Users\UAMC\.gitconfig`:

```ini
[core]
    sshCommand = C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:/Users/UAMC/git-tools/git-ssh-github-443.ps1

[url "git@github.com:"]
    insteadOf = https://github.com/
    insteadOf = http://github.com/
```

Effect:

- GitHub `https://github.com/...` remotes are transparently rewritten to `git@github.com:...`
- Git SSH calls go through a GitHub-specific wrapper

### SSH wrapper

Wrapper path:

```text
C:\Users\UAMC\git-tools\git-ssh-github-443.ps1
```

Behavior:

- if the target host is `github.com` or `ssh.github.com`
- force the connection to `git@ssh.github.com`
- force port `443`
- add `StrictHostKeyChecking=accept-new`
- use `C:\Windows\System32\OpenSSH\ssh.exe`

For non-GitHub SSH targets, the wrapper passes arguments through without forcing port `443`.

## Verification Commands

```bash
git config --global --get core.sshCommand
git config --global --get-regexp '^url\\..*\\.insteadOf$'
git ls-remote https://github.com/UAM11/Master-Thesis.git
git ls-remote git@github.com:UAM11/Master-Thesis.git
```

If both `ls-remote` commands return commit hashes, the GitHub SSH-over-443 path is active.

## Verified Outcome

The affected repository was successfully pushed after switching away from `HTTPS`:

```text
To ssh.github.com:UAM11/Master-Thesis.git
   2d24ace..3bd4bff  main -> main
```

## Rollback

To remove the global Git configuration:

```bash
git config --global --unset core.sshCommand
git config --global --unset-all url.git@github.com:.insteadOf
```

To fully remove the wrapper, also delete:

```text
C:\Users\UAMC\git-tools\git-ssh-github-443.ps1
```

## Caveats

- This fix is tuned for GitHub, not for every SSH host.
- Network environments can change; if `HTTPS` later becomes stable, this setup can still remain in place.
- If the local SSH key changes, GitHub SSH authentication must still be valid.
