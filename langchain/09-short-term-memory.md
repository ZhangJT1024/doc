# 短期记忆（Short-term memory）

> 原文：https://docs.langchain.com/oss/javascript/langchain/short-term-memory
> LangChain (TypeScript) 官方文档中文整理

## 概述

记忆（Memory）是一个记住先前交互信息的系统。对于 AI 智能体而言，记忆至关重要，因为它让智能体能够记住先前的交互、从反馈中学习，并适应用户偏好。随着智能体处理包含大量用户交互的越来越复杂的任务，这一能力对效率和用户满意度都变得不可或缺。

短期记忆让你的应用能够在单个线程或对话中记住先前的交互。

> 【提示】
> 线程（thread）将一次会话中的多个交互组织在一起，类似于电子邮件将消息分组到一个对话中的方式。

对话历史是最常见的短期记忆形式。长对话对当今的 LLM 构成挑战；完整的历史可能无法放入 LLM 的上下文窗口，从而导致上下文丢失或错误。

即使你的模型支持完整的上下文长度，大多数 LLM 在长上下文上仍然表现不佳。它们会被陈旧或跑题的内容“分心”，同时还伴随着更慢的响应时间和更高的成本。

聊天模型通过 [消息（messages）](/oss/javascript/langchain/messages) 接受上下文，其中包括指令（系统消息）和输入（人类消息）。在聊天应用中，消息在人类输入和模型响应之间交替，导致消息列表随着时间推移越来越长。由于上下文窗口有限，许多应用可以从使用移除或“遗忘”陈旧信息的技术中受益。

> 【提示】
> 需要 **跨** 对话记住信息？使用 [长期记忆（long-term memory）](/oss/javascript/langchain/long-term-memory) 在不同线程和会话之间存储和召回用户特定或应用级别的数据。

## 使用

要为智能体添加短期记忆（线程级持久化），你需要在创建智能体时指定 `checkpointer`。

> 【信息】
> LangChain 的智能体将短期记忆作为智能体状态的一部分进行管理。
>
> 通过将这些存储在图的 state 中，智能体可以访问给定对话的完整上下文，同时保持不同线程之间的隔离。
>
> 使用 checkpointer 将 state 持久化到数据库（或内存），以便线程可以在任何时候恢复。
>
> 当智能体被调用或某个步骤（如工具调用）完成时，短期记忆会更新；在每个步骤开始时读取 state。

#### Google

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### OpenAI

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### Anthropic

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### OpenRouter

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### Fireworks

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### Baseten

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

#### Ollama

```ts
import { createAgent, tool } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const getUserInfo = tool(() => "No user profile on file.", {
  name: "get_user_info",
  description: "Look up information about the current user.",
  schema: z.object({}),
});

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [getUserInfo],
  checkpointer,
});

const threadConfig = { configurable: { thread_id: "1" } };
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Hi! My name is Bob." }] },
  threadConfig,
);
let response = result.messages.at(-1)?.content;
console.log(response); // "Hi Bob! Nice to see you here. How are you doing?"

result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  threadConfig,
);
response = result.messages.at(-1)?.content;
console.log(response); // "You are Bob!"
```

> 【卡片】查看示例追踪（View example trace）
> 打开此示例对应的公开 LangSmith 运行记录：https://smith.langchain.com/public/eaa48cfd-c3f0-4951-a496-f448a8dd7a04/r

### 在生产环境中

在生产环境中，使用由数据库支持的 checkpointer：

```ts
import { PostgresSaver } from "@langchain/langgraph-checkpoint-postgres";

const DB_URI = "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const checkpointer = PostgresSaver.fromConnString(DB_URI);
```

> 【提示】
> 关于包括 SQLite、Postgres 和 Azure Cosmos DB 在内的更多 checkpointer 选项，请参阅 Persistence 文档中的 [checkpointer 库列表](/oss/javascript/langgraph/checkpointers#checkpointer-libraries)。

## 自定义智能体记忆

你可以通过创建带有 state schema 的自定义中间件来扩展智能体状态。自定义 state schema 可以通过中间件中的 `stateSchema` 参数传入。state 定义建议使用 `StateSchema` 类（也支持普通的 Zod 对象）。

```typescript
import { createAgent, createMiddleware } from "langchain";
import { StateSchema, MemorySaver } from "@langchain/langgraph";
import * as z from "zod";

const CustomState = new StateSchema({
    userId: z.string(),
    preferences: z.record(z.string(), z.any()),
});

const stateExtensionMiddleware = createMiddleware({
    name: "StateExtension",
    stateSchema: CustomState,
});

const checkpointer = new MemorySaver();
const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    middleware: [stateExtensionMiddleware],
    checkpointer,
});

// Custom state can be passed in invoke
const result = await agent.invoke({
    messages: [{ role: "user", content: "Hello" }],
    userId: "user_123",
    preferences: { theme: "dark" },
});
```

## 常见模式

启用 [短期记忆](#usage) 后，长对话可能会超出 LLM 的上下文窗口。常见的解决方案有：

* **裁剪消息（Trim messages）：** 移除最前或最后 N 条消息（在调用 LLM 之前）。
* **删除消息（Delete messages）：** 从 LangGraph state 中永久删除消息。
* **总结消息（Summarize messages）：** 总结历史中较早的消息，并用摘要替换它们。
* **自定义策略（Custom strategies）：** 自定义策略（例如消息过滤等）。

这使智能体能够跟踪对话，而不会超出 LLM 的上下文窗口。

### 裁剪消息

大多数 LLM 都有一个支持的最大上下文窗口（以 token 计）。

决定何时截断消息的一种方式是统计消息历史中的 token 数，并在接近该限制时截断。如果你使用 LangChain，可以使用裁剪消息工具（trim messages utility），并指定要从列表中保留的 token 数，以及用于处理边界的 `strategy`（例如保留最后的 `maxTokens`）。

要在智能体中裁剪消息历史，请使用 [`createMiddleware`](https://reference.langchain.com/javascript/langchain/index/createMiddleware) 配合 `beforeModel` 钩子：

```typescript
import { RemoveMessage } from "@langchain/core/messages";
import { createAgent, createMiddleware } from "langchain";
import { MemorySaver, REMOVE_ALL_MESSAGES } from "@langchain/langgraph";

const trimMessages = createMiddleware({
  name: "TrimMessages",
  beforeModel: (state) => {
    const messages = state.messages;

    if (messages.length <= 3) {
      return; // No changes needed
    }

    const firstMsg = messages[0];
    const recentMessages =
      messages.length % 2 === 0 ? messages.slice(-3) : messages.slice(-4);
    const newMessages = [firstMsg, ...recentMessages];

    return {
      messages: [
        new RemoveMessage({ id: REMOVE_ALL_MESSAGES }),
        ...newMessages,
      ],
    };
  },
});

const checkpointer = new MemorySaver();
const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [trimMessages],
  checkpointer,
});
```

### 删除消息

你可以从图 state 中删除消息，以管理消息历史。

当你想移除特定消息或清空整个消息历史时，这很有用。

要从图 state 中删除消息，你可以使用 `RemoveMessage`。要让 `RemoveMessage` 生效，你需要使用带有 [`messagesStateReducer`](https://reference.langchain.com/javascript/langchain-langgraph/index/messagesStateReducer) [reducer](/oss/javascript/langgraph/graph-api#reducers) 的 state 键，例如 `MessagesValue`。

删除特定消息：

```typescript
import { RemoveMessage } from "@langchain/core/messages";

const deleteMessages = (state) => {
    const messages = state.messages;
    if (messages.length > 2) {
        // remove the earliest two messages
        return {
        messages: messages
            .slice(0, 2)
            .map((m) => new RemoveMessage({ id: m.id })),
        };
    }
};
```

> 【警告】
> 删除消息时，**务必确保** 结果消息历史是有效的。检查你正在使用的 LLM 提供商的限制。例如：
>
> * 一些提供商期望消息历史以 `user` 消息开头
> * 大多数提供商要求带工具调用的 `assistant` 消息后面必须跟随对应的 `tool` 结果消息。

```typescript
import { RemoveMessage } from "@langchain/core/messages";
import { createAgent, createMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const deleteOldMessages = createMiddleware({
  name: "DeleteOldMessages",
  afterModel: (state) => {
    const messages = state.messages;
    if (messages.length > 2) {
      // remove the earliest two messages
      return {
        messages: messages
          .slice(0, 2)
          .map((m) => new RemoveMessage({ id: m.id! })),
      };
    }
    return;
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [],
  systemPrompt: "Please be concise and to the point.",
  middleware: [deleteOldMessages],
  checkpointer: new MemorySaver(),
});

const config = { configurable: { thread_id: "1" } };

const streamA = await agent.streamEvents(
  { messages: [{ role: "user", content: "hi! I'm bob" }] },
  { ...config, version: "v3" }
);
for await (const snapshot of streamA.values) {
  const messageDetails = snapshot.messages.map((message) => [
    message.getType(),
    message.content,
  ]);
  console.log(messageDetails);
}

const streamB = await agent.streamEvents(
  { messages: [{ role: "user", content: "write a short poem about cats" }] },
  { ...config, version: "v3" }
);
for await (const snapshot of streamB.values) {
  const messageDetails = snapshot.messages.map((message) => [
    message.getType(),
    message.content,
  ]);
  console.log(messageDetails);
}

const streamC = await agent.streamEvents(
  { messages: [{ role: "user", content: "what's my name?" }] },
  { ...config, version: "v3" }
);
for await (const snapshot of streamC.values) {
  const messageDetails = snapshot.messages.map((message) => [
    message.getType(),
    message.content,
  ]);
  console.log(messageDetails);
}
```

```
[["human", "hi! I'm bob"]]
[["human", "hi! I'm bob"], ["ai", "Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code."]]
[["human", "hi! I'm bob"], ["ai", "Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code."], ["human", "write a short poem about cats"]]
[["human", "hi! I'm bob"], ["ai", "Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code."], ["human", "write a short poem about cats"], ["ai", "There once was a cat on a wall, Who barely moved at all..."]]
[["human", "write a short poem about cats"], ["ai", "There once was a cat on a wall, Who barely moved at all..."]]
[["human", "write a short poem about cats"], ["ai", "There once was a cat on a wall, Who barely moved at all..."], ["human", "what's my name?"]]
[["human", "write a short poem about cats"], ["ai", "There once was a cat on a wall, Who barely moved at all..."], ["human", "what's my name?"], ["ai", "I don't know your name - you haven't told me!"]]
[["human", "what's my name?"], ["ai", "I don't know your name - you haven't told me!"]]
```

### 总结消息

如上所示，裁剪或移除消息的问题在于，你可能会因为剔除消息队列而丢失信息。
因此，一些应用受益于一种更复杂的方法：使用聊天模型来总结消息历史。

![Summary](https://mintcdn.com/langchain-5e9cc07a/ybiAaBfoBvFquMDz/oss/images/summary.png?fit=max&auto=format&n=ybiAaBfoBvFquMDz&q=85&s=c8ed3facdccd4ef5c7e52902c72ba938)

要在智能体中总结消息历史，请使用内置的 [`summarizationMiddleware`](/oss/javascript/langchain/middleware#summarization)：

```typescript
import { createAgent, summarizationMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: { tokens: 4000 },
      keep: { messages: 20 },
    }),
  ],
  checkpointer,
});

const config = { configurable: { thread_id: "1" } };
await agent.invoke({ messages: "hi, my name is bob" }, config);
await agent.invoke({ messages: "write a short poem about cats" }, config);
await agent.invoke({ messages: "now do the same but for dogs" }, config);
const finalResponse = await agent.invoke({ messages: "what's my name?" }, config);

console.log(finalResponse.messages.at(-1)?.content);
// Your name is Bob!
```

关于更多配置选项，请参阅 [`summarizationMiddleware`](/oss/javascript/langchain/middleware#summarization)。

## 访问记忆

你可以通过以下几种方式访问和修改智能体的短期记忆（state）：

### 工具

#### 在工具中读取短期记忆

使用 `runtime` 参数（类型为 `ToolRuntime`）在工具中访问短期记忆（state）。

`runtime` 参数对工具签名是隐藏的（因此模型看不到它），但工具可以通过它访问 state。

```typescript
import { createAgent, tool, type ToolRuntime } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const CustomState = new StateSchema({
  userId: z.string(),
});

const getUserInfo = tool(
  async (_, config: ToolRuntime<typeof CustomState.State>) => {
    const userId = config.state.userId;
    return userId === "user_123" ? "John Doe" : "Unknown User";
  },
  {
    name: "get_user_info",
    description: "Get user info",
    schema: z.object({}),
  }
);

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getUserInfo],
  stateSchema: CustomState,
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "what's my name?" }],
    userId: "user_123",
  },
  {
    context: {},
  }
);

console.log(result.messages.at(-1)?.content);
// Outputs: "Your name is John Doe."
```

#### 从工具写入短期记忆

要在执行期间修改智能体的短期记忆（state），你可以直接从工具返回 state 更新。

这对于持久化中间结果，或让信息可供后续工具或提示词访问非常有用。

```typescript
import { tool, createAgent, ToolMessage, type ToolRuntime } from "langchain";
import { Command, StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const CustomState = new StateSchema({
  userId: z.string().optional(),
  userName: z.string().optional(),
});

const updateUserInfo = tool(
  async (_, config: ToolRuntime<typeof CustomState.State>) => {
    const userId = config.state.userId;
    const name = userId === "user_123" ? "John Smith" : "Unknown user";
    return new Command({
      update: {
        userName: name,
        // update the message history
        messages: [
          new ToolMessage({
            content: "Successfully looked up user information",
            tool_call_id: config.toolCall?.id ?? "",
          }),
        ],
      },
    });
  },
  {
    name: "update_user_info",
    description: "Look up and update user info.",
    schema: z.object({}),
  }
);

const greet = tool(
  async (_, config: ToolRuntime<typeof CustomState.State>) => {
    const userName = config.state.userName;
    return `Hello ${userName}!`;
  },
  {
    name: "greet",
    description: "Use this to greet the user once you found their info.",
    schema: z.object({}),
  }
);

const agent = createAgent({
  model: "openai:gpt-5-mini",
  tools: [updateUserInfo, greet],
  stateSchema: CustomState,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "greet the user" }],
  userId: "user_123",
});

console.log(result.messages.at(-1)?.content);
// Output: "Hello John Smith! It's great to meet you. How can I help you today?"
```

### 提示词

在中间件中访问短期记忆（state），以根据对话历史或自定义 state 字段创建动态提示词。

```typescript
import * as z from "zod";
import { createAgent, tool, dynamicSystemPromptMiddleware } from "langchain";

const contextSchema = z.object({
  userName: z.string(),
});
type ContextSchema = z.infer<typeof contextSchema>;

const getWeather = tool(
  async ({ city }) => {
    return `The weather in ${city} is always sunny!`;
  },
  {
    name: "get_weather",
    description: "Get user info",
    schema: z.object({
      city: z.string(),
    }),
  }
);

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [getWeather],
  contextSchema,
  middleware: [
    dynamicSystemPromptMiddleware<ContextSchema>((_, config) => {
      return `You are a helpful assistant. Address the user as ${config.context?.userName}.`;
    }),
  ],
});

const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is the weather in SF?" }],
  },
  {
    context: {
      userName: "John Smith",
    },
  }
);

for (const message of result.messages) {
  console.log(message);
}
/**
 * HumanMessage {
 *   "content": "What is the weather in SF?",
 *   // ...
 * }
 * AIMessage {
 *   // ...
 *   "tool_calls": [
 *     {
 *       "name": "get_weather",
 *       "args": {
 *         "city": "San Francisco"
 *       },
 *       "type": "tool_call",
 *       "id": "call_tCidbv0apTpQpEWb3O2zQ4Yx"
 *     }
 *   ],
 *   // ...
 * }
 * ToolMessage {
 *   "content": "The weather in San Francisco is always sunny!",
 *   "tool_call_id": "call_tCidbv0apTpQpEWb3O2zQ4Yx"
 *   // ...
 * }
 * AIMessage {
 *   "content": "John Smith, here's the latest: The weather in San Francisco is always sunny!\n\nIf you'd like more details (temperature, wind, humidity) or a forecast for the next few days, I can pull that up. What would you like?",
 *   // ...
 * }
 */
```

### 模型之前（Before model）

```mermaid
%%{
    init: {
        "fontFamily": "monospace",
        "flowchart": {
        "curve": "basis"
        }
    }
}%%
graph TD
    S(["`\_\_start\_\_`"])
    PRE(before_model)
    MODEL(model)
    TOOLS(tools)
    END(["`\_\_end\_\_`"])
    S --> PRE
    PRE --> MODEL
    MODEL -.-> TOOLS
    MODEL -.-> END
    TOOLS --> PRE
    classDef blueHighlight fill:#E5F4FF,stroke:#006DDD,color:#030710;
    classDef neutral fill:#F2FAFF,stroke:#40668D,stroke-width:2px,color:#2F4B68;
    class S blueHighlight;
    class END blueHighlight;
    class PRE,MODEL,TOOLS neutral;
```

```typescript
import { RemoveMessage } from "@langchain/core/messages";
import { createAgent, createMiddleware, trimMessages } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import { REMOVE_ALL_MESSAGES } from "@langchain/langgraph";

const trimMessageHistory = createMiddleware({
  name: "TrimMessages",
  beforeModel: async (state) => {
    const trimmed = await trimMessages(state.messages, {
      maxTokens: 384,
      strategy: "last",
      startOn: "human",
      endOn: ["human", "tool"],
      tokenCounter: (msgs) => msgs.length,
    });
    return {
      messages: [new RemoveMessage({ id: REMOVE_ALL_MESSAGES }), ...trimmed],
    };
  },
});

const checkpointer = new MemorySaver();
const agent = createAgent({
  model: "gpt-5-nano",
  tools: [],
  middleware: [trimMessageHistory],
  checkpointer,
});
```

### 模型之后（After model）

```mermaid
%%{
    init: {
        "fontFamily": "monospace",
        "flowchart": {
        "curve": "basis"
        }
    }
}%%
graph TD
    S(["\_\_start\_\_"])
    MODEL(model)
    POST(after_model)
    TOOLS(tools)
    END(["\_\_end\_\_"])
    S --> MODEL
    MODEL --> POST
    POST -.-> END
    POST -.-> TOOLS
    TOOLS --> MODEL
    classDef blueHighlight fill:#E5F4FF,stroke:#006DDD,color:#030710;
    classDef greenHighlight fill:#F6FFDB,stroke:#6E8900,color:#2E3900;
    classDef neutral fill:#F2FAFF,stroke:#40668D,stroke-width:2px,color:#2F4B68;
    class S blueHighlight;
    class END blueHighlight;
    class POST greenHighlight;
    class MODEL,TOOLS neutral;
```

```typescript
import { RemoveMessage } from "@langchain/core/messages";
import { createAgent, createMiddleware } from "langchain";
import { REMOVE_ALL_MESSAGES } from "@langchain/langgraph";

const validateResponse = createMiddleware({
  name: "ValidateResponse",
  afterModel: (state) => {
    const lastMessage = state.messages.at(-1)?.content;
    if (
      typeof lastMessage === "string" &&
      lastMessage.toLowerCase().includes("confidential")
    ) {
      return {
        messages: [
          new RemoveMessage({ id: REMOVE_ALL_MESSAGES }),
        ],
      };
    }
    return;
  },
});

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [],
  middleware: [validateResponse],
});
```

***

> 【提示】
> 通过 MCP 将这些文档连接到 Claude、VSCode 等，以获得实时答案：[连接这些文档](/use-these-docs)。
>
> 在 GitHub 上编辑此页面：https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/short-term-memory.mdx ，或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
