# 流式传输（Streaming）

> 原文：Streaming
> 来源：https://docs.langchain.com/oss/javascript/deepagents/streaming

---

> 从 deep agent 运行和 subagent 执行中流式传输实时更新。

> **提示**：对于新应用程序，我们推荐 [event streaming](/oss/javascript/deepagents/event-streaming)——这是 Deep Agents v0.6 引入的类型化投影 API。Event streaming 为每个投影（subagents、messages、tool calls、values）提供独立的迭代器，这样你就可以独立消费它们，而不必根据 `stream_mode` 块进行分支判断。

Deep Agents 构建在 LangGraph 的流式传输基础设施之上，并为一等的 subagent 流提供支持。当 deep agent 将工作委派给 subagents 时，你可以独立地从每个 subagent 流式传输更新——实时跟踪进度、LLM tokens 和工具调用。

deep agent streaming 可以实现的功能：

* [**Stream subagent progress**](#subagent-progress)——在并行运行时跟踪每个 subagent 的执行。
* [**Stream LLM tokens**](#llm-tokens)——从主 agent 和每个 subagent 流式传输 tokens。
* [**Stream tool calls**](#tool-calls)——查看 subagent 执行内部的工具调用和结果。
* [**Stream custom updates**](#custom-updates)——从 subagent 节点内部发出用户自定义信号。

## 启用 subgraph streaming

Deep Agents 使用 LangGraph 的 subgraph streaming 来呈现 subagent 执行的事件。要接收 subagent 事件，请在流式传输时启用 `stream_subgraphs`。

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt: "You are a helpful research assistant",
  subagents: [
    {
      name: "researcher",
      description: "Researches a topic in depth",
      systemPrompt: "You are a thorough researcher.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research quantum computing advances" },
    ],
  },
  {
    streamMode: "updates",
    subgraphs: true, // [!code highlight]
  },
)) {
  if (namespace.length > 0) {
    // Subagent event - namespace identifies the source
    console.log(`[subagent: ${namespace.join("|")}]`);
  } else {
    // Main agent event
    console.log("[main agent]");
  }
  console.log(chunk);
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/c168526d-4120-4f09-8714-4a45a6446598/r)）：Open a public LangSmith run for this example.

## Namespaces

当启用 `subgraphs` 时，每个流式事件都包含一个 **namespace**，用于标识它由哪个 agent 产生。namespace 是表示 agent 层级的节点名称和任务 ID 的路径。

| Namespace                                  | 来源                                                             |
| ------------------------------------------ | ---------------------------------------------------------------- |
| `()`（空）                                 | 主 agent                                                         |
| `("tools:abc123",)`                        | 由主 agent 的 `task` 工具调用 `abc123` 生成的一个 subagent       |
| `("tools:abc123", "model_request:def456")` | subagent 内部的 model request 节点                               |

使用 namespaces 将事件路由到正确的 UI 组件：

```ts
for await (const [namespace, chunk] of await agent.stream(
  { messages: [{ role: "user", content: "Plan my vacation" }] },
  { streamMode: "updates", subgraphs: true },
)) {
  // Check if this event came from a subagent
  const isSubagent = namespace.some((segment: string) =>
    segment.startsWith("tools:"),
  );

  if (isSubagent) {
    // Extract the tool call ID from the namespace
    const toolCallId = namespace
      .find((s: string) => s.startsWith("tools:"))
      ?.split(":")[1];
    console.log(`Subagent ${toolCallId}:`, chunk);
  } else {
    console.log("Main agent:", chunk);
  }
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/29fabe2e-590d-45a1-bb2e-6d7491103625/r)）：Open a public LangSmith run for this example.

## Subagent 进度

使用 `stream_mode="updates"` 在每个步骤完成时跟踪 subagent 进度。这对于显示哪些 subagents 处于活动状态以及它们完成了什么工作很有用。

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt:
    "You are a project coordinator with no research knowledge. " +
    "For every user request, you must call the task() tool with " +
    "subagent_type set to researcher. Never answer research questions yourself. " +
    "Keep your final response to one sentence.",
  subagents: [
    {
      name: "researcher",
      description: "Researches topics thoroughly",
      systemPrompt:
        "You are a thorough researcher. Research the given topic " +
        "and provide a concise summary in 2-3 sentences.",
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Write a short summary about AI safety" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  // Main agent updates (empty namespace)
  if (namespace.length === 0) {
    for (const [nodeName, data] of Object.entries(chunk)) {
      if (nodeName === "tools") {
        // Subagent results returned to main agent
        for (const msg of (data as any).messages ?? []) {
          if (msg.type === "tool") {
            console.log(`\nSubagent complete: ${msg.name}`);
            console.log(`  Result: ${String(msg.content).slice(0, 200)}...`);
          }
        }
      } else {
        console.log(`[main agent] step: ${nodeName}`);
      }
    }
  }
  // Subagent updates (non-empty namespace)
  else {
    for (const [nodeName] of Object.entries(chunk)) {
      console.log(`  [${namespace[0]}] step: ${nodeName}`);
    }
  }
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/6892dbbe-d797-44da-974f-d78cd433299e/r)）：Open a public LangSmith run for this example.

#### Output

```shell
Main agent step: model_request
  [tools:call_abc123] step: model_request
  [tools:call_abc123] step: tools
  [tools:call_abc123] step: model_request
Subagent complete: task
Result: ## AI Safety Report...
Main agent step: model_request
  [tools:call_def456] step: model_request
  [tools:call_def456] step: model_request
Subagent complete: task
Result: # Comprehensive Report on AI Safety...
Main agent step: model_request
```

## LLM tokens

使用 `stream_mode="messages"` 从主 agent 和 subagents 流式传输单个 tokens。每个消息事件都包含标识来源 agent 的元数据。

```ts
let currentSource = "";

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Research quantum computing advances",
      },
    ],
  },
  { streamMode: "messages", subgraphs: true },
)) {
  const [message] = chunk;

  // Check if this event came from a subagent (namespace contains "tools:")
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));

  if (isSubagent) {
    // Token from a subagent
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    if (subagentNs !== currentSource) {
      process.stdout.write(`\n\n--- [subagent: ${subagentNs}] ---\n`);
      currentSource = subagentNs;
    }
    if (message.text) {
      process.stdout.write(message.text);
    }
  } else {
    // Token from the main agent
    if ("main" !== currentSource) {
      process.stdout.write(`\n\n--- [main agent] ---\n`);
      currentSource = "main";
    }
    if (message.text) {
      process.stdout.write(message.text);
    }
  }
}

process.stdout.write("\n");
```

> **View example trace**（[链接](https://smith.langchain.com/public/eae65ae1-ab35-412c-8769-465a7ee1ac0e/r)）：Open a public LangSmith run for this example.

## 工具调用

当 subagents 使用工具时，你可以流式传输工具调用事件，以显示每个 subagent 正在做什么。工具调用块出现在 `messages` stream 模式中。

```ts
import { AIMessageChunk, ToolMessage } from "langchain";

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Research recent quantum computing advances",
      },
    ],
  },
  { streamMode: "messages", subgraphs: true },
)) {
  const [message] = chunk;

  // Identify source: "main" or the subagent namespace segment
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  const source = isSubagent
    ? namespace.find((s: string) => s.startsWith("tools:"))!
    : "main";

  // Tool call chunks (streaming tool invocations)
  if (AIMessageChunk.isInstance(message) && message.tool_call_chunks?.length) {
    for (const tc of message.tool_call_chunks) {
      if (tc.name) {
        console.log(`\n[${source}] Tool call: ${tc.name}`);
      }
      // Args stream in chunks - write them incrementally
      if (tc.args) {
        process.stdout.write(tc.args);
      }
    }
  }

  // Tool results
  if (ToolMessage.isInstance(message)) {
    console.log(
      `\n[${source}] Tool result [${message.name}]: ${message.text?.slice(0, 150)}`,
    );
  }

  // Regular AI content (skip tool call messages)
  if (
    AIMessageChunk.isInstance(message) &&
    message.text &&
    !message.tool_call_chunks?.length
  ) {
    process.stdout.write(message.text);
  }
}

process.stdout.write("\n");
```

> **View example trace**（[链接](https://smith.langchain.com/public/b691dfbe-3eea-472b-92a3-7200ce7fd48d/r)）：Open a public LangSmith run for this example.

## 自定义更新

在你的 subagent tools 内部使用 `config.writer` 发出自定义进度事件：

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { tool, type ToolRuntime } from "langchain";
import { z } from "zod";

/**
 * A tool that emits custom progress events via config.writer.
 * The writer sends data to the "custom" stream mode.
 */
const analyzeData = tool(
  async ({ topic }: { topic: string }, config: ToolRuntime) => {
    const writer = config.writer;

    writer?.({ status: "starting", topic, progress: 0 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "analyzing", progress: 50 });
    await new Promise((r) => setTimeout(r, 500));

    writer?.({ status: "complete", progress: 100 });
    return `Analysis of "${topic}": Customer sentiment is 85% positive, driven by product quality and support response times.`;
  },
  {
    name: "analyze_data",
    description:
      "Run a data analysis on a given topic. " +
      "This tool performs the actual analysis and emits progress updates. " +
      "You MUST call this tool for any analysis request.",
    schema: z.object({
      topic: z.string().describe("The topic or subject to analyze"),
    }),
  },
);

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt:
    "You are a coordinator. For any analysis request, you MUST delegate " +
    "to the analyst subagent using the task tool. Never try to answer directly. " +
    "After receiving the result, summarize it in one sentence.",
  subagents: [
    {
      name: "analyst",
      description: "Performs data analysis with real-time progress tracking",
      systemPrompt:
        "You are a data analyst. You MUST call the analyze_data tool " +
        "for every analysis request. Do not use any other tools. " +
        "After the analysis completes, report the result.",
      tools: [analyzeData],
    },
  ],
});

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze customer satisfaction trends",
      },
    ],
  },
  { streamMode: "custom", subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  if (isSubagent) {
    const subagentNs = namespace.find((s: string) => s.startsWith("tools:"))!;
    console.log(`[${subagentNs}]`, chunk);
  } else {
    console.log("[main]", chunk);
  }
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/8b328f38-ab1e-42f3-be42-4364118ec3f1/r)）：Open a public LangSmith run for this example.

#### Output

```shell
[tools:call_abc123] { status: 'fetching', progress: 0 }
[tools:call_abc123] { status: 'analyzing', progress: 50 }
[tools:call_abc123] { status: 'complete', progress: 100 }
```

## 流式传输多种模式

组合多种 stream 模式，以获得 agent 执行的完整画面：

```ts
// Skip internal middleware steps - only show meaningful node names
const INTERESTING_NODES = new Set(["model", "tools"]);

let lastSource = "";
let midLine = false; // true when we've written tokens without a trailing newline

for await (const [namespace, mode, data] of await agent.stream(
  {
    messages: [
      {
        role: "user",
        content: "Analyze the impact of remote work on team productivity",
      },
    ],
  },
  { streamMode: ["updates", "messages", "custom"], subgraphs: true },
)) {
  const isSubagent = namespace.some((s: string) => s.startsWith("tools:"));
  const source = isSubagent ? "subagent" : "main";

  if (mode === "updates") {
    for (const nodeName of Object.keys(data)) {
      if (!INTERESTING_NODES.has(nodeName)) continue;
      if (midLine) {
        process.stdout.write("\n");
        midLine = false;
      }
      console.log(`[${source}] step: ${nodeName}`);
    }
  } else if (mode === "messages") {
    const [message] = data;
    if (message.text) {
      // Print a header when the source changes
      if (source !== lastSource) {
        if (midLine) {
          process.stdout.write("\n");
          midLine = false;
        }
        process.stdout.write(`\n[${source}] `);
        lastSource = source;
      }
      process.stdout.write(message.text);
      midLine = true;
    }
  } else if (mode === "custom") {
    if (midLine) {
      process.stdout.write("\n");
      midLine = false;
    }
    console.log(`[${source}] custom event:`, data);
  }
}

process.stdout.write("\n");
```

> **View example trace**（[链接](https://smith.langchain.com/public/abec3125-178c-474a-b0c4-9e4cb553eb63/r)）：Open a public LangSmith run for this example.

## 常见模式

### 跟踪 subagent 生命周期

监控 subagents 何时启动、运行和完成：

```ts
function getToolCalls(message: unknown): Array<{
  id?: string;
  name?: string;
  args?: Record<string, unknown>;
}> {
  if (!message || typeof message !== "object") {
    return [];
  }
  const record = message as Record<string, unknown>;
  const toolCalls = record.tool_calls ?? record.toolCalls;
  return Array.isArray(toolCalls)
    ? (toolCalls as Array<{
        id?: string;
        name?: string;
        args?: Record<string, unknown>;
      }>)
    : [];
}

const activeSubagents = new Map<
  string,
  { type?: string; description?: string; status: string }
>();

for await (const [namespace, chunk] of await agent.stream(
  {
    messages: [
      { role: "user", content: "Research the latest AI safety developments" },
    ],
  },
  { streamMode: "updates", subgraphs: true },
)) {
  for (const [nodeName, data] of Object.entries(chunk)) {
    // ─── Phase 1: Detect subagent starting ────────────────────────
    // When the main agent emits a task tool call, a subagent has been spawned.
    if (namespace.length === 0) {
      for (const msg of (data as { messages?: unknown[] }).messages ?? []) {
        for (const tc of getToolCalls(msg)) {
          if (tc.name === "task" && tc.id) {
            activeSubagents.set(tc.id, {
              type: tc.args?.subagent_type as string | undefined,
              description: String(tc.args?.description ?? "").slice(0, 80),
              status: "pending",
            });
            console.log(
              `[lifecycle] PENDING  → subagent "${tc.args?.subagent_type}" (${tc.id})`,
            );
          }
        }
      }
    }

    // ─── Phase 2: Detect subagent running ─────────────────────────
    // When we receive events from a tools:UUID namespace, that
    // subagent is actively executing.
    if (namespace.length > 0 && namespace[0].startsWith("tools:")) {
      const pregelId = namespace[0].split(":")[1];
      // Check if any pending subagent needs to be marked running.
      // Note: the pregel task ID differs from the tool_call_id,
      // so we mark any pending subagent as running on first subagent event.
      let markedRunning = false;
      for (const [, sub] of activeSubagents) {
        if (sub.status === "pending") {
          sub.status = "running";
          markedRunning = true;
          console.log(
            `[lifecycle] RUNNING  → subagent "${sub.type}" (pregel: ${pregelId})`,
          );
          break;
        }
      }
      if (!markedRunning && activeSubagents.size === 0) {
        activeSubagents.set(pregelId, {
          type: "researcher",
          status: "running",
        });
        console.log(
          `[lifecycle] RUNNING  → subagent "researcher" (pregel: ${pregelId})`,
        );
      }
    }

    // ─── Phase 3: Detect subagent completing ──────────────────────
    // When the main agent's tools node returns a tool message,
    // the subagent has completed and returned its result.
    if (namespace.length === 0 && nodeName === "tools") {
      for (const msg of (data as { messages?: Array<Record<string, unknown>> })
        .messages ?? []) {
        if (msg.type === "tool") {
          const toolCallId = String(msg.tool_call_id ?? msg.toolCallId ?? "");
          const subagent = activeSubagents.get(toolCallId);
          if (subagent) {
            subagent.status = "complete";
            console.log(
              `[lifecycle] COMPLETE → subagent "${subagent.type}" (${toolCallId})`,
            );
            console.log(
              `  Result preview: ${String(msg.content).slice(0, 120)}...`,
            );
          }
        }
      }
    }
  }
}

// Print final state
console.log("\n--- Final subagent states ---");
for (const [id, sub] of activeSubagents) {
  console.log(`  ${sub.type}: ${sub.status}`);
}
```

> **View example trace**（[链接](https://smith.langchain.com/public/7c7546e0-f99a-4655-bf0c-550f94ebdff3/r)）：Open a public LangSmith run for this example.

## 相关

* [Subagents](/oss/javascript/deepagents/subagents)——使用 Deep Agents 配置和使用 subagents
* [Frontend streaming](/oss/javascript/deepagents/frontend/overview)——使用 [`useStream`](https://reference.langchain.com/javascript/langchain-react/index/useStream) 为 Deep Agents 构建 React UIs
* [LangChain Event Streaming](/oss/javascript/langchain/event-streaming)——LangChain agents 的通用流式传输概念
