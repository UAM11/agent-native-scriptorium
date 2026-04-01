# 新 Agent 启动 Prompt

这个 prompt 用于在新的线程、新的 harness，或新的 agent 接手这个仓库时，快速完成自举。

适用场景：

- 切换到新的对话线程
- 换了新的 agent / harness
- 希望让一个新 agent 先熟悉仓库，再独立维护

## 推荐用法

把下面整段发给新的 agent，然后再补你的具体任务。

```text
请先把这个仓库理解为一个 Git-first 的 agent-native 外脑与成长操作系统，而不是普通笔记仓库。

请按顺序阅读以下文件：
1. AGENT_INDEX.md
2. USER.md
3. AGENTS.md
4. NOW.md
5. playbooks/repository/growth-loop-workflow.md
6. tracks/job-hunt-2026/README.md

阅读后请先用简短结构总结：
- 这个项目是什么
- 当前维护模式是什么
- 当前 active mission 和 active tracks 是什么
- 你会遵守哪些关键规则
- 你准备如何维护它

在完成这段总结之前，不要直接改文件。

如果我的任务只涉及某个局部主题，你可以在完成上述总结后，再继续阅读对应目录下最相关的 README 或条目。
```

## 更短版本

如果只想快速拉起上下文，也可以使用这个短版：

```text
请先阅读 AGENT_INDEX.md、USER.md、AGENTS.md、NOW.md 和当前 mission 的 track README，然后先总结你的理解，再执行我的具体任务。
```
