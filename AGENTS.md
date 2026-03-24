# AGENTS.md

这个文件定义的是“任何 agent 在这个仓库里都应遵守的通用维护规则”。

它不负责记录用户的个性化偏好。用户偏好请看 `USER.md`。

## 职责边界

- `AGENTS.md`：仓库级、通用型、长期有效的协作规则
- `USER.md`：用户个人偏好、默认风格、特殊约定、当前习惯

如果两者都存在，建议遵循：

1. 用户当下的直接指令
2. `USER.md`
3. `AGENTS.md`

## 仓库使命

保存由 agent 参与产生的高价值成果，让它们对未来的人类读者和未来的 agent 都仍然可理解、可复用、可继续扩展。

## 双仓协作规则

- `agent-native-scriptorium` 是公开层，只收录可公开分享、完成脱敏、适合复用的内容。
- `agent-native-vault` 是私有层，用于未公开、敏感、半成品或带有强个人上下文的材料。
- 判断不准时，默认先放 `vault`，不要冒险直接提交到 `scriptorium`。
- 从 `vault` 晋升到 `scriptorium` 时，先做脱敏、提炼、改写，再保留清晰的承接关系。
- 即使在私仓中，也不要提交密钥、令牌、口令、cookie、身份证件、银行卡信息或其他高风险秘密。
- 具体操作流程见 `playbooks/repository/dual-repo-workflow.md`。

## 主维护仓与同步规则

- 用户可以按当前任务或当前阶段指定主维护仓；在该任务中，它就是 source of truth。
- 未收到明确同步请求时，不要自动把一侧改动同步到另一侧。
- 同步请求默认解析为：主维护仓、目标仓、同步方式、包含范围、排除范围、覆盖策略。
- 完全同步会对共享面做完整对齐，但默认仍是非破坏式同步，不删除目标仓独有文件。
- 部分同步只处理用户点名的文件、目录、主题，或“本次某类改动”。
- 只有当用户明确说出“镜像”“删除目标仓多余共享文件”“完全以 X 为准覆盖”之类表述时，才允许 destructive mirror。
- 具体执行流程见 `playbooks/repository/repo-sync-workflow.md`，重复执行时优先使用 `skills/repo-sync/SKILL.md`。

## 默认规则

- 除非有明确理由，否则不要随意重构已有资料。
- 优先做增量修改，而不是大面积推倒重写。
- 只要历史上下文仍然有解释价值，就不要轻易删除。
- 新建文件时，文件名默认使用小写 kebab-case。
- 默认使用 Markdown，除非有更合适的格式。
- 严禁提交密钥、令牌、凭证、cookie、私密个人数据或其他敏感信息。
- 每次有实质性变更时，更新 `CHANGELOG.md`。
- 如果根目录同时维护 `README.md` 与 `README.zh-CN.md`，修改仓库定位、结构或入口说明时，应尽量同步两者的核心信息。

## 目录意图

- `playbooks/`：可执行的操作指南与经验证的排障流程
- `skills/`：面向 agent 复用的操作封装
- `ideas/`：待孵化的想法目录，可包含产品、科研、生活、创业、工作和日常灵感
- `solutions/`：具体问题、推理过程与最终解决方案
- `learning/`：学习笔记、概念整理、阅读痕迹与解释性资料
- `prompts/`：可复用的提示词模式、工作流和交互配方
- `templates/`：统一风格与结构时可复用的模板

## 笔记质量要求

每一份值得长期保留的笔记，通常应尽量包含：

- 它解决了什么问题或回答了什么问题
- 这个问题为什么重要
- 结论、方案或提炼后的解释
- 如果内容受环境影响，需要附上验证信息
- 足够独立的上下文，让未来读者不必回看原始对话

## 推荐元数据块

当笔记需要更强的可追踪性时，可在开头使用：

```md
- Created: YYYY-MM-DD
- Updated: YYYY-MM-DD
- Type: playbook | solution | learning | prompt | skill | idea
- Status: draft | verified | needs-review
- Tags: tag1, tag2, tag3
- Model: GPT-5.4
- Harness: Codex
- Source: one-line origin note
```

如果某条记录使用了不同模型或不同 agent 形态，应显式覆盖默认值。

字段定义与常见混淆说明见 `playbooks/repository/metadata-field-reference.md`。
尤其注意：

- `Model` 指底层模型本身，例如 `GPT-5.4`
- `Harness` 指让模型以 agent 形态运行的那层系统，例如 `Codex`

## 维护流程

1. 在新增内容前，先理解仓库现有结构。
2. 把新材料放入最合适的目录。
3. 从零开始写时，优先复用 `templates/knowledge-note-template.md`。
4. 如果新增内容是高价值入口，记得同步更新 `README.md`。
5. 如果新增内容是重复可执行流程，评估它是否应从 `playbooks/` 提升到 `skills/`。
6. 如果一份新笔记取代了旧笔记，优先通过链接建立承接关系，而不是悄悄抹去历史。

## playbook 与 skill 的区分

- `playbook` 更偏“给人看”，强调背景、解释、过程和可读性
- `skill` 更偏“给 agent 复用”，强调触发条件、步骤、输入输出和执行习惯

对于高复用、强操作型、步骤稳定的方案，推荐同时保留：

- 一个 `playbook` 作为人类可读的知识条目
- 一个 `skill` 作为 agent 可复用的操作封装

## Skill 编写规范

当在本仓库中新增或修改 `skill` 时，默认应遵循 Anthropic 官方 agent skills 的结构与最佳实践。

本仓库中的默认要求：

- 每个 skill 使用独立目录。
- skill 目录中必须有 `SKILL.md`。
- `SKILL.md` 必须包含 YAML frontmatter。
- frontmatter 至少包含：
  - `name`
  - `description`
- `description` 必须同时回答两件事：
  - 这个 skill 做什么
  - 应该在什么情况下使用
- `SKILL.md` 应保持短小、可触发、可执行。
- 更长的说明应下沉到：
  - `references/`
  - `scripts/`
  - `assets/`
- 不要在单个 skill 目录中再放一个 `README.md` 去替代 `SKILL.md`。
- 新 skill 优先从 `skills/template/SKILL.md` 开始。

简化理解：

- `SKILL.md` 负责“告诉 agent 何时触发、如何开始”
- `references/` 负责“需要时再读取的长说明”
- `scripts/` 负责“可直接执行的程序”
- `assets/` 负责“模板、样例、静态资源”

如果一个 skill 过长、过杂、过像说明书，应优先拆分 supporting files，而不是继续把所有细节堆进 `SKILL.md`。

## 写作风格

- 优先为“未来复用”而写，而不是为“即时聊天”而写。
- 具体，不空泛。
- 步骤要能执行。
- 假设要写明。
- 任何可能过时的结论，都尽量注明验证方式与验证时间。

## 变更纪律

- 保持提交聚焦。
- 避免顺手式的大范围改写。
- 如果引入了新规范，请同步记录到 `README.md`、`PLAN.md`、`CHANGELOG.md` 或本文件。
- 更新 `CHANGELOG.md` 前，先用 `git log --oneline --date=short` 核对相关提交的真实日期，再按绝对日期分组，不要仅凭会话顺序或相对时间判断。
