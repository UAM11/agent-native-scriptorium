# Harness Engineering 初探与演进比较

- Created: 2026-04-10
- Updated: 2026-04-28
- Type: learning
- Status: draft
- Tags: harness-engineering, prompt-engineering, context-engineering, agents
- Model: GPT-5.4
- Harness: Codex
- Source: organized from a user-provided summary and mind map of the Bilibili video “最近爆火的 Harness Engineering 到底是啥？一期讲透！”

## 背景

这份笔记刻意保持轻量，只做一个第一眼可读的 `Harness Engineering` 入门卡片。
重点不是展开复杂细节，而是先回答三个问题：

- `Harness Engineering` 到底在说什么？
- 它和 `Prompt Engineering`、`Context Engineering` 的区别是什么？
- 为什么 AI 落地的重点会逐步从“模型更聪明”转向“系统更稳定”？

## 脑图

![Harness Engineering 脑图](/F:/agent-native-scriptorium/learning/harness-engineering-primer-mindmap.svg)

## 简要说明

可以先把 `Harness Engineering` 理解为：

> 在 AI 系统里，除模型本身之外，那套决定系统是否能稳定交付的运行系统。

如果说：

- `Prompt Engineering` 更关注“怎么说”
- `Context Engineering` 更关注“给什么信息”

那么：

- `Harness Engineering` 更关注“系统怎样持续做对、做稳、出错后还能恢复”

这也是为什么可以把很多 agent 粗略理解成：

> `Agent = Model + Harness`

## 三阶段演进比较

### Prompt Engineering

- 关注点：模型是否听懂指令
- 核心：塑造局部概率空间，优化语言表达
- 局限：难以弥补知识缺失，也不擅长管理长链路状态

### Context Engineering

- 关注点：模型是否获取足够、正确、及时的信息
- 核心：把当前决策真正需要的信息送进去
- 常见做法：检索增强、渐进式披露、上下文裁剪与组织

### Harness Engineering

- 关注点：模型在真实执行中能否持续做对
- 核心：全流程驾驭、监督、约束、纠偏
- 典型范围：工具、状态、执行编排、观测、校验、恢复机制

## 成熟 Harness 的六层结构

从这张脑图看，一个更成熟的 harness 通常至少会覆盖六层：

1. `上下文管理`
   角色目标定义、信息裁剪选择、结构化组织
2. `工具系统`
   工具合理配置、调用时机、结果提炼回喂
3. `执行编排`
   目标理解、信息补全、分析生成、输出校验修正
4. `记忆与状态`
   任务状态、中间结果、长期记忆、用户偏好
5. `评估与观测`
   输出验收、环境验证、自动测试、日志、指标、错误归因
6. `约束校验与失败恢复`
   行为约束、输出校验、失败重试、回滚

## 实践直觉

这份总结里提到的企业实践，可以先用很直觉的方式理解：

- `Anthropic`
  更强调处理上下文溢出、自评失真等问题，例如 `Context Reset`、生产与验收分离。
- `OpenAI`
  更强调工程师角色从“直接写结果”转向“设计环境、拆解任务、建立反馈”，以及渐进式披露、自主验证、自动治理。
- `其他企业`
  即使不改底层模型，只优化 harness，本身也可能带来明显的稳定性和效率提升。

## 一句话总结

`Prompt` 解决“怎么说”，`Context` 解决“给什么”，`Harness` 解决“怎样稳定工作”。  
当任务进入多步执行、外部工具、状态管理和真实环境验证时，`Harness Engineering` 就会从“加分项”变成“必需项”。

## 来源视频

- Bilibili：[最近爆火的 Harness Engineering 到底是啥？一期讲透！](https://www.bilibili.com/video/BV1Zk9FBwELs/?share_source=copy_web&vd_source=e11d1aa23ea058d42b676f314fe822a0)

## 备注

- 这份笔记是基于你提供的脑图与文字总结整理而成，不是完整逐字稿复原。
- 这里有意保持轻量，只保留第一层理解框架，方便后续继续扩展。
