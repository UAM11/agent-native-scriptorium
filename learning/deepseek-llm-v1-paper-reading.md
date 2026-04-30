# DeepSeek LLM 论文阅读笔记

- Created: 2026-04-30
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, deepseek-llm, scaling-law, pretraining, alignment, paper-reading
- Model: GPT-5.4
- Harness: Codex
- Source: structured reading of the DeepSeek LLM paper, following the original paper sections and figures

## 背景

这篇笔记是对 `DeepSeek LLM: Scaling Open-Source Language Models with Longtermism` 的专题精读。

它和 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 的分工是：

- 路线笔记：负责讲清 `DeepSeek LLM / V1 -> V4` 的整体演变
- 这篇专题笔记：负责把第一阶段 `DeepSeek LLM / V1` 按原文目录完整梳理

如果你现在想真正吃透 DeepSeek 的起点，这篇笔记应该和原论文一起看，而不是替代原论文。

## 问题或目标

> 按照原文目录结构，把 `DeepSeek LLM` 论文每一节在解决什么、给了哪些关键结论、每张关键图表在说明什么，系统梳理清楚。

## 如何使用这篇笔记

建议按下面顺序使用：

1. 先看每一节的“这一节在解决什么”
2. 再看“关键结论”
3. 最后看“图表阅读卡”

这样可以避免一上来就被公式、表格和 benchmark 淹没。

说明：

- 为了保持笔记可读性，这里没有直接复制原论文图片，而是把关键图表以内嵌“图表阅读卡”的方式整理到对应小节中
- 如果需要核对原图，请对照论文 PDF 中对应的 `Figure / Table` 编号查看

## 论文信息

- 标题：[DeepSeek LLM: Scaling Open-Source Language Models with Longtermism](https://arxiv.org/abs/2401.02954)
- arXiv：`2401.02954`
- 首版日期：`2024-01-05`
- 对应阶段：`DeepSeek LLM / V1 起点`
- 对应路线笔记位置：[第一阶段总纲](deepseek-v1-to-v4-research-route.md)

## 原文目录地图

论文正文结构是：

1. `Introduction`
2. `Pre-Training`
   - `2.1 Data`
   - `2.2 Architecture`
   - `2.3 Hyperparameters`
   - `2.4 Infrastructures`
3. `Scaling Laws`
   - `3.1 Scaling Laws for Hyperparameters`
   - `3.2 Estimating Optimal Model and Data Scaling`
   - `3.3 Scaling Laws with Different Data`
4. `Alignment`
5. `Evaluation`
   - `5.1 Public Benchmark Evaluation`
   - `5.1.1 Base Model`
   - `5.1.2 Chat Model`
   - `5.2 Open-Ended Evaluation`
   - `5.2.1 Chinese Open-Ended Evaluation`
   - `5.2.2 English Open-Ended Evaluation`
   - `5.3 Held-Out Evaluation`
   - `5.4 Safety Evaluation`
   - `5.5 Discussion`
6. `Conclusion, Limitation, and Future Work`

附录里当前最值得看的有：

- `A.2 Different Model Scale Representations`
- `A.3 Benchmark Metrics Curves`

## Abstract

### 这一部分在解决什么

摘要在回答一个总问题：

> DeepSeek 到底做的是“又一个开源大模型”，还是“把开源大模型扩展规律重新系统研究了一遍”？

### 关键结论

- 他们发布了 `7B` 和 `67B` 两个规模的 base/chat 模型
- 预训练语料是 `2T` bilingual corpus
- 真正的主角不是模型大小本身，而是：
  - `hyperparameter scaling laws`
  - `optimal model/data scaling`
  - `data quality` 对 scaling 的影响

### 怎么理解

如果只看摘要，这篇论文最该被理解为：

> 一篇“把 DeepSeek 后续大模型路线的地基先打稳”的论文。

## 1. Introduction

### 这一节在解决什么

Introduction 在回答：

> 为什么还要重新研究 scaling law？不是已经有 Kaplan 和 Chinchilla 了吗？

### 关键结论

- 早期 scaling work 给出的最优模型/数据扩展结论并不一致
- 很多工作没有把超参数本身的 scaling 一起研究清楚
- 训练数据质量在 scaling 讨论中经常被忽略
- DeepSeek 想做的是：把“真实训练决策”也纳入 scaling 研究，而不是只拟合一条漂亮曲线

### 为什么这一节重要

这一节决定了你怎么读整篇论文：

- 如果你把它当“模型发布文”，会觉得后面很多讨论太细
- 如果你把它当“训练规律研究 + 真实训练实践”，整篇结构就会很顺

## 2. Pre-Training

### 2.1 Data

#### 这一节在解决什么

如何为 bilingual base model 准备一套质量足够高、结构足够稳的预训练数据。

#### 关键结论

- 数据处理被拆成 3 个阶段：
  - `deduplication`
  - `filtering`
  - `remixing`
- 目标不是简单堆数据量，而是提高：
  - 丰富度
  - 多样性
  - 信息密度

#### 图表阅读卡

`Table 1 | Deduplication ratios for various Common Crawl dumps`

- 这张表在说明什么：
  - 去重不是“做一下就行”，而是去重范围会直接决定最终数据质量
- 关键观察：
  - 从 `1` 个 dump 到 `91` 个 dump，去重率从 `22.2%` 增长到 `89.8%`
- 你应该带走什么：
  - DeepSeek 很早就把“数据工程”当成模型能力的一部分，而不是前置杂活

#### 为什么它重要

后面 `3.3 Scaling Laws with Different Data` 能成立，一个前提就是：他们真的在迭代数据质量，而不只是复读同一份数据集。

### 2.2 Architecture

#### 这一节在解决什么

在第一阶段，DeepSeek 还没进入 `MoE / MLA` 时代，所以这一节的重点是：

> 在 dense transformer 框架下，如何选一个足够稳、足够适合继续扩展的基线架构？

#### 关键结论

- 总体设计大体沿用 `LLaMA`
- 采用：
  - `Pre-Norm`
  - `RMSNorm`
  - `SwiGLU`
  - `RoPE`
- tokenizer 词表训练后扩展到 `102400`
- `67B` 已经使用 `GQA`，以降低推理成本
- `67B` 选了更深的结构：`95` 层，而不是只继续加宽 FFN

#### 图表阅读卡

`Table 2 | Detailed specs of DeepSeek LLM family of models`

- 这张表在说明什么：
  - 7B 和 67B 不是随便长出来的，而是后面 scaling 结论驱动出来的实际训练配置
- 关键信息：
  - `7B`: `30` 层，`d_model=4096`
  - `67B`: `95` 层，`d_model=8192`
  - 都训练在 `2T` tokens 上，context 都是 `4096`
- 你应该带走什么：
  - 第一阶段就已经开始体现 DeepSeek 的风格：理论结论会被立刻落成工程配置

### 2.3 Hyperparameters

#### 这一节在解决什么

在真实预训练里，除了模型结构，最关键的其实是：

- optimizer
- learning rate schedule
- batch size
- gradient clipping

这一节在回答：

> 要把第一阶段真的训起来，默认训练配方是什么？

#### 关键结论

- optimizer：`AdamW`
- 初始化标准差：`0.006`
- `β1 = 0.9`
- `β2 = 0.95`
- `weight_decay = 0.1`
- `gradient clipping = 1.0`
- 使用 `multi-step learning rate scheduler`，而不是典型 cosine

#### 图表阅读卡

`Figure 1 | Training loss curves with different learning rate schedulers or different parameters for schedulers`

- 这张图在说明什么：
  - `multi-step LR scheduler` 的最终效果和 cosine 基本一致
  - 但它更适合 continual training 和阶段复用
- 子图重点：
  - `Figure 1(a)`：multi-step vs cosine
  - `Figure 1(b)`：不同 multi-step 比例
- 你应该带走什么：
  - DeepSeek 不是盲目追求单次训练最优，而是会优先考虑“这套策略后面还能不能复用”

### 2.4 Infrastructures

#### 这一节在解决什么

这一节在回答：

> 这些预训练实验具体是靠什么训练框架和基础设施跑起来的？

#### 关键结论

- 使用了自研的 `HAI-LLM` 训练框架
- 强调 training system 的高效与轻量
- 这说明从第一阶段开始，DeepSeek 就不是只做算法，也在做训练系统

#### 为什么这一节重要

这一节虽然短，但意义很大：

> DeepSeek 后来之所以能把 `V3 / V4` 做成 frontier 级工程系统，不是从后面某一代突然开始，而是第一阶段就已经把系统工程当主线了。

## 3. Scaling Laws

### 3.1 Scaling Laws for Hyperparameters

#### 这一节在解决什么

这一节在回答：

> `batch size` 和 `learning rate` 是否也存在稳定的 scaling 规律？

#### 关键结论

- 他们先在小规模实验上做 grid search
- 再在多个 compute budget 上拟合最优超参数
- 发现：
  - 最优 `batch size` 会随 compute 增大而变大
  - 最优 `learning rate` 会随 compute 增大而变小
- 拟合公式：
  - `η_opt = 0.3118 * C^-0.1250`
  - `B_opt = 0.2920 * C^0.3271`

#### 图表阅读卡

`Figure 2 | Training loss w.r.t. batch size and learning rate with 1e17 and 1e20 FLOPs`

- 这张图在说明什么：
  - near-optimal 区域不是一个特别尖的点，而是一块比较宽的区域
- 为什么重要：
  - 这说明实际训练中，超参数不是“只能命中一个神秘真值”，而是有可靠可操作的工作区间

`Figure 3 | Scaling curves of batch size and learning rate`

- 这张图在说明什么：
  - 用小模型拟合出来的 scaling 曲线，能较好预测大模型的近优超参数
- 你应该看什么：
  - 灰点：近优区间
  - 虚线：幂律拟合
  - 蓝色星标：7B / 67B 落点
- 你应该带走什么：
  - 第一阶段真正建立的是“超参数 scaling 的实用预测框架”

### 3.2 Estimating Optimal Model and Data Scaling

#### 这一节在解决什么

这一节在回答：

> 在给定 compute budget 的情况下，模型和数据到底该怎么一起扩展？

#### 关键结论

- 他们没有继续只用参数量 `N` 表示模型规模
- 而是改用：
  - `M = non-embedding FLOPs/token`
- 把 compute budget 表达成：
  - `C = M * D`
- 然后拟合出：
  - `M_opt = M_base * C^a`
  - `D_opt = D_base * C^b`
- 对他们当前数据分布，得到：
  - `a = 0.5243`
  - `b = 0.4757`

#### 图表阅读卡

`Table 3 | Difference in model scale representations`

- 这张表在说明什么：
  - 单纯看参数量会低估或高估不同结构下的真实计算量
- 你应该带走什么：
  - DeepSeek 不是只换了符号，而是在修正“什么才算模型规模”

`Figure 4 | IsoFLOP curve and optimal model/data allocation`

- 这张图在说明什么：
  - 在不同计算预算下，可以拟合出更合理的最优模型/数据分配曲线
- 你应该看什么：
  - `IsoFLOP curve`
  - `optimal model scaling`
  - `optimal data scaling`

`Figure 5 | Performance scaling curve`

- 这张图在说明什么：
  - 用小规模实验拟合出的 scaling curve，可以较好预测 7B 和 67B 的泛化性能
- 你应该带走什么：
  - 第一阶段不是“纸上谈 scaling”，而是把 scaling 真正变成了大模型训练前的规划工具

### 3.3 Scaling Laws with Different Data

#### 这一节在解决什么

这一节在回答：

> 如果数据分布和数据质量发生变化，最优 model/data scaling 会不会也变？

#### 关键结论

- 会变，而且变化明显
- 数据质量越高，追加 compute 时越应该把预算更多分给模型扩展
- 不是永远“多加数据”最划算

#### 图表阅读卡

`Table 4 | Coefficients of model scaling and data scaling vary with training data distribution`

- 这张表在说明什么：
  - 不同数据分布会改变最优模型/数据 scaling 指数
- 关键观察：
  - 数据质量越好，模型 scaling 指数 `a` 越大，数据 scaling 指数 `b` 越小
- 你应该带走什么：
  - 数据质量不是附属因素，它会反过来改变最优扩展策略

#### 为什么这一节重要

这是第一阶段里最值得长期记住的一节之一，因为它直接解释了：

> 为什么后面 DeepSeek 会持续重视数据工程、数据迭代和数据质量提升。

## 4. Alignment

### 这一节在解决什么

这一节在回答：

> 基础模型训完之后，怎样把它变成一个有帮助、较安全、较稳定的 chat model？

### 关键结论

- 总共收集了约 `1.5M` instruction data
- 其中：
  - `1.2M` helpfulness data
  - `300K` safety data
- helpfulness 数据分布：
  - `31.2%` general language
  - `46.6%` math
  - `22.2%` coding
- alignment pipeline 分两步：
  - `SFT`
  - `DPO`

### 这一步最值得学的地方

- `67B` 在 SFT 阶段更容易过拟合，所以训练 epoch 更少
- 数学和代码数据会强化能力，但也可能带来重复输出和对话体验下降
- `DPO` 对标准 benchmark 的数值帮助不大，但对 open-ended generation 和 alignment 很有帮助

### 为什么它重要

这一节是 later `R1` 的重要对照物：

- 第一阶段的 alignment 主线还是 `SFT + DPO`
- reasoning 还没有被当成单独的 RL 路线处理

## 5. Evaluation

### 5.1 Public Benchmark Evaluation

#### 5.1.1 Base Model

##### 这一节在解决什么

base model 在标准 benchmark 上的综合水平如何。

##### 图表阅读卡

`Table 5 | Main results`

- 这张表在说明什么：
  - 即使是 bilingual 2T 语料，DeepSeek 仍能在英文 benchmark 上和 LLaMA2 打得很近
  - `67B` 在数学、代码和中文 benchmark 上尤其强
- 你应该带走什么：
  - 第一阶段的基础模型已经显示出明显的中英双语和 reasoning 潜力

#### 5.1.2 Chat Model

##### 这一节在解决什么

SFT / DPO 后的 chat model 与 base model 相比，哪些能力提升，哪些能力会下降。

##### 图表阅读卡

`Table 6 | The comparison between base and chat models`

- 这张表在说明什么：
  - 对齐后的 chat model 在大多数任务上有提升
  - 但某些 cloze / sentence completion 类任务会下降
- 你应该带走什么：
  - alignment 不是“全维度无代价增强”，而是有能力 trade-off 的

### 5.2 Open-Ended Evaluation

#### 5.2.1 Chinese Open-Ended Evaluation

##### 这一节在解决什么

在更接近真实对话和复杂中文任务的环境里，chat model 表现如何。

##### 图表阅读卡

`Table 7 | AlignBench leaderboard`

- 这张表在说明什么：
  - DeepSeek 67B Chat 在中文 open-ended evaluation 上非常强
  - DPO 基本在各维度都有帮助
- 你应该带走什么：
  - 第一阶段虽然还没进入 `R1`，但在 open-ended 中文对齐上已经很有竞争力

#### 5.2.2 English Open-Ended Evaluation

##### 这一节在解决什么

在英文多轮开放问答环境中，DeepSeek chat model 的表现如何。

##### 图表阅读卡

`Table 8 | MT-Bench Evaluation`

- 这张表在说明什么：
  - DeepSeek 67B Chat 在英文 open-ended evaluation 上已经能压过不少开源基线
  - DPO 进一步提升 MT-Bench 总分
- 你应该带走什么：
  - `DPO` 的主要收益更偏 open-ended generation，而不是标准选择题 benchmark

### 5.3 Held-Out Evaluation

#### 这一节在解决什么

如何尽量避免 benchmark contamination 和过拟合带来的虚高结果。

#### 图表阅读卡

`Table 9 | Held-out Dataset Evaluation`

- 这张表在说明什么：
  - 作者专门找了更近、更新、难污染的数据集看泛化
- 你应该带走什么：
  - DeepSeek 在第一阶段就已经非常重视“不要只看公开 benchmark 分数”

### 5.4 Safety Evaluation

#### 这一节在解决什么

DeepSeek 67B Chat 在安全问题上的表现究竟如何。

#### 图表阅读卡

`Table 10 | Our taxonomy for safety evaluation`

- 这张表在说明什么：
  - 作者不只做一个简单安全 benchmark，而是先建立自己的安全分类法，再做人审
- 你应该带走什么：
  - 安全评测在这里被看作系统工程，不是只跑一个 public benchmark

`Table 11 | Do-Not-Answer Score`

- 这张表在说明什么：
  - DeepSeek 67B Chat 在这个数据集上超过了 ChatGPT 和 GPT-4
- 你应该带走什么：
  - 第一阶段的 alignment 已经明显把“安全拒答能力”纳入目标

### 5.5 Discussion

#### 这一节在解决什么

这一节最有价值，因为它不只是报结果，而是在总结训练中真正踩出来的经验。

#### 图表阅读卡

`Table 12 | Two-stage fine-tuning results`

- 这张表在说明什么：
  - `two-stage SFT` 可以降低 repetition，同时提升 instruction following，而不显著伤害 code / math
- 你应该带走什么：
  - alignment 不是一刀切，阶段化训练很重要

`Table 13 | The impact of adding multi-choice question data`

- 这张表在说明什么：
  - 加入 `20M` 多选题数据后，多选题 benchmark 提升明显
  - 但 open-ended knowledge 类任务未必提升
- 你应该带走什么：
  - 训练数据形态会强烈影响“模型到底学会了什么能力”

`Table 14 | The impact of adding a system prompt`

- 这张表在说明什么：
  - system prompt 对 `67B` 有帮助，对 `7B` 反而略有负面影响
- 你应该带走什么：
  - 大小模型对提示词结构的吸收能力并不一样

#### 为什么这一节非常值得反复看

如果说前面的章节更像“研究结论”，那 `Discussion` 更像“训练团队真正踩出来的经验教训”。  
后续无论你学 `V2`、`V3` 还是 `R1`，这一节都会反复有用。

## 6. Conclusion, Limitation, and Future Work

### 这一节在解决什么

这一节在总结：

- 这篇论文到底完成了什么
- 还缺什么
- 后面准备往哪走

### 关键结论

- `DeepSeek LLM` 证明了：
  - scaling law 可以用于更真实的训练决策
  - `batch size / learning rate` 也存在可拟合规律
  - `M = non-embedding FLOPs/token` 是更合理的规模度量
  - 数据质量会改变最优扩展策略
- 论文也明确承认：
  - 知识更新不足
  - 可能出现 hallucination
  - 中文数据仍不完整
  - 其他语言能力有限

### 为什么它重要

这一节最重要的不是 limitations 本身，而是它暗示了后面几代的自然方向：

- 更经济的扩展：通向 `MoE`
- 更便宜的长上下文：通向 `MLA`
- 更强的推理与 agent：通向 `R1 / V3.2 / V4`

## 附录里最值得看的部分

### A.2 Different Model Scale Representations

#### 图表阅读卡

`Figure 6 | Performance scaling curves using different model scale representations`

- 这张图在说明什么：
  - `6N1` 会高估大模型性能
  - `6N2` 会低估大模型性能
  - `M` 的预测最准确
- 你应该带走什么：
  - DeepSeek 在第一阶段就已经开始重新定义“应该怎么度量模型规模”

### A.3 Benchmark Metrics Curves

#### 图表阅读卡

`Figure 7 | Benchmark metrics curves of DeepSeek LLM Base`

- 这张图在说明什么：
  - 训练曲线整体还在持续增长，没有明显见顶
- 你应该带走什么：
  - 第一阶段的很多配置，不是因为“模型已经完全到极限”，而是因为研究和工程节奏先停在了一个合理点

## 读完整篇后，最该带走的 10 句话

1. `DeepSeek LLM` 的核心不是“发了 7B/67B”，而是把 scaling 地基先研究清楚。
2. 它不只研究模型和数据，也研究 `batch size / learning rate` 的 scaling。
3. 最优超参数不是一个点，而是一条 near-optimal 带。
4. 用 `non-embedding FLOPs/token` 表示模型规模，比只看参数量更合理。
5. 数据质量会改变最优 model/data allocation。
6. 第一阶段已经体现出 DeepSeek 的工程气质：研究结论必须落到真实训练配置。
7. alignment 在这一代仍然是 `SFT + DPO` 主线。
8. `DPO` 的主要收益更偏 open-ended generation，而不是标准 benchmark。
9. `Discussion` 部分非常值钱，因为它沉淀了 staged fine-tuning、multiple-choice data、system prompt 等经验。
10. 第一阶段自然导向第二阶段：dense `FFN` 太贵、`KV cache` 太贵，于是才会出现 `DeepSeekMoE + MLA`。

## 和路线笔记怎么配合

如果你已经读完这篇，可以回到总纲：

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)

建议的回跳方式是：

1. 先回看路线笔记里 `DeepSeek LLM / V1` 那一段
2. 再直接进入 `DeepSeekMoE + V2`
3. 特别带着两个问题去看 V2：
   - 为什么 dense `FFN` 成本开始成为主瓶颈？
   - 为什么 `KV cache` 成本开始被单独拿出来解决？

## 验证

- 2026-04-30：基于原论文目录与关键图表，完成第一版结构化阅读笔记
- 主要材料：
  - [DeepSeek LLM arXiv 摘要页](https://arxiv.org/abs/2401.02954)
  - [DeepSeek LLM PDF](https://arxiv.org/pdf/2401.02954)

## 相关笔记

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)
