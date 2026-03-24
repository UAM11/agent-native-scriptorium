# Sync Model

## Core Terms

- **Primary repo**: the source of truth for the current task or sync request
- **Target repo**: the paired repository receiving synced content
- **Full sync**: align the shared surface from the primary repo to the target repo
- **Partial sync**: sync only the files, paths, topics, or recent changes the user names
- **Mirror sync**: an explicit destructive form of full sync that may delete target-side shared files absent from the source

## Safe Defaults

- Do not sync both repos automatically unless the user explicitly asks.
- Treat sync as one-way from the primary repo to the target repo.
- Preserve target-only files by default.
- If the target repo is public, exclude private or unsanitized content unless the user explicitly asks to publish a sanitized version.
- Never sync secrets, tokens, credentials, personal IDs, or other high-risk sensitive data.

## Current File Classification

### Direct-sync by default

These paths are normally safe to copy from the primary repo to the target repo:

- `ideas/README.md`
- `ideas/portable-agent-memory-system.md`
- `learning/README.md`
- `playbooks/github/`
- `playbooks/repository/metadata-field-reference.md`
- `playbooks/repository/repo-sync-workflow.md`
- `prompts/README.md`
- `skills/README.md`
- `skills/github-push-stable-setup/`
- `skills/repo-sync/`
- `skills/template/`
- `solutions/README.md`
- `templates/`

### Paired-sync by default

These files should stay aligned in meaning, but not be blindly overwritten because each repo has role-specific framing:

- `README.md`
- `README.zh-CN.md`
- `AGENTS.md`
- `USER.md`
- `PLAN.md`
- `CHANGELOG.md`
- `playbooks/repository/dual-repo-workflow.md`

### Repo-exclusive by default

- `vault`-only:
  - `ideas/private-vault-privacy-test.md`
  - anything the user marks as private, vault-only, unsanitized, or not for publication
- `scriptorium`-only:
  - currently no fixed path
  - any file the user explicitly says should remain public-only

### Never-sync

- `.git/`
- editor junk, temporary files, local cache
- secrets, tokens, credentials, or high-risk personal data

## Parsing a Natural-Language Sync Request

Resolve these fields before editing:

- primary repo
- target repo
- mode: full or partial
- include scope
- exclude scope
- overwrite policy

If the user names a topic rather than file paths, derive the candidate file set from the current task context or recent diffs. If multiple interpretations remain plausible, show a short proposed sync scope before editing.

## Shortcut Grammar

Accept this compressed but readable format:

```text
sync <source> -> <target> <mode> [scope] [exclude] [policy]
```

Alias table:

- source/target:
  - `vault` / `private` / `pri`
  - `scriptorium` / `public` / `pub`
- mode:
  - `full`
  - `part` / `partial`

Common fields:

- scope:
  - `path:...`
  - `topic:...`
  - `change:this-round`
- exclude:
  - `exclude:...`
- policy:
  - `keep-exclusive`
  - `no-delete`
  - `mirror`
  - `public-safe`

Treat this syntax as a convenience layer, not a rigid parser. If the meaning is clear, proceed. If multiple interpretations remain plausible, summarize the proposed sync scope before editing.
