# Agent Index

这个文件是给新 agent 的最短入口页。

如果你刚进入这个仓库，先不要急着改文件；先用这里的索引快速建立对项目的工作模型。

## 仓库是什么

`agent-native-scriptorium` 是一个 Git-first 的 agent-native 外脑与成长操作系统。

它不是单纯的知识库，也不是纯日志本，而是把这几层放在同一个轻量结构里：

- `journal/`：记录真实推进过程
- `tracks/`：承接长期主线与 workstream
- 资产层：`learning/`、`solutions/`、`playbooks/`、`skills/`
- `reviews/`：做周期复盘
- `metrics/`：只在有用时记录轻量指标

最小工作闭环：

`journal -> track -> asset -> review -> metrics`

## 当前模式

- 当前默认只维护公开仓 `agent-native-scriptorium`
- `agent-native-vault` 暂时冻结，除非用户明确要求使用私仓或执行同步
- 公仓中的日志、求职记录、面试复盘与投递信息都必须默认 `public-safe`

## 必读顺序

1. `USER.md`
2. `AGENTS.md`
3. `NOW.md`
4. `playbooks/repository/growth-loop-workflow.md`
5. 当前 mission 对应的 `tracks/.../README.md`

如果任务明显只涉及某个局部目录，可以在读完前 4 项后再跳到对应条目。

## 当前主线

- 当前 active mission：`job-hunt-2026`
- 当前 active tracks：`rl`、`agents`、`resume`、`interviews`、`projects`、`applications`
- 当前重点方向：RL / agent 学习，源码与论文阅读，简历打磨，面试准备，项目准备与求职推进

## 默认工作流

1. 先把这次 session 的真实推进记进 `journal/`
2. 再把内容挂到合适的 `tracks/` 主线上
3. 当内容稳定且值得长期保留时，再晋升到资产层
4. 通过 `journal/weekly/` 和 `reviews/` 做阶段汇总
5. 只有在确实有帮助时，才把结果提炼进 `metrics/`

## 写到哪里

- 当天/当次过程记录：`journal/`
- 长期主线组织：`tracks/`
- 学习沉淀：`learning/`
- 问题解法：`solutions/`
- 操作指南：`playbooks/`
- agent 可复用流程：`skills/`
- 灵感与构想：`ideas/`
- 可复用的提示词脚手架：`prompts/`
- 统一模板：`templates/`

## 护栏

- 不要默认动私仓
- 不要自动发起双仓同步
- 不要把敏感、未脱敏或强个人上下文内容直接写进公仓
- 更新 `CHANGELOG.md` 前先核对 `git log --oneline --date=short`
- 修改仓库定位、结构或顶层入口时，同步检查 `README.md` 与 `README.zh-CN.md`
- 优先做增量改动，避免把结构做重

## 启动动作

当你完成上述阅读后，先用简短结构总结：

- 这个仓库是什么
- 当前维护模式是什么
- 当前 mission 和 active tracks 是什么
- 你准备遵守哪些关键规则
- 你准备如何推进当前任务

然后再开始实际编辑。
