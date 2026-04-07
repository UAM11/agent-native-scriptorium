# LLM 流式输出与 LangGraph 流机制

- Created: 2026-04-07
- Updated: 2026-04-07
- Type: learning
- Status: draft
- Tags: llm, streaming, langgraph, inference, kv-cache, sse
- Model: GPT-5.4
- Harness: Codex
- Source: a learning session on how LLM streaming works and how LangGraph exposes `messages`, `updates`, and `custom` streams

## 背景

这份笔记是为了把几个很容易混在一起的概念重新梳开：

- LLM 为什么能“一个字一个字往外冒”
- SSE 到底是什么
- KV cache 到底是什么
- LangGraph 的 `messages`、`updates`、`custom` 几种流分别在流什么

## 问题或目标

搞清楚一条完整链路：

从用户点击发送，到屏幕上开始出现流式输出，中间到底发生了什么。

## 先用 3 层把整件事拆开

刚开始学这部分时，最容易把“模型怎么生成”“网络怎么传”“框架怎么包装”混在一起。

一个更稳的理解方式是先拆成 3 层：

1. 模型层：LLM 到底是怎么“一个字一个字往外吐”的
2. 传输层：这些内容为什么能实时显示在屏幕上
3. 框架层：LangGraph 这种框架又在上面多做了什么

先有这个分层，后面再看 `KV cache`、`SSE`、`messages`、`updates`、`custom`，就不容易乱。

## 一张总图

```text
用户提问
  ↓
模型开始逐步生成 token
  ↓
服务端把 token 增量发出来
  ↓
前端一边接收一边显示
  ↓
LangGraph 还可以顺手把“节点进度 / 状态更新 / 自定义事件”一起流出来
```

这张图背后的真正含义是：

- 模型负责生成
- 网络负责传输
- 框架负责组织和包装

## 第 1 层：模型到底怎么生成文本

LLM 不是“一次性想完整段话再发出来”。

它更像这样：

- 先看你给的上下文
- 预测“下一个 token”最可能是什么
- 把这个 token 接到上下文后面
- 再预测下一个 token
- 一直重复，直到结束

这里最重要的词是 `token`。

`token` 不一定等于一个汉字，也不一定等于一个英文单词，它只是模型内部切分文本的一种单位。

所以模型生成的真实过程更接近：

```text
输入: "今天天气"
预测下一个 token
再预测下一个 token
再预测下一个 token
...
```

你最后看到的是一句完整的话，但模型底层实际做的是 `next-token prediction`。

## 第 2 层：什么叫“流式输出”

模型本来就是一步一步生成的。

区别只在于：

- 非流式：服务端等整段都生成完，再一次性返回
- 流式：服务端每生成一点，就立刻往外发一点

所以：

- 流式输出不等于“模型换了一种推理方式”
- 它更多是“把模型本来就逐步生成的过程实时暴露给你看”

可以把它想成：

- 非流式：写完整封信再寄给你
- 流式：写一句递一句给你看

## 第 3 层：什么是 KV cache

这是理解推理加速最关键的概念之一。

假设模型已经生成了前 100 个 token，现在要生成第 101 个。

如果没有 cache，模型每次都要把前面 100 个 token 再重新参与一遍注意力计算，这会非常浪费。

Transformer 的注意力里会产生：

- `K = Key`
- `V = Value`

模型在前面 token 上算出来的这些 `K/V`，后面还会继续用，所以推理时会把它们缓存起来。

下一步生成时：

- 不再重算前面所有 token 的 K/V
- 只算新 token 的部分
- 再和历史缓存拼起来继续推理

直觉理解：

- 没有 KV cache：每说一句新话，都把前面重新读一遍
- 有 KV cache：前面的“理解结果”先记下来，后面直接接着算

这里最重要的边界是：

- `KV cache` 是推理缓存
- 它不是 agent memory
- 也不是长期知识库

## 第 4 层：什么是 SSE

`SSE = Server-Sent Events`

它是服务端往客户端持续推消息的一种方式。

你只要先记住这几个特征：

- 基于 HTTP
- 单向：`服务端 -> 客户端`
- 连接会保持一段时间
- 服务端可以不断发“事件”

所以它特别适合做：

- LLM 流式输出
- 实时日志
- 任务进度推送

为什么 LLM 常用 SSE？

因为模型每生成一个 token 或一个小片段，服务端就可以立刻发一个事件出去；客户端收到后马上显示，于是你就看到“字在往外冒”。

## 把这几个概念重新串起来

真正的流式输出过程其实就是：

- 模型层：模型一个 token 一个 token 地生成
- 加速层：`KV cache` 让后续 token 生成更快
- 传输层：服务端用 `SSE` 把这些 token 增量发给客户端

所以你看到的“流式输出”背后，本质上是：

> 逐 token 生成 + KV cache 加速 + SSE 增量传输

## 再往深一层：prefill、decode、TTFT、ITL

前面那套理解已经够用，但如果想再深入一点，还需要再认识四个词。

### 1. prefill 和 decode

- `prefill`：模型先把整段 prompt“读进去”
- `decode`：模型开始往后一个 token 一个 token 生成

直觉上可以理解为：

- `prefill`：先读题
- `decode`：开始答题

### 2. TTFT 和 ITL

- `TTFT`：`Time To First Token`，第一个 token 出现需要多久
- `ITL`：`Inter-Token Latency`，相邻 token 之间的间隔有多大

可以粗暴记成：

- `TTFT`：多久开始说话
- `ITL`：开始说之后说得快不快

这一层的意义在于：

- `prefill` 更影响第一个 token 什么时候出来
- `KV cache` 和 decode 性能更影响后续 token 的连续输出速度

## LangGraph 在这件事里多做了什么

LangGraph 不是一个模型，而是一个图式工作流运行时。

你可以把它理解成：

- 里面有很多节点 `node`
- 节点之间按图连接
- 有一份共享状态 `state`
- 某些节点会调用 LLM
- 某些节点会调工具
- LangGraph 负责整个流程怎么跑

所以它的“流式输出”不只是模型吐字。

这是 LangGraph 最关键的地方。

它可以流的东西不止一种：

- `messages`：LLM token 流
- `updates`：状态增量更新
- `values`：完整状态
- `custom`：你自己定义的事件

所以 LangGraph 的流式不是只有“模型说到哪了”，它还可以是：

- 图跑到哪一步了
- 某个节点刚完成了什么
- 工具执行进度如何

## LangGraph 流机制的心智模型

可以把 LangGraph 的流式分成两类：

### 第一类：模型 token 流

如果某个节点里调用了支持流式的 LLM：

- LLM 不断产出 token chunk
- LangGraph 通过回调拿到这些 chunk
- 再把它们包装成 `messages` 事件流出来

所以：

`模型流 -> LangGraph 接住 -> 你在 graph.stream(..., stream_mode="messages") 里读到`

### 第二类：图执行过程流

除了 token，LangGraph 自己也知道：

- 哪个节点执行了
- state 更新了什么
- 你是否主动发了进度消息

所以它还能流：

- `updates`
- `values`
- `custom`

这就是为什么 LangGraph 比“直接调 OpenAI 流式接口”更强：

- 它不仅流文本
- 它还流工作流本身

## LangGraph 最小示例

下面示例统一使用：

- `version="v2"`：LangGraph 1.1 之后推荐的统一流格式
- `part["type"]`：区分流类型
- `part["data"]`：拿具体 payload

### 示例 1：`messages` 流

作用：

- 看 LLM token 是怎么流出来的

```python
from dataclasses import dataclass

from langchain.chat_models import init_chat_model
from langgraph.graph import START, StateGraph


@dataclass
class State:
    topic: str
    joke: str = ""


model = init_chat_model(model="gpt-4.1-mini")


def call_model(state: State):
    result = model.invoke(
        [{"role": "user", "content": f"Write a short joke about {state.topic}."}]
    )
    return {"joke": result.content}


graph = (
    StateGraph(State)
    .add_node("call_model", call_model)
    .add_edge(START, "call_model")
    .compile()
)


for part in graph.stream(
    {"topic": "ice cream"},
    stream_mode="messages",
    version="v2",
):
    if part["type"] == "messages":
        message_chunk, metadata = part["data"]
        if message_chunk.content:
            print(message_chunk.content, end="", flush=True)
```

为什么它会流起来：

- 模型底层在逐 token 生成
- LangGraph 在 `messages` 模式下通过 callback 接住 token chunk
- `graph.stream(...)` 把这些 chunk 暴露成可迭代流
- 你的 `for` 循环每收到一个 chunk，就立刻打印一次

### 示例 2：`updates` 流

作用：

- 看 graph state 在每个节点后更新了什么

```python
from typing import TypedDict

from langgraph.graph import START, StateGraph


class State(TypedDict):
    topic: str
    query: str
    answer: str


def plan_query(state: State):
    return {"query": f"facts about {state['topic']}"}


def build_answer(state: State):
    return {"answer": f"stub answer for {state['query']}"}


graph = (
    StateGraph(State)
    .add_node("plan_query", plan_query)
    .add_node("build_answer", build_answer)
    .add_edge(START, "plan_query")
    .add_edge("plan_query", "build_answer")
    .compile()
)


for part in graph.stream(
    {"topic": "LangGraph", "query": "", "answer": ""},
    stream_mode="updates",
    version="v2",
):
    if part["type"] == "updates":
        for node_name, update in part["data"].items():
            print(f"{node_name}: {update}")
```

你会看到的不是 token，而是类似：

```python
{"plan_query": {"query": "facts about LangGraph"}}
{"build_answer": {"answer": "stub answer for facts about LangGraph"}}
```

所以 `updates` 的重点是：

- 哪个节点更新了 state
- 更新了哪几个字段

### 示例 3：`custom` 流

作用：

- 在节点内部手动发进度、自定义状态、业务事件

```python
from typing import TypedDict

from langgraph.config import get_stream_writer
from langgraph.graph import START, StateGraph


class State(TypedDict):
    topic: str
    answer: str


def custom_streaming_node(state: State):
    writer = get_stream_writer()

    writer({"progress": 10, "stage": "received topic"})
    writer({"progress": 50, "stage": "collecting facts"})
    writer({"progress": 90, "stage": "assembling answer"})

    return {"answer": f"final answer about {state['topic']}"}


graph = (
    StateGraph(State)
    .add_node("custom_streaming_node", custom_streaming_node)
    .add_edge(START, "custom_streaming_node")
    .compile()
)


for part in graph.stream(
    {"topic": "streaming", "answer": ""},
    stream_mode="custom",
    version="v2",
):
    if part["type"] == "custom":
        print(part["data"])
```

这里的重点不是模型输出，而是：

- 你想发什么，就可以发什么
- 非常适合进度条、阶段提示、工具执行状态

### 示例 4：`messages + updates + custom` 混合流

作用：

- 同时观察 token、状态更新和自定义进度

```python
from typing import TypedDict

from langchain.chat_models import init_chat_model
from langgraph.config import get_stream_writer
from langgraph.graph import START, StateGraph


class State(TypedDict):
    topic: str
    query: str
    answer: str


model = init_chat_model(model="gpt-4.1-mini")


def plan_query(state: State):
    writer = get_stream_writer()
    writer({"progress": 20, "stage": "planning query"})
    return {"query": f"one short joke about {state['topic']}"}


def call_model(state: State):
    writer = get_stream_writer()
    writer({"progress": 60, "stage": "calling model"})
    result = model.invoke(
        [{"role": "user", "content": state["query"]}]
    )
    writer({"progress": 100, "stage": "finished"})
    return {"answer": result.content}


graph = (
    StateGraph(State)
    .add_node("plan_query", plan_query)
    .add_node("call_model", call_model)
    .add_edge(START, "plan_query")
    .add_edge("plan_query", "call_model")
    .compile()
)


for part in graph.stream(
    {"topic": "cats", "query": "", "answer": ""},
    stream_mode=["messages", "updates", "custom"],
    version="v2",
):
    if part["type"] == "messages":
        msg, metadata = part["data"]
        if msg.content:
            print(msg.content, end="", flush=True)

    elif part["type"] == "updates":
        print("\n[UPDATE]", part["data"])

    elif part["type"] == "custom":
        print("\n[CUSTOM]", part["data"])
```

这个例子最适合建立直觉：

- `messages`：看模型正在吐什么字
- `updates`：看节点刚改了哪些状态
- `custom`：看你手动定义的业务进度

## 怎么理解这三种流的边界

### `messages`

- 来源：LLM token 流
- 粒度：细
- 用途：前端实时显示“模型正在说什么”

### `updates`

- 来源：graph 每一步后的状态变化
- 粒度：中
- 用途：追踪工作流推进到了哪一环

### `custom`

- 来源：节点内部显式调用 `writer(...)`
- 粒度：由你决定
- 用途：暴露业务进度、阶段提示、自定义 telemetry

## 一句话心智模型

- `messages` 是模型视角
- `updates` 是图状态视角
- `custom` 是应用视角

## 注意事项

- 示例默认假设你已经配置好 LangChain / LangGraph 对应的模型提供方环境变量
- `messages` 流是否可用，取决于底层模型适配层是否支持 streaming callback
- `version="v2"` 需要较新的 LangGraph 版本，推荐 LangGraph >= 1.1
- `custom` 特别适合接任意 LLM 或外部服务的自定义流，不一定要依赖 LangChain 标准 chat model 接口

## 验证

- 2026-04-07：基于 LangGraph 官方 streaming 文档、LangGraph 源码、OpenAI / Anthropic / Hugging Face / vLLM 官方资料整理

## 相关笔记

- https://docs.langchain.com/oss/python/langgraph/streaming
- https://developers.openai.com/api/docs/guides/streaming-responses
- https://platform.claude.com/docs/en/build-with-claude/streaming
- https://huggingface.co/docs/transformers/main/cache_explanation
- https://docs.vllm.ai/en/stable/configuration/optimization/
