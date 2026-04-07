# CHANGELOG

这个文件用于记录仓库中每一次有意义的变动。

约定：

- 不是每一个字符级修改都要记录，但每一次“有实际价值的内容变更”都应留下条目。
- 记录重点不是形式化流水账，而是帮助未来快速理解“仓库为什么变成现在这样”。

## 2026-03-23

### 仓库初始化

- 建立根目录 `README.md`
- 建立 `PLAN.md`
- 建立 `AGENTS.md`
- 建立 `templates/knowledge-note-template.md`
- 建立第一份知识条目 `playbooks/github/github-push-stable-setup.md`

### 文档语言与本地路径整理

- 根 `README.md` 保持英文
- 其余维护文档调整为中文
- 本地主工作副本迁移到 `F:\agent-native-scriptorium`

### 维护协议升级

- 新增 `CHANGELOG.md`，作为仓库变动日志
- 新增 `USER.md`，用于记录用户个性化偏好
- 在知识条目元数据中加入 `Model` 与 `Harness`
- 将 `AGENTS.md` 与 `USER.md` 的职责边界明确拆分
- 为 `github-push-stable-setup` 增加对应的 `skill` 版本

### 元数据与 Skill 规范化

- 将知识条目中的元数据字段名统一为英文
- 按 Anthropic skills 规范重写 `skills/github-push-stable-setup/SKILL.md`
- 为该 skill 增加 `references/` 辅助文档，采用渐进式加载结构

### Skill 维护规范升级

- 在 `AGENTS.md` 中明确规定：本仓库的 skill 默认遵循 Anthropic 官方 agent skills 规范
- 在 `skills/README.md` 中补充 skill 编写规则、推荐目录结构和官方参考链接
- 新增 `skills/template/SKILL.md`，作为后续新 skill 的默认起点

### Idea 孵化能力

- 新增 `ideas/` 目录，用于存放待孵化的产品方向与系统构想
- 记录第一条产品 idea：可迁移、可迭代的 agent-native 知识库/记忆系统

### Idea 目录范围扩展

- 将 `ideas/` 从偏产品导向的孵化区，扩展为通用灵感目录
- 明确 `ideas/` 可同时容纳产品、科研、生活、创业、工作与日常想法

### 双仓结构落地

- 建立 `agent-native-scriptorium` 与 `agent-native-vault` 的双仓协作边界
- 新增 `playbooks/repository/dual-repo-workflow.md`，明确内容路由与私仓晋升公仓流程
- 在 `README.md`、`AGENTS.md`、`USER.md`、`PLAN.md` 中补充双仓职责与维护偏好

## 2026-03-24

### 元数据字段命名修正

- 将错误的 `Hardness` 统一更正为 `Harness`
- 同步修正模板、现有知识条目与相关说明文档中的字段名

### 元数据字段说明补充

- 新增 `playbooks/repository/metadata-field-reference.md`，统一定义知识条目元数据字段
- 在模板中加入 `Model` 与 `Harness` 的速记说明，减少后续维护时的概念混淆
- 在 `AGENTS.md` 中加入元数据说明文档入口

### 双语 README 入口

- 保留英文 `README.md` 作为根入口
- 新增中文伴随版 `README.zh-CN.md`，用于中文协作与共建
- 在 `AGENTS.md` 与 `USER.md` 中补充双语 README 的同步维护约定

### 双仓同步工作流

- 新增 `playbooks/repository/repo-sync-workflow.md`，定义用户指定主维护仓的双仓同步方式
- 新增 `skills/repo-sync/`，将自然语言同步请求包装为可复用的 agent skill
- 在 `AGENTS.md`、`USER.md`、`PLAN.md`、双语 `README` 中补充同步规则、自然语言同步约定与入口说明

### 同步短口令

- 为双仓同步补充最小句式：`sync <源> -> <目标> <mode> [scope] [exclude] [policy]`
- 在 `repo-sync` playbook 与 skill references 中加入别名表、最短示例与口令解析说明
- 在入口文档中补充短口令入口，降低实际使用门槛

### 当前维护模式调整

- 将当前工作模式调整为“公仓优先”
- 默认只维护 `agent-native-scriptorium`
- `agent-native-vault` 暂时冻结，只有在用户明确要求时才重新参与维护或同步

### Changelog 日期校验规则

- 在 `AGENTS.md` 中补充 changelog 维护纪律：更新前先核对 `git log --oneline --date=short`
- 在同步工作流中补充 `CHANGELOG.md` 的日期校验注意事项，避免再按会话顺序误分日期

## 2026-03-31

### Growth OS V1 骨架

- 将项目定位进一步明确为一个 Git-first 的 agent-native 个人成长操作系统
- 新增 `journal/`、`tracks/`、`reviews/`、`metrics/` 四层骨架
- 新增 `playbooks/repository/growth-loop-workflow.md`，明确“日志 -> track -> 资产 -> 复盘 -> 指标”的最小闭环
- 为 `daily journal`、`weekly review`、`monthly review` 与 `track` 新增专用模板
- 落地 `tracks/job-hunt-2026/` 主线，并为 `rl`、`agents`、`resume`、`interviews`、`projects`、`applications` 建立轻量入口
- 在根 `README`、`README.zh-CN`、`AGENTS.md`、`USER.md`、`PLAN.md` 中同步更新新定位与维护约定

## 2026-04-01

### Agent Onboarding 入口层

- 新增 `AGENT_INDEX.md`，作为新 agent 进入仓库时的最短索引页
- 新增 `NOW.md`，承接当前维护模式、active mission、优先级与默认写入路径
- 新增 `prompts/new-agent-bootstrap.md`，用于在新线程或新 harness 中快速完成仓库自举
- 在 `README.md`、`README.zh-CN.md`、`AGENTS.md`、`USER.md`、`PLAN.md` 与 `prompts/README.md` 中补充对应入口与维护约定

### 第一篇真实 Daily Journal

- 新增 `journal/daily/2026-04-01.md`，记录仓库 onboarding 落地、`nanobot` 第一轮架构学习与简历修正
- 将 `PLAN.md` 中“第一篇真实的 daily journal”标记为已完成

## 2026-04-02

### Claw 组织实验 Idea

- 新增 `ideas/claw-organization-experiments.md`，记录关于“自给自足 claw / 科研团队 claw / 公司 claw”的三类长期实验设想

## 2026-04-03

### 句子与片段入口

- 新增 `ideas/quotes-and-fragments.md`，用于收纳值得保留的句子、片段与生活灵感

### 高收入用户切入口 Idea

- 新增 `ideas/premium-user-wedge.md`，记录“先挣有钱人的零花钱”这类偏商业化的用户切入思路

### 未来科研方向 Idea

- 新增 `ideas/future-research-directions.md`，记录 Agentic RL、数据飞轮、安全与隐私、推理加速与压缩、自进化等后续重点科研方向

## 2026-04-07

### 流式输出学习笔记

- 新增 `learning/llm-streaming-and-langgraph-streaming.md`，整理 `prefill / decode / KV cache / SSE` 及 LangGraph `messages / updates / custom` 流机制
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该学习笔记入口
- 重写该笔记的开头结构，用“模型层 / 传输层 / 框架层”的三层教程式引导来提升可读性
