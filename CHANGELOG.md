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
- 在知识条目元数据中加入 `Model` 与 `Hardness`
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
