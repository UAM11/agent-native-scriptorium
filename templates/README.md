# templates

这个目录用于存放可复用的写作模板和结构脚手架。

## 怎么选模板

可以先按条目类型来判断：

- `journal`：优先用 `daily-journal-template.md`、`weekly-review-template.md`、`monthly-review-template.md`
- `track`：优先用 `track-template.md`
- `learning`：优先用 `learning-note-template.md`
- 其他知识条目：默认用 `knowledge-note-template.md`

## `knowledge-note-template` 和 `learning-note-template` 的区别

### `knowledge-note-template.md`

适合：

- 通用知识条目
- `playbook`
- `solution`
- `prompt`
- `idea`
- 其他不需要强教程感的资料

特点：

- 更通用
- 更轻量
- 更适合“把一件事记下来”
- 更强调结论、步骤、验证和注意事项

一句话理解：

> 它是默认模板，适合大多数知识条目。

### `learning-note-template.md`

适合：

- `Type: learning`
- 需要从头讲清楚概念的笔记
- 容易混淆、需要分层解释的主题
- 希望写成教程感更强的学习资料

特点：

- 更像教程
- 更强调分层、承接和总地图
- 更适合“帮助别人真的理解”
- 默认包含“先拆开 -> 总图 -> 核心概念 -> 串起来 -> 再深化 -> 示例”的结构

一句话理解：

> 它不是记录模板，而是教学模板。

## 一个简单决策法

如果你在写的时候更像是在回答：

- “这件事最后怎么做？”
- “这个问题怎么解决？”
- “这份资料以后怎么复用？”

优先用 `knowledge-note-template.md`。

如果你在写的时候更像是在回答：

- “这件事从头该怎么理解？”
- “这些概念为什么会混在一起？”
- “我该怎么一步一步学会这个主题？”

优先用 `learning-note-template.md`。

## 给 Agent 的默认规则

如果新 agent 不确定该用哪个模板，默认按下面顺序判断：

1. 如果条目类型已经明确是 `learning`，直接用 `learning-note-template.md`
2. 如果目标是“帮助理解”，并且需要分层解释、总图、概念承接和示例，优先用 `learning-note-template.md`
3. 如果目标是“保存知识”或“沉淀方案”，默认用 `knowledge-note-template.md`
4. 如果是日志、复盘、track 之类固定条目，优先用专用模板而不是通用模板
