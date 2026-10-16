# 工具（Tools）

> 原文：https://docs.langchain.com/oss/javascript/langchain/tools
> LangChain (TypeScript) 官方文档中文整理

工具（Tools）扩展了 [智能体（agents）](/oss/javascript/langchain/agents) 的能力——让它们能够获取实时数据、执行代码、查询外部数据库，并在现实世界中采取行动。

在底层，工具是带有明确定义输入和输出的可调用函数，会被传递给 [聊天模型（chat model）](/oss/javascript/langchain/models)。模型会根据对话上下文决定何时调用工具，以及提供什么输入参数。

> 【提示】
> 关于模型如何处理工具调用的细节，请参阅 [工具调用（Tool calling）](/oss/javascript/langchain/models#tool-calling)。使用 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-langchain-tools) 跟踪工具调用并调试错误。按照 [追踪快速入门（tracing quickstart）](/langsmith/trace-with-langchain) 完成设置。
>
> 我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它可以监控你的追踪记录、检测问题并提出修复建议。

## 创建工具

### 基本工具定义

创建工具最简单的方式是从 `langchain` 包中导入 `tool` 函数。你可以使用 [zod](https://zod.dev/) 来定义工具的输入 schema：

```ts
import * as z from "zod"
import { tool } from "langchain"

const searchDatabase = tool(
  ({ query, limit }) => `Found ${limit} results for '${query}'`,
  {
    name: "search_database",
    description: "Search the customer database for records matching the query.",
    schema: z.object({
      query: z.string().describe("Search terms to look for"),
      limit: z.number().describe("Maximum number of results to return"),
    }),
  }
);
```

> 【提示】
> **服务端工具使用（Server-side tool use）：** 一些聊天模型具有内置工具（网络搜索、代码解释器），它们在服务端执行。详情请参阅 [服务端工具使用](#server-side-tool-use)。

> 【警告】
> 工具名称建议使用 `snake_case`（例如使用 `web_search` 而不是 `Web Search`）。一些模型提供商会因为名称包含空格或特殊字符而出现问题，或直接报错拒绝。坚持使用字母数字字符、下划线和连字符，有助于提高跨提供商的兼容性。

## 访问上下文

当工具能够访问运行时信息（如对话历史、用户数据和持久化记忆）时，它们最为强大。本节介绍如何在工具内部访问和更新这些信息。

### 上下文（Context）

上下文提供在调用时传入的不可变配置数据。可将它用于用户 ID、会话详情，或对话期间不应更改的应用程序特定设置。

> 【提示】
> `thread_id`（通过 `config={"configurable": {"thread_id": ...}}` 传入）界定的是 *对话* 范围：消息历史和检查点；而 `context` 携带的是 *每次运行（per-run）* 的数据，供你的工具和中间件在调用时读取。在生产环境中，你通常会将两者一起传入：每次对话使用一个稳定的 `thread_id`，并在每次调用时传入一个 `context` 对象。

工具可以通过 `config` 参数访问智能体的运行时上下文。将 `context` 与 `thread_id` 一起传入，以便对话在多次轮次之间持久化：

#### Google

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "google-genai:gemini-3.6-flash" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### OpenAI

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "openai:gpt-5.5" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### Anthropic

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "anthropic:claude-sonnet-4-6" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### OpenRouter

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "openrouter:openrouter:z-ai/glm-5.2" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### Fireworks

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "fireworks:accounts/fireworks/models/glm-5p2" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### Baseten

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "baseten:zai-org/GLM-5.2" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

#### Ollama

```ts
import * as z from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";

const getUserName = tool(
  (_, config) => {
    return config.context.user_name;
  },
  {
    name: "get_user_name",
    description: "Get the user's name.",
    schema: z.object({}),
  },
);

const contextSchema = z.object({
  user_name: z.string(),
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "ollama:north-mini-code-1.0" }),
  tools: [getUserName],
  contextSchema,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is my name?" }],
  },
  {
    configurable: { thread_id: crypto.randomUUID() },
    context: { user_name: "John Smith" },
  },
);
```

> 【卡片】查看示例追踪（View example trace）
> 打开此示例对应的公开 LangSmith 运行记录：https://smith.langchain.com/public/59163065-33b5-4b9e-a8d0-dc882e31f97c/r

### 长期记忆（Store）

[`BaseStore`](https://reference.langchain.com/javascript/langchain-core/stores/BaseStore) 提供可跨对话持久保存的存储。与状态（短期记忆）不同，保存到 store 中的数据在未来的会话中仍然可用。

通过 `config.store` 访问 store。store 使用命名空间/键（namespace/key）模式来组织数据：

> 【提示】
> 对于生产部署，请使用持久化的 store 实现，例如 [`PostgresStore`](https://reference.langchain.com/javascript/langchain-langgraph-checkpoint-postgres/store/PostgresStore)、`MongoDBStore` 或 `RedisStore`，而不是 `InMemoryStore`。设置细节请参阅 [记忆文档（memory documentation）](/oss/javascript/langgraph/add-memory)。

```ts
import * as z from "zod";
import { createAgent, tool } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";
import { ChatOpenAI } from "@langchain/openai";

const store = new InMemoryStore();

// Access memory
const getUserInfo = tool(
  async ({ user_id }) => {
    const value = await store.get(["users"], user_id);
    console.log("get_user_info", user_id, value);
    return value;
  },
  {
    name: "get_user_info",
    description: "Look up user info.",
    schema: z.object({
      user_id: z.string(),
    }),
  }
);

// Update memory
const saveUserInfo = tool(
  async ({ user_id, name, age, email }) => {
    console.log("save_user_info", user_id, name, age, email);
    await store.put(["users"], user_id, { name, age, email });
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info.",
    schema: z.object({
      user_id: z.string(),
      name: z.string(),
      age: z.number(),
      email: z.string(),
    }),
  }
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "gpt-5.5" }),
  tools: [getUserInfo, saveUserInfo],
  store,
});

// First session: save user info
await agent.invoke({
  messages: [
    {
      role: "user",
      content: "Save the following user: userid: abc123, name: Foo, age: 25, email: foo@langchain.dev",
    },
  ],
});

// Second session: get user info
const result = await agent.invoke({
  messages: [
    { role: "user", content: "Get user info for user with id 'abc123'" },
  ],
});

console.log(result);
// Here is the user info for user with ID "abc123":
// - Name: Foo
// - Age: 25
// - Email: foo@langchain.dev
```

### 流式写入器（Stream writer）

在工具执行期间实时流式推送更新。这对于在长时间运行的操作中向用户提供进度反馈非常有用。

使用 `config.writer` 发出自定义更新：

```ts
import * as z from "zod";
import { tool, ToolRuntime } from "langchain";

const getWeather = tool(
  ({ city }, config: ToolRuntime) => {
    const writer = config.writer;

    // Stream custom updates as the tool executes
    if (writer) {
      writer(`Looking up data for city: ${city}`);
      writer(`Acquired data for city: ${city}`);
    }

    return `It's always sunny in ${city}!`;
  },
  {
    name: "get_weather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string(),
    }),
  }
);
```

### 执行信息（Execution info）

通过 `runtime.execution_info` 在工具内部访问线程 ID、运行 ID 和重试状态：

```ts
import { tool } from "langchain";
import * as z from "zod";

const logExecutionContext = tool(
  async (_input, runtime) => {
    const info = runtime.executionInfo;
    console.log(`Thread: ${info.threadId}, Run: ${info.runId}`);
    console.log(`Attempt: ${info.nodeAttempt}`);
    return "done";
  },
  {
    name: "log_execution_context",
    description: "Log execution identity information.",
    schema: z.object({}),
  }
);
```

> 【提示】
> 需要 `deepagents>=1.9.0`（或 `@langchain/langgraph>=1.2.8`）。

### 服务端信息（Server info）

当你的工具运行在 LangGraph Server 上时，可通过 `runtime.server_info` 访问助手 ID、图 ID 以及已认证用户：

```ts
import { tool } from "langchain";
import * as z from "zod";

const getAssistantScopedData = tool(
  async (_input, runtime) => {
    const server = runtime.serverInfo;
    if (server != null) {
      console.log(`Assistant: ${server.assistantId}, Graph: ${server.graphId}`);
      if (server.user != null) {
        console.log(`User: ${server.user.identity}`);
      }
    }
    return "done";
  },
  {
    name: "get_assistant_scoped_data",
    description: "Fetch data scoped to the current assistant.",
    schema: z.object({}),
  }
);
```

当工具不在 LangGraph Server 上运行时，`serverInfo` 为 `null`。

> 【提示】
> 需要 `deepagents>=1.9.0`（或 `@langchain/langgraph>=1.2.8`）。

## 工具执行

在 LangChain 中，工具由智能体使用（例如通过 [`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent)），而工具错误处理通过 [中间件（middleware）](/oss/javascript/langchain/middleware) 进行配置。

对于 LangGraph 工作流，工具执行由 [`ToolNode`](https://reference.langchain.com/javascript/langchain-langgraph/prebuilt/ToolNode) 处理。Graph API 用法请参阅 [ToolNode](/oss/javascript/langgraph/workflows-agents#toolnode)，其中包括工具如何访问当前图状态和运行范围（run-scoped）的上下文。

### 工具返回值

你可以为工具选择不同的返回值：

* 返回 `string`，用于人类可读的结果。
* 返回 `object`，用于模型应解析的结构化结果。
* 返回 `Command`（可带消息），当你需要写入状态时使用。

#### 返回字符串

当工具应提供纯文本供模型阅读并在其下一次响应中使用时，返回字符串。

```ts
import { tool } from "langchain";
import * as z from "zod";

const getWeather = tool(({ city }) => `It is currently sunny in ${city}.`, {
  name: "get_weather",
  description: "Get weather for a city.",
  schema: z.object({ city: z.string() }),
});
```

行为：

* 返回值会被转换为 `ToolMessage`。
* 模型看到该文本，并决定下一步做什么。
* 除非模型或其他工具之后这么做，否则不会更改任何智能体状态字段。

当结果本身自然就是人类可读文本时使用这种方式。

#### 返回对象

当你的工具产生模型应检查的结构化数据时，返回对象（例如 `dict`）。

```ts
import { tool } from "langchain";
import * as z from "zod";

const getWeatherData = tool(
  ({ city }) => ({
    city,
    temperature_c: 22,
    conditions: "sunny",
  }),
  {
    name: "get_weather_data",
    description: "Get structured weather data for a city.",
    schema: z.object({ city: z.string() }),
  },
);
```

行为：

* 对象会被序列化，并作为工具输出发回。
* 模型可以读取特定字段并据此推理。
* 与字符串返回一样，这不会直接更新图状态。

当下游推理能从显式字段（而非自由格式文本）中获益时使用这种方式。

#### 返回多模态内容

工具不限于纯文本。当模型支持多模态工具结果时，工具可以返回 [标准内容块（standard content blocks）](/oss/javascript/langchain/messages#standard-content-blocks)，让模型在一次工具结果中接收文本、图像和其他媒体。

```typescript
import { tool } from "langchain";
import { z } from "zod";

const captureScreenshot = tool(
  async () => [
    { type: "text", text: "Screenshot of the current page:" },
    { type: "image", url: "https://example.com/page.png" },
  ],
  {
    name: "capture_screenshot",
    description: "Capture a screenshot of the current page.",
    schema: z.object({}),
  }
);
```

行为：

* 返回值会被转换为带有 `content` 的 `ToolMessage`（多模态 `content`）。
* 工具运行后，使用 `message.content_blocks` 读取规范化后的块列表。
* 模型必须支持你返回的模态。在返回图像、音频或视频之前，请检查你的 [模型能力](/oss/javascript/integrations/chat)。

关于块类型和提供商特定要求，请参阅 [多模态消息（Multimodal messages）](/oss/javascript/langchain/messages#multimodal)。

#### 返回 Command

当工具需要更新图状态（例如设置用户偏好或应用状态）时，返回 [`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command)。
当 `Command` 以当前图为目标时，请在更新中包含一个 `ToolMessage`，其工具调用 ID 与当前工具调用匹配。
消息历史中的每个工具调用都必须有对应的 `ToolMessage`。

使用 `runtime.toolCallId` 作为 `tool_call_id` 参数。

```ts
import { tool, ToolMessage, type ToolRuntime } from "langchain";
import { Command } from "@langchain/langgraph";
import * as z from "zod";

const setLanguage = tool(
  async ({ language }, config: ToolRuntime) => {
    return new Command({
      update: {
        preferredLanguage: language,
        messages: [
          new ToolMessage({
            content: `Language set to ${language}.`,
            tool_call_id: config.toolCallId,
          }),
        ],
      },
    });
  },
  {
    name: "set_language",
    description: "Set the preferred response language.",
    schema: z.object({ language: z.string() }),
  },
);
```

行为：

* 该 command 使用 `update` 更新状态。
* 更新后的状态可用于同一次运行中的后续步骤。
* 对于可能被并行工具调用更新的字段，请使用 reducer。

当工具不只是返回数据，还会改变智能体状态时使用这种方式。

#### 直接从工具返回

在工具上设置直接返回（return direct），以短路智能体循环：智能体立即将工具输出返回给调用者，而不将其送回模型进行进一步处理。

##### Google

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "google-genai:gemini-3.6-flash" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### OpenAI

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "openai:gpt-5.5" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### Anthropic

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "anthropic:claude-sonnet-4-6" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### OpenRouter

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "openrouter:openrouter:z-ai/glm-5.2" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### Fireworks

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "fireworks:accounts/fireworks/models/glm-5p2" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### Baseten

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "baseten:zai-org/GLM-5.2" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

##### Ollama

```ts
import { ChatOpenAI } from "@langchain/openai";
import { createAgent, tool } from "langchain";
import * as z from "zod";

const fetchOrderStatus = tool(
  ({ order_id }) => {
    return `Order ${order_id} is shipped and will arrive in 2 days.`;
  },
  {
    name: "fetch_order_status",
    description: "Fetch the current status of a customer order.",
    schema: z.object({ order_id: z.string() }),
    returnDirect: true,
  },
);

const agent = createAgent({
  model: new ChatOpenAI({ model: "ollama:north-mini-code-1.0" }),
  tools: [fetchOrderStatus],
});

const result = await agent.invoke({
  messages: [
    { role: "user", content: "What is the status of order #12345?" },
  ],
});
// The agent returns the tool output directly without another LLM call:
// "Order 12345 is shipped and will arrive in 2 days."
```

> 【卡片】查看示例追踪（View example trace）
> 打开此示例对应的公开 LangSmith 运行记录：https://smith.langchain.com/public/8f9bb682-fa0b-4264-8495-246703f0624b/r

行为：

* 工具正常执行，其输出被包装在 `ToolMessage` 中。
* 智能体停止循环，并将工具的输出作为最终响应返回，跳过任何额外的模型调用。
* **多个并行工具调用：** 当模型在一步中调用多个工具时，它们都会先执行。所有工具完成后，只有当该批次中 **每一个** 工具都设置了 `return_direct=True` 时，智能体才会路由到 `END`。最终响应包含该步骤中被调用的每个工具的 `ToolMessage` 输出。

在以下情况使用这种方式：

* 工具的输出就是完整、可直接呈现给用户的答案（例如返回可直接显示结果的查询）。
* 当不需要额外推理时，你希望避免额外的模型调用。
* 你需要确定性、未经修改的输出：模型无法对工具结果进行改写、总结或进一步处理。

> 【警告】
> 因为模型不处理工具的输出，`return_direct=True` 不适用于其结果需要进一步推理、总结或与其他工具调用串联的工具。

> 【警告】
> **混合并行调用：** 如果模型同时调用了一个 `return_direct=True` 的工具和一些没有 `return_direct=True` 的工具，智能体在该步骤后 **不会** 退出。它会带着该批次中的所有 `ToolMessage` 回到模型，让模型可以推理所有结果。只有当该步骤中的每个工具调用都设置了 `return_direct=True` 时，`return_direct` 才会短路循环。

### 错误处理

使用 LangChain 智能体 [中间件（middleware）](/oss/javascript/langchain/middleware) 处理工具错误，以重试失败的工具调用或返回自定义错误消息：

#### Google

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### OpenAI

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### Anthropic

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### OpenRouter

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### Fireworks

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### Baseten

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  middleware: [handleToolErrors],
});
```

#### Ollama

```ts
import { createAgent, createMiddleware, ToolMessage } from "langchain";

const handleToolErrors = createMiddleware({
  name: "HandleToolErrors",
  wrapToolCall: async (request, handler) => {
    try {
      return await handler(request);
    } catch (error) {
      return new ToolMessage({
        content: `Tool error: Please check your input and try again. (${error})`,
        tool_call_id: request.toolCall.id!,
      });
    }
  },
});

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  middleware: [handleToolErrors],
});
```

### 状态注入

工具通过 [`ToolRuntime`](https://reference.langchain.com/javascript/langchain/index/Runtime) 访问图状态。状态、上下文、store 和流式 API 请参阅 [访问上下文](#access-context)。

关于从工具访问状态、上下文和长期记忆的更多细节，请参阅 [访问上下文](#access-context)。

## 动态工具选择

使用动态工具时，智能体可用的工具集在运行时被修改，而不是全部预先定义。并非每个工具都适合每种情况。工具过多可能会使模型不堪重负（上下文过载）并增加错误；工具过少则会限制能力。动态工具选择使得可以根据认证状态、用户权限、功能开关（feature flags）或对话阶段来调整可用工具集。

根据工具是否事先已知，有两种方法：

### 过滤预先注册的工具

当所有可能的工具在创建智能体时就已知时，你可以预先注册它们，并根据状态、权限或上下文动态过滤哪些工具暴露给模型。

#### 状态（State）

仅在达到某些对话里程碑后启用高级工具：

```typescript
import { createMiddleware, tool } from "langchain";
import { createDeepAgent } from "deepagents";

const stateBasedTools = createMiddleware({
    name: "StateBasedTools",
    wrapModelCall: (request, handler) => {
        // Read from State: check authentication and conversation length
        const state = request.state as typeof request.state & {
            authenticated?: boolean;
        };
        const isAuthenticated = state.authenticated ?? false;
        const messageCount = state.messages.length;

        let filteredTools = request.tools;

        // Only enable sensitive tools after authentication
        if (!isAuthenticated) {
            filteredTools = request.tools.filter(
                (t: any) => typeof t.name === "string" && t.name.startsWith("public_"),
            );
        } else if (messageCount < 5) {
            filteredTools = request.tools.filter(
                (t: any) => typeof t.name === "string" && t.name !== "advanced_search",
            );
        }

        return handler({ ...request, tools: filteredTools });
    },
});

const agent = await createDeepAgent({
    model: "claude-sonnet-4-6",
    tools: tools,
    middleware: [stateBasedTools] as any,
});
```

#### Store

根据 Store 中的用户偏好或功能开关过滤工具：

```typescript
import { createMiddleware } from "langchain";
import { createDeepAgent, StoreBackend } from "deepagents";
import * as z from "zod";
import { InMemoryStore } from "@langchain/langgraph";

const contextSchema = z.object({
  userId: z.string(),
});

const storeBasedTools = createMiddleware({
  name: "StoreBasedTools",
  contextSchema,
  wrapModelCall: async (request, handler) => {
    const userId =
      (request.runtime?.context as { userId?: string } | undefined)?.userId ??
        "user-123";

    // Read from Store: get user's enabled features
    const runtimeStore = request.runtime?.store as InMemoryStore | undefined;
    const rawFlags = (await runtimeStore?.get(
      ["features"],
      userId as string,
    )) as unknown;
    const featureFlags = rawFlags as FeatureFlags | undefined;

    let filteredTools = request.tools;

    if (featureFlags) {
      const enabledFeatures = featureFlags.enabledTools || [];
      filteredTools = request.tools.filter((t) =>
        enabledFeatures.includes(t.name as string)
      );
    }

    return handler({ ...request, tools: filteredTools });
  },
});

const agent = await createDeepAgent({
  model: "claude-sonnet-4-6",
  backend: new StoreBackend(),
  store,
  checkpointer,
  tools,
  middleware: [storeBasedTools] as any,
});
```

#### 运行时上下文（Runtime Context）

根据运行时上下文中的用户权限过滤工具：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";

const contextSchema = z.object({
  userRole: z.string(),
});

const contextBasedTools = createMiddleware({
  name: "ContextBasedTools",
  contextSchema,
  wrapModelCall: (request, handler) => {
    // Read from Runtime Context: get user role
    const userRole = request.runtime.context.userRole;

    let filteredTools = request.tools;

    if (userRole === "admin") {
      // Admins get all tools
    } else if (userRole === "editor") {
      filteredTools = request.tools.filter((t) => t.name !== "delete_data");
    } else {
      filteredTools = request.tools.filter(
        (t) => (t.name as string).startsWith("read_"),
      );
    }

    return handler({ ...request, tools: filteredTools });
  },
});

const agent = await createDeepAgent({
  model: "claude-sonnet-4-6",
  store,
  checkpointer,
  tools,
  middleware: [contextBasedTools] as any,
});
```

这种方法在以下情况最佳：

* 所有可能的工具在编译/启动时就已知
* 你希望根据权限、功能开关或对话状态进行过滤
* 工具是静态的，但它们的可用性是动态的

更多示例请参阅 [动态选择工具（Dynamically selecting tools）](/oss/javascript/langchain/middleware/custom#dynamically-selecting-tools)。

### 运行时工具注册

当工具在运行时被发现或创建时（例如从 MCP 服务器加载、根据用户数据生成，或从远程注册表获取），你需要既注册这些工具，又动态处理它们的执行。

这需要两个中间件钩子：

1. `wrap_model_call` —— 将动态工具添加到请求中
2. `wrap_tool_call` —— 处理动态添加的工具的执行

```typescript
import { createAgent, createMiddleware, tool } from "langchain";
import * as z from "zod";

// A tool that will be added dynamically at runtime
const calculateTip = tool(
  ({ billAmount, tipPercentage = 20 }) => {
    const tip = billAmount * (tipPercentage / 100);
    return `Tip: $${tip.toFixed(2)}, Total: $${(billAmount + tip).toFixed(2)}`;
  },
  {
    name: "calculate_tip",
    description: "Calculate the tip amount for a bill",
    schema: z.object({
      billAmount: z.number().describe("The bill amount"),
      tipPercentage: z.number().default(20).describe("Tip percentage"),
    }),
  }
);

const dynamicToolMiddleware = createMiddleware({
  name: "DynamicToolMiddleware",
  wrapModelCall: (request, handler) => {
    // Add dynamic tool to the request
    // This could be loaded from an MCP server, database, etc.
    return handler({
      ...request,
      tools: [...request.tools, calculateTip],
    });
  },
  wrapToolCall: (request, handler) => {
    // Handle execution of the dynamic tool
    if (request.toolCall.name === "calculate_tip") {
      return handler({ ...request, tool: calculateTip });
    }
    return handler(request);
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [getWeather], // Only static tools registered here
  middleware: [dynamicToolMiddleware],
});

// The agent can now use both getWeather AND calculateTip
const result = await agent.invoke({
  messages: [{ role: "user", content: "Calculate a 20% tip on $85" }],
});
```

这种方法在以下情况最佳：

* 工具在运行时被发现（例如来自 MCP 服务器）
* 工具根据用户数据或配置动态生成
* 你正在与外部工具注册表集成

> 【提示】
> 对于运行时注册的工具，`wrap_tool_call` 钩子是必需的，因为智能体需要知道如何执行不在原始工具列表中的工具。没有它，智能体将不知道如何调用动态添加的工具。

## 无头工具（Headless tools）

有些工具应该在 **用户应用运行的地方**（通常是浏览器）运行，而不是在进程内部运行。**无头工具（Headless tools）** 是工具定义（包含名称、描述和参数 schema），你在 **服务端** 将它们注册到智能体。**实现** 只在 **客户端** 注册，并在一次简短的 interrupt/resume 握手之后执行。

这与函数体在服务端运行的普通工具不同，也与 [服务端工具使用](#server-side-tool-use)（模型提供商远程执行内置工具）不同。

### 何时使用无头工具

当工作依赖于只有客户端才存在的 **环境、设备或 UI** 时使用它们。例如：

* **浏览器 API：** 地理位置（Geolocation）、IndexedDB、剪贴板（Clipboard）、Canvas 2D、文件选择器（file pickers）、电池 API（Battery API）等。
* **隐私与本地性：** 数据留在设备上（例如 IndexedDB 中的本地“记忆”）。
* **延迟：** 纯本地操作无需额外的服务端往返。
* **结构化、安全的副作用：** 优先使用许多小而类型化的工具（例如每个 canvas 原语一个工具），而不是把任意代码发送给 `eval`。

### 该模式如何工作

在两种运行时中，模型看到的都是一个它可以调用的普通工具，但实际执行发生在服务端进程之外。

1. **定义** 工具：使用 `langchain` 的 `tool({ name, description, schema })`，仅包含元数据和验证，没有服务端运行器。
2. **附加** 真实行为：使用 `.implement(async (args) => { ... })`，它返回一个 **无头工具实现**（定义 + `execute` 函数）。
3. **注册** 第 1 步中的定义：通过 `createAgent` 或你的图注册，使模型在其常规工具调用循环中看到该工具。
4. **传递** 第 2 步中的实现：传递给你的流式钩子的 `tools` 选项。

> 【信息】
> 将 **工具定义**（`tool({ name, description, schema })`）和 **实现**（`.implement(...)`）放在 **独立的模块** 中。从你的服务端智能体和前端都导入共享的定义文件，以保持名称和 schema 一致；将仅客户端的执行逻辑放在服务端永远不会加载的实现模块中。

当模型为这些工具之一发出工具调用时，运行会 **中断（interrupts）**，而不是在本地执行该工具。你的应用可以检查 payload，在正确的环境中执行操作（例如浏览器、另一个服务，或人工审核步骤），然后用工具结果 **恢复（resume）** 图。当你使用受支持的 JS SDK 钩子时，它们可以检测到无头工具中断，运行匹配的客户端实现，并为你提交 resume 命令。

使用可选的 **`onTool`** 回调来观察生命周期事件（`start`、`success`、`error`），以实现 UI 反馈，例如加载指示器（spinners）或提示条（toasts）。

> 【卡片】无头工具前端模式（Headless tools frontend pattern）
> 查看一个使用 `useStream` 在客户端执行仅含 schema 的工具的端到端示例：/oss/javascript/langchain/frontend/headless-tools

## 预构建工具

LangChain 提供了大量预构建的工具和工具包，用于常见任务，如网络搜索、代码解释、数据库访问等。这些开箱即用的工具可以直接集成到你的智能体中，无需编写自定义代码。

请参阅 [工具与工具包（tools and toolkits）](/oss/javascript/integrations/tools) 集成页面，以获取按类别组织的可用工具完整列表。

## 来自 MCP 服务器的工具

[模型上下文协议（Model Context Protocol，MCP）](https://modelcontextprotocol.io) 是一个开放协议，标准化了应用程序如何向语言模型暴露工具。无需手动编写工具，你可以连接到一个 MCP 服务器，并将它声明的工具适配为 LangChain 工具，然后像任何其他工具一样传递给智能体。

LangChain 智能体通过 [`@langchain/mcp-adapters`](https://github.com/langchain-ai/langchainjs/tree/main/libs/langchain-mcp-adapters) 库调用 MCP 服务器上定义的工具，该库会发现服务器的工具并将它们适配为 LangChain 工具。

关于设置、传输方式（transports）和认证，请参阅 [模型上下文协议（MCP）](/oss/javascript/langchain/mcp)。

## 服务端工具使用

一些聊天模型具有由模型提供商在服务端执行的内置工具。这些包括网络搜索和代码解释器等能力，无需你定义或托管工具逻辑。

关于启用和使用这些内置工具的细节，请参阅各个 [聊天模型集成页面](/oss/javascript/integrations/providers) 以及 [工具调用文档](/oss/javascript/langchain/models#server-side-tool-use)。

***

> 【提示】
> 通过 MCP 将这些文档连接到 Claude、VSCode 等，以获得实时答案：[连接这些文档](/use-these-docs)。
>
> 在 GitHub 上编辑此页面：https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/tools.mdx ，或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
