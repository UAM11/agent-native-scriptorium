# 知识条目元数据字段说明

- Created: 2026-03-24
- Updated: 2026-03-24
- Type: playbook
- Status: verified
- Tags: metadata, schema, glossary, repository
- Model: GPT-5.4
- Harness: Codex
- Source: added to prevent future confusion in repository maintenance

## 用途

这份说明用于统一仓库内知识条目的元数据含义，减少不同 agent 或未来维护者对字段的误解。

推荐元数据块：

```md
- Created: YYYY-MM-DD
- Updated: YYYY-MM-DD
- Type: playbook | solution | learning | prompt | skill | idea | journal | review | track
- Status: draft | verified | needs-review
- Tags: tag1, tag2, tag3
- Model: GPT-5.4
- Harness: Codex
- Source: one-line origin note
```

## 字段定义

### Created

- 这条记录第一次建立的日期
- 一般保持不变

### Updated

- 这条记录最近一次做出实质性修改的日期
- 只要内容真的更新了，就应该同步改动

### Type

- 这条记录属于哪一类知识条目
- 当前推荐值：`playbook`、`solution`、`learning`、`prompt`、`skill`、`idea`、`journal`、`review`、`track`

### Status

- 这条记录当前的成熟度或可靠性状态
- `draft`：草稿或待继续完善
- `verified`：已经验证过，当前可用
- `needs-review`：内容可能过时、需要重新确认

### Tags

- 用于检索、聚类和未来整理
- 推荐使用短标签，避免整句

### Model

- 指底层模型本身
- 它回答“这条内容主要是在什么模型上产出的？”
- 例子：`GPT-5.4`

### Harness

- 指让模型以 agent 形态运行的那层系统、运行框架或工作台
- 它回答“这条内容主要是通过什么 agent/runtime 形态产出的？”
- 例子：`Codex`

### Source

- 记录这条知识从哪里来
- 通常用一句话说明来源场景即可

## 新增类型建议

### journal

- 用于日常 session 记录、daily log、weekly log 等过程层内容
- 重点是保留真实推进、阻塞点与下一步

### review

- 用于周度、月度或阶段性复盘
- 重点是总结、回看、评估与下一阶段规划

### track

- 用于一条长期主线、mission 或 workstream 的入口页
- 重点是定义范围、当前焦点、进入方式和产出方向

## Model 与 Harness 的区别

- `Model` 更像大脑，负责理解、推理、生成
- `Harness` 更像工作台，负责把模型包进真实可执行的 agent 运行环境

一个常见组合可以写成：

```md
- Model: GPT-5.4
- Harness: Codex
```

这表示：

- 内容主要由 `GPT-5.4` 这个模型生成或协助生成
- 内容是在 `Codex` 这个 agent harness 中完成的

## 常见错误

- 不要把 `Harness` 写成 `Hardness`
- 不要把 `Model` 和 `Harness` 当成同一个维度
- 不要把临时网页、项目名或仓库名直接写进 `Model`
- 如果只是换了运行环境，没有换模型，也不要把模型字段一起改掉

## 维护建议

- 如果一条记录明显依赖具体环境，除了元数据外，还应在正文写清验证条件
- 如果后续引入新的模型或新的 agent runtime，直接覆盖默认值即可
- 如果未来需要更强的可追踪性，可以再扩展字段，但应优先保持当前核心字段稳定
- 公仓中的 `journal`、`review` 与 `track` 记录默认应保持 public-safe，避免写入敏感求职或隐私细节
