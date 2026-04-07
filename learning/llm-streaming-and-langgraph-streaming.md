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

## 提炼后的结论

最值得记住的一句话是：

> 流式输出 = 模型逐 token 生成 + KV cache 提速 + 服务端增量传输 + 框架把这些增量包装成应用可消费的事件流。

进一步拆开：

1. 模型先做 `prefill`，处理整段输入。
2. 然后进入 `decode`，开始一个 token 一个 token 地生成。
3. `KV cache` 负责复用历史 attention 的 K/V，避免后续 token 重复计算。
4. 服务端通过 `SSE` 持续把 token 或 chunk 推给前端。
5. LangGraph 在上层不仅能流 token，还能流图执行状态、自定义进度和任务事件。

## 核心概念

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

### 3. KV cache

`KV cache` 是 Transformer 推理阶段的缓存机制。

原因是：

- 模型生成第 N+1 个 token 时，需要参考前面的 1...N 个 token
- 如果每次都把前面所有 token 的 attention 重新算一遍，会非常浪费

所以会把前面 token 对应的 `Key/Value` 缓存起来，后续直接复用。

最重要的边界：

- `KV cache` 是推理缓存
- 它不是 agent memory
- 也不是长期知识库

### 4. SSE

`SSE = Server-Sent Events`

它是基于 HTTP 的服务端推送机制，适合把模型生成出来的 token/chunk 一点点推给客户端。

你可以把它想成：

- 非流式：整段话写完再发
- SSE 流式：写一句发一句

### 5. LangGraph 的几种流

LangGraph 官方支持多种 `stream_mode`：

- `messages`：LLM token + metadata
- `updates`：状态增量更新
- `values`：完整状态快照
- `custom`：节点里通过 `get_stream_writer()` 手动发出的自定义数据

最常用的三种是：

- `messages`
- `updates`
- `custom`

## 一条完整链路

```text
用户点击发送
  -> 前端发请求
  -> 服务端收到 prompt
  -> 模型 prefill
  -> 模型 decode
  -> KV cache 加速后续 token
  -> 服务端拿到 token/chunk
  -> SSE 持续推送
  -> 前端边收边渲染
  -> 用户看到文字一点点冒出来
```

如果中间用的是 LangGraph，那么它会在这条链路上额外接管：

- token 流
- graph state 更新
- 节点自定义进度

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
