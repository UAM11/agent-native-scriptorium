---
name: repo-sync
description: Coordinate one-way synchronization between `agent-native-scriptorium` and `agent-native-vault` from a user-designated primary repo. Use when the user wants to fully sync shared content, partially sync selected files or topics, or describe include/exclude rules in natural language. Do not use for blind publication of private content or destructive mirroring unless the user explicitly authorizes it.
---

# Repo Sync

Use this skill to turn a natural-language dual-repo sync request into a safe, reviewable action.

## Workflow

1. Identify the primary repo, target repo, sync mode, include scope, exclude scope, and overwrite policy.
2. Read `references/sync-model.md` for the default safety rules and the current file classification.
3. Read `references/request-examples.md` if the request is phrased by topic, recent change, or loosely specified intent.
4. For full sync, sync the shared surface from the primary repo to the paired repo, then align paired files semantically instead of blindly overwriting repo-specific wording.
5. For partial sync, sync only the requested paths, files, or the files implied by the requested topic or change set.
6. Preserve target-only files by default. Delete or mirror only when the user explicitly says to do so.
7. If the target repo is public, do not publish private or unsanitized content. Exclude it or rewrite it first.
8. Verify with `git status` and, when useful, `git diff --stat` in both repos. Report what changed and what was intentionally left unsynced.

## Success Criteria

- The chosen primary repo remains the source of truth for this task.
- The target repo receives only the requested shared content.
- Repo-exclusive or privacy-sensitive content stays unsynced unless the user explicitly overrides.
- The result is reviewable, non-destructive by default, and accurately summarized.

## Bundled References

- `references/sync-model.md`
- `references/request-examples.md`
