# DeepSeekMoE 论文阅读笔记

- Created: 2026-04-30
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, deepseekmoe, moe, expert-specialization, paper-reading
- Model: GPT-5.4
- Harness: Codex
- Source: close reading of the official DeepSeekMoE paper, rewritten section by section against the original paper structure

## 背景

这篇笔记只做一件事：

> 按 `DeepSeekMoE` 论文原文的目录、论证顺序、实验结构和关键图表，把作者到底说了什么重新梳理清楚。

它和路线笔记 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 的分工是：

- 路线笔记：负责讲 `DeepSeek LLM / V1 -> V4` 的整体演化
- 这篇笔记：只负责 `DeepSeekMoE` 这篇论文本身，不替后续版本讲故事

## 这篇笔记怎么用

建议按下面的顺序看：

1. 先看每一节的“这一节在原文里做什么”
2. 再看“按段落顺序梳理”
3. 最后看“关键图表与表格”

这篇笔记会尽量贴着原文走，但不会逐句硬翻。重点是把：

- 每一节承担什么功能
- 关键实验到底在证明什么
- 哪些数字最值得记住

讲清楚。

## 论文信息

- 标题：[DeepSeekMoE: Towards Ultimate Expert Specialization in Mixture-of-Experts Language Models](https://arxiv.org/abs/2401.06066)
- arXiv：`2401.06066`
- 首版日期：`2024-01-11`
- 对应路线位置：[DeepSeek V1-V4 研究路线笔记中的第二阶段前半段](deepseek-v1-to-v4-research-route.md)

## 原文目录地图

论文正文结构是：

1. `Introduction`
2. `Preliminaries: Mixture-of-Experts for Transformers`
3. `DeepSeekMoE Architecture`
   - `3.1 Fine-Grained Expert Segmentation`
   - `3.2 Shared Expert Isolation`
   - `3.3 Load Balance Consideration`
4. `Validation Experiments`
   - `4.1 Experimental Setup`
   - `4.2 Evaluations`
   - `4.3 DeepSeekMoE Aligns Closely with the upper bound of MoE Models`
   - `4.4 Ablation Studies`
   - `4.5 Analysis on Expert Specialization`
5. `Scaling up to DeepSeekMoE 16B`
   - `5.1 Experimental Setup`
   - `5.2 Evaluations`
6. `Alignment for DeepSeekMoE 16B`
   - `6.1 Experimental Setup`
   - `6.2 Evaluations`
7. `DeepSeekMoE 145B Ongoing`
   - `7.1 Experimental Setup`
   - `7.2 Evaluations`
8. `Related Work`
9. `Conclusion`

当前最值得补看的附录部分是：

- `Appendix A`：不同规模下的超参数总表
- `Appendix B`：与更大 `GShard / Dense` 基线的补充比较
- `Appendix C`：`DeepSeekMoE 16B` 与 `DeepSeek 7B` 的训练曲线

## Abstract

### 这一节在原文里做什么

摘要的任务很集中：先点出传统 `MoE` 的核心缺陷，再交代 `DeepSeekMoE` 的两条主方法，最后用一串缩略结论说明这条路不仅在小规模验证有效，而且能扩到 `16B` 和 `145B`。

### 按原文信息顺序梳理

1. 开头先设定目标：作者不是泛泛谈 `MoE`，而是直接把目标定义成 `ultimate expert specialization`。
2. 然后指出传统 `MoE` 在专家专门化上做得还不够，存在：
   - `knowledge hybridity`
   - `knowledge redundancy`
3. 接着提出两条核心策略：
   - `fine-grained expert segmentation`
   - `shared expert isolation`
4. 再给小规模验证结果：
   - `DeepSeekMoE 2B` 明显强于 `GShard 2B`
   - 甚至能接近 `GShard 2.9B`
   - 还接近同等总参数 dense 上界
5. 然后把视角拉大到 `16B`：
   - 训练在 `2T` tokens 上
   - 只用约 `40%` 计算量就能接近 `DeepSeek 7B` 和 `LLaMA2 7B`
6. 再补一句对齐结果：
   - `DeepSeekMoE Chat 16B` 经过 `SFT` 后也能维持这种“低算力、强表现”的特点
7. 最后用 `145B ongoing` 收尾：
   - 说明这不是一次性 demo，而是一条准备继续扩大的路线

### 这一节应该怎么理解

摘要真正传达的是：

- 这篇论文关心的不是“有没有做出一个新的 `MoE` 变体”
- 而是“能不能把 `MoE` 真正改造成一套更高专门化、更低冗余、更容易扩展的架构”

## 1. Introduction

### 这一节在原文里做什么

Introduction 的任务不是直接讲公式，而是回答：

> 为什么在已经有很多 `MoE` 工作的情况下，DeepSeek 还要再做一篇新的 `MoE` 架构论文？

### 按段落顺序梳理

#### 段落 1：先给大背景

作者先从大家熟悉的大模型趋势讲起：

- 只要训练数据和计算预算足够，大模型继续扩展通常会更强
- 但 dense 模型继续做大，代价也会越来越高

这里的功能，是把 `MoE` 引入成一个“算力压力下的自然选择”。

#### 段落 2：承认已有 `MoE` 成果很多，但还不够

作者没有否认已有 `MoE` 的价值，反而先强调：

- `GShard`
- `Switch Transformer`
- 以及其他 Transformer-MoE 路线

已经证明了 `MoE` 作为扩展大模型的方法是可行的。

问题不是“`MoE` 行不行”，而是“现有 `MoE` 是不是已经接近理想状态”。

#### 段落 3：提出两个核心问题

这里是整个论文最关键的动机段。

作者把传统 `MoE` 的问题概括成两类：

- `knowledge hybridity`
  含义是：单个 expert 学到的知识类型太杂，不够专门
- `knowledge redundancy`
  含义是：不同 experts 之间重复学习了太多共通知识

这两个问题的共同后果是：

- expert 并没有真正形成清晰分工
- 稀疏激活省下来的计算，没有最大化转化成更高质量的专家专门化

#### 段落 4：提出两条架构策略

作者接着给出整篇论文的两条主方法：

1. `Fine-Grained Expert Segmentation`
   把 expert 切得更细，同时让每个 token 激活更多个更小的 expert
2. `Shared Expert Isolation`
   把公共知识交给始终激活的 shared experts，减轻 routed experts 的重复学习

这两条方法共同服务于一个目标：

> 让 routed experts 更像“真正的专科医生”，而不是一群能力相似的泛化子网络。

#### 段落 5：先给 2B 验证结果预告

作者在 Introduction 里就先预告了 `2B` 规模的验证结论：

- `DeepSeekMoE 2B` 明显强于 `GShard 2B`
- 能接近 `GShard 2.9B`
- 甚至接近同总参数 dense 模型这个“严格上界”

这一段很重要，因为它提前说明：

- 这篇论文不是只讲 intuition
- 作者已经用实验表明这套架构不是小修小补

#### 段落 6：再给 16B 扩展和对齐预告

接着作者把结果推进到更真实的规模：

- 把模型扩到 `16B`
- 在 `2T` tokens 上训练
- 用约 `40%` 计算量获得接近 `DeepSeek 7B` 与 `LLaMA2 7B` 的性能
- 还能做 `SFT` 变成 chat model

这一段的作用是把整篇论文从“架构验证”升级成“架构 + 可扩展性 + 可对齐性”的完整故事。

#### 段落 7：最后预告 145B ongoing

作者没有把 `16B` 当终点，而是明确说：

- 他们已经在做 `145B`
- 初步结果仍然显示出对 `GShard` 的稳定优势

这段的意义是告诉读者：

> `DeepSeekMoE` 不是一个只在小中规模成立的技巧，而是打算继续向前沿规模扩展的路线。

#### 段落 8：贡献列表

Introduction 后半段给出贡献清单，基本可以整理成四项：

1. `Architectural Innovation`
   提出 `DeepSeekMoE`，核心是 finer-grained experts 和 shared experts
2. `Empirical Validation`
   通过 `2B` 规模验证它确实更接近 `MoE` 的能力上界
3. `Scalability`
   扩到 `16B` 与 `145B`，证明路线可继续放大
4. `Alignment for MoE`
   证明这条 `MoE` 路线不是只能做 base model，也能顺利做 chat alignment

#### 段落 9：全文结构说明

最后是标准的 roadmap：

- Section 2 先复述通用 Transformer-MoE
- Section 3 提出 DeepSeekMoE
- Section 4 做小规模验证、消融和专门化分析
- Section 5 做 `16B` 扩展
- Section 6 做 `16B` 对齐
- Section 7 做 `145B` 初步放大

### 关键图表与表格

#### Figure 1

虽然严格说它在后文 `5.2.2` 才被详细解释，但图在 Introduction 首页就出现了，作用是提前给读者一个结果级直觉：

- 横轴是 `activated parameters`
- 纵轴是 Open LLM Leaderboard 平均表现
- `DeepSeekMoE 16B` 明显高于同等激活参数的开源模型
- 它接近 `LLaMA2 7B`，但后者大约有 `2.5x` 的激活参数

这张图在 Introduction 里承担的是“提前建立信心”的作用。

### 小结

Introduction 真正在做四件事：

- 说明为什么 `MoE` 仍值得继续研究
- 指出传统 `MoE` 的结构性问题不是算力，而是专门化不足
- 提前展示这条路线在 `2B / 16B / 145B` 上都不是空话
- 把论文定位成一条可扩展、可对齐的架构路线，而不是单个 trick

## 2. Preliminaries: Mixture-of-Experts for Transformers

### 这一节在原文里做什么

这一节不提出新方法，而是先把“普通 Transformer-MoE 是怎么写的”讲清楚，给后面的架构改造提供统一基线。

### 按段落顺序梳理

#### 段落 1：先写标准 Transformer block

作者先写出标准 Transformer block 的两步：

1. self-attention
2. FFN

并把残差写进公式里。

这一步的功能是提醒读者：

- `MoE` 改造的位置主要在 `FFN`
- 不是在整个 Transformer 上重造一遍

#### 段落 2：再写典型 MoE 替换方式

接着作者说标准做法是：

- 把部分或全部 `FFN` 替换成 `MoE layer`
- 每个 expert 本质上还是一个 `FFN`
- 每个 token 被分配到 `top-k` experts

这里的关键点是：

- 总参数可以很大
- 但每个 token 只计算少量 experts
- 所以 `MoE` 的核心价值是“稀疏激活”

#### 段落 3：给出公式 3 到 5

作者把 generic MoE 的写法正式化：

- `h_t^l` 是多个专家输出的加权和再加残差
- `g_i,t` 是稀疏 gate
- `s_i,t` 是 token 对 expert 的 affinity
- `Topk` 决定只有少数 gate 非零

这一组公式的本质不是数学炫技，而是给后面的 DeepSeekMoE 一个可比较的起点：

> 后文所有改动，都是在“专家怎么切、怎么激活、哪些专家永远激活、如何保持平衡”这几件事上做文章。

### 关键图表与表格

#### Figure 2（前半作用）

Figure 2 在这节末尾开始出现，它先把传统 `top-2 routing` 的结构摆出来，作为后面 `DeepSeekMoE` 改造的起点。

### 小结

这节真正要读懂的是：

- 传统 `MoE` 的工作方式并不复杂
- `DeepSeekMoE` 也不是另起炉灶
- 它是基于标准 routed MoE 的结构性修正

## 3. DeepSeekMoE Architecture

### 这一章在原文里做什么

Section 3 是整篇论文的技术核心。它要回答：

> 如果传统 `MoE` 的问题是专家不够专门、还彼此重复，那架构上到底应该怎么改？

### 关键总图：Figure 2

Figure 2 是整篇论文最重要的一张结构图。它按三步展示了架构演化：

1. `(a)` 传统 `top-2 routing`
2. `(b)` 加入 `fine-grained expert segmentation`
3. `(c)` 再加入 `shared expert isolation`，形成完整 `DeepSeekMoE`

作者特别强调：

- 三种结构在比较时保持总 expert 参数量不变
- 计算量也保持不变

这很关键，因为后面的提升就不能被简单解释成“你只是多给了参数或算力”。

### 3.1 Fine-Grained Expert Segmentation

#### 这一小节在原文里做什么

回答的是：

> 如果一个 expert 太大、太粗，为什么会妨碍专门化？把它切细之后，到底改变了什么？

#### 按段落顺序梳理

##### 段落 1：先说问题

作者先指出：

- 当 expert 数量有限时，一个 expert 很容易同时覆盖多种知识
- 这样 expert 内部会挤进很多不同类型的信息
- 这些知识很难在同一组参数里被同时高效利用

换句话说，传统 `MoE` 的一个问题不是没有分工，而是分工粒度还太粗。

##### 段落 2：提出解决思路

作者的思路是：

- 在总 expert 参数量不变的前提下
- 把每个 expert 切成更小的 experts
- 由于每个 expert 更小了，就让每个 token 同时激活更多个专家
- 从而保持总体计算量不变

也就是：

- expert 更小
- 激活个数更多
- 总计算量不变

##### 段落 3：给出公式 6 到 8

这组三个公式对应的变化是：

- 总 expert 数从 `N` 变成 `mN`
- 原来只激活 `K` 个 expert，现在激活 `mK` 个
- 每个 expert 的中间维度缩小为原来的 `1/m`

数学上最关键的点不是形式，而是约束：

- 总参数不变
- 总 FLOPs 不变
- 但组合空间大得多

##### 段落 4：给出组合数例子

作者给了一个特别直观的组合学例子：

- 设 `N = 16`
- 传统 `top-2` 只有 `C(16, 2) = 120` 种组合
- 如果每个 expert 再切成 `4` 份，那么就会变成 `64` 个更小 expert
- 同时激活 `8` 个
- 可选组合数会变成 `C(64, 8) = 4,426,165,368`

这组数字的意义不是让你记住精确组合数，而是让你感受到：

> 一旦粒度变细、激活更灵活，模型可以用更多不同的稀疏子结构来承载不同知识。

#### 小结

`Fine-Grained Expert Segmentation` 解决的是：

- expert 太粗
- token 的专家组合不够灵活
- 多种知识被迫混装在同一 expert 里

作者想要的效果是：

- 把知识切得更细
- 让不同知识更容易落到不同 experts 上

### 3.2 Shared Expert Isolation

#### 这一小节在原文里做什么

回答的是：

> 即便把 expert 切细了，为什么 experts 之间还是可能重复学同样的公共知识？怎么把这种重复压下去？

#### 按段落顺序梳理

##### 段落 1：先说问题

作者指出：

- 不同 token 即使被路由到不同 experts，也经常需要一部分共同知识
- 如果没有单独的公共承载位，这些共通知识就会在多个 routed experts 里重复出现

这就是 `knowledge redundancy`。

##### 段落 2：提出 shared experts

作者的解决方案是：

- 单独拿出 `K_s` 个 experts 作为 `shared experts`
- 这些 experts 不通过 router 竞争
- 每个 token 都会固定经过它们

也就是说：

- `shared experts` 负责公共知识
- `routed experts` 负责差异化知识

##### 段落 3：为了保持计算量不变，要减少 routed 激活数

因为 shared experts 是始终激活的，所以如果不做调整，计算量就会增加。作者因此同步做了约束：

- shared experts 增加多少
- routed experts 的激活数就相应减少多少

这样总计算量仍然保持不变。

##### 段落 4：给出公式 9 到 11

这组公式把完整 `DeepSeekMoE` 写成：

- 一部分共享 experts 的输出总是加入
- 另一部分 routed experts 仍由 top-k routing 决定

在记法上：

- shared experts 数量是 `K_s`
- 总 routed experts 数是 `mN - K_s`
- 非零 routed gates 数是 `mK - K_s`

##### 段落 5：作者特别说明 shared experts 不是完全原创概念

作者特别加了一句：

- `shared expert isolation` 的原型可以追溯到 Rajbhandari 等人的工作
- 但他们当时更多是从工程角度使用
- DeepSeek 这里是把它作为“提升专家专门化”的算法设计来系统化

这句话很重要，因为它说明作者在做两件事：

- 一方面承认历史脉络
- 另一方面明确自己的创新点在“目标解释和整体整合”

#### 小结

`Shared Expert Isolation` 解决的是：

- 许多 routed experts 会重复学习共通知识
- 这会浪费参数，也削弱 routed expert 的差异化

它的本质可以记成一句话：

> 让 shared experts 吸收公共底座，让 routed experts 更专注于分化。

### 3.3 Load Balance Consideration

#### 这一小节在原文里做什么

回答的是：

> 你设计了更复杂的 routing 以后，怎么避免训练时只学少数 experts，或者多机部署时算力严重不均衡？

#### 按段落顺序梳理

##### 段落 1：先定义两类负载不均衡问题

作者一开始就说，routing 学出来后常见两类风险：

1. `routing collapse`
   只有少数 experts 被频繁选中，其他 experts 训不起来
2. 多设备部署时的 `device bottleneck`
   某些设备特别忙，导致整体吞吐下降

##### 段落 2：引入 expert-level balance loss

为了缓解 `routing collapse`，作者加了 `expert-level balance loss`。

直觉上它在约束两件事：

- 专家被选中的频率不要太失衡
- router 给不同专家的概率质量不要过度集中

公式 12 到 14 里：

- `f_i` 更接近“被选频率”
- `P_i` 更接近“概率质量”

两者乘起来，是为了避免“概率上偏爱某些专家，同时又经常真正选中它们”。

##### 段落 3：再引入 device-level balance loss

作者接着说：

- 如果模型跨设备部署
- 真正需要平衡的不一定是每个 expert 都严格一样忙
- 而是每台设备的总负载不要差太多

所以他们又设计了 `device-level balance loss`，公式 15 到 17。

##### 段落 4：解释为什么不把平衡约束设得太死

这段特别值得注意。

作者明确说：

- 如果对 expert-level balance 约束太强
- 会伤害模型性能

因此实践上采用的是：

- 较小的 expert-level balance factor，主要防 collapse
- 较大的 device-level balance factor，主要保吞吐

这体现出 DeepSeek 的一个很典型风格：

> 他们不是追求“数学上最均匀”，而是追求“性能和系统效率之间的工程平衡”。

#### 小结

这一节虽然不像前两节那么“新颖”，但它非常关键，因为它把一个好想法变成了能训练、能部署的架构。

## 4. Validation Experiments

### 这一章在原文里做什么

Section 4 的任务不是直接冲大规模，而是先回答：

> `DeepSeekMoE` 这个架构本身，值不值得继续放大？

所以它的设计很像“受控验证”：

- 先在 `2B` 左右规模验证
- 比的不是绝对最强模型
- 而是看在同等参数、同等激活参数、同等计算量约束下，它是否真的优于现有 `MoE`

### 4.1 Experimental Setup

#### 这一小节在原文里做什么

交代验证实验的训练数据、基础设施、超参数和评测集，保证后面的比较不是口头比较。

### 4.1.1 Training Data and Tokenization

#### 按段落顺序梳理

1. 训练数据来自 DeepSeek-AI 自建的大规模多语种语料
2. 主要是英文和中文，也包含其他语言
3. 数据来源很杂：
   - web text
   - math materials
   - code
   - literature
   - 其他文本材料
4. 这轮验证实验只抽了 `100B tokens` 子集
5. tokenizer 用 HuggingFace tokenizer 工具训练 `BPE`
6. 验证实验使用 `8K` 词表

#### 这一小节的意义

这里不是在炫数据规模，而是在告诉你：

- 小规模架构验证已经放在真实多领域语料上做
- 不是只在单一合成集上玩结构技巧

### 4.1.2 Infrastructures

#### 按段落顺序梳理

1. 实验基于 `HAI-LLM` 框架
2. 集成了多种并行策略：
   - tensor parallelism
   - ZeRO data parallelism
   - PipeDream pipeline parallelism
   - expert parallelism
3. 为了优化性能，还自己写了：
   - CUDA kernels
   - Triton kernels
4. 硬件环境是：
   - `A100`
   - `H800`
   - 节点内 `NVLink / NVSwitch`
   - 节点间 `InfiniBand`

#### 这一小节的意义

这节等于在说：

- 这不是纸面架构论文
- 从一开始就考虑到了训练系统怎么把它跑起来

### 4.1.3 Hyper-Parameters

#### 按段落顺序梳理

##### 模型设置

验证实验模型的主要配置是：

- `9` 层 Transformer
- hidden size `1280`
- `10` 个 attention heads
- 每个 head 维度 `128`
- 初始化标准差 `0.006`

MoE 相关配置是：

- 所有 `FFN` 都替换成 `MoE layers`
- 总 expert 参数量 = 标准 FFN 的 `16x`
- 激活 expert 参数量 = 标准 FFN 的 `2x`
- 因而每个 MoE 模型总参数约 `2B`
- 激活参数约 `0.3B`

##### 训练设置

训练设置包括：

- `AdamW`
- `β1 = 0.9`
- `β2 = 0.95`
- `weight_decay = 0.1`
- warmup + step decay 学习率调度
- 前 `2K` steps 线性 warmup
- 在 `80%` 和 `90%` 训练步数时各乘一次 `0.316`
- 最大学习率 `1.08e-3`
- gradient clipping `1.0`
- batch size `2K`
- seq length `2K`
- 每 batch `4M tokens`
- 总训练 `25,000` steps，对应 `100B tokens`
- 不用 dropout

##### 负载平衡相关工程选择

由于这批验证模型规模还不大：

- 所有参数都部署在单 GPU 上
- 不做 token dropping
- 不用 device-level balance loss
- 只保留 expert-level balance factor `0.01` 来防 collapse

#### 这一小节的意义

这一组设置说明作者想先在尽量干净的环境里看架构收益：

- 先不要让多机负载平衡复杂化结果
- 先看“结构本身值不值”

### 4.1.4 Evaluation Benchmarks

#### 按段落顺序梳理

作者把评测分成几类：

- `Language Modeling`
  - `Pile`
  - 指标是交叉熵 loss
- `Language Understanding and Reasoning`
  - `HellaSwag`
  - `PIQA`
  - `ARC-easy`
  - `ARC-challenge`
- `Reading Comprehension`
  - `RACE-middle`
  - `RACE-high`
- `Code Generation`
  - `HumanEval`
  - `MBPP`
- `Closed-Book QA`
  - `TriviaQA`
  - `NaturalQuestions`

这组 benchmark 的作用是覆盖：

- 语言建模
- 常识 / 推理
- 阅读理解
- 代码
- 知识问答

也就是让作者可以观察：

> 这个架构收益到底是广谱的，还是只在某一类任务上偶然占优。

### 4.2 Evaluations

#### 这一小节在原文里做什么

先拿 `DeepSeekMoE 2B` 与几类代表性基线正面对比，验证“它是不是比已有 MoE 真更好”。

#### 按段落顺序梳理

##### Baselines

作者对比了五类模型：

- `Dense 0.2B`
- `Hash Layer 2.0B`
- `Switch Transformer 2.0B`
- `GShard 2.0B`
- `DeepSeekMoE 2.0B`

其中最关键的公平性约束是：

- 所有模型共享同一训练语料和超参数
- 所有 MoE 总参数相同
- `GShard` 与 `DeepSeekMoE` 的激活参数相同

##### Results

作者对 `Table 1` 的解释可以整理成三步：

1. 稀疏模型相比同激活规模 dense，普遍更强
2. `GShard` 因为比 top-1 路由多激活一个 expert，略优于 `Hash Layer` 和 `Switch`
3. `DeepSeekMoE` 在与 `GShard` 同总参数、同激活参数下仍显著更强

#### 关键图表与表格

##### Table 1

最值得记住的不是所有分数，而是比较关系：

- `Pile loss`
  - Dense：`2.060`
  - Hash：`1.932`
  - Switch：`1.881`
  - GShard：`1.867`
  - DeepSeekMoE：`1.808`
- `HellaSwag`
  - `50.5 -> 54.8`（GShard 到 DeepSeekMoE）
- `PIQA`
  - `70.6 -> 72.3`
- `ARC-challenge`
  - `31.6 -> 34.3`
- `TriviaQA`
  - `10.2 -> 16.6`
- `NaturalQuestions`
  - `3.2 -> 5.7`

这张表支撑的核心结论是：

> 不是只要上 `MoE` 就够了；在已有 `MoE` 家族里，`DeepSeekMoE` 也显著更强。

### 4.3 DeepSeekMoE Aligns Closely with the upper bound of MoE Models

#### 这一小节在原文里做什么

这一节不再满足于“比同规模 `GShard` 强”，而是继续问：

> 如果给 `GShard` 更多 expert 参数和更多计算，它需要加到多大，才能追上 `DeepSeekMoE`？  
> 如果拿 dense 模型当能力上界，`DeepSeekMoE` 离这个上界还有多远？

#### 按段落顺序梳理

##### Comparison with GShard×1.5

作者先做了一个很强的对比：

- 把 `GShard` expert size 直接放大到 `1.5x`
- 也就是 expert 参数和 expert 计算都变成 `1.5x`

结果是：

- `DeepSeekMoE` 仍然能和它打平或非常接近

作者据此得到的结论是：

- 不是 `DeepSeekMoE` 靠额外计算取胜
- 而是结构效率更高

##### Comparison with Dense×16

接着作者又拿 dense 上界来比。

这里作者故意不用普通“attention:FFN = 1:2”的常规 dense 设计，而是构造了一个 `Dense×16`：

- 等价于把 `FFN` 容量做到与所有专家总容量一致

这样比较更像是在问：

> 如果我们完全不用稀疏，而是把同样的 FFN 总容量都做成 dense，上界有多高？

作者发现：

- `DeepSeekMoE` 已经非常接近这个严格上界

#### 关键图表与表格

##### Table 2

这张表最值得记住的是三组关系：

1. `DeepSeekMoE` 与 `GShard×1.5`
   - `Pile loss` 都是 `1.808`
   - 说明 `DeepSeekMoE` 接近拿更大 `GShard` 才能换来的水平
2. `DeepSeekMoE` 与 `Dense×16`
   - `Pile loss`
     - Dense×16：`1.806`
     - DeepSeekMoE：`1.808`
   - 已经非常接近
3. 计算量差距
   - `DeepSeekMoE`：`4.3T FLOPs / 2K tokens`
   - `GShard×1.5`：`5.8T`
   - `Dense×16`：`24.6T`

这张表支撑的核心论断是：

> 在这个规模上，`DeepSeekMoE` 几乎把 `MoE` 的潜力逼近到了 dense 上界附近。

### 4.4 Ablation Studies

#### 这一小节在原文里做什么

回答的是：

> `DeepSeekMoE` 的提升到底来自哪里？  
> 是 shared experts 起作用？是 finer-grained segmentation 起作用？还是只是巧合？

#### 按段落顺序梳理

##### 消融目标

作者先说明：

- 为了公平，所有消融模型都保持相同总参数和相同激活参数

所以任何变化都更容易被解释为架构设计带来的。

##### Shared Expert Isolation 的影响

作者先从 `GShard` 出发：

- 在其基础上孤立出一个 shared expert

结果发现：

- 大多数 benchmark 都有提升

这意味着：

- shared expert 不是无关紧要的工程细节
- 它确实在减少 routed experts 的公共知识重复

##### Fine-Grained Expert Segmentation 的影响

作者再做更细的粒度实验：

- 把 expert 继续切成 `32` 个或 `64` 个更小 experts
- 保持总参数与激活参数不变

Figure 3 里看到的趋势很明确：

- expert 切得越细
- 整体性能越稳定提升

也就是说：

- 粒度更细不是噪声
- 它确实提升了模型的知识分解能力

##### Shared / Routed 比例探索

在最细 `64` experts 的配置下，作者又比较了：

- `1`
- `2`
- `4`

个 shared experts 的情况。

他们观察到：

- 不同比例总体性能差异不算特别大
- 但 `1:3` 这个比例在 `Pile loss` 上略好
  - `1` 个 shared：`1.808`
  - `2` 个 shared：`1.806`
  - `4` 个 shared：`1.811`

因此后续放大时，他们保留了：

> shared experts : activated routed experts = `1:3`

#### 关键图表与表格

##### Figure 3

Figure 3 做的不是单个任务对比，而是归一化后的总体表现比较。

它支撑的结论很直接：

- `shared expert isolation` 有收益
- `fine-grained segmentation` 也有收益
- 两者叠加后效果最好

#### 小结

这节的价值很大，因为它把“这是一个组合方法”拆开验证了：

- 不是只有 shared experts 在起作用
- 也不是只有切细 experts 在起作用
- 两个设计确实都对性能有独立贡献

### 4.5 Analysis on Expert Specialization

#### 这一小节在原文里做什么

这一节是整篇论文最值钱的部分之一。它不再问“分数更高了吗”，而是直接问：

> 你说自己提高了 expert specialization，有没有更贴近这个概念本身的证据？

#### 按段落顺序梳理

##### 开头先定义分析对象

作者明确说：

- 这里分析的是 `DeepSeekMoE 2B`
- 具体指 Table 1 里那个配置
- 即总参数 `2.0B`
- `1` 个 shared expert
- `63` 个 routed experts
- 每次激活 `1 + 7`

##### 分析 1：routed experts 的冗余更低

作者的做法是：

- 对每个 token
- 把 routing 概率最高的一部分 experts 屏蔽掉
- 再从剩余 routed experts 里重新选 top-k
- 看 `Pile loss` 会变差多少

这个实验背后的逻辑是：

- 如果专家之间高度冗余
- 屏蔽掉最常用 experts 后，其他 experts 可以较好顶上
- 性能下降就不会太剧烈

结果见 `Figure 4`：

- `DeepSeekMoE` 比 `GShard×1.5` 对“屏蔽 top routed experts”更敏感

作者把这种敏感解释为：

- routed experts 更不可替代
- 专家之间冗余更低

##### 分析 2：shared experts 不能被 routed experts 替代

作者又做了一个非常关键的对照：

- 关掉 shared expert
- 同时多激活一个 routed expert
- 保持总计算量不变

结果：

- `Pile loss` 从 `1.808` 上升到 `2.414`

这说明：

- shared expert 不只是“额外多放了个专家”
- 它承载的是 routed experts 没有稳定吸收的基础知识

也就是说，shared expert 的作用不是可有可无，而是结构性的。

##### 分析 3：知识获取更准确

作者接着验证另一个核心主张：

> 如果组合更灵活，模型是否可以用更少的激活 routed experts 学到足够准确的知识？

做法是：

- 把激活 routed experts 数从 `3` 到 `7` 逐步变化
- 看 `Pile loss` 如何变化

`Figure 5` 显示：

- 只激活 `4` 个 routed experts 时
- `DeepSeekMoE` 的 `Pile loss` 就已经能接近 `GShard`

作者据此认为：

- `DeepSeekMoE` 的知识获取更精准
- 它不需要像传统 `GShard` 那样依赖更多激活专家来弥补粗粒度路由

##### 分析 4：半激活设置仍然强

作者最后又进一步验证：

- 在从头训练的前提下
- 让 `DeepSeekMoE` 只用一半的激活 routed experts

`Figure 6` 显示：

- 即便只用一半激活 expert 参数
- `DeepSeekMoE` 依然能超过 `GShard`

这组证据很重要，因为它把前面的“更准确知识获取”进一步推进成了：

- 更少激活也行
- 说明专门化和组合效率确实更高

#### 关键图表与表格

##### Figure 4

作用：

- 看“禁用高路由专家”后模型掉得有多快

结论：

- `DeepSeekMoE` 掉得更快
- 不是坏事
- 反而说明 routed experts 更不可替代、冗余更低

##### Figure 5

作用：

- 看需要多少个 activated routed experts 才能达到可接受表现

结论：

- 只用 `4` 个 routed experts，`DeepSeekMoE` 就能达到接近 `GShard` 的 `Pile loss`

##### Figure 6

作用：

- 比较“半激活”版本的 `DeepSeekMoE` 和 `GShard`

结论：

- 即使 activated expert 参数减半，`DeepSeekMoE` 仍强于 `GShard`

#### 小结

如果只选一个章节来理解“为什么 DeepSeek 认为自己真的提高了专家专门化”，那就是这一节。

它给出的证据不是单纯 benchmark 更高，而是更接近概念本身：

- experts 更不可替代
- shared expert 真在承载公共知识
- 更少激活也能更准确获取知识

## 5. Scaling up to DeepSeekMoE 16B

### 这一章在原文里做什么

Section 5 的任务是把故事从“2B 验证”推进到：

> 这套架构能不能在真正有竞争力的模型规模上成立？

### 5.1 Experimental Setup

### 5.1.1 Training Data and Tokenization

#### 按段落顺序梳理

1. 训练语料仍来自 Section 4 那套多语种语料
2. 但规模从 `100B` 提升到 `2T tokens`
3. 这个 token 数刻意与 `LLaMA2 7B` 对齐
4. tokenizer 仍然是 BPE
5. 词表扩大到 `100K`

#### 这一小节的意义

这里表明作者已经从“结构验证数据量”切换到“真正做出可竞争 base model 的训练量级”。

### 5.1.2 Hyper-Parameters

#### 按段落顺序梳理

##### 模型设置

`DeepSeekMoE 16B` 的关键配置是：

- `28` 层
- hidden size `2048`
- `16` 个 attention heads
- 每个 head 维度 `128`
- 初始化标准差 `0.006`

MoE 部分配置是：

- 除了第一层之外，其余 `FFN` 都替换成 `MoE`
- 第一层保留 dense，因为作者观察到第一层 load balance 收敛更慢
- 每层 `2` 个 shared experts
- `64` 个 routed experts
- 每个 expert 大小是标准 FFN 的 `0.25x`
- 每个 token 激活：
  - `2` 个 shared experts
  - `6 / 64` 个 routed experts

总体结果：

- 总参数约 `16.4B`
- 激活参数约 `2.8B`

##### 为什么没有继续切得更细

作者明确提到：

- 更细的 expert segmentation 在理论上还能做
- 但在 `16B` 这个规模下，过小 expert 可能损害计算效率

这其实是一句很重要的工程判断：

- 论文不是说“越细越好”
- 而是在“专门化收益”和“kernel / 通信效率”之间取平衡

##### 训练设置

训练设置延续类似思路：

- `AdamW`
- `β1 = 0.9`
- `β2 = 0.95`
- `weight_decay = 0.1`
- warmup + step decay
- 前 `2K` steps warmup
- `80%` 和 `90%` 处各乘 `0.316`
- 最大学习率 `4.2e-4`
- gradient clipping `1.0`
- batch size `4.5K`
- seq length `4K`
- 每 batch `18M tokens`
- 总步数 `106,449`
- 总训练量 `2T tokens`
- 不用 dropout

##### 并行与 balance 选择

这一版使用：

- pipeline parallelism
- 不同层放在不同设备
- 但每层所有 experts 放在同一设备

因此作者在这一版：

- 不做 token drop
- 不用 device-level balance loss
- 只用很小的 expert-level balance factor `0.001`

作者还解释了原因：

- 在这种并行方式下，提高 expert-level balance factor 不能提升计算效率
- 反而会损害模型性能

这句话非常像 DeepSeek 后续论文的一贯风格：他们很早就在做“系统约束下的最优 balance”。

### 5.1.3 Evaluation Benchmarks

#### 按段落顺序梳理

与 Section 4 相比，这里把 benchmark 扩充了。

新增或变化包括：

- `Pile` 改用 `BPB` 作为公平指标
  - 因为 `DeepSeekMoE 16B` 与 `LLaMA2 7B` tokenizer 不同
- 新增阅读理解：
  - `DROP`
- 新增数学：
  - `GSM8K`
  - `MATH`
- 新增多学科选择题：
  - `MMLU`
- 新增消歧：
  - `WinoGrande`
- 新增中文 benchmark：
  - `CLUEWSC`
  - `CEval`
  - `CMMLU`
  - `CHID`
- 还额外评测 `Open LLM Leaderboard`
  - 其中包括 `ARC`
  - `HellaSwag`
  - `MMLU`
  - `TruthfulQA`
  - `Winogrande`
  - `GSM8K`

#### 这一小节的意义

作者从这里开始，已经不只是验证“结构值不值”，而是在验证：

> 这条路线能不能在英中双语、代码、数学、常识和通用榜单上都进入真正的竞争区间。

### 5.2 Evaluations

### 5.2.1 Internal Comparison with DeepSeek 7B

#### 这一小节在原文里做什么

回答的是：

> 在完全相同语料、相同训练 token 数下，`DeepSeekMoE 16B` 和自家 dense `DeepSeek 7B` 相比，究竟值不值？

#### 按段落顺序梳理

##### Baselines

这里的对比对象只有一个，但非常重要：

- `DeepSeek 7B (Dense)`

公平性在于：

- 两者都训练在同样的 `2T` 语料上

因此比较能尽量隔离掉语料差异的影响。

##### Results

作者对 Table 3 的解读可以拆成四层：

1. 整体上，`DeepSeekMoE 16B` 用约 `40.5%` 计算量做到接近 `DeepSeek 7B`
2. 它在语言建模和知识密集型任务上有明显优势
   - `Pile`
   - `HellaSwag`
   - `TriviaQA`
   - `NaturalQuestions`
3. 它在多项选择题上偏弱
4. 这个弱点被解释为 attention 参数不足，而不是 FFN 路线失效

作者给了一个非常重要的解释：

- `DeepSeekMoE 16B` attention 参数约 `0.5B`
- `DeepSeek 7B` attention 参数约 `2.5B`

也就是说：

- 稀疏化主要发生在 `FFN`
- 但选择题类任务可能更吃 attention capacity

这是这篇论文里最关键的“优点与短板同时说清楚”的地方之一。

#### 关键图表与表格

##### Table 3

最值得记住的几组数字：

- 计算量：
  - DeepSeek 7B：`183.5T FLOPs / 4K tokens`
  - DeepSeekMoE 16B：`74.4T`
- `Pile (BPB)`
  - `0.75 -> 0.74`
- `HellaSwag`
  - `75.4 -> 77.1`
- `GSM8K`
  - `17.4 -> 18.8`
- `MATH`
  - `3.3 -> 4.3`
- `TriviaQA`
  - `59.7 -> 64.8`
- `NaturalQuestions`
  - `22.2 -> 25.5`

但在多选题上：

- `MMLU`
  - `48.2 -> 45.0`
- `CEval`
  - `45.0 -> 40.6`
- `CMMLU`
  - `47.2 -> 42.5`

##### 表后文字里的工程结论

这一节后面的文字还额外给了两个非常实用的信息：

- `DeepSeekMoE 16B` 能单卡 `40GB GPU` 部署
- 配合算子优化，推理速度接近 `7B dense` 的 `2.5x`

这说明这篇论文的目标不只是“训练便宜”，还有：

- 可部署
- 可实际跑得快

### 5.2.2 Comparison with Open Source Models

#### 这一小节在原文里做什么

把比较对象从“自家 dense 基线”扩展到：

- `LLaMA2 7B`
- 以及一整组开源模型

目的是回答：

> DeepSeekMoE 16B 放到更广阔的开源生态里，位置到底在哪？

#### 按段落顺序梳理

##### 与 LLaMA2 7B 的内部 benchmark 对比

作者先比 `LLaMA2 7B`：

- 两者都训练了 `2T tokens`
- `DeepSeekMoE 16B` 总参数是它的 `245%`
- 但只需要 `39.6%` 计算量

作者给出的观察包括：

1. 以约 `40%` 计算量，在大多数 benchmark 上超过 `LLaMA2 7B`
2. 数学和代码能力明显更强
3. 中文 benchmark 明显更强
4. 即便英文语料少于 `LLaMA2 7B`，在英文理解和知识任务上仍能持平或更强

##### Open LLM Leaderboard 对比

接着作者把视野放到更大的开源集合：

- `LLaMA 7B`
- `Falcon 7B`
- `GPT-J 6B`
- `RedPajama-INCITE`
- `Open LLaMA`
- `OPT 2.7B`
- `Pythia 2.8B`
- `GPT-Neo 2.7B`
- `BLOOM 3B`

Figure 1 的核心结论是：

- 在相似 activated parameters 条件下，`DeepSeekMoE 16B` 明显更强
- 它接近 `LLaMA2 7B`
- 而 `LLaMA2 7B` 约有 `2.5x` activated parameters

#### 关键图表与表格

##### Table 4

最值得记住的是：

- 计算量：
  - `LLaMA2 7B`: `187.9T`
  - `DeepSeekMoE 16B`: `74.4T`
- `HumanEval`
  - `14.6 -> 26.8`
- `MBPP`
  - `21.8 -> 39.2`
- `GSM8K`
  - `15.5 -> 18.8`
- `MATH`
  - `2.6 -> 4.3`
- 中文 benchmark 提升尤为明显
  - `CLUEWSC`: `64.0 -> 72.1`
  - `CEval`: `33.9 -> 40.6`
  - `CMMLU`: `32.6 -> 42.5`
  - `CHID`: `37.9 -> 89.4`

##### Figure 1

Figure 1 放在这里再看，意义更完整：

- 它不是单次 benchmark 表
- 而是一个“激活参数效率图”

这张图支撑的是整篇论文最核心的对外结论之一：

> `DeepSeekMoE` 把“同等激活规模下的能力密度”做高了。

### 小结

Section 5 真正证明的是：

- `DeepSeekMoE` 不只在 `2B` 上有效
- 它能扩成真实可竞争的 base model
- 而且其优势主要体现在：
  - 语言建模
  - 知识型任务
  - 数学 / 代码
  - 中文场景
- 但在多选题上仍有 attention-capacity 短板

## 6. Alignment for DeepSeekMoE 16B

### 这一章在原文里做什么

这章回答一个很实际的问题：

> 很多 `MoE` 模型预训练后还行，但微调和对齐收益不明显。`DeepSeekMoE 16B` 能不能顺利做成 chat model？

### 6.1 Experimental Setup

#### 按段落顺序梳理

##### 训练数据

作者使用的是自家整理的 `1.4M` SFT 样本，覆盖：

- math
- code
- writing
- question answering
- reasoning
- summarization
- 更多泛任务

并强调：

- 以英语和中文为主
- 目标是做双语通用 chat model

##### 超参数

SFT 设置相对直接：

- batch size `1024`
- `8` epochs
- `AdamW`
- max seq length `4K`
- 尽量 pack 满
- 不用 dropout
- 常数学习率 `1e-5`
- 不做额外学习率调度

##### 评测集改动

chat 模型评测与 Section 5 类似，但做了三点调整：

1. 去掉 `Pile`
   - 因为 chat model 很少拿来做纯语言建模
2. 去掉 `CHID`
   - 因为结果不够稳定
3. 新增 `BBH`
   - 更完整评测 reasoning

#### 这一小节的意义

这里传达的是：

- 作者没有把 alignment 做成复杂的多阶段 RLHF
- 而是先用标准 SFT 验证：这个架构能不能正常吃对齐数据并转成 chat model

### 6.2 Evaluations

#### 按段落顺序梳理

##### Baselines

为了公平，作者拿同样 SFT 数据训练了三套 chat 模型：

- `LLaMA2 SFT 7B`
- `DeepSeek Chat 7B`
- `DeepSeekMoE Chat 16B`

##### Results

作者对 Table 5 的总结可以拆成四点：

1. 只用约 `40%` 计算量，`DeepSeekMoE Chat 16B` 与两个 `7B dense` chat 模型整体可比
2. 在代码任务上尤其亮眼
   - `HumanEval`
   - `MBPP`
3. 在 `MMLU / CEval / CMMLU` 这类多选题上，仍落后于 `DeepSeek Chat 7B`
4. 但相比 base model，这个差距在 SFT 后缩小了

作者还强调：

- bilingual pretraining 让它在中文 benchmark 上明显强于 `LLaMA2 SFT 7B`

#### 关键图表与表格

##### Table 5

最值得记住的关系：

- 计算量：
  - `LLaMA2 SFT 7B`: `187.9T`
  - `DeepSeek Chat 7B`: `183.5T`
  - `DeepSeekMoE Chat 16B`: `74.4T`
- 代码：
  - `HumanEval`: `35.4 / 45.1 / 45.7`
  - `MBPP`: `27.8 / 39.0 / 46.2`
- reasoning / knowledge：
  - `PIQA`: `76.9 / 78.4 / 79.7`
  - `TriviaQA`: `60.1 / 59.5 / 63.3`
- 中文：
  - `CLUEWSC`: `48.4 / 66.2 / 68.2`

这张表支撑的结论是：

> `DeepSeekMoE` 不是只能做 base model，它也能被正常对齐，并保留“低计算、强能力”的优势。

### 小结

这一章把论文从“架构 + 预训练”推进到了“架构 + 预训练 + 对齐”。

对整条路线来说，这很关键，因为它说明：

- `DeepSeekMoE` 不是一个实验室结构
- 它能顺利接到产品化更近的 chat setting

## 7. DeepSeekMoE 145B Ongoing

### 这一章在原文里做什么

Section 7 的任务不是交付最终版 `145B`，而是回答：

> 如果把这条路继续放大到百亿级以上，它的优势还在不在？

### 7.1 Experimental Setup

#### 按段落顺序梳理

##### 数据与 tokenizer

- 语料和 tokenizer 与 `16B` 相同
- 只是这轮初步研究只训练了 `245B tokens`

##### 模型设置

`DeepSeekMoE 145B` 的核心配置：

- `62` 层
- hidden size `4096`
- `32` 个 attention heads
- 每个 head 维度 `128`
- 初始化标准差 `0.006`
- 除第一层外其余 `FFN` 都改成 `MoE`
- 每层：
  - `4` 个 shared experts
  - `128` 个 routed experts
  - 每个 expert 大小是标准 FFN 的 `0.125x`
- 每 token 激活：
  - `4` 个 shared experts
  - `12 / 128` 个 routed experts

总体规模：

- 总参数约 `144.6B`
- 激活参数约 `22.2B`

##### 训练设置

训练设置包括：

- `AdamW`
- `β1 = 0.9`
- `β2 = 0.95`
- `weight_decay = 0.1`
- warmup + constant LR
- 前 `2K` steps warmup
- 最大 LR `3.0e-4`
- gradient clipping `1.0`
- batch size `4.5K`
- seq length `4K`
- 每 batch `18M tokens`
- 总训练 `13,000` steps = `245B tokens`
- 不用 dropout

##### 并行与 balance 选择

这一版开始真正用到：

- expert parallelism + data parallelism
- 每层 routed experts 均匀部署到 `4` 台设备

因此这次 device-level balance 开始变得重要：

- `device-level balance factor = 0.05`
- `expert-level balance factor = 0.003`

这里可以看出：

- 小规模验证里 device-level balance 可以先省
- 到更大规模，它就必须显式进入训练设计

### 7.2 Evaluations

#### 按段落顺序梳理

##### Baselines

作者比较了三类基线：

- `DeepSeek 67B (Dense)`
- `GShard 137B`
- `DeepSeekMoE 142B (Half Activated)`

其中 `Half Activated` 特别有意思：

- 保持相近总参数
- 但 shared experts 只保留 `2`
- routed 激活数只保留 `6 / 128`

它是在问：

> 大规模下，是否还能继续减少 activated expert 参数，而不明显掉太多能力？

##### Results

作者总结 Table 6 的结论主要有三条：

1. `DeepSeekMoE 145B` 明显优于 `GShard 137B`
2. 用仅 `28.5%` 计算量，就能整体接近 `DeepSeek 67B (Dense)`
3. 半激活版 `DeepSeekMoE 142B` 也仍然很强
   - 仅 `18.2%` 计算量就能接近 `DeepSeek 67B`
   - 还超过 `GShard 137B`

#### 关键图表与表格

##### Table 6

最值得记住的关系：

- `DeepSeek 67B`：`2057.5T FLOPs / 4K tokens`
- `GShard 137B`：`572.7T`
- `DeepSeekMoE 145B`：`585.6T`
- `DeepSeekMoE 142B Half Activated`：`374.6T`

几个代表性结果：

- `Pile loss`
  - Dense：`1.905`
  - GShard：`1.961`
  - MoE 145B：`1.876`
  - Half Activated：`1.888`
- `TriviaQA`
  - `57.2 / 52.5 / 61.1 / 59.8`
- `NaturalQuestions`
  - `22.6 / 19.0 / 25.0 / 23.5`

和 `16B` 一样，多选题仍然是相对短板：

- `MMLU`
  - `45.1 / 26.3 / 39.4 / 37.5`
- `CEval`
  - `40.3 / 26.2 / 37.1 / 32.8`

#### 小结

这一节真正证明的是：

- `DeepSeekMoE` 的优势不是只停留在中等规模
- 到 `145B` 级别，仍然能维持：
  - 对 `GShard` 的结构优势
  - 对 dense 的高计算效率

## 8. Related Work

### 这一节在原文里做什么

Related Work 的任务是把 `DeepSeekMoE` 放回历史脉络里，说明它不是凭空出现的。

### 按段落顺序梳理

1. 最早的 `MoE` 可追溯到 `Jacobs` 和 `Jordan`
2. `Shazeer` 把 `MoE` 用到语言模型
3. 进入 Transformer 时代后：
   - `GShard`
   - `Switch Transformer`
   - `Hash Layer`
   - `StableMoE`
   - `expert-choice routing`
   - `ST-MoE`
   等路线分别从：
   - 路由方式
   - 训练稳定性
   - fine-tuning 难度
   做过改造
4. 作者最后明确自己的定位：
   - 现有很多工作仍围绕常见 `top-1 / top-2 routing`
   - 但 expert specialization 还有很大提升空间
   - `DeepSeekMoE` 就是把“提高专门化”作为首要目标来设计

### 小结

这节的重点不是记住所有前人工作，而是理解作者怎样给自己定位：

> 他们不是主要在做训练稳定性或工程 patch，而是在把“专家真正专门化”当成主问题。

## 9. Conclusion

### 这一节在原文里做什么

Conclusion 负责把全文的主张收束成一句清晰的话。

### 按原文信息顺序梳理

1. 先重申目标：
   - `DeepSeekMoE` 要追求的是更高 expert specialization
2. 再重申方法：
   - `fine-grained expert segmentation`
   - `shared expert isolation`
3. 再重申证据链：
   - `2B` 验证它优于现有 MoE，并接近 MoE 上界
   - `16B` 证明它能在 `2T` tokens 上扩展到真实可竞争模型
   - `SFT` 证明它可做对齐
   - `145B ongoing` 证明它仍能向更大规模推进
4. 最后补一个很实践的信息：
   - `DeepSeekMoE 16B` checkpoint 会公开
   - 可单卡 `40GB` 部署

### 小结

Conclusion 真正收束的不是“我们又发了一个模型”，而是：

> `DeepSeekMoE` 这条路同时满足了更高专门化、更高参数效率、更低激活计算，以及继续扩展到真实模型规模的可行性。

## 读完整篇后最该带走的 7 个点

1. 这篇论文的真正目标不是“做个更大的 `MoE`”，而是“把 expert specialization 这件事做真”。
2. 作者把传统 `MoE` 的核心问题概括成两类：`knowledge hybridity` 和 `knowledge redundancy`。
3. `fine-grained expert segmentation` 的价值，不是简单切小，而是在固定参数和计算下大幅提升 expert 组合灵活性。
4. `shared expert isolation` 的价值，不是多放几个永远激活的专家，而是把公共知识从 routed experts 中剥离出来。
5. `Section 4.5` 是整篇论文最值钱的证据段，因为它真正围绕“专门化”本身做了分析，而不只是报 benchmark。
6. `DeepSeekMoE 16B` 的优势主要体现在更高的激活参数效率、更强的知识 / 数学 / 代码能力，以及更好的双语覆盖；短板主要在 attention 参数偏少带来的多选题能力不足。
7. 对 DeepSeek 路线来说，这篇论文真正完成的是：把 `FFN -> MoE` 这条经济化扩展路线打通。

## 如何回到路线笔记

回到 [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md) 时，可以只记一句话：

> 第二阶段前半段，DeepSeek 给 `FFN` 这一侧的答案，是 `DeepSeekMoE`：更细粒度、更低冗余、更高专门化的稀疏 FFN。

## 相关笔记

- [DeepSeek V1-V4 研究路线笔记](deepseek-v1-to-v4-research-route.md)
- [DeepSeek LLM 论文阅读笔记](deepseek-llm-v1-paper-reading.md)
- [DeepSeek-V2 论文阅读笔记](deepseek-v2-paper-reading.md)
