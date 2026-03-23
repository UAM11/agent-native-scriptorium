# agent-native-scriptorium

> A delirious scriptorium where agents, bugs, midnight questions, and stray sparks are fermented into strange manuals for learning, making, and surviving the future.

`agent-native-scriptorium` is the public shelf of a dual-repository system for agent-born solutions, distilled study notes, reusable playbooks, and practical learning artifacts. The goal is simple: when a useful answer appears once, we do not let it vanish into chat history. When that answer is still sensitive, unfinished, or too personal to share, it should stay in the private vault until it is ready to be promoted.

## What Belongs Here

- Shareable technical incidents and verified fixes that are worth preserving
- Study notes refined from agent-assisted learning sessions
- Reusable workflows, prompts, skills, and operational playbooks
- Public-safe idea notes that are ready to be seen, cited, or extended
- Structured records that future agents can extend instead of recreating

## Dual-Repo Topology

- `agent-native-scriptorium`: the public, shareable layer for sanitized and reusable knowledge
- `agent-native-vault`: the private incubation layer for unfinished, sensitive, or strongly personal but non-secret material
- Default routing rule: if privacy is uncertain, start in the vault
- Promotion rule: only move material here after distilling it into a standalone note and removing private details
- Workflow reference: see `playbooks/repository/dual-repo-workflow.md`

## Repository Shape

- `playbooks/`: operational guides and step-by-step fixes
- `playbooks/repository/`: repository governance and cross-repo workflows
- `skills/`: agent-oriented operational packaging for reusable procedures
- `ideas/`: incubating ideas across product, research, life, work, entrepreneurship, and everyday imagination
- `solutions/`: concrete problem/solution writeups
- `learning/`: study notes, conceptual summaries, reading traces
- `prompts/`: reusable prompt patterns and workflows
- `templates/`: note templates and documentation scaffolds
- `CHANGELOG.md`: a running record of meaningful repository changes
- `USER.md`: personalized user preferences for how this archive should be maintained

## Current Holdings

- [GitHub Push Stable Setup](playbooks/github/github-push-stable-setup.md): a verified fix for unstable GitHub `HTTPS` pushes by routing Git traffic through `SSH over 443`
- [GitHub Push Stable Setup Skill](skills/github-push-stable-setup/SKILL.md): the same procedure packaged in a more agent-executable form
- [Portable Agent Memory System](ideas/portable-agent-memory-system.md): an incubating idea for turning high-value chat outputs into portable, iterable knowledge assets
- [Dual Repo Workflow](playbooks/repository/dual-repo-workflow.md): the routing and promotion rules for operating the public `scriptorium` alongside the private `vault`

## Maintenance Philosophy

- Prefer preservation over reinvention
- Prefer additive updates over broad rewrites
- Keep notes reusable by a future self and by future agents
- Separate raw context from distilled conclusions
- Record dates and verification details whenever advice is environment-sensitive

## How Humans And Agents Should Contribute

1. Capture the useful thing while it is still fresh.
2. Distill it into a reusable note instead of a conversation fragment.
3. File it into the best-matching folder.
4. Keep filenames stable, descriptive, and machine-friendly.
5. When adding a high-value artifact, update this `README` so the archive remains legible at a glance.

## Operating Rules For Agents

- See [AGENTS.md](AGENTS.md) for repository-wide operating rules that apply to any agent.
- See [USER.md](USER.md) for user-specific preferences, defaults, and maintenance style choices.
- Update [CHANGELOG.md](CHANGELOG.md) whenever you make a meaningful repository change.

## Roadmap

See [PLAN.md](PLAN.md) for the current build-out plan.
