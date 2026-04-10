# agent-native-scriptorium

English | [简体中文](README.zh-CN.md)

> A delirious scriptorium where agents, bugs, midnight questions, and stray sparks are fermented into strange manuals for learning, making, and surviving the future.

`agent-native-scriptorium` is the public, Git-first half of an agent-native external brain and growth operating system. It combines a journal layer, mission-oriented track workspaces, reusable knowledge assets, and lightweight metrics so that long-term learning and job-hunt work do not disappear into chat history. When a piece of work is still private, unfinished, or not safe to publish, it should stay out of this public repo until it has been sanitized or promoted intentionally.

## What Belongs Here

- Public-safe daily and weekly journals of meaningful work
- Mission-oriented track notes for learning, projects, interviews, and job-hunt preparation
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

- `AGENT_INDEX.md`: the fastest onboarding index for a new agent entering the repo
- `NOW.md`: a lightweight current-state snapshot for cross-thread and cross-harness continuity
- `journal/`: public-safe daily and weekly progress logs
- `tracks/`: mission-oriented workspaces for ongoing lines of effort
- `playbooks/`: operational guides and step-by-step fixes
- `playbooks/repository/`: repository governance and cross-repo workflows
- `reviews/`: periodic reviews that summarize progress across logs and tracks
- `metrics/`: lightweight derived snapshots and dashboards
- `skills/`: agent-oriented operational packaging for reusable procedures
- `ideas/`: incubating ideas across product, research, life, work, entrepreneurship, and everyday imagination
- `solutions/`: concrete problem/solution writeups
- `learning/`: study notes, conceptual summaries, reading traces
- `prompts/`: reusable prompt patterns and workflows
- `templates/`: note templates and documentation scaffolds
- `CHANGELOG.md`: a running record of meaningful repository changes
- `USER.md`: personalized user preferences for how this archive should be maintained

## Current Holdings

- [Agent Index](AGENT_INDEX.md): the shortest agent-facing entry point for understanding the repo, current mode, and default workflow
- [Now](NOW.md): the current-state snapshot for the active mission, priorities, and guardrails
- [Growth Loop Workflow](playbooks/repository/growth-loop-workflow.md): the lightweight operating model for capture, promotion, review, and metrics
- [Job Hunt 2026 Track](tracks/job-hunt-2026/README.md): the active mission workspace for RL, agents, resume, interviews, projects, and applications
- [New Agent Bootstrap Prompt](prompts/new-agent-bootstrap.md): a reusable prompt for bringing a new thread or harness up to speed before it starts editing
- [Daily Journal Template](templates/daily-journal-template.md): the public-safe daily session template
- [Weekly Review Template](templates/weekly-review-template.md): the roll-up template for weekly reflection and progress checks
- [Harness Engineering Primer and Evolution Comparison](learning/harness-engineering-primer-and-evolution-comparison.md): a lightweight learning note that compares `Prompt Engineering`, `Context Engineering`, and `Harness Engineering`, and outlines the six-layer shape of a mature harness
- [LLM Streaming and LangGraph Streaming](learning/llm-streaming-and-langgraph-streaming.md): a learning note on `prefill`, `decode`, `KV cache`, `SSE`, and LangGraph `messages / updates / custom` streaming
- [Nanobot Agent Control Center Analysis](learning/nanobot-agent-control-center-analysis.md): a structured reading of `nanobot`'s main agent, control center, scheduling, queueing, and context management, with a concrete design proposal for a stronger service-control system
- [Data Flywheels, Agentic RL, and Vertical AI Agents](learning/data-flywheel-agentic-rl-and-vertical-ai-agents.md): a learning note on what data flywheels really are, how they differ from raw data accumulation, where `RFT` and agentic RL fit, and how support, legal, and coding agents can build compounding improvement loops
- [GitHub Push Stable Setup](playbooks/github/github-push-stable-setup.md): a verified fix for unstable GitHub `HTTPS` pushes by routing Git traffic through `SSH over 443`
- [GitHub Push Stable Setup Skill](skills/github-push-stable-setup/SKILL.md): the same procedure packaged in a more agent-executable form
- [Portable Agent Memory System](ideas/portable-agent-memory-system.md): an incubating idea for turning high-value chat outputs into portable, iterable knowledge assets
- [Dual Repo Workflow](playbooks/repository/dual-repo-workflow.md): the routing and promotion rules for operating the public `scriptorium` alongside the private `vault`
- [Repo Sync Workflow](playbooks/repository/repo-sync-workflow.md): a user-directed method for choosing a primary repo and running full or partial sync between the paired archives
- [Repo Sync Skill](skills/repo-sync/SKILL.md): the agent-facing sync procedure for parsing natural-language sync requests safely

## Maintenance Philosophy

- Prefer preservation over reinvention
- Prefer additive updates over broad rewrites
- Prefer lightweight structure over deep placeholder-heavy trees
- Capture progress first, then promote only what deserves long-term status
- Keep notes reusable by a future self and by future agents
- Separate raw context from distilled conclusions
- Record dates and verification details whenever advice is environment-sensitive
- Treat token usage as a secondary signal, not the main score
- Keep public journaling public-safe; anonymize sensitive job-hunt details

## How Humans And Agents Should Contribute

1. Capture the session in `journal/` while it is still fresh.
2. Anchor it to the right mission or track in `tracks/`.
3. Promote durable value into `learning/`, `solutions/`, `playbooks/`, or `skills/`.
4. Roll up the week in `journal/weekly/` and periodically summarize in `reviews/`.
5. Record lightweight derived metrics in `metrics/` when they help decision-making.
6. Keep filenames stable, descriptive, and machine-friendly.
7. When adding a high-value entry point, update `README.md` and `README.zh-CN.md` so the archive remains legible at a glance.

## Operating Rules For Agents

- Start from [Agent Index](AGENT_INDEX.md) when a new agent or harness enters the repo.
- See [AGENTS.md](AGENTS.md) for repository-wide operating rules that apply to any agent.
- See [USER.md](USER.md) for user-specific preferences, defaults, and maintenance style choices.
- Update [CHANGELOG.md](CHANGELOG.md) whenever you make a meaningful repository change.

## Roadmap

See [PLAN.md](PLAN.md) for the current build-out plan.
