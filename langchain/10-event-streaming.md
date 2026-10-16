# 事件流式传输

> 原文：https://docs.langchain.com/oss/javascript/langchain/event-streaming
> LangChain (TypeScript) 官方文档中文整理

LangChain agent 构建在 LangGraph 之上，因此它们支持相同的流式传输栈，并针对 agent 场景提供了面向消息、工具调用、状态和自定义更新的投影（projection）。

对于大多数应用和前端用例，请通过 `stream_events(..., version="v3")` 使用 **事件流式传输（Event Streaming）**。事件流式传输返回一个带有类型化投影的 run 对象，因此每个投影都可以被独立消费，而无需解析流模式元组。

```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  async ({ city }) => `It's always sunny in ${city}!`,
  {
    name: "get_weather",
    description: "Get weather for a city.",
    schema: z.object({ city: z.string() }),
  }
);

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getWeather],
});

const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "What is the weather in SF?" }] },
  { version: "v3" }
);

for await (const message of stream.messages) {
  for await (const delta of message.text) {
    process.stdout.write(delta);
  }
}

const finalState = await stream.output;
```

## 可以流式传输的内容

| 投影                  | 用途                                                                       |
| --------------------- | -------------------------------------------------------------------------- |
| `for event in stream` | 带有完整信封（envelope）并可访问每个通道的原始协议事件。                    |
| `stream.messages`     | 模型消息流，每次 LLM 调用对应一个。                                        |
| `message.text`        | 一条消息的文本增量（delta）和最终文本。                                    |
| `message.reasoning`   | 对暴露推理内容的模型而言，是其推理增量。                                    |
| `message.toolCalls`   | 工具调用参数分块以及最终确定的工具调用。                                    |
| `message.output`      | 模型调用完成后得到的最终消息对象。                                          |
| `message.usage`       | 当提供商返回时的 token 用量元数据。                                         |
| `stream.values`       | Agent 状态快照。                                                            |
| `stream.output`       | 最终 agent 状态。                                                           |
| `stream.subgraphs`    | 嵌套的图运行（子 agent 和普通子图）。                                       |
| `stream.extensions`   | 自定义转换器投影。                                                          |
| `stream.toolCalls`    | 工具执行生命周期、输入、输出增量、最终输出以及错误。                        |

`stream.messages` 会产出消息流。每个消息流都暴露 `.text`、`.reasoning`、`.toolCalls`、`.output` 和 `.usage`。异步投影既可以被迭代以获取实时增量，也可以被 await 以获取最终值。

## Agent 消息

当你想要获取每次 LLM 调用的模型输出时，使用 `stream.messages`。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  process.stdout.write(`[${message.node}] `);
  for await (const delta of message.text) {
    process.stdout.write(delta);
  }

  const fullMessage = await message.output;
  console.log(fullMessage.content);

  const usage = await message.usage;
  if (usage) {
    console.log(usage);
  }
}
```

`message.output` 会给出最终确定的 AI 消息，其中包含提供商特定的内容块。在 TypeScript 中，当你只需要 token 计数或其他用量元数据时，使用 `message.usage`；在 Python 中，则从 `message.output.usage_metadata` 读取用量信息。

## 推理内容

推理内容与文本内容具有相同的结构，但只有在所选模型会输出推理块时才可用。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const message of stream.messages) {
  for await (const delta of message.reasoning) {
    process.stdout.write(`[thinking] ${delta}`);
  }

  for await (const delta of message.text) {
    process.stdout.write(delta);
  }
}
```

有关模型配置的细节，请参阅[推理指南](/oss/javascript/langchain/models#reasoning)以及你所使用提供商的集成页面。

## 工具调用

有两个有用的工具调用投影：

* `message.tool_calls` 在模型生成工具调用时流式传输工具调用参数分块。
* `stream.tool_calls` 在工具调用开始后流式传输工具执行的生命周期。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      for await (const chunk of message.toolCalls) {
        console.log("tool call chunk", chunk);
      }
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(call.name, call.input);
      console.log(await call.output, await call.error);
    }
  })(),
]);
```

## 流式传输子 agent

当一次 `createAgent` 调用（通常是通过一个包装工具）调用了另一个具名的 `createAgent` 时，内部 agent 的事件会在嵌套的命名空间中流动。你传给 `createAgent` 的 `name` 会在流中标识该内部 agent，因此你可以按 agent 进行过滤和标注。

具名的子 agent 会出现在专用的 `stream.subagents` 投影上。每个句柄都暴露内部 agent 自己的 `.messages`、`.toolCalls` 和 `.output`，以及 `.name`（你传入的 `name=`）、`.cause`（派发该子 agent 的工具调用）和嵌套的 `.subagents`。由于只有具名的 `createAgent` 运行会出现在这里，因此你无需把普通子图过滤掉。

```ts
import { createAgent, tool } from "langchain";
import { z } from "zod";

const getWeather = tool(
  async ({ city }) => `It's always sunny in ${city}!`,
  { name: "get_weather", schema: z.object({ city: z.string() }) }
);

const weatherAgent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getWeather],
  name: "weather_agent",
});

const callWeather = tool(
  async ({ query }) => {
    const result = await weatherAgent.invoke({
      messages: [{ role: "user", content: query }],
    });
    return result.messages.at(-1)?.text ?? "";
  },
  { name: "call_weather", schema: z.object({ query: z.string() }) }
);

const supervisor = createAgent({
  model: "openai:gpt-5.5",
  tools: [callWeather],
  name: "supervisor",
});

const stream = await supervisor.streamEvents(
  { messages: [{ role: "user", content: "What's the weather in Boston?" }] },
  { version: "v3" }
);

for await (const subagent of stream.subagents) {
  process.stdout.write(`${subagent.name}: `);
  for await (const message of subagent.messages) {
    for await (const token of message.text) {
      process.stdout.write(token);
    }
  }
  process.stdout.write("\n");
}
//Output: "weather_agent: The weather in Boston is sunny!"
```

从工具中调用的普通 `StateGraph` 子图也会出现在 `stream.subgraphs` 上——在 `.compile(name=...)` 上设置 `name=`，即可在 `subagent.graph_name` 中获得标签。

`stream.subagents` 是具名 `createAgent` 子 agent 的聚焦视图，而 `stream.subgraphs` 则涵盖每一个嵌套图。根据你的 UI 需要选择合适的一个。

## 状态与最终输出

使用 `stream.values` 获取状态快照，使用 `stream.output` 获取最终 agent 状态。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

for await (const snapshot of stream.values) {
  console.log(snapshot);
}

const finalState = await stream.output;
```

## 多个投影

当你想在 JavaScript 中同时使用多个投影时，请使用并发消费者：

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      console.log(await message.text);
    }
  })(),
  (async () => {
    for await (const call of stream.toolCalls) {
      console.log(call.name, call.input);
    }
  })(),
]);
```

若要访问未以类型化投影形式暴露的通道，或检查完整的事件信封，请迭代原始协议事件：

```ts
for await (const event of stream) {
  console.log(event.method, event.params.namespace, event.params.data);
}
```

## 自定义更新

当你的应用需要某个并非内置的投影时（例如检索进度、工件或领域特定事件），请使用自定义流转换器。

```ts
const stream = await agent.streamEvents(input, {
  version: "v3",
  transformers: [toolActivityTransformer],
});

for await (const activity of stream.extensions.toolActivity) {
  console.log(activity);
}
```

### 在中间件上注册转换器

> 【提示】通过中间件注册的转换器需要 `langchain@1.4.3` 或更高版本。

中间件可以在其钩子和工具之外声明流转换器工厂。工厂的形状在不同语言之间有所不同：

将 `streamTransformers` 作为工厂元组传给 `createMiddleware`。每个工厂的形状为 `() => StreamTransformer<any>`（零参数），并且每个作用域调用一次。每次调用都返回一个全新的转换器可以保持每个子图相互隔离。

```ts
import { createAgent, createMiddleware } from "langchain";

const toolActivityMiddleware = createMiddleware({
  name: "ToolActivityMiddleware",
  streamTransformers: [toolActivityTransformer],
});

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getWeather],
  middleware: [toolActivityMiddleware],
});
```

在编译时，`createAgent` 会将通过中间件注册的工厂与传给其自身 `streamTransformers` 选项的任何内容合并。编译后图上的最终顺序为：

1. 内置的 `ToolCallTransformer`。
2. 通过中间件注册的工厂，按中间件顺序。
3. 由调用方通过 `createAgent` 传入的 `streamTransformers`。

这样可以确保内置的工具调用投影位于消费者转换器之前，并让调用方传入的条目拥有最终决定权。

有关转换器的契约，请参阅[构建你自己的投影](/oss/javascript/langgraph/event-streaming#build-your-own-projection)。

## 相关内容

* [Streaming](/oss/javascript/langchain/streaming) 涵盖底层 Pregel 流模式。
* [构建你自己的投影](/oss/javascript/langgraph/event-streaming#build-your-own-projection) 涵盖如何编写应用特定的投影。
* [前端流式传输模式](/oss/javascript/langchain/frontend/overview) 展示了基于流式状态构建的 UI 用例。

***
