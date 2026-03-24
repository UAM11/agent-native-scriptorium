# Request Examples

## Full Sync

User request:

```text
This round, treat vault as the primary repo and fully sync the shared content to scriptorium. Keep target-only files and do not delete anything.
```

Interpretation:

- primary repo: `agent-native-vault`
- target repo: `agent-native-scriptorium`
- mode: full
- include scope: shared surface
- exclude scope: repo-exclusive content
- overwrite policy: non-destructive

## Partial Sync By Paths

User request:

```text
Treat scriptorium as primary. Sync only templates/, skills/repo-sync/, and playbooks/repository/, but do not sync CHANGELOG.md.
```

Interpretation:

- primary repo: `agent-native-scriptorium`
- target repo: `agent-native-vault`
- mode: partial
- include scope: named paths
- exclude scope: `CHANGELOG.md`
- overwrite policy: preserve target-only files

## Partial Sync By Topic

User request:

```text
Use vault as primary and sync the metadata-field and repo-sync workflow changes to public, but exclude any private ideas.
```

Interpretation:

- primary repo: `agent-native-vault`
- target repo: `agent-native-scriptorium`
- mode: partial
- include scope: files implied by the metadata-field and repo-sync topics
- exclude scope: private ideas and other vault-only content
- overwrite policy: preserve target-only files

Action note:

- derive the candidate file list from the current task or recent diffs
- if the topic resolves to multiple plausible file sets, show a short proposed scope first

## Explicit Mirror

User request:

```text
Use scriptorium as primary and mirror the shared surface into vault. Delete extra shared files on the target if they are not in the source.
```

Interpretation:

- primary repo: `agent-native-scriptorium`
- target repo: `agent-native-vault`
- mode: full
- include scope: shared surface
- exclude scope: repo-exclusive content unless explicitly overridden
- overwrite policy: destructive mirror

Action note:

- mirror deletion is allowed only because the user explicitly requested it
