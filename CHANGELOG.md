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

### 学习笔记模板升级

- 新增 `templates/learning-note-template.md`，用于承载更偏教程式、强调逻辑承接与分层解释的学习笔记
- 在 `templates/knowledge-note-template.md` 与 `learning/README.md` 中补充该模板入口
- 新增 `templates/README.md`，明确 `knowledge-note-template` 与 `learning-note-template` 的区别及模板选择规则
- 在 `AGENTS.md` 中补充模板选择规则，帮助新 agent 自动判断该调用哪个模板

### Daily Journal 回填

- 补写 `journal/daily/2026-04-02.md`，记录 OpenClaw 组织实验 idea 的形成与“经济效益”表述修正
- 补写 `journal/daily/2026-04-03.md`，记录句子片段入口、商业化切入口与未来科研方向的集中沉淀
- 新增 `journal/daily/2026-04-07.md`，记录流式输出学习、LangGraph streaming 笔记与 `nanobot` 学习路线重定向

## 2026-04-08

### Nanobot 中控系统学习笔记

- 新增 `learning/nanobot-agent-control-center-analysis.md`，系统拆解 `nanobot` 中与主 agent / 中控 / 调度 / 队列 / 上下文管理最相关的核心代码
- 在笔记中分别梳理每个部分的抽象设计逻辑、关键源码路径、值得学习之处与需要加强之处
- 基于 `nanobot`、Claude Code、OpenClaw 与 OpenAI harness engineering 的思路，外推出一版适用于“万能服务中控系统”的可执行设计方案
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该笔记入口

## 2026-04-10

### 当前状态与入口同步

- 更新 `NOW.md` 的快照日期、当前优先级、结构性里程碑与下一批优先内容，使其与现有 `journal/` 与 `learning/` 资产保持一致
- 更新 `PLAN.md` 的“近期优先补充”，移除已完成项，聚焦 weekly review、metrics snapshot、求职向 solution/playbook 与交叉链接
- 为 `prompts/new-agent-bootstrap.md` 补齐元数据块，使该 prompt 也遵循仓库的可追踪性约定

### 数据飞轮与 Agentic RL 学习笔记

- 新增 `learning/data-flywheel-agentic-rl-and-vertical-ai-agents.md`，系统整理数据飞轮、eval 飞轮、`RFT` 与 Agentic RL 的关系，并结合 TikTok、Google Maps、Tesla、OpenAI、Anthropic、GitHub、Harvey、Graphite 的官方案例建立整体框架
- 在笔记中补充客服 / 法务 / coding agent 三类垂直场景的闭环设计思路，包括建议记录的数据对象、eval 维度、reward 理解方式与迭代顺序
- 新增 `journal/daily/2026-04-10.md`，记录本次围绕数据飞轮与 Agentic RL 的学习与沉淀过程
- 在 `ideas/future-research-directions.md` 中把“Agentic RL 与数据飞轮”的方向和新学习笔记建立承接关系
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该学习笔记入口

### Harness Engineering 初探笔记

- 新增 `learning/harness-engineering-primer-and-evolution-comparison.md`，基于用户提供的视频摘要与脑图，轻量整理 `Prompt Engineering`、`Context Engineering` 与 `Harness Engineering` 的演进关系与差异
- 新增并优化 `learning/harness-engineering-primer-mindmap.svg`，调整脑图排版、卡片布局与中文显示可读性
- 在笔记中压缩整理成熟 harness 的六层结构：上下文管理、工具系统、执行编排、记忆与状态、评估与观测、约束校验与失败恢复
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该入门笔记入口

## 2026-04-29

### DeepSeek 研究路线笔记

- 新增 `learning/deepseek-v1-to-v4-research-route.md`，以官方论文和报告为主线，梳理 `DeepSeek LLM / V1` 到 `V4` 的研究路线，并拆成预训练、架构、后训练与系统工程四条主线
- 新增 `journal/daily/2026-04-29.md`，记录本次围绕 DeepSeek 技术演变与学习路径搭建的学习 session
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该研究笔记入口
- 在后续迭代中补充 `DeepSeek mHC` 节点，把连接结构演化明确写成 `Residual -> HC -> mHC -> V4`
- 在后续迭代中补充第一阶段精读，展开 `DeepSeek LLM / V1` 中的 `scaling law`、超参数规律、数据质量影响与其如何自然导向 `V2`

### 维护偏好更新

- 更新 `USER.md`，将默认协作习惯从“自动本地提交”升级为“任务完成后默认本地提交并推送远端，除非用户明确要求暂不 push”

## 2026-04-30

### DeepSeek LLM 论文阅读笔记

- 新增 `learning/deepseek-llm-v1-paper-reading.md`，按原文目录完整梳理 `DeepSeek LLM`，并在对应小节内嵌关键图表与表格阅读卡
- 在 `learning/deepseek-v1-to-v4-research-route.md` 中为第一阶段加入专题精读回链，形成“总纲 + 专题精读”双层结构
- 新增 `journal/daily/2026-04-30.md`，记录本次围绕 `DeepSeek LLM` 论文阅读笔记的学习 session
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中补充该论文阅读笔记入口
- 后续重写 `learning/deepseek-llm-v1-paper-reading.md`，减少与路线笔记的重复内容，并改为严格按 `Abstract -> Introduction -> Pre-Training -> Scaling Laws -> Alignment -> Evaluation -> Discussion -> Conclusion` 的原文顺序逐节解读
- 在重写版本中补齐 `Evaluation / Discussion / Conclusion` 的细节，并把每节统一整理为“这一节在原文里做什么、按段落顺序梳理、关键图表/表格、与路线笔记的轻量关联”

### DeepSeek 第二阶段启动

- 压缩 `learning/deepseek-v1-to-v4-research-route.md` 中第一阶段过长的精读段，把路线笔记重新收回到“总纲 / 地图”职责
- 清理 `learning/deepseek-llm-v1-paper-reading.md` 中重复出现的“和路线笔记的关联”小节，并收束为文末统一回链
- 将原本合并的第二阶段入口笔记拆分为 `learning/deepseek-moe-paper-reading.md` 与 `learning/deepseek-v2-paper-reading.md`，保持与第一阶段一致的“两层结构”
- 在 `learning/README.md`、`README.md` 与 `README.zh-CN.md` 中改为分别补充两篇单文陪读入口
- 在 `journal/daily/2026-04-30.md` 中记录本次结构清理、拆分与第二阶段启动过程
