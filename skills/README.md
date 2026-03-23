# skills

这个目录用于存放更适合 agent 直接复用的技能化资料。

和 `playbooks/` 的区别：

- `playbooks/` 更偏向人类阅读与理解
- `skills/` 更偏向 agent 执行与复用

推荐做法：

- 如果一份内容只是偶发性的经验总结，先写成 `playbook`
- 如果一份内容已经稳定、可重复、步骤清晰、经常会再次用到，再补成 `skill`
- 对高价值工具型方案，可以同时保留 `playbook + skill`

## 本仓库采用的 Skill 规范

本仓库默认采用 Anthropic 官方 agent skills 的思路来组织 skill。

建议后续维护者把下面这些视为默认规范：

- 一个 skill 对应一个独立目录
- 技能入口文件固定为 `SKILL.md`
- `SKILL.md` 使用 YAML frontmatter
- frontmatter 至少要有 `name` 和 `description`
- `description` 要清楚说明：
  - skill 做什么
  - 在什么情况下应该用它
- `SKILL.md` 保持短小、可触发、可执行
- 更长的说明应放进 `references/`
- 可执行程序应放进 `scripts/`
- 静态模板或资源应放进 `assets/`

## 本仓库中的推荐目录结构

```text
skills/
  skill-name/
    SKILL.md
    references/
    scripts/
    assets/
```

其中：

- `references/`：长说明、检查清单、背景知识
- `scripts/`：命令脚本、工具脚本、自动化程序
- `assets/`：模板、示例文件、静态材料

## 写 Skill 时的实操建议

1. 先判断它是不是真的值得成为 skill，而不是一篇普通 playbook。
2. 优先把“何时触发”写清楚，而不是先写实现细节。
3. 把 `SKILL.md` 控制在 agent 能快速扫完并决定是否使用的长度。
4. 如果内容越来越长，优先拆到 supporting files。
5. 新增 skill 时，从 `skills/template/SKILL.md` 开始，而不是从零散发挥。

## 官方参考

- Anthropic 官方 skills 仓库：
  [github.com/anthropics/skills](https://github.com/anthropics/skills)
- Anthropic 官方技能指南：
  [The Complete Guide to Building Skill for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
