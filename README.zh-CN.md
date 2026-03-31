# agent-native-scriptorium

[English](README.md) | 简体中文

> 一个微醺却清醒的知识工坊。agent、bug、深夜问题与灵光乍现，都在这里慢慢发酵成可复用的手册、笔记与方法。

`agent-native-scriptorium` 现在更像一个公开的、Git-first 的 agent-native 外脑与成长操作系统。它把 `journal`、`tracks`、长期知识资产与轻量化指标放进同一套结构里，让学习、求职、项目推进与 agent 协作不再散落在聊天记录里。凡是还不适合公开、尚未脱敏、仍带有强个人上下文的内容，都不应直接进入这个公仓。

## 这里适合放什么

- 公开安全的日常工作日志与周度复盘
- 围绕长期主线展开的 track 工作区
- 值得公开分享的排障经验与修复方案
- 经过整理的学习笔记与概念说明
- 可复用的工作流、playbook、prompt 与 skill
- 适合公开展示、可继续延展的 idea 条目
- 能被未来的你，或未来的 agent，直接接住并继续推进的知识资产

## 双仓结构

- `agent-native-scriptorium`：公开层，用于放可分享、可复用、已脱敏的内容
- `agent-native-vault`：私有层，用于放未公开、敏感、半成品或带有强个人上下文的内容
- 默认规则：拿不准能不能公开时，先放 `vault`
- 晋升规则：只有在内容被提炼成可独立成立、且已移除私密细节后，才迁入 `scriptorium`
- 工作流说明：见 `playbooks/repository/dual-repo-workflow.md`

## 仓库结构

- `journal/`：公开安全的日常日志与周度记录
- `tracks/`：围绕长期主线组织的工作区
- `playbooks/`：操作指南、排障流程与经验证的方法
- `playbooks/repository/`：仓库治理、双仓协作、元数据等维护规则
- `reviews/`：跨日志与 track 的周期复盘
- `metrics/`：轻量化的派生指标与仪表板产物
- `skills/`：面向 agent 复用的操作封装
- `ideas/`：产品、科研、生活、工作、创业与日常灵感的孵化区
- `solutions/`：问题、分析与最终解法的整理稿
- `learning/`：学习笔记、阅读痕迹与概念解释
- `prompts/`：可复用的提示词与交互配方
- `templates/`：模板、脚手架与统一格式
- `CHANGELOG.md`：记录仓库重要变更
- `USER.md`：记录用户偏好与维护风格

## 当前馆藏

- [成长闭环工作流](playbooks/repository/growth-loop-workflow.md)：把日志、track、资产、复盘与指标串起来的最小工作模型
- [Job Hunt 2026 主线](tracks/job-hunt-2026/README.md)：当前围绕 RL、agents、简历、面试、项目与投递展开的主工作区
- [Daily Journal Template](templates/daily-journal-template.md)：公开安全的日常日志模板
- [Weekly Review Template](templates/weekly-review-template.md)：周度复盘模板
- [GitHub Push 稳定方案](playbooks/github/github-push-stable-setup.md)：一个已经验证可用的 GitHub 推送修复方案
- [GitHub Push Stable Setup Skill](skills/github-push-stable-setup/SKILL.md)：同一方案的 agent 可执行封装
- [可迁移的 Agent 记忆系统](ideas/portable-agent-memory-system.md)：关于“把高价值对话沉淀成长期资产”的想法条目
- [双仓协作工作流](playbooks/repository/dual-repo-workflow.md)：public/private 双仓的路由与晋升规则
- [知识条目元数据字段说明](playbooks/repository/metadata-field-reference.md)：`Model`、`Harness` 等元数据字段的统一定义
- [双仓同步工作流](playbooks/repository/repo-sync-workflow.md)：允许用户指定主维护仓，并通过自然语言发起完全同步或部分同步
- [Repo Sync Skill](skills/repo-sync/SKILL.md)：把自然语言同步请求转成安全可执行操作的 agent 技能

## 维护哲学

- 让资料沉淀，而不是让答案淹没在会话里
- 尽量做增量整理，而不是大面积重写
- 让结构保持轻量，不用过深的目录和过多的空占位文件
- 先记录真实推进，再把真正有价值的部分晋升为长期资产
- 让条目既能被人理解，也能被 agent 接续
- 把原始上下文和提炼后的结论区分开
- 涉及环境、工具、网络、平台差异时，尽量写清验证条件和日期
- token 可以记录，但更适合作为投入信号，而不是唯一目标
- 公仓中的日志默认要 public-safe，求职与面试细节应尽量匿名化

## 共建方式

1. 先把当天或当次 session 记进 `journal/`。
2. 再把它挂到合适的 `track` 上，而不是只留在零散日志里。
3. 当内容足够稳定、有复用价值时，再晋升到 `learning/`、`solutions/`、`playbooks/` 或 `skills/`。
4. 每周做一次周度复盘，必要时在 `reviews/` 里做更长周期总结。
5. 需要量化时，再把结果提炼进 `metrics/`，不要一开始就追求复杂指标。
6. 文件名保持稳定、明确、机器可处理。
7. 新增高价值入口时，同步更新 `README.md` 和 `README.zh-CN.md`，让仓库一眼可读。

如果只是想快速发起同步，也可以直接用短口令，例如：

```text
sync vault -> pub full keep-exclusive
```

## 给 Agent 的说明

- 通用协作规则见 `AGENTS.md`
- 用户偏好与默认值见 `USER.md`
- 每次有实质性改动时，更新 `CHANGELOG.md`

## 路线图

当前建设计划见 `PLAN.md`。
