# DeepSeek V1-V4 研究路线笔记

- Created: 2026-04-29
- Updated: 2026-04-30
- Type: learning
- Status: draft
- Tags: deepseek, llm-training, pretraining, posttraining, moe, long-context, rl
- Model: GPT-5.4
- Harness: Codex
- Source: organized from a user-provided Bilibili video summary and verified against official DeepSeek papers, reports, and transparency pages

## 背景

这份笔记的目标，不是把 DeepSeek 每一篇论文都逐页复述一遍，而是先建立一条可持续研究的主线：

- 用 `DeepSeek LLM / V1` 到 `V4` 的演变，理解近两年前沿大模型的训练范式变化
- 把视频里的通俗叙事，重新对齐到官方论文里的真实技术节点
- 为后续再看 `MiMo-V2-Flash` 做一个更稳的参照系

这份笔记会持续增量更新。当前版本优先解决“总路线、关键动机、代表方法和学习顺序”，而不是把所有公式一次讲完。

## 问题或目标

> 用 DeepSeek 这条技术路线，建立对最新大模型预训练、架构优化、后训练和系统工程范式的整体理解。

## 先把名字讲清楚

在视频或二手总结里，常会把 DeepSeek 的路线简化成 `V1 -> V2 -> V3 -> V4`。这个讲法有助于建立直觉，但如果直接拿来记技术史，很容易混乱。

当前更准确的理解是：

1. 视频里说的 `V1`，更接近官方的早期基础模型阶段，也就是 **DeepSeek LLM**，重点是 `scaling law` 与基础预训练。
2. `R1` 不是 `V4` 的一个附属章节，而是 **后训练范式** 的独立跃迁：它在问“推理能力能不能主要靠 RL 长出来”。
3. `V3.2` 是连接 `V3/R1` 与 `V4` 的桥，尤其重要，但本笔记当前把它视为“需要下一轮精读”的过渡节点。
4. 在 `V3.2` 和 `V4` 之间，还应该单独看到一个 **DeepSeek mHC** 节点。它代表的不是又一个普通缩写，而是 DeepSeek 对连接结构本身的一次系统性改造：从传统残差连接，走向 `HC`（Hyper-Connections），再走向 `mHC`（Manifold-Constrained Hyper-Connections）。

所以更适合学习的路线不是单纯版本故事，而是：

- `预训练路线`
- `架构路线`
- `后训练路线`
- `系统工程路线`

## 先给总判断

如果只用一句话概括 DeepSeek 的演变：

> DeepSeek 不是在单纯把模型做大，而是在同时推进四件事：更合理的 scaling、更经济的稀疏架构、更强的 RL 推理、更可控的长上下文与大规模训练系统。

把这条路线压成最短版本，大致是：

- `DeepSeek LLM / V1`：先把基础模型和 `scaling law` 弄清楚
- `DeepSeekMoE + V2`：解决“怎么更便宜地把模型做大、把推理做快”
- `V3`：解决“怎么把稀疏大模型稳定地训练到超大规模”
- `R1`：解决“推理能力能不能主要靠 RL 训练出来”
- `mHC`：解决“连接结构怎样在更强表达力下仍保持大规模训练稳定”
- `V4`：解决“怎么把长上下文、稳定训练、specialist RL 和能力融合整合到同一代模型里”

## 一张总图

```mermaid
flowchart LR
    A["DeepSeek LLM / V1<br/>scaling law + dense base model"] --> B["DeepSeekMoE<br/>shared experts + finer expert segmentation"]
    B --> C["DeepSeek-V2<br/>DeepSeekMoE + MLA"]
    C --> D["DeepSeek-V3<br/>larger MoE + aux-loss-free load balancing + MTP"]
    D --> E["DeepSeek-R1<br/>pure RL reasoning line"]
    D --> F["DeepSeek-V3.2<br/>DSA + stronger RL + agentic task synthesis"]
    F --> H["DeepSeek mHC<br/>Residual -> HC -> mHC"]
    E --> G["DeepSeek-V4<br/>CSA/HCA + specialist RL + GRM + OPD"]
    H --> G
```

## 版本对照表

| 阶段 | 代表材料 | 主要动机 | 核心方法 | 一句话本质 |
| --- | --- | --- | --- | --- |
| DeepSeek LLM / `V1` 起点 | [DeepSeek LLM](https://arxiv.org/abs/2401.02954), 2024-01-05 | 先把开源基础模型的 `scaling law`、数据和训练配比做对 | dense transformer、scaling law、后训练以 `SFT + DPO` 为主 | 探究`scaling law`配比，先把“基础盘”打稳 |
| DeepSeekMoE | [DeepSeekMoE](https://arxiv.org/abs/2401.06066), 2024-01-11 | dense 模型继续变大时，计算成本过高 | finer-grained experts、shared experts；**FFN -> DeepSeekMoE** | 用更聪明的稀疏化替代简单堆参数 |
| DeepSeek-V2 | [DeepSeek-V2](https://arxiv.org/abs/2405.04434), 2024-05-07 | 同时降低训练成本、推理成本和 KV cache 成本 | `DeepSeekMoE + MLA`；**MHA -> MLA** | 稀疏 FFN 和压缩注意力一起落地 |
| DeepSeek-V3 | [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437), 2024-12-27 | 让超大 MoE 模型在真实工程中稳定、便宜地训练出来 | `MLA`、`DeepSeekMoE`、aux-loss-free load balancing、`MTP`、 FP8 Training | 真正把“超大稀疏模型训练系统”跑通 |
| DeepSeek-R1 | [DeepSeek-R1](https://arxiv.org/abs/2501.12948), 2025-01-22 | 让 reasoning 不只靠 SFT，而是靠 RL 真正长出来 | `R1-Zero`、pure RL、multi-stage RL、cold-start data | 推理能力成为一条独立后训练主线 |
| DeepSeek-V3.2 | [DeepSeek-V3.2](https://arxiv.org/abs/2512.02556), 2025-12-01 | 把 reasoning、agent 与长上下文效率进一步结合 | **`DSA`**、更大规模 RL、agentic task synthesis | 是通向 V4 的过渡桥梁 |
| DeepSeek mHC | [mHC: Manifold-Constrained Hyper-Connections](https://arxiv.org/abs/2512.24880), 2025-12-31 | `HC` 相比传统残差更灵活，但会破坏 identity mapping，带来训练不稳和扩展性问题 | **Residual -> HC -> mHC**，把 `HC` 的残差映射投影到特定流形上，恢复 identity mapping，并结合基础设施优化保证效率 | 是 `V4` 之前连接结构演进的关键桥梁 |
| DeepSeek-V4 | [DeepSeek-V4](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/resolve/main/DeepSeek_V4.pdf), 2026-04-24 | 把超长上下文、稳定训练、specialist post-training 和能力融合整合起来 | **`CSA/HCA`**、`mHC`、`GRM`、`OPD`、million-token curriculum | 前几代技术积累的阶段性总整合 |

## 四条演进线

### 1. 预训练线

这一条线在回答：

> 数据、模型规模、上下文长度和训练目标，怎样组合才能让基础模型继续有效扩张？

从 DeepSeek 路线看，重点依次变成：

- `scaling law`
- 稀疏架构下的大规模预训练
- 更长上下文的 curriculum
- `MTP` 这类更强训练目标
- 优化器和数值稳定性

### 2. 架构线

这一条线在回答：

> 不只是把 transformer 堆大，而是怎样改它，才能让训练和推理都更便宜？

DeepSeek 的代表动作是：

- `FFN -> DeepSeekMoE`
- `MHA / GQA / MQA` 之外的 `MLA`
- 更长上下文里的 `DSA`、`CSA/HCA`
- `Residual -> HC -> mHC` 这条连接结构演化线

### 3. 后训练线

这一条线在回答：

> 模型会说话，不等于模型会推理；模型会推理，也不等于模型能在复杂任务里持续做对。那后训练应该怎么演化？

这条线的大致变化是：

- `SFT + DPO`
- `SFT + RL`
- `R1-Zero` 代表的 pure RL 推理路线
- `specialist SFT + GRPO`
- `GRM`
- `OPD`

### 4. 系统工程线

这一条线在回答：

> 再好的算法，如果训练不稳、服务不起、KV cache 爆炸、长上下文跑不动，就无法真正成为一代模型。

所以 DeepSeek 的路线里，系统工程不是附属品，而是主线的一部分：

- 训练成本控制
- 长上下文计算与缓存管理
- 稀疏路由稳定性
- rollout / RL 基础设施
- serving 和 deployment 的可行性

## 逐代拆解

### 1. DeepSeek LLM / `V1` 起点：先把 scaling law 和基础盘做对

专题精读见：

- [DeepSeek LLM 论文阅读笔记](deepseek-llm-v1-paper-reading.md)

#### 它在解决什么

DeepSeek 的起点不是“先做一个很花哨的新架构”，而是更底层的问题：

> 如果要持续训练自己的大模型，数据规模、模型规模、超参数和算力到底该怎么配？

#### 它的方法

在 [DeepSeek LLM](https://arxiv.org/abs/2401.02954) 里，重点是：

- 建立 base model 与 chat model
- 研究 `scaling laws for hyperparameters`
- 估计最优的模型和数据 scaling
- 在后训练阶段采用更传统的 `SFT + DPO`

#### 这一步为什么重要

这一步的重要性不在于“性能最炸裂”，而在于：

- 它奠定了后续所有更大模型的配比逻辑
- 它说明 DeepSeek 一开始就把“训练规律”当成核心研究对象
- 它让后面的 MoE、长上下文、RL 不至于建立在不稳的基础盘上

#### 直觉理解

这一步更像是：

> 先学会怎么正确地造发动机，再谈怎么加涡轮和上赛车套件。

#### 第一阶段精读：为什么 `scaling law` 是整条路线的起点

如果只看后来的 `MoE`、`RL`、`mHC`，很容易以为 DeepSeek 的强项只是“后面几代的花活很多”。  
但从 [DeepSeek LLM](https://arxiv.org/abs/2401.02954) 来看，DeepSeek 一开始做的恰恰是最底层、最不花哨、但决定后续一切上限的工作：

> 在固定 compute budget 下，到底该把资源分配给模型规模、数据规模、batch size、learning rate 中的哪一部分？

这件事之所以重要，是因为如果这一层没做对，后面所有更大的 dense 模型、MoE 模型和 RL 训练都会建立在一个“配比错误”的基础上。

#### 第一阶段精读：这篇论文到底在回答哪几个问题

这篇论文可以拆成 4 个核心问题：

1. 在开源常用配置里，`7B` 和 `67B` 这种规模要怎么训才更合理？
2. `batch size` 和 `learning rate` 是否也服从稳定的 scaling 规律？
3. 在固定计算预算下，模型和数据应该按什么比例一起放大？
4. 不同数据质量，会不会直接改变 scaling law 的结论？

这 4 个问题，就是 DeepSeek 后面所有路线的地基。

#### 第一阶段精读：论文最关键的 4 个发现

##### 1. 不只是模型和数据，`batch size` 与 `learning rate` 也应该按 scaling law 来选

论文在小规模实验上先做了网格搜索，再拟合了 compute budget `C` 与最优 `batch size / learning rate` 的幂律关系。  
他们给出的拟合结果是：

- `η_opt = 0.3118 * C^-0.1250`
- `B_opt = 0.2920 * C^0.3271`

这背后的直觉很简单：

- compute 变大时，最优 batch size 会逐渐变大
- 最优 learning rate 会逐渐变小

更重要的是，论文还指出近优解并不是一个极窄点，而是一个“较宽的带状区域”。  
这说明：

> scaling law 不是在追求一个神秘精确参数，而是在给你一个可靠的近优工作区间。

##### 2. 用 `非 embedding FLOPs/token` 表示模型规模，比单纯看参数量更合理

这是这篇论文里一个很容易被忽略、但其实非常重要的点。

论文指出，传统用参数量 `N` 表示模型规模时，会忽略注意力计算的真实开销，容易在不同规模下产生明显误差。  
所以他们改用：

- `M = non-embedding FLOPs/token`

并把 compute budget 写成：

- `C = M * D`

其中 `D` 是数据 token 数。

这一步的意义是：

> DeepSeek 不只是想“复述已有 scaling law”，而是在修正什么才是更合理的“模型规模”度量。

这也是为什么这篇论文虽然还处在 dense 时代，但已经明显带有很强的“工程化视角”。

##### 3. 数据质量会改变最优的模型/数据扩展策略

这是第一阶段里最值得你牢牢记住的发现之一。

论文明确写到：

- 数据质量越高
- 追加 compute 时就越应该把更多预算分配给模型扩展，而不是单纯继续堆更多数据

直觉上很好理解：

- 如果数据本身更干净、更有逻辑、更少噪声
- 那模型更容易从每个 token 中学到更多东西
- 这时继续增大模型，收益会更高

这条发现很重要，因为它直接改变你看“大模型扩展”的方式：

> 不是永远“模型更大 + 数据更多”就行，而是数据质量本身会改变最优扩展方向。

##### 4. `DeepSeek LLM` 不是一篇纯理论 scaling paper，它立即把这些规律用到了真实模型上

论文不是停在 curve fitting 上，而是立刻把结论用于实际训练：

- 预训练数据：`2T` tokens
- 主要语言：中文和英文
- 模型规模：`7B` 和 `67B`
- context length：`4096`

其中架构层面：

- 总体大体沿用 `LLaMA`
- 使用 `Pre-Norm + RMSNorm + SwiGLU + RoPE`
- `67B` 为了优化推理成本，已经使用了 `GQA`
- `67B` 采用了更深的网络设计：`95` 层，而不是单纯把 FFN 中间层继续加宽

这说明第一阶段虽然还不是 `MLA / MoE` 时代，但已经开始出现一个很重要的 DeepSeek 风格：

> 每一代都不是只改一个理论点，而是立刻把理论发现变成工程配置和真实模型决策。

#### 第一阶段精读：为什么它用了 `multi-step LR scheduler`

论文里还有一个很有 DeepSeek 风格的小点：  
它没有沿用最典型的 cosine scheduler，而是选了 `multi-step learning rate scheduler`。

原因不是因为它在单次训练里一定更神，而是因为它方便：

- 复用第一阶段训练
- 做 continual training
- 在扩大训练规模时减少浪费

论文明确说，虽然它和 cosine 在最终性能上基本一致，但 `multi-step` 在“可复用训练阶段”这件事上更方便。

这其实也是后面 DeepSeek 一条很稳定的工程哲学：

> 不只看单次最优，还看这套策略是不是适合持续扩展和反复复用。

#### 第一阶段精读：后训练为什么还是 `SFT + DPO`

这一阶段的后训练还不是 `R1` 那种“让 reasoning 靠 RL 自长”的路线。

它更接近 2024 年初的主流范式：

- `SFT`
- `DPO`

但论文对这一层也给了几个很值得记的实践细节：

- 收集了超过 `1M` 条 `SFT` 数据
- `67B` 在 SFT 阶段更容易过拟合，所以只训了 `2` epochs，而 `7B` 训了 `4` epochs
- 数学 SFT 数据会显著提高重复输出风险
- 为了降低 repetition，他们尝试了 `two-stage SFT` 和 `DPO`
- `DPO` 对 benchmark 基础能力影响不大，但能明显改善 open-ended generation 和 alignment

这一步的意义是：

> DeepSeek 在第一阶段并没有激进地改造后训练范式，它先用了相对稳妥的方法，把基础模型和对话模型都做扎实。

这也解释了为什么后来 `R1` 会显得那么关键：

- 第一阶段回答的是“基础模型怎么训”
- `R1` 才开始回答“推理能力怎么长”

#### 第一阶段精读：这一阶段的实践细节，为什么今天还值得学

从今天回头看，`DeepSeek LLM` 最值得学的不是它某个 benchmark 分数，而是它体现出的 4 个研究习惯：

1. 先研究规律，再上规模  
   不是盲目把模型越堆越大，而是先问 scaling 是否真的成立、参数该怎么配。

2. 先修正度量方式，再谈结论  
   他们没有照抄“参数量 = 模型规模”，而是引入 `non-embedding FLOPs/token`。

3. 把数据质量放进 scaling 讨论  
   这一步很前沿，因为它让 scaling law 不再只是“算力-模型-数据”三元关系，而开始显式纳入数据质量。

4. 理论结论必须立刻落到真实训练决策  
   batch size、learning rate、scheduler、模型深度、SFT 节奏，全都跟前面的研究结论相连接。

#### 第一阶段精读：为什么它会自然导向 `V2`

把第一阶段真正理解透后，你会发现 `V2` 几乎是顺着这个逻辑“必然长出来”的。

因为到这里为止，DeepSeek 已经基本回答了：

- 数据和模型该怎么配
- 超参数怎么配
- dense 基础模型怎样训得比较对

那接下来最自然的问题就变成了：

> 如果这些配比已经相对正确了，而我们还想继续扩展能力，最大的瓶颈是什么？

答案很快会落到两个地方：

- dense `FFN` 太贵
- attention 的 `KV cache` 太贵

于是第二阶段才会自然出现：

- `FFN -> DeepSeekMoE`
- `MHA -> MLA`

也就是说：

> `V1` 不是被 `V2` 推翻了，  
> 而是 `V1` 先把“怎么正确扩展基础模型”弄明白，`V2` 才开始解决“怎么更便宜地扩展”。

#### 第一阶段精读：这一阶段最值得带走的心智模型

- `DeepSeek LLM` 的第一目标不是发明新架构，而是校准“训练规律”
- `scaling law` 在这里不是论文装饰，而是整条路线的出发点
- 数据质量不是附属问题，它会反过来改变最优扩展策略
- 第一阶段的成功，直接为第二阶段的 `MoE + MLA` 铺路

### 2. DeepSeekMoE + V2：解决“更便宜地做大模型”

#### 它在解决什么

dense 模型继续变大后，瓶颈非常直接：

- 参数多
- 计算贵
- 推理慢
- KV cache 显存压力大

所以这一步的核心，不是只提升 benchmark，而是：

> 怎样在不按 dense 成本线性爆炸的前提下，把模型继续做强？

#### DeepSeekMoE 的关键点

[DeepSeekMoE](https://arxiv.org/abs/2401.06066) 最关键的两点是：

- 把专家切得更细，形成更灵活的 expert 组合
- 单独保留 shared experts，捕捉公共知识，减少 routed experts 的冗余

它本质上在解决一个经典 MoE 问题：

> MoE 不只是“少算一点”，还要让专家真的产生更清晰的专业分工，而不是学成一堆高度重叠的子网络。

#### DeepSeek-V2 的关键点

到了 [DeepSeek-V2](https://arxiv.org/abs/2405.04434)，这条线第一次非常完整地落地：

- `236B` total parameters
- `21B` activated parameters per token
- `128K` context
- 预训练 `8.1T` tokens
- 继续采用 `SFT + RL`

更重要的是两项代表技术：

- `DeepSeekMoE`
- `MLA`（Multi-head Latent Attention）

#### MLA 为什么重要

`MLA` 的直觉不是“又发明了一个注意力缩写”，而是：

> 想办法把推理里最贵的 `KV cache` 压缩到 latent 表示里，让长上下文和高吞吐更现实。

V2 报告明确给出的结果是：

- 相比 DeepSeek 67B，训练成本下降 `42.5%`
- `KV cache` 降低 `93.3%`
- 最大生成吞吐提升到 `5.76x`

#### 这一步的本质

V2 不是单点性能优化，而是第一次把“**经济的 MoE + 经济的注意力**”组合成一条完整路线。

### 3. V3：解决“怎么把超大稀疏模型真正训练出来”

#### 它在解决什么

V2 证明了这套方向可行，但还不等于：

- 能稳定训练到超大规模
- 能把训练成本压到足够低
- 能在不崩的情况下持续提升能力

V3 的核心问题因此变成：

> 不是“有架构”，而是“如何把这套架构真正训练成前沿模型”。

#### 它的方法

[DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437) 给出的关键点非常清楚：

- `671B` total parameters
- `37B` activated parameters per token
- 继承 `MLA + DeepSeekMoE`
- 引入 `auxiliary-loss-free` 的 load balancing
- 引入 `MTP`（multi-token prediction）
- 预训练 `14.8T` tokens
- 全训练仅用 `2.788M H800 GPU hours`

而且报告还特别强调：

- 整个训练过程中没有出现不可恢复的 loss spike
- 没有进行 rollback

#### 为什么 `MTP` 值得特别学

`MTP` 的意义不只在训练目标更强，还在于它把训练和推理效率联系起来了：

- 训练时：模型不只学下一个 token，而是更丰富的未来预测
- 推理时：可以支持更激进的 speculative decoding 设计

#### 这一步的本质

V3 代表的不是“又大一点的 V2”，而是：

> DeepSeek 已经把“超大稀疏前沿模型的预训练与工程稳定性”跑成了一条成熟路线。

### 4. R1：把“推理能力来自哪里”单独拉成一条主线

#### 它在解决什么

V3 很强，但“强 base / chat 模型”和“强 reasoning 模型”不是一回事。

R1 这条线真正想回答的是：

> 推理能力到底能不能主要靠 RL 自己长出来，而不是只能靠监督链路灌进去？

#### 它的方法

[DeepSeek-R1](https://arxiv.org/abs/2501.12948) 的关键信息是：

- `DeepSeek-R1-Zero` 先做了不以 `SFT` 为前置的大规模 RL
- 纯 RL 下出现了显著 reasoning 行为
- 也出现了可读性差、语言混杂等问题
- 于是 `DeepSeek-R1` 又引入 cold-start data 和 multi-stage training

#### 为什么这一步重要

R1 的意义不是“彻底抛弃 SFT”，而是证明了：

- RL 不只是最后一层微调
- RL 可以成为 reasoning 能力形成的核心驱动力
- 但 pure RL 直接产出的模型，不一定已经满足可用性和可读性要求

#### 这一步的本质

这条线你可以这样理解：

> `R1-Zero` 证明“能力可以长出来”，`R1` 证明“长出来的能力还需要被整形成一个可用产品”。

### 5. V3.2：从强 reasoning 走向强 long-context + agent

#### 当前应如何看待它

在这条学习路线上，`V3.2` 最适合被看成一个桥梁节点，而不是简单的“小版本更新”。

根据 [DeepSeek Transparency](https://www.deepseek.com/en/transparency/)、[DeepSeek-V3.2 Release](https://api-docs.deepseek.com/news/news251201) 和 [DeepSeek-V3.2-Exp](https://api-docs.deepseek.com/news/news250929) 的官方说明，它至少做了三件重要的事：

- 在 `V3.2-Exp` 中率先验证 `DSA`（DeepSeek Sparse Attention），探索长文本训练和推理的稀疏注意力提效
- 在正式版 `V3.2` 中进一步强化 reasoning-first 的 post-training 路线
- 建立面向复杂交互环境的 `agentic task synthesis pipeline`
- 首次把思考过程真正融入工具使用

#### 为什么它重要

官方说明里给出的细节也很值得记：

- 训练数据合成覆盖 `1800+` 个环境
- 包含 `85k+` 条复杂指令
- `V3.2` 被官方定义为首个把 thinking 直接融入 tool-use 的模型

这意味着 DeepSeek 的关注点已经不只是：

- 让模型更会答题

而是进一步转向：

- 让模型在长上下文场景里更高效
- 让 reasoning 与 tool-use / agent 任务结合

#### 当前这部分的处理方式

本笔记先把 `V3.2` 标记为：

- 已识别其关键位置
- 尚未做逐章细读

这也是我们后续很适合单独补一轮的节点。

### 6. DeepSeek mHC：把“连接结构”单独拉成一个关键节点

#### 它在解决什么

传统 transformer 里的残差连接非常成功，但它也有明显边界：

- 表达形式简单
- 跨层连接方式固定
- 当模型规模和训练深度继续增加时，连接结构本身也可能成为稳定性瓶颈

`HC`（[Hyper-Connections](https://arxiv.org/abs/2409.19606)）这条线试图解决这个问题。它的核心想法是：

> 不再把残差连接看成固定的“`x + f(x)`”，而是允许网络在不同深度间更灵活地调节和重排连接强度。

但 `HC` 也带来了新的问题。根据 [mHC 论文](https://arxiv.org/abs/2512.24880) 的摘要：

- `HC` 虽然提升了表达力
- 但它会破坏残差连接里非常关键的 `identity mapping` 性质
- 进而带来训练不稳定、可扩展性受限和额外的 memory access 开销

#### 它的方法

`mHC` 的关键动作是：

- 不是放弃 `HC`
- 而是对 `HC` 的残差映射空间施加流形约束
- 通过投影恢复 identity mapping
- 同时配合基础设施优化保证效率

如果用更直觉的话说：

> `HC` 是在说“连接可以更灵活”，`mHC` 是在说“这种灵活性必须被约束在一个可稳定训练的几何空间里”。

#### 为什么这一步应该单独看

如果直接把 `mHC` 淹没在 `V4` 的方法列表里，会看不出它真正的重要性。

更好的学习顺序是：

- 传统残差：稳定，但表达形式固定
- `HC`：更灵活，但训练更难稳
- `mHC`：在保留灵活性的同时，把稳定性问题重新补回来

这一步很关键，因为它说明 DeepSeek 在 `V4` 前，不只是加了更长上下文注意力和更强后训练，而是连 **transformer 里最基础的连接拓扑** 都在重新设计。

#### 和你提到的 Kimi 线索怎么对应

你提到 `HC` 可以记作“来源于 Kimi 的那条连接结构思路”，这个直觉对理解很有帮助。  
为了保持正文严谨，这份笔记目前先按公开论文命名写成：

- `Residual -> HC -> mHC`

如果后续我们单独补到 Kimi / Moonshot 的一手材料，再把这条关联写得更明确会更稳。

### 7. V4：把前几代积累整合成一代 frontier 系统

#### 它在解决什么

到了 V4，问题已经不再是单点突破，而是：

> 如何把超长上下文、稳定训练、specialist post-training、能力融合和部署可行性整合到同一代模型里？

#### 它的方法

[DeepSeek-V4](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/resolve/main/DeepSeek_V4.pdf) 和 [Transparency](https://www.deepseek.com/en/transparency/) 里能确认的关键信息包括：

- 官方发布时间：`2026-04-24`
- `Flash` 预训练 `32T` tokens，`Pro` 预训练 `33T` tokens
- context curriculum：`4K -> 16K -> 64K -> 1M`
- `Flash`: `284B` total / `13B` active
- `Pro`: `1.6T` total / `49B` active
- 长上下文注意力：`CSA + HCA`
- 残差连接优化：`mHC`（Manifold-Constrained Hyper-Connections）
- 后训练：`specialist SFT + GRPO`
- 对难验证任务引入 `GRM`
- 最后用 `OPD`（On-Policy Distillation）把多个专家能力融合到统一模型

#### 这些技术各自在解决什么

- `CSA/HCA`
  解决 million-token 级上下文里的注意力计算与信息保留问题
- `mHC`
  把 `HC` 的灵活连接能力转化为可在超深/超大训练中稳定工作的连接结构
- `GRM`
  解决“难任务 reward 不好写成一个简单标量”的问题
- `OPD`
  解决“多个 specialist 很强，但最终还需要一个统一 student 模型”的问题

#### 这一步的本质

V4 的意义，不是某一个孤立新技巧，而是：

> DeepSeek 把 “大规模预训练 + 长上下文 + specialist RL + reward 建模 + 能力融合” 串成了一条更完整的 frontier pipeline。

## 怎么一步一步把这条路线学明白

如果你接下来要继续深挖，我建议按这个顺序来：

### 第一步：先只抓“每代在解决什么问题”

不要先背术语，先回答：

- V1 在解决什么？
- V2 在解决什么？
- V3 在解决什么？
- R1 在解决什么？
- V4 在解决什么？

如果这个问题答不清，后面所有缩写都会变成噪声。

### 第二步：再抓“每代只有 1 到 2 个真正该记住的创新”

- V1：`scaling law`
- V2：`DeepSeekMoE + MLA`
- V3：`aux-loss-free load balancing + MTP`
- R1：`pure RL + multi-stage RL`
- V4：`CSA/HCA + mHC + GRM + OPD`

### 第三步：把术语翻成人话

例如：

- `MoE`：让不同 token 只激活少量专家，减少计算
- `MLA`：压缩 `KV cache`，让长上下文更便宜
- `MTP`：不只预测下一个 token，而是更丰富地预测多个未来 token
- `GRM`：让 reward 本身也更像一个会生成解释和判断的模型
- `OPD`：让 student 在自己的轨迹上向一个或多个 teacher 学

### 第四步：把 DeepSeek 拆成两张表来读

表 1：预训练 / 架构

- 训练数据规模
- 模型规模
- 稀疏结构
- 注意力结构
- 训练目标
- 优化器
- context 长度

表 2：后训练 / agent

- 是否有 SFT
- RL 算法
- reward 来自哪里
- 是否有 specialist
- 是否有 distillation
- 是否面向工具使用或 agent 环境

### 第五步：最后再拿 MiMo 做对照

在把 DeepSeek 路线理顺之后，再看 `MiMo-V2-Flash`，你会更清楚地比较：

- 两家在预训练目标上的不同取舍
- 两家在 post-training / agentic RL 上的不同发力点
- DeepSeek 更偏“frontier 全栈整合”，MiMo 更偏“高效 reasoning / agent / code 路线”

## 最容易混淆的点

### 1. `DeepSeek LLM` 不应粗暴等同于后来统一命名下的产品代号 `V1`

视频里这么讲有助于理解，但写技术笔记时最好标成：

- `DeepSeek LLM / V1 起点`

### 2. `R1` 是后训练路线，不只是版本号中的一个节点

它对应的是一类问题：

- reasoning 能力如何通过 RL 形成

而不只是“某一代聊天模型升级版”。

### 3. `V3` 很强，不等于 `V3` 已经等于 `R1`

- `V3` 更偏基础能力和稳定训练
- `R1` 更偏 reasoning post-training

### 4. 视频里的技术压缩叙事，不等于论文里的严格结构

比如 `DSA / CSA / HCA / mHC / OPD / GRM` 这些点，必须回到对应官方报告里逐项确认，不适合只靠二手视频记忆。

## 一句话心智模型

- DeepSeek 的起点不是“先发明花哨结构”，而是先把 `scaling law` 做明白。
- V2 的本质是：用 `MoE + MLA` 把“大模型更便宜”这件事真正落地。
- V3 的本质是：把超大稀疏模型的稳定训练和成本控制跑通。
- R1 的本质是：把“推理能力能否靠 RL 自发生长”单独拉成一条主线。
- `mHC` 的本质是：在连接结构层面，把“更强表达力”和“训练稳定性”重新兼得。
- V4 的本质是：把长上下文、specialist RL、reward 建模和能力融合整合成一套 frontier pipeline。

## 注意事项

- 这份笔记当前优先保证“主线正确、术语不乱、学习路径清晰”，不是一份逐章精读稿。
- `V3.2` 部分当前只做桥梁定位，后续值得单独扩写一轮。
- `mHC` 部分当前先按公开论文与 V4 报告建立最小主线，后续可以再补更细的数学与实现细节。
- 视频是很好的直觉引子，但涉及版本定义、术语归属和时间线时，应以官方论文和官方页面为准。

## 验证

- 2026-04-29：核对了用户提供的视频总结与脑图，并对照官方论文/报告梳理关键节点
- 主要使用的官方或一手材料：
  - [DeepSeek LLM](https://arxiv.org/abs/2401.02954)
  - [DeepSeekMoE](https://arxiv.org/abs/2401.06066)
  - [DeepSeek-V2](https://arxiv.org/abs/2405.04434)
  - [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437)
  - [DeepSeek-R1](https://arxiv.org/abs/2501.12948)
  - [DeepSeek-V3.2](https://arxiv.org/abs/2512.02556)
  - [DeepSeek-V3.2 Release](https://api-docs.deepseek.com/news/news251201)
  - [DeepSeek-V3.2-Exp](https://api-docs.deepseek.com/news/news250929)
  - [Hyper-Connections](https://arxiv.org/abs/2409.19606)
  - [mHC: Manifold-Constrained Hyper-Connections](https://arxiv.org/abs/2512.24880)
  - [DeepSeek-V4 Technical Report](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/resolve/main/DeepSeek_V4.pdf)
  - [DeepSeek Transparency](https://www.deepseek.com/en/transparency/)
  - [Bilibili 视频：深入解读 DeepSeek V1~V4](https://www.bilibili.com/video/BV1rpovBCEGH/?share_source=copy_web&vd_source=e11d1aa23ea058d42b676f314fe822a0)

## 相关笔记

- [DeepSeek LLM 论文阅读笔记](deepseek-llm-v1-paper-reading.md)
- [从数据飞轮到 Agentic RL：垂直 AI Agent 的持续改进闭环](data-flywheel-agentic-rl-and-vertical-ai-agents.md)
- 后续计划：补一篇 `MiMo-V2-Flash` 对照笔记，与本篇形成“双案例学习组”
