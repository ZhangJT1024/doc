# Agents（智能体）

> 原文：https://docs.langchain.com/oss/javascript/langchain/agents
> LangChain (TypeScript) 官方文档中文整理

智能体（Agent）是一个在循环中调用工具的模型，直到完成给定任务。

![核心智能体循环示意图](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/core_agent_loop.svg?fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=4b4cbb497b6273758a565de1bc90ece0)

Harness（框架/载体）是围绕该循环的一切：提示词、工具，以及任何塑造模型行为的中间件。

> 【提示】**智能体 = 模型 + Harness**
>
> Harness 的职责：在合适的时机为模型提供正确的上下文，以完成给定任务。

[`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 是一个高度可配置的 harness。最简单的用法如下：

#### Google

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "google-genai:gemini-3.6-flash", tools });
```

#### OpenAI

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "openai:gpt-5.5", tools });
```

#### Anthropic

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "anthropic:claude-sonnet-4-6", tools });
```

#### OpenRouter

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "openrouter:openrouter:z-ai/glm-5.2", tools });
```

#### Fireworks

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "fireworks:accounts/fireworks/models/glm-5p2", tools });
```

#### Baseten

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "baseten:zai-org/GLM-5.2", tools });
```

#### Ollama

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "ollama:north-mini-code-1.0", tools });
```

在此基础上，你可以直接通过 `model=`、`tools=` 和 `system_prompt=` 参数配置基础项。对于更高级的能力，可以用[中间件](#configure-the-harness)扩展 harness。

> 【提示】[Deep Agents](/oss/javascript/deepagents/overview) 构建在 `create_agent` 之上，已经预先组装了常用能力，例如规划、文件系统工具、子智能体和记忆。当你需要自行配置 harness 时，请使用 `create_agent`。

## 核心组件

![智能体模型与 Harness 组件示意图](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/agent_model_harness.svg?fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=5ac6a7e0343af7cb5ba3ca632e2224af)

### 模型

传入一个模型标识符字符串（`"provider:model"`）或一个已初始化的模型实例，来为你的智能体选择模型。有关参数、provider 设置和动态模型选择，请参阅[模型](/oss/javascript/langchain/models)。

#### Google

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "google-genai:gemini-3.6-flash", tools });
```

#### OpenAI

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "openai:gpt-5.5", tools });
```

#### Anthropic

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "anthropic:claude-sonnet-4-6", tools });
```

#### OpenRouter

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "openrouter:openrouter:z-ai/glm-5.2", tools });
```

#### Fireworks

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "fireworks:accounts/fireworks/models/glm-5p2", tools });
```

#### Baseten

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "baseten:zai-org/GLM-5.2", tools });
```

#### Ollama

```ts
import { createAgent } from "langchain";

var agent = createAgent({ model: "ollama:north-mini-code-1.0", tools });
```

### 工具

要为智能体提供工具，可以传入任何可调用对象（callable）、LangChain 工具或工具字典。有关工具定义、上下文访问和动态工具选择，请参阅[工具](/oss/javascript/langchain/tools)。

#### Google

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "google-genai:gemini-3.6-flash", tools: [search] });
```

#### OpenAI

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "openai:gpt-5.5", tools: [search] });
```

#### Anthropic

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "anthropic:claude-sonnet-4-6", tools: [search] });
```

#### OpenRouter

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "openrouter:openrouter:z-ai/glm-5.2", tools: [search] });
```

#### Fireworks

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "fireworks:accounts/fireworks/models/glm-5p2", tools: [search] });
```

#### Baseten

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "baseten:zai-org/GLM-5.2", tools: [search] });
```

#### Ollama

```ts
import { tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Results for: ${query}`, {
  name: "search",
  description: "Search for information",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({ model: "ollama:north-mini-code-1.0", tools: [search] });
```

### 系统提示词

塑造智能体处理任务的方式。系统提示词参数接受字符串或 `SystemMessage`。若要在运行时使用动态提示词，请使用[中间件](/oss/javascript/langchain/middleware)。

#### Google

```ts
var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### OpenAI

```ts
var agent = createAgent({
  model: "openai:gpt-5.5",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### Anthropic

```ts
var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### OpenRouter

```ts
var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### Fireworks

```ts
var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### Baseten

```ts
var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

#### Ollama

```ts
var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools,
  systemPrompt: "You are a helpful assistant. Be concise and accurate.",
});
```

### 结构化输出

使用 `response_format=` 从智能体返回经过校验的 schema。有关策略和示例，请参阅[结构化输出](/oss/javascript/langchain/structured-output)。

#### Google

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### OpenAI

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### Anthropic

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### OpenRouter

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### Fireworks

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### Baseten

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

#### Ollama

```ts
const Answer = z.object({ summary: z.string(), confidence: z.number() });

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools,
  responseFormat: Answer,
});
const result = await agent.invoke({
  messages: [{ role: "user", content: "Summarize AI trends" }],
});
result.structuredResponse; // { summary: ..., confidence: ... }
```

### 智能体状态

每个智能体都通过一个 `AgentState` 对象管理其执行上下文，该对象保存当前对话历史以及你的工具和中间件所需的任何自定义字段。

内置字段为：

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| `messages` | `BaseMessage[]` | 当前线程的完整对话历史。仅追加：新消息会被添加，而不会被替换。 |

`AgentState` 也是传递给每个节点式中间件钩子（`beforeModel`、`afterModel` 等）的类型。钩子接收当前状态，并可以返回一个更新对象，用于合并回该状态。

要添加自定义字段，请在中间件上使用 `stateSchema`（配合 `StateSchema` 或 Zod 对象）定义一个状态 schema：

#### Google

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### OpenAI

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### Anthropic

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### OpenRouter

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### Fireworks

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### Baseten

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  middleware: [stateMiddleware],
});
```

#### Ollama

```ts
import { createAgent, createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const MyState = new StateSchema({
  userId: z.string(),
  callCount: z.number().default(0),
});

const stateMiddleware = createMiddleware({
  name: "StateExtension",
  stateSchema: MyState,
});

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  middleware: [stateMiddleware],
});
```

有关完整细节、示例以及中间件级别的状态 schema，请参阅[短期记忆](/oss/javascript/langchain/short-term-memory#customizing-agent-memory)和[自定义中间件](/oss/javascript/langchain/middleware/custom#state-updates)。

## 调用

> 【提示】使用 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-langchain-agents) 追踪该循环的每一步、调试工具调用并评估智能体输出。按照[追踪快速入门](/langsmith/trace-with-langchain)进行设置。我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的 trace、检测问题并提出修复建议。

你可以用一条消息来调用智能体。在底层，这会向智能体的 [`State`](/oss/javascript/langgraph/graph-api#state) 传递一个更新。所有智能体的状态中都包含一个[消息序列](/oss/javascript/langgraph/use-graph-api#messagesvalue)；要调用智能体，请传入一条新消息以及一个 `thread_id`，这样智能体就能持久化并恢复对话历史：

#### Google

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### OpenAI

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### Anthropic

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### OpenRouter

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### Fireworks

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### Baseten

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

#### Ollama

```ts
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: crypto.randomUUID() } };

let result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  config,
);

// A follow-up turn on the same conversation: reuse the same thread_id to keep history
result = await agent.invoke(
  { messages: [{ role: "user", content: "What about tomorrow?" }] },
  config,
);
```

[查看示例 trace](https://smith.langchain.com/public/5e5926b4-3156-477c-83e5-969404aeb92e/r) —— 打开此示例的公开 LangSmith 运行记录。

> 【提示】使用 `thread_id` 持久化对话历史，需要为智能体配置[检查点保存器（checkpointer）](/oss/javascript/langchain/long-term-memory)。部署到 [LangSmith](/langsmith/deployment) 时，会自动提供 checkpointer。在本地，需要显式传入一个，例如 `create_agent(..., checkpointer=InMemorySaver())`。

如果你还需要向工具和中间件传递每次运行（per-run）的配置（例如用户 ID、API 密钥或功能开关），请将它作为 `context` 与 config 一起传入。使用 `contextSchema` 定义该数据的结构，并通过 `runtime.context` 访问它：

#### Google

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### OpenAI

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### Anthropic

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### OpenRouter

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### Fireworks

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### Baseten

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

#### Ollama

```ts
import * as z from "zod";
import { AIMessage } from "@langchain/core/messages";
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const contextSchema = z.object({
  user_id: z.string(),
});

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  contextSchema,
  checkpointer: new MemorySaver(),
});

const result = await agent.invoke(
  {
    messages: [
      { role: "user", content: "What's the weather in San Francisco?" },
    ],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_id: "user-123" },
  },
);
```

[查看示例 trace](https://smith.langchain.com/public/8b453131-3d7f-4218-a407-f724b96fb4e7/r) —— 打开此示例的公开 LangSmith 运行记录。

`thread_id` 用于界定*对话*的范围（消息历史、检查点），而 `context` 承载你的工具和中间件在调用时读取的*每次运行*数据。两者通常一起传入。更多内容请参阅[工具上下文](/oss/javascript/langchain/tools#context)和 [Runtime](/oss/javascript/langchain/runtime)。

## 流式输出

`invoke` 在运行结束时返回最终响应。如果智能体执行多次工具调用，用户通常需要在完成之前获得进度更新。使用流式输出，可以在中间消息和工具活动发生时即时呈现它们。

```ts
const stream = await agent.streamEvents(
  {
    messages: [
      {
        role: "user",
        content: "Search for AI news and summarize the findings",
      },
    ],
  },
  { version: "v3" },
);

for await (const snapshot of stream.values) {
  // Each snapshot contains the full state at that point
  const latestMessage = snapshot.messages.at(-1);
  if (latestMessage?.content) {
    if (latestMessage.type === "human") {
      console.log(`User: ${latestMessage.content}`);
    } else if (latestMessage.type === "ai") {
      console.log(`Agent: ${latestMessage.content}`);
    }
  } else if (latestMessage?.tool_calls?.length) {
    const toolCallNames = latestMessage.tool_calls.map((tc) => tc.name);
    console.log(`Calling tools: ${toolCallNames.join(", ")}`);
  }
}
```

[查看示例 trace](https://smith.langchain.com/public/311f121f-4400-4709-a202-2f11e972669d/r) —— 打开此示例的公开 LangSmith 运行记录。

> 【提示】有关流式模式、事件类型和 UI 模式，请参阅[流式输出](/oss/javascript/langchain/streaming)。

## 配置 Harness

`create_agent` 具有高度可扩展性。中间件是自定义的基元（primitive）：每一块中间件负责处理一个关注点，在合适的时机钩入智能体循环，并可与任何其他中间件自由组合。只取你的用例所需的，其余略过。

常见模式已作为一等中间件预置。其他任何需求都可以构建为[自定义中间件](/oss/javascript/langchain/middleware/custom)。

![按类别划分的智能体 Harness 能力](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/agent_harness_capabilities.svg?fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=0ff671d72badd0844826660dfcb04391)

随着智能体承担复杂工作，它们需要在几个关键领域获得支持。中间件生态提供：

- **执行环境**（[#execution-environment](#execution-environment)）：工具、文件系统、沙箱和代码执行
- **上下文管理**（[#context-management](#context-management)）：摘要、记忆、技能和提示词缓存
- **规划与委派**（[#planning-and-delegation](#planning-and-delegation)）：待办清单和子智能体，用于并行、隔离的工作
- **容错**（[#fault-tolerance](#fault-tolerance)）：重试、回退和调用限制
- **护栏**（[#guardrails](#guardrails)）：PII 检测和内容控制
- **人工引导**（[#steering](#steering)）：在高影响操作前进行 human-in-the-loop 审批

> 【提示】`create_deep_agent` 为长时间运行的编码和研究任务预先组装了这套技术栈（默认包含文件系统、摘要、子智能体和提示词缓存）。完整预置 harness 请参阅 [Deep Agents](/oss/javascript/deepagents/harness)。

### 执行环境

当智能体能够采取行动而不只是生成文本时，它们尤其有用。执行环境为智能体提供了一个工作空间：可调用的工具、可跨轮次读写文件的文件系统，以及用于运行脚本或 shell 命令的代码执行能力。

#### Google

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### OpenAI

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### Anthropic

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### OpenRouter

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### Fireworks

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### Baseten

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

#### Ollama

```ts
import { createAgent } from "langchain";
import { createFilesystemMiddleware, StateBackend } from "deepagents";

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search],
  middleware: [createFilesystemMiddleware({ backend: new StateBackend() })],
});
```

参见 [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware)、[Sandboxes](/oss/javascript/deepagents/sandboxes)、[Interpreters](/oss/javascript/deepagents/interpreters)。

> 【提示】此示例从 `deepagents` 包导入。请按以下方式安装：
>
> **npm**
>
> ```bash
> npm install deepagents
> ```
>
> **yarn**
>
> ```bash
> yarn add deepagents
> ```
>
> **pnpm**
>
> ```bash
> pnpm add deepagents
> ```

### 上下文管理

每次模型调用都有一个固定的上下文窗口。随着智能体运行，该窗口会被不断累积的历史、工具结果和中间步骤填满。摘要（Summarization）会在溢出发生前压缩历史；记忆（memory）在启动时加载持久化指令，使知识能够跨会话延续；技能（skills）按需呈现领域知识，而不是一次性加载全部内容。

#### Google

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "google-genai:gemini-3.6-flash";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### OpenAI

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "openai:gpt-5.5";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### Anthropic

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "anthropic:claude-sonnet-4-6";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### OpenRouter

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "openrouter:openrouter:z-ai/glm-5.2";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### Fireworks

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "fireworks:accounts/fireworks/models/glm-5p2";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### Baseten

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "baseten:zai-org/GLM-5.2";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

#### Ollama

```ts
import { createAgent } from "langchain";
import {
  StateBackend,
  createFilesystemMiddleware,
  createSkillsMiddleware,
  createSummarizationMiddleware,
} from "deepagents";

var backend = new StateBackend();
const model = "ollama:north-mini-code-1.0";

var agent = createAgent({
  model,
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    createSummarizationMiddleware({ model, backend }),
    createSkillsMiddleware({ backend, sources: ["./skills/"] }),
  ],
});
```

参见 [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware)、[`MemoryMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createMemoryMiddleware)、[技能（Skills）](/oss/javascript/langchain/multi-agent/skills)、[上下文工程](/oss/javascript/deepagents/context-engineering)。

> 【提示】此示例从 `deepagents` 包导入。请按以下方式安装：
>
> **npm**
>
> ```bash
> npm install deepagents
> ```
>
> **yarn**
>
> ```bash
> yarn add deepagents
> ```
>
> **pnpm**
>
> ```bash
> pnpm add deepagents
> ```

### 规划与委派

复杂任务常常超出单个上下文窗口所能处理的范围。委派（Delegation）让主智能体把工作拆分成多个片段，交给各自在独立上下文中运行的子智能体，而自身专注于协调而非执行。工作可以并行运行；主智能体的上下文保持干净。

#### Google

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "google-genai:gemini-3.6-flash",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### OpenAI

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "openai:gpt-5.5",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### Anthropic

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "anthropic:claude-sonnet-4-6",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### OpenRouter

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "openrouter:openrouter:z-ai/glm-5.2",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### Fireworks

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "fireworks:accounts/fireworks/models/glm-5p2",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### Baseten

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "baseten:zai-org/GLM-5.2",
          middleware: [],
        },
      ],
    }),
  ],
});
```

#### Ollama

```ts
import { createAgent, todoListMiddleware, tool } from "langchain";
import {
  createFilesystemMiddleware,
  createSubAgentMiddleware,
  StateBackend,
} from "deepagents";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var backend = new StateBackend();

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search],
  middleware: [
    createFilesystemMiddleware({ backend }),
    todoListMiddleware(),
    createSubAgentMiddleware({
      defaultModel: "anthropic:claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "researcher",
          description: "Searches and returns a structured summary.",
          systemPrompt:
            "Use the search tool to research the question and summarize key points.",
          tools: [search],
          model: "ollama:north-mini-code-1.0",
          middleware: [],
        },
      ],
    }),
  ],
});
```

参见[子智能体（Subagents）](/oss/javascript/langchain/multi-agent/subagents)。

> 【提示】此示例从 `deepagents` 包导入。请按以下方式安装：
>
> **npm**
>
> ```bash
> npm install deepagents
> ```
>
> **yarn**
>
> ```bash
> yarn add deepagents
> ```
>
> **pnpm**
>
> ```bash
> pnpm add deepagents
> ```

### 为智能体命名

可选地为智能体使用一个标识符。当把智能体作为子图嵌入[多智能体](/oss/javascript/langchain/multi-agent)系统时，这尤其有用。

#### Google

```ts
var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools,
  name: "research_assistant",
});
```

#### OpenAI

```ts
var agent = createAgent({
  model: "openai:gpt-5.5",
  tools,
  name: "research_assistant",
});
```

#### Anthropic

```ts
var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools,
  name: "research_assistant",
});
```

#### OpenRouter

```ts
var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools,
  name: "research_assistant",
});
```

#### Fireworks

```ts
var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools,
  name: "research_assistant",
});
```

#### Baseten

```ts
var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools,
  name: "research_assistant",
});
```

#### Ollama

```ts
var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools,
  name: "research_assistant",
});
```

### 容错

生产环境中的智能体会遇到开发阶段很少出现的故障：速率限制、模型超时、瞬时的 API 错误。容错中间件在基础设施层面处理这些问题，因此你的工具和业务逻辑无需在每次调用周围包裹 try/catch。

#### Google

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### OpenAI

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### Anthropic

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### OpenRouter

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### Fireworks

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### Baseten

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

#### Ollama

```ts
import {
  createAgent,
  modelRetryMiddleware,
  tool,
  toolRetryMiddleware,
} from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3 }),
    toolRetryMiddleware({ maxRetries: 2 }),
  ],
});
```

参见 [`modelRetryMiddleware`](https://reference.langchain.com/javascript/langchain/index/modelRetryMiddleware)、[`toolRetryMiddleware`](https://reference.langchain.com/javascript/langchain/index/toolRetryMiddleware)、[预置中间件](/oss/javascript/langchain/middleware/built-in)。

### 护栏

有些策略无法只存在于提示词中——无论模型做什么，它们都需要被确定性地强制执行。护栏在数据流经智能体循环时对其进行拦截，在工具结果进入模型上下文之前应用合规规则或内容策略。

#### Google

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### OpenAI

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### Anthropic

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### OpenRouter

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### Fireworks

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### Baseten

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

#### Ollama

```ts
import { createAgent, piiMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search],
  middleware: [piiMiddleware("email")],
});
```

参见 [`piiMiddleware`](https://reference.langchain.com/javascript/langchain/index/piiMiddleware)、[预置中间件](/oss/javascript/langchain/middleware/built-in)。

### 人工引导

完全自主并不总是合适的。人工引导（Steering）让你在特定的决策点安置人类——在破坏性写入、昂贵的 API 调用或任何需要判断的操作之前——而无需重构你的智能体。智能体会暂停并等待；由人类批准、编辑或拒绝；然后继续执行。

#### Google

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### OpenAI

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### Anthropic

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### OpenRouter

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### Fireworks

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### Baseten

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

#### Ollama

```ts
import { createAgent, humanInTheLoopMiddleware, tool } from "langchain";
import * as z from "zod";

var search = tool(({ query }) => `Search results for: ${query}`, {
  name: "search",
  description: "Search for a query and return a short summary.",
  schema: z.object({ query: z.string() }),
});

var agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { writeFile: true } })],
});
```

参见 [`humanInTheLoopMiddleware`](https://reference.langchain.com/javascript/langchain/middleware/humanInTheLoopMiddleware)、[Human-in-the-loop](/oss/javascript/langchain/human-in-the-loop)。

### 中间件资源

- [中间件概览](/oss/javascript/langchain/middleware/overview)：中间件栈如何工作以及钩子何时触发
- [预置中间件](/oss/javascript/langchain/middleware/built-in)：完整参考与配置示例
- [自定义中间件](/oss/javascript/langchain/middleware/custom)：为业务逻辑、PII 擦除等编写你自己的钩子

---

> 【信息】将这些文档连接到 Claude、VSCode 等，通过 MCP 获得实时答案：[连接这些文档](/use-these-docs)。
>
> 在 GitHub 上[编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/agents.mdx)或[提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。


