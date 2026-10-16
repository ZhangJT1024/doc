# Agent User Interaction Protocol (AG-UI)

> 原文：Agent User Interaction Protocol (AG-UI)
> 来源：https://docs.langchain.com/oss/javascript/deepagents/ag-ui

---

> 通过 Agent User Interaction Protocol (AG-UI) 暴露 deep agents，以便向任何 AG-UI 客户端或前端流式传输事件。

[Agent User Interaction Protocol (AG-UI)](https://docs.ag-ui.com) 是一个开放、轻量、基于事件的协议，它标准化了 AI agents 与面向用户的应用程序之间的连接方式。通过 AG-UI 暴露一个 deep agent，会将其 run 转换为一个类型化的事件流（messages、tool calls、reasoning、state 和 lifecycle），任何 AG-UI 客户端都可以消费它，这样你就可以驱动前端，而无需将其与 LangGraph 内部实现耦合。

> **注意**：AG-UI 是为 agent 到用户交互而设计的：即 agentic backend 与面向用户的前端之间的连接。它与其他 Deep Agents 协议有所不同：
>
> * **AG-UI** 将一个 deep agent 连接到前端应用程序（本页）。
> * [Agent Client Protocol (ACP)](/oss/javascript/deepagents/acp) 将一个 deep agent 连接到代码编辑器和 IDE。
> * [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp) 让一个 deep agent 调用由外部服务器托管的工具。
> * [Agent2Agent (A2A)](/oss/javascript/deepagents/a2a) 将一个 deep agent 连接到其他 agent。

## 快速开始

将 deep agent 作为 LangGraph graph 来提供服务，然后连接 TypeScript `@ag-ui/langgraph` adapter，以便 AG-UI 客户端可以驱动它。

### 安装依赖

安装 Deep Agents、LangGraph CLI 和 AG-UI LangGraph adapter：

#### npm

```bash
npm install @ag-ui/langgraph deepagents @langchain/langgraph-cli
```

#### yarn

```bash
yarn add @ag-ui/langgraph deepagents @langchain/langgraph-cli
```

#### pnpm

```bash
pnpm add @ag-ui/langgraph deepagents @langchain/langgraph-cli
```

使用 `createDeepAgent`（Python 中为 `create_deep_agent`）创建的 deep agent 就是一个 LangGraph graph。将它作为 LangGraph server 来提供服务，从而将其暴露给 AG-UI 客户端，然后让 AG-UI adapter 指向该 server。

### 创建一个 deep agent

定义 agent 并导出该 graph，以便 LangGraph server 能够加载它。

```ts title="agent.ts"
import { createDeepAgent } from "deepagents";

export const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  systemPrompt: "You are an expert researcher.",
});
```

### 提供 agent 服务

在项目根目录的 `langgraph.json` 文件中注册该 graph。

```json title="langgraph.json"
{
  "dependencies": ["."],
  "graphs": {
    "deep_agent": "./agent.ts:agent"
  },
  "env": ".env"
}
```

启动 LangGraph 开发服务器。它会在 `http://localhost:2024` 上通过 HTTP 暴露该 graph。

```bash
npx @langchain/langgraph-cli dev
```

### 连接 AG-UI adapter

TypeScript `@ag-ui/langgraph` adapter 将正在运行的 graph 包装为一个任何客户端都可以驱动的 AG-UI agent。让它指向 LangGraph server 并指定要加载的 graph 名称。无论该 graph 是用 Python 还是 TypeScript 编写的，这都有效。

```ts title="ag-ui agent"
import { LangGraphAgent } from "@ag-ui/langgraph";

const agent = new LangGraphAgent({
  graphId: "deep_agent",
  deploymentUrl: "http://localhost:2024",
});
```

> **AG-UI LangGraph adapter on npm**（[链接](https://www.npmjs.com/package/@ag-ui/langgraph)）：`@ag-ui/langgraph` 包为 LangGraph graphs（包括 deep agents）实现了 AG-UI 协议。

## 流式事件

AG-UI 是一个事件流。当 deep agent 运行时，adapter 会将其 LangGraph 执行转换为类型化的 AG-UI events。客户端订阅这些 events 并在它们到达时进行更新，而不是等待最终答案。

一次 deep agent run 会映射到多种 AG-UI event 类型。其中主要的几种如下（以及其他）：

| Deep agent 活动                              | AG-UI 事件                                                               |
| -------------------------------------------- | ------------------------------------------------------------------------ |
| 运行生命周期                                 | `RUN_STARTED`, `RUN_FINISHED`, `RUN_ERROR`                               |
| Graph 节点进度                               | `STEP_STARTED`, `STEP_FINISHED`                                          |
| 助手文本                                     | `TEXT_MESSAGE_START`, `TEXT_MESSAGE_CONTENT`, `TEXT_MESSAGE_END`         |
| 工具调用                                     | `TOOL_CALL_START`, `TOOL_CALL_ARGS`, `TOOL_CALL_END`, `TOOL_CALL_RESULT` |
| 推理                                         | `REASONING_START`, `REASONING_MESSAGE_CONTENT`, `REASONING_END`          |
| 共享状态（todos、subagents、自定义 keys）     | `STATE_SNAPSHOT`, `STATE_DELTA`                                          |
| 对话历史                                     | `MESSAGES_SNAPSHOT`                                                      |

状态更新使用 `STATE_SNAPSHOT` 提供完整基线，使用 `STATE_DELTA`（JSON Patch，RFC 6902）提供增量变更，这样客户端就可以保持 todos、plans 和 subagent status 同步，而无需在每一步都重新发送整个 state。

要直接观察该 stream，请使用 subscriber 运行 agent。每个 event 都有一个匹配的 `on…Event` 回调：

```ts title="observe.ts"
import { LangGraphAgent } from "@ag-ui/langgraph";

const agent = new LangGraphAgent({
  graphId: "deep_agent",
  deploymentUrl: "http://localhost:2024",
  initialMessages: [
    { id: "1", role: "user", content: "Research the AG-UI protocol." },
  ],
});

await agent.runAgent(
  {},
  {
    onTextMessageContentEvent({ event }) {
      process.stdout.write(event.delta);
    },
    onToolCallStartEvent({ event }) {
      console.log(`\n[tool] ${event.toolCallName}`);
    },
    onStateSnapshotEvent({ event }) {
      console.log("\n[state]", event.snapshot);
    },
    onRunFinishedEvent() {
      console.log("\n[done]");
    },
  },
);
```

当 deep agent 为等待人工输入而暂停时，run 会以一个 interrupt 结束。默认情况下，`@ag-ui/langgraph` 会在 `RUN_FINISHED` 的同时发出一个 legacy 的 `on_interrupt` custom event。要接收 `RUN_FINISHED` 上结构化的 AG-UI interrupt outcome（`outcome.type === "interrupt"`），请在构造 agent 时设置 `emitInterruptOutcome: true`：

```ts
const agent = new LangGraphAgent({
  graphId: "deep_agent",
  deploymentUrl: "http://localhost:2024",
  emitInterruptOutcome: true,
});
```

在下一个 input 中发送标准的 AG-UI `resume` 字段来恢复该 run：

```ts
const input = {
  threadId: "t1",
  runId: "r2",
  messages: [],
  resume: [
    { interruptId: "int-abc", status: "resolved", payload: { approved: true } },
  ],
};
```

关于 interrupt 模型本身，请参阅 [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)。

> **说明**：完整的事件 schema 请参阅 [AG-UI events reference](https://docs.ag-ui.com/concepts/events)。

## 连接前端

任何 AG-UI 客户端都可以驱动通过该协议暴露的 deep agent，因此你无需自己构建 message rendering、streaming 或 state sync。让客户端指向该 adapter（或包装它的某个 runtime），并通过其 `graphId` 选择 agent。

> **CopilotKit**（[链接](/oss/javascript/langchain/frontend/integrations/copilotkit)）：支持 AG-UI 的 React chat runtime，适用于 LangGraph 和 Deep Agents，包括 Python FastAPI bridge。

> **AG-UI clients**（[链接](https://docs.ag-ui.com/integrations)）：AG-UI 客户端和 SDKs 的完整列表，包括 terminal 和 mobile 客户端。

> **Build a custom client**（[链接](https://docs.ag-ui.com/quickstart/clients)）：直接使用 AG-UI SDK 消费事件流来构建你自己的界面。

## 编程式 API

`LangGraphAgent` 连接到 LangGraph server 上的 deep agent，并将其暴露为一个 AG-UI agent。用要加载的 graph 构造它，然后用几个方法驱动它。

```ts
import { LangGraphAgent } from "@ag-ui/langgraph";

const agent = new LangGraphAgent({
  graphId: "deep_agent",              // matches a key in langgraph.json
  deploymentUrl: "http://localhost:2024",
});
```

* **`runAgent(parameters?, subscriber?)`**：运行 agent 并将 AG-UI events 流式传输到 subscriber 的 `on…Event` 回调（参见 [流式事件](#流式事件)）。当 run 结束时 resolve。
* **`subscribe(subscriber)`**：附加一个持久 subscriber，它会在每一次 run 中接收 events，而不只是针对某一次调用。
* **`abortRun()`**：取消正在进行的 run。

要给 input 设定初始内容，请向构造函数传入 `initialMessages`，或在运行前调用 `addMessage` 或 `setMessages`。

## 另请参阅

* [CopilotKit](/oss/javascript/langchain/frontend/integrations/copilotkit)：面向 Deep Agents over AG-UI 的 Python 和 TypeScript CopilotKit runtime 模式
* [Frontend overview](/oss/javascript/deepagents/frontend/overview)：使用 LangChain frontend SDKs 构建流式展示 deep agent 进度的 UI
* [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)：deep agents 的 interrupt 与 resume 模型
* [Agent Client Protocol (ACP)](/oss/javascript/deepagents/acp)：将 deep agents 连接到代码编辑器和 IDE
* [AG-UI documentation](https://docs.ag-ui.com)：协议概念、events 和 client SDKs
