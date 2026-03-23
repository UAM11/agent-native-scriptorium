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
