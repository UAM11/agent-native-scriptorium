# AGENTS.md

This repository is a long-term knowledge archive. Treat it like a maintained library, not a scratchpad.

## Mission

Preserve high-value outputs produced with agents so they remain reusable, composable, and understandable by future humans and future agents.

## Default Rules

- Preserve existing material unless there is a clear reason to restructure it.
- Prefer additive edits over destructive rewrites.
- Do not remove historical context if it still explains why a note exists.
- Keep filenames in lowercase kebab-case when creating new files.
- Markdown is the default format unless another format is clearly better.
- Never commit secrets, credentials, tokens, private keys, cookies, or raw personal data.

## Folder Intent

- `playbooks/`: operational procedures and verified step-by-step fixes
- `solutions/`: specific problems paired with reasoning and outcome
- `learning/`: study notes, concept maps, and explanatory material
- `prompts/`: reusable prompt patterns, workflows, and interaction recipes
- `templates/`: reusable note structures for consistency

## What A Good Note Should Contain

Each durable note should usually include:

- what problem or question it addresses
- why the issue mattered
- the solution or distilled explanation
- verification details when the content is environment-sensitive
- enough context that a future reader does not need the original chat

## Recommended Metadata Block

Use this near the top of durable notes when it helps:

```md
- Created: YYYY-MM-DD
- Updated: YYYY-MM-DD
- Type: playbook | solution | learning | prompt
- Status: draft | verified | needs-review
- Tags: tag1, tag2, tag3
- Source: short origin note
```

## Maintenance Workflow

1. Read the existing repository shape before adding new files.
2. Place new material in the most appropriate folder.
3. Use the template in `templates/knowledge-note-template.md` when starting from scratch.
4. Update `README.md` when the new artifact is a top-level, high-value entry point.
5. If a note supersedes another note, link the older note to the newer one instead of silently replacing history.

## Writing Style

- Optimize for future reuse, not chat-like immediacy.
- Be concrete.
- Keep procedures executable.
- Mark assumptions clearly.
- If a claim may go stale, say how and when it was verified.

## Change Discipline

- Keep commits focused.
- Avoid large drive-by rewrites.
- If you introduce a new convention, document it in `README.md`, `PLAN.md`, or this file.
