# learning

这个目录用于存放蒸馏后的学习笔记、概念解释、阅读痕迹和结构化学习资料。

当一份内容的主要价值在于“帮助理解”，而不只是“修掉一个问题”时，就应该优先放在这里。

当前入口：

- `deepseek-v1-to-v4-research-route.md`：以官方论文和报告为主线，梳理 `DeepSeek LLM / V1` 到 `V4` 的技术演变，并把版本故事拆成预训练、架构、后训练和系统工程四条主线
- `harness-engineering-primer-and-evolution-comparison.md`：一篇更轻量的入门笔记，用来快速区分 `Prompt Engineering`、`Context Engineering` 与 `Harness Engineering`
- `llm-streaming-and-langgraph-streaming.md`：从 `prefill / decode / KV cache / SSE` 到 LangGraph `messages / updates / custom` 的一份流式输出学习笔记
- `nanobot-agent-control-center-analysis.md`：围绕主 agent / 中控 / 调度 / 队列 / 上下文管理拆解 `nanobot`，并外推到一个可执行的万能服务中控系统设计
- `data-flywheel-agentic-rl-and-vertical-ai-agents.md`：系统梳理数据飞轮、eval 飞轮、`RFT` 与 Agentic RL 的关系，并给出客服 / 法务 / coding agent 的具体闭环设计框架

推荐模板：

- `templates/learning-note-template.md`：适合写教程感更强、层次更分明、重视逻辑承接的学习笔记
