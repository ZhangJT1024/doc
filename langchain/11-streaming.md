# 流式传输

> 原文：https://docs.langchain.com/oss/javascript/langchain/streaming
> LangChain (TypeScript) 官方文档中文整理

> 【提示】对于新应用，我们推荐使用[事件流式传输](/oss/javascript/langchain/event-streaming)——这是 LangChain v1.3 引入的类型化投影 API。事件流式传输为每个投影（消息、值、工具调用、子图）提供了独立的迭代器，因此你可以独立消费它们，而无需根据 `stream_mode` 分块进行分支判断。

LangChain 实现了一套流式传输系统，用于呈现实时更新。

流式传输对于增强基于 LLM 构建的应用的响应性至关重要。通过逐步显示输出，即便在完整响应尚未就绪之前也能展示内容，流式传输显著改善了用户体验（UX），尤其是在面对 LLM 的延迟时。

## 概述

LangChain 的流式传输系统让你能够将 agent 运行的实时反馈呈现给应用。

使用 LangChain 流式传输可以实现：

* [**流式传输 agent 进度**](#agent-进度)——在每一步 agent 步骤后获取状态更新。
* [**流式传输 LLM token**](#llm-token)——在语言模型 token 生成时进行流式传输。
* [**流式传输思考 / 推理 token**](#流式传输思考--推理-token)——在模型推理生成时进行呈现。
* [**流式传输自定义更新**](#自定义更新)——发出用户自定义信号（例如 `"Fetched 10/100 records"`）。
* [**流式传输多种模式**](#流式传输多种模式)——从 `updates`（agent 进度）、`messages`（LLM token + 元数据）或 `custom`（任意用户数据）中进行选择。

有关更多端到端示例，请参阅下方的[常见模式](#常见模式)章节。

## 支持的流模式

将以下流模式中的一个或多个作为列表传给 [`stream`](https://reference.langchain.com/javascript/classes/_langchain_langgraph.index.CompiledStateGraph.html#stream) 方法：

| 模式       | 描述                                                                                                                        |
| ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| `updates`  | 在每一步 agent 步骤后流式传输状态更新。如果在同一步骤中产生了多个更新（例如运行了多个节点），这些更新会分别流式传输。        |
| `messages` | 从任何调用了 LLM 的图节点中流式传输 `(token, metadata)` 元组。                                                               |
| `custom`   | 使用流写入器（stream writer）从图节点内部流式传输自定义数据。                                                                 |

## Agent 进度

要流式传输 agent 进度，请使用 [`stream`](https://reference.langchain.com/javascript/classes/_langchain_langgraph.index.CompiledStateGraph.html#stream) 方法并设置 `streamMode: "updates"`。这会在每一步 agent 步骤后发出一个事件。

例如，如果你有一个只调用一次工具的 agent，你应该会看到以下更新：

* **LLM 节点**：带有工具调用请求的 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage)
* **工具节点**：带有执行结果的 [`ToolMessage`](https://reference.langchain.com/javascript/langchain-core/messages/ToolMessage)
* **LLM 节点**：最终的 AI 响应

通过 `configurable` 传入 `thread_id`，以便对话被检查点保存，后续轮次可以恢复相同的历史记录。`thread_id` 独立于 `streamMode`；你还可以在传入它的同时传入 `context`，用于工具从 `runtime.context` 读取的每次运行数据。

#### Google

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### OpenAI

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### Anthropic

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### OpenRouter

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### Fireworks

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### Baseten

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

#### Ollama

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import z from "zod";

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [getWeather],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "what is the weather in sf" }] },
  { ...config, version: "v3" },
);
await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const token of message.text) {
        process.stdout.write(token);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(`\nTool call: ${call.name}(${JSON.stringify(call.input)})`);
      console.log(`Tool result: ${await call.output}`);
    }
  })(),
]);

const finalState = await stream.output;
// Tool call: get_weather({"city":"San Francisco"})
// Tool result: [object ToolMessage]
// According to the data I have, the weather in San Francisco is always sunny! Would you like current conditions or a short forecast for today or the next few days?
```

- **[查看示例 trace](https://smith.langchain.com/public/20bac62c-c7af-4ee2-9dd3-129e24ce6834/r)**：为此示例打开一个公开的 LangSmith 运行记录。

> 【注意】使用 `thread_id` 持久化对话历史要求 agent 配置了[检查点保存器（checkpointer）](/oss/javascript/langchain/long-term-memory)。在 [LangSmith 部署](/langsmith/deployment)中会自动预置检查点保存器。在本地，需要显式传入一个，例如 `createAgent({ ..., checkpointer: new MemorySaver() })`。本页其余的代码片段为简洁起见省略了 `thread_id`，但在生产环境中你应该传入它。

## LLM token

要流式传输 LLM 生成的 token，请使用 `streamMode: "messages"`：

```typescript
import z from "zod";
import { createAgent, tool } from "langchain";

const getWeather = tool(
    async ({ city }) => {
        return `The weather in ${city} is always sunny!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string(),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const [token, metadata] of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: "messages" }
)) {
    console.log(`node: ${metadata.langgraph_node}`);
    console.log(`content: ${JSON.stringify(token.contentBlocks, null, 2)}`);
}
```

> 【注意】**要把 agent 包装为父级 `StateGraph` 中的一个节点？** [`createAgent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 返回的是 `ReactAgent` 包装器；在将其添加为节点时应传入 `agent.graph`。使用 `subgraphs: true` 以便消息分块包含子图命名空间。请参阅[子图输出](/oss/javascript/langgraph/streaming#subgraph-outputs)。

## 自定义更新

要在工具执行时流式传输来自工具的更新，你可以使用配置中的 `writer` 参数。

```typescript
import z from "zod";
import { tool, createAgent } from "langchain";
import { LangGraphRunnableConfig } from "@langchain/langgraph";

const getWeather = tool(
    async (input, config: LangGraphRunnableConfig) => {
        // Stream any arbitrary data
        config.writer?.(`Looking up data for city: ${input.city}`);
        // ... fetch city data
        config.writer?.(`Acquired data for city: ${input.city}`);
        return `It's always sunny in ${input.city}!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string().describe("The city to get weather for."),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const chunk of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: "custom" }
)) {
    console.log(chunk);
}
```

**文件：Output**
```shell
Looking up data for city: San Francisco
Acquired data for city: San Francisco
```

> 【注意】如果你在工具中添加了 `writer` 参数，那么在不提供 writer 函数的情况下，你将无法在 LangGraph 执行上下文之外调用该工具。

## 流式传输多种模式

你可以通过将 streamMode 作为数组传入来指定多种流式传输模式：`streamMode: ["updates", "messages", "custom"]`。

流式输出将是 `[mode, chunk]` 元组，其中 `mode` 是流模式的名称，`chunk` 是该模式流式传输的数据。

```typescript
import z from "zod";
import { tool, createAgent } from "langchain";
import { LangGraphRunnableConfig } from "@langchain/langgraph";

const getWeather = tool(
    async (input, config: LangGraphRunnableConfig) => {
        // Stream any arbitrary data
        config.writer?.(`Looking up data for city: ${input.city}`);
        // ... fetch city data
        config.writer?.(`Acquired data for city: ${input.city}`);
        return `It's always sunny in ${input.city}!`;
    },
    {
        name: "get_weather",
        description: "Get weather for a given city.",
        schema: z.object({
        city: z.string().describe("The city to get weather for."),
        }),
    }
);

const agent = createAgent({
    model: "gpt-5.4-mini",
    tools: [getWeather],
});

for await (const [streamMode, chunk] of await agent.stream(
    { messages: [{ role: "user", content: "what is the weather in sf" }] },
    { streamMode: ["updates", "messages", "custom"] }
)) {
    console.log(`${streamMode}: ${JSON.stringify(chunk, null, 2)}`);
}
```

## 常见模式

以下示例展示了流式传输的常见用例。

### 流式传输思考 / 推理 token

有些模型在给出最终答案之前会进行内部推理。你可以通过过滤[标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)中 `type` 为 `"reasoning"` 的内容，来在思考 / 推理 token 生成时进行流式传输。

> 【注意】必须在模型上启用推理输出。
>
> 有关配置细节，请参阅[推理章节](/oss/javascript/langchain/models#reasoning)以及你[提供商的集成页面](/oss/javascript/integrations/providers/overview)。
>
> 要快速查看某个模型是否支持推理，请参阅 [models.dev](https://models.dev)。

要从 agent 流式传输思考 token，请使用 `streamMode: "messages"` 并过滤推理内容块。在模型支持时，使用启用了扩展思考（extended thinking）的模型实例（例如 `ChatAnthropic`）：

```ts
import z from "zod";
import { createAgent, tool } from "langchain";
import { ChatAnthropic } from "@langchain/anthropic";

const getWeather = tool(
  async ({ city }) => {
    return `It's always sunny in ${city}!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({ city: z.string() }),
  },
);

const agent = createAgent({
  model: new ChatAnthropic({
    model: "claude-sonnet-4-6",
    thinking: { type: "enabled", budget_tokens: 5000 },
  }),
  tools: [getWeather],
});

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "What is the weather in SF?" }] },
  { version: "v3" },
);
for await (const message of stream.messages) {
  for await (const token of message.reasoning) {
    process.stdout.write(`[thinking] ${token}`);
  }
  for await (const token of message.text) {
    process.stdout.write(token);
  }
}
```

- **[查看示例 trace](https://smith.langchain.com/public/a8cf65f3-e85e-448c-844e-67019558ddbc/r)**：为此示例打开一个公开的 LangSmith 运行记录。

**文件：Output**
```shell
[thinking] The user is asking about the weather in San Francisco. I have a tool
[thinking]  available to get this information. Let me call the get_weather tool
[thinking]  with "San Francisco" as the city parameter.
The weather in San Francisco is: It's always sunny in San Francisco!
```

无论模型提供商是谁，这种方式都以相同的方式工作——LangChain 会通过 [`content_blocks`](/oss/javascript/langchain/messages#standard-content-blocks) 属性，将提供商特定的格式（Anthropic 的 `thinking` 块、OpenAI 的 `reasoning` 摘要等）规范化为标准的 `"reasoning"` 内容块类型。

要直接从聊天模型（不使用 agent）流式传输推理 token，请参阅[使用聊天模型进行流式传输](/oss/javascript/langchain/models#reasoning)。

## 禁用流式传输

在某些应用中，你可能需要为给定模型禁用单个 token 的流式传输。这在以下情况中很有用：

* 在使用[多 agent](/oss/javascript/langchain/multi-agent) 系统时控制哪些 agent 流式传输其输出
* 将支持流式传输的模型与不支持流式传输的模型混合使用
* 部署到 [LangSmith](/langsmith/observability) 并希望阻止某些模型输出被流式传输到客户端

在初始化模型时设置 `streaming: false`。

```typescript
import { ChatOpenAI } from "@langchain/openai";

const model = new ChatOpenAI({
  model: "gpt-5.5",
  streaming: false,
});
```

> 【提示】在部署到 LangSmith 时，对任何你不希望其输出被流式传输到客户端的模型设置 `streaming=False`。这会在部署前于你的图代码中配置。

> 【注意】并非所有聊天模型集成都支持 `streaming` 参数。如果你的模型不支持，请改用 `disableStreaming: true`。该参数通过基类在所有聊天模型上都可用。

有关更多细节，请参阅 [LangGraph 流式传输指南](/oss/javascript/langgraph/streaming#disable-streaming-for-specific-chat-models)。

## 相关内容

* [前端流式传输](/oss/javascript/langchain/frontend/overview)——使用 [`useStream`](https://reference.langchain.com/javascript/langchain-react/index/useStream) 构建 React UI，以实现实时 agent 交互
* [使用聊天模型进行流式传输](/oss/javascript/langchain/models#stream)——不使用 agent 或图，直接从聊天模型流式传输 token
* [使用聊天模型进行推理](/oss/javascript/langchain/models#reasoning)——配置并访问聊天模型的推理输出
* [标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)——了解用于推理、文本及其他内容类型的规范化内容块格式
* [带 human-in-the-loop 的流式传输](/oss/javascript/langchain/human-in-the-loop#streaming-with-human-in-the-loop)——在处理中断以进行人工审核的同时流式传输 agent 进度
* [LangGraph 流式传输](/oss/javascript/langgraph/streaming)——高级流式传输选项，包括 `values`、`debug` 模式以及子图流式传输

***
