# 事件流（Event streaming）

> 原文：Event streaming
> 来源：https://docs.langchain.com/oss/javascript/deepagents/event-streaming

---

> 从 Deep Agents 流式传输 subagents、消息、工具调用和最终输出。

本页涵盖 Deep Agents 特有的流式传输关注点——最重要的是通过 `stream.subagents` 从被委派的 subagents 进行流式传输。关于通用的 agent 流式传输（`stream.messages`、`stream.values`、工具调用、自定义更新），请参阅 [LangChain Event Streaming](/oss/javascript/langchain/event-streaming)。

## 流式传输 subagents

Deep Agents 在 LangGraph 流式传输之上增加了一个 subagent 投影。当你希望每次被委派的 `task` 调用都有一个流句柄时，使用 `stream.subagents`。该投影很轻量：它首先发现 subagent 任务，而消息、工具调用和值流仅在你于某个 subagent 句柄上访问它们时才打开。

每个句柄的 `name` 就是该子代理配置的名称：即协调者在调用 `task` 工具时传入的 `subagent_type`。Deep Agents 将这个名字绑定到被委派的运行上，因此你在 subagent 规格中定义的同一标签，也正是你在流中用于过滤和路由的标签。

```ts
const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "Write me a haiku about the sea" }] },
  { version: "v3" },
);

const subagentNames: string[] = [];
for await (const subagent of stream.subagents) {
  console.log(subagent.name);
  console.log(await subagent.taskInput);

  for await (const message of subagent.messages) {
    console.log(await message.text);
  }

  subagentNames.push(subagent.name);
}

await stream.output;
```

## Subagent stream 字段

每个 subagent stream 暴露的投影种类与父运行相同，例如消息、工具调用、嵌套 subagents 和最终输出。关于通用的父运行流式传输模型，请参阅 [LangChain Event Streaming](/oss/javascript/langchain/event-streaming)。

TypeScript 使用 camelCase 的投影名称，例如 `toolCalls` 和 `taskInput`。每个 subagent stream 都可以暴露 `.messages`、`.toolCalls`、`.values`、`.subagents` 和 `.output`。

| 字段        | 描述                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------ |
| `name`      | 子代理名称，取自协调者在其 `task` 调用中选择的 `subagent_type`。                            |
| `messages`  | 由该 subagent 发出的消息。                                                                  |
| `subagents` | 嵌套的 subagent 调用。                                                                      |
| `output`    | 最终 subagent 状态，或被委派任务的完成信号。                                                |
| `taskInput` | 传递给 task 工具的 prompt 的 Promise。                                                      |
| `toolCalls` | 作用于该 subagent 的工具调用。                                                              |

## 跟踪 subagent 生命周期

当你只需要显示哪些 subagents 已启动和完成时，使用 `stream.subagents`。除非你在单个 subagent 上访问那些投影，否则无需订阅消息流或值流。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

let running = 0;
let completed = 0;
let failed = 0;
const watchers: Promise<void>[] = [];

for await (const subagent of stream.subagents) {
  running += 1;
  console.log(`${subagent.name}: started`);

  watchers.push(
    subagent.output.then(
      () => {
        running -= 1;
        completed += 1;
        console.log(`${subagent.name}: completed`);
      },
      () => {
        running -= 1;
        failed += 1;
        console.log(`${subagent.name}: failed`);
      },
    ),
  );
}

await Promise.all(watchers);
console.log({ running, completed, failed });
```

> **View example trace**（[链接](https://smith.langchain.com/public/1d3f1455-4a9c-4607-aa1d-115948f2e1fa/r)）：Open a public LangSmith run for this example.

## 流式传输消息

Deep Agents 可以发出协调者 agent 和已委派 subagents 的消息。对顶层消息使用 `stream.messages`，对每个已委派的 subagent 使用 `subagent.messages`。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

const coordinatorMessages: string[] = [];
for await (const message of stream.messages) {
  const text = await message.text;
  console.log("[coordinator]", text);
  coordinatorMessages.push(text);
}

for await (const subagent of stream.subagents) {
  for await (const message of subagent.messages) {
    console.log(`[${subagent.name}]`, await message.text);
  }
}

await stream.output;
```

## 流式传输工具调用

Deep Agents 在 agent 树的每一层都暴露工具调用。对协调者工具使用顶层 `stream.tool_calls`，对已委派的工作使用每个 `subagent.tool_calls`。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

const coordinatorToolNames: string[] = [];
for await (const call of stream.toolCalls) {
  console.log("[coordinator tool]", call.name, call.input);
  console.log(await call.status);
  coordinatorToolNames.push(call.name);
}

for await (const subagent of stream.subagents) {
  for await (const call of subagent.toolCalls) {
    console.log(`[${subagent.name} tool]`, call.name, call.input);

    const status = await call.status;
    if (status === "finished") {
      console.log(await call.output);
    } else if (status === "error") {
      console.error(await call.error);
    }
  }
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/3fdda16a-c42c-4931-81f5-36e4a4ecfc5c/r)）：Open a public LangSmith run for this example.

## 流式传输嵌套工作

你可以递归进入某个 subagent stream，以观察嵌套的 subagents、消息和工具调用。

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

const subagentNames: string[] = [];
for await (const subagent of stream.subagents) {
  console.log(`subagent ${subagent.name}: started`);

  for await (const toolCall of subagent.toolCalls) {
    console.log(`${toolCall.name}(${JSON.stringify(toolCall.input)})`);

    const status = await toolCall.status;
    if (status === "finished") {
      console.log(await toolCall.output);
    } else if (status === "error") {
      console.error(await toolCall.error);
    }
  }

  for await (const nested of subagent.subagents) {
    console.log(`nested subagent ${nested.name}: started`);
  }

  subagentNames.push(subagent.name);
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/0b947581-de0c-491e-96c1-ba37cc862b0e/r)）：Open a public LangSmith run for this example.

## 并发消费

协调者和 subagent 的输出经常交错。当你需要实时 UI 更新时，请并发消费各投影。

在 JavaScript 中使用并发消费者：

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      console.log("[coordinator]", await message.text);
    }
  })(),
  (async () => {
    for await (const subagent of stream.subagents) {
      void (async () => {
        for await (const message of subagent.messages) {
          console.log(`[${subagent.name}]`, await message.text);
        }
      })();
    }
  })(),
]);
```

> **View example trace**（[链接](https://smith.langchain.com/public/467e4c96-80a0-42ba-9f5f-d869b74e5899/r)）：Open a public LangSmith run for this example.

当你需要跨协调者和所有 subagents 的精确到达顺序时，请迭代原始协议事件，并使用 `namespace` 来标识来源：

```ts
const stream = await agent.streamEvents(input, { version: "v3" });

const textDeltas: string[] = [];
for await (const event of stream) {
  if (event.method !== "messages") continue;

  const data = event.params.data;
  if (data.event !== "content-block-delta") continue;

  const block = data.delta ?? {};
  if (block.type === "text-delta") {
    const isSubagent = event.params.namespace.some((seg) =>
      seg.startsWith("tools:"),
    );
    const source = isSubagent ? "subagent" : "coordinator";
    console.log(`[${source}] ${block.text}`);
    textDeltas.push(block.text);
  }
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/4113a4d2-b6d8-4f2a-ad1d-2ab9e29859e6/r)）：Open a public LangSmith run for this example.

## Subagents 与 subgraphs 对比

`stream.subgraphs` 展示的是图执行结构。`stream.subagents` 展示的是产品层面的 Deep Agents 任务委派。面向用户的 UI 请使用 `stream.subagents`，因为它隐藏了内部图节点，并直接暴露 subagent 概念。

## 相关

* [LangChain Event Streaming](/oss/javascript/langchain/event-streaming) 涵盖通用的 agent 消息和工具调用流式传输概念。
* [Subagent frontend streaming](/oss/javascript/deepagents/frontend/subagent-streaming) 展示了将协调者消息与 subagent 卡片分开的 UI 模式。
* [LangGraph Event Streaming](/oss/javascript/langgraph/event-streaming) 涵盖底层的图流式传输模型。
