# 成长闭环工作流

- Created: 2026-03-31
- Updated: 2026-03-31
- Type: playbook
- Status: verified
- Tags: growth, workflow, journal, promotion, review, metrics
- Model: GPT-5.4
- Harness: Codex
- Source: designed while turning the repository into a lightweight growth operating system

## 背景

如果仓库只保存“提炼后的好东西”，就会失去真实推进过程；如果仓库把所有过程原样堆进来，又会迅速变成噪音堆。

这个工作流的目标，是在“足够轻”与“足够长期”之间找到平衡：先留痕，再提炼，再复盘，最后在需要时量化。

## 最小工作模型

当前推荐采用 5 层闭环：

1. `journal/`：记录真实 session、daily log、weekly log
2. `tracks/`：承接长期主线、mission 和 workstream
3. 资产层：把高价值内容晋升到 `learning/`、`solutions/`、`playbooks/`、`skills/`
4. `reviews/`：做月度或阶段性总结
5. `metrics/`：只在有用时记录轻量指标与可视化结果

简化理解：

`journal -> track -> asset -> review -> metrics`

## 每日工作流

1. 用 `journal/daily/YYYY-MM-DD.md` 记录当天真实推进。
2. 在日志中标出当前关联的 track。
3. 记录输入、工作过程、产出、阻塞点和下一步。
4. 如果当天出现了值得长期保留的内容，在 `Promotions` 段落里标记它应晋升到哪里。
5. 当天不必强行晋升；宁可轻量记录，也不要为了“完美沉淀”而中断推进。

## 每周工作流

1. 用 `journal/weekly/YYYY-Www.md` 汇总本周 daily logs。
2. 回看本周真正推进了什么，而不是只看忙碌感。
3. 记录本周新增的长期资产、主要阻塞和下周优先级。
4. 如有必要，把关键结果再整理到 `reviews/` 或 `metrics/`。

## 晋升规则

### 留在 `journal/`

- 仍然带有强过程性
- 还没有稳定结论
- 只是当天推进痕迹

### 晋升到 `tracks/`

- 某条主线需要一个长期入口页
- 内容更适合作为 workstream 的组织骨架

### 晋升到资产层

- `learning/`：可复用的学习笔记、源码阅读、论文拆解、概念解释
- `solutions/`：某个问题的最终解法、思路与验证结果
- `playbooks/`：稳定、可复用、以操作步骤为主的方法
- `skills/`：适合 agent 反复执行的稳定流程

## 轻量化原则

- 新目录优先先放一个 `README.md` 作为入口，不要一开始就创建大量空文件。
- 只有某类材料真的反复出现时，才把它拆成更深的子目录。
- 默认先记录，再提炼；不要为了“格式完整”而让记录成本变高。
- 每日条目可以很短，但要让未来的你和未来的 agent 看得懂。

## Public-Safe 原则

由于当前默认维护的是公开仓：

- 公司名、面试官信息、联系方式、敏感 JD 细节应尽量匿名化
- 投递记录建议使用 `company-a`、`company-b` 这类代称
- 简历内容可以记录修改原则、公开安全的 bullet bank 和版本策略，但不应直接暴露敏感个人信息
- 面试复盘可以保留题型、盲点、反思，不应机械记录受保护的具体题面或内部细节

## 指标原则

- token 估计可以记录，但它只是辅助信号
- 更值得长期跟踪的是：`Days Logged`、`Deep Work Blocks`、`Assets Promoted`、`Papers Read`、`Source Modules Read`、`Interview Questions Practiced`、`Resume Revisions`、`Applications Sent`
- 指标只在能帮助判断节奏和方向时才有意义，不应演变成表演式 KPI

## 推荐节奏

- 每天：`journal/daily/`
- 每周：`journal/weekly/`
- 每月：`reviews/monthly/`
- 按需：`metrics/snapshots/`

## 相关条目

- `templates/daily-journal-template.md`
- `templates/weekly-review-template.md`
- `templates/monthly-review-template.md`
- `templates/track-template.md`
