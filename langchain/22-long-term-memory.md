# 长期记忆

> 原文：https://docs.langchain.com/oss/javascript/langchain/long-term-memory
> LangChain (TypeScript) 官方文档中文整理

> **文档索引**
> 获取完整文档索引：https://docs.langchain.com/llms.txt
> 使用该文件可在进一步探索前发现所有可用页面。

> 为 LangChain 智能体添加长期记忆，以跨对话和会话存储和回忆数据。

长期记忆让你的智能体能够跨不同的对话和会话存储和回忆信息。
与[短期记忆](/oss/javascript/langchain/short-term-memory)（作用域限定在单个线程）不同，长期记忆跨线程持久存在，可以随时被回忆。

长期记忆构建在 [LangGraph 存储](/oss/javascript/langgraph/stores)之上，它将以命名空间和键组织的 JSON 文档保存数据。

## 用法

要为智能体添加长期记忆，创建一个存储并将其传递给 [`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent)：

### InMemoryStore

#### Google

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  store,
});
```

#### OpenAI

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  store,
});
```

#### Anthropic

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  store,
});
```

#### OpenRouter

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  store,
});
```

#### Fireworks

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  store,
});
```

#### Baseten

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  store,
});
```

#### Ollama

```ts
import { createAgent } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore();

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  store,
});
```

### PostgreSQL

```shell
npm install @langchain/langgraph-checkpoint-postgres
```

#### Google

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [],
  store,
});
```

#### OpenAI

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [],
  store,
});
```

#### Anthropic

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [],
  store,
});
```

#### OpenRouter

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [],
  store,
});
```

#### Fireworks

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [],
  store,
});
```

#### Baseten

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [],
  store,
});
```

#### Ollama

```ts
import { createAgent } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [],
  store,
});
```

> **【提示】** 有关其他存储后端（包括 Redis 和 MongoDB），请参阅[存储集成](/oss/javascript/integrations/long-term-memory)列表。有关 MongoDB 演练，请参阅[使用 MongoDB 的长期记忆](/oss/javascript/integrations/memory/mongodb-long-term-memory)。

工具随后可以使用 `runtime.store` 参数从存储中读取和写入。有关示例，请参阅[在工具中读取长期记忆](#read-long-term-memory-in-tools)和[从工具中写入长期记忆](#write-long-term-memory-from-tools)。

> **【提示】** 有关记忆类型（语义、情景、程序性）和写入记忆策略的深入探讨，请参阅[记忆概念指南](/oss/javascript/concepts/memory#long-term-memory)。

## 记忆存储

LangGraph 将长期记忆作为 JSON 文档存储在[存储](/oss/javascript/langgraph/stores)中。

每个记忆都组织在一个自定义 `namespace`（类似于文件夹）和一个独特的 `key`（类似文件名）下。命名空间通常包含用户或组织 ID 或其他标签，以便于组织信息。

这种结构支持记忆的层次化组织。随后通过内容过滤器支持跨命名空间搜索。

### InMemoryStore

```ts
import { InMemoryStore } from "@langchain/langgraph";

const embed = (texts: string[]): number[][] => {
  // Replace with an actual embedding function or LangChain embeddings object
  return texts.map(() => [1.0, 2.0]);
};

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production use.
const store = new InMemoryStore({ index: { embed, dims: 2 } });
const userId = "my-user";
const applicationContext = "chitchat";
const namespace = [userId, applicationContext];

await store.put(namespace, "a-memory", {
  rules: [
    "User likes short, direct language",
    "User only speaks English & TypeScript",
  ],
  "my-key": "my-value",
});

// get the "memory" by ID
const item = await store.get(namespace, "a-memory");

// search for "memories" within this namespace, filtering on content equivalence, sorted by vector similarity
const items = await store.search(namespace, {
  filter: { "my-key": "my-value" },
  query: "language preferences",
});
```

### PostgreSQL

```ts
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const embed = (texts: string[]): number[][] => {
  return texts.map(() => [1.0, 2.0]);
};

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI, {
  index: { embed, dims: 2 },
});
await store.setup();

const userId = "my-user";
const applicationContext = "chitchat";
const namespace = [userId, applicationContext];

await store.put(namespace, "a-memory", {
  rules: [
    "User likes short, direct language",
    "User only speaks English & TypeScript",
  ],
  "my-key": "my-value",
});

const item = await store.get(namespace, "a-memory");
const items = await store.search(namespace, {
  filter: { "my-key": "my-value" },
  query: "language preferences",
});
```

有关记忆存储的更多信息，请参阅[持久化](/oss/javascript/langgraph/stores)指南。

## 在工具中读取长期记忆

### InMemoryStore

#### Google

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### OpenAI

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### Anthropic

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### OpenRouter

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### Fireworks

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### Baseten

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

#### Ollama

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();
const contextSchema = z.object({
  userId: z.string(),
});

// Write sample data to the store using the put method
await store.put(
  ["users"], // Namespace to group related data together (users namespace for user data)
  "user_123", // Key within the namespace (user ID as key)
  {
    name: "John Smith",
    language: "English",
  }, // Data to store for the given user
);

const getUserInfo = tool(
  // Look up user info.
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    // Access the store - same as that provided to `createAgent`
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Retrieve data from store - returns StoreValue object with value and metadata
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [getUserInfo],
  contextSchema,
  // Pass store to agent - enables agent to access store when running tools
  store,
});

// Run the agent
const result = await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);

console.log(result.messages.at(-1)?.content);

/**
 * Outputs:
 * User Information:
 * - **Name:** John Smith
 * - **Language:** English
 */
```

> **【信息】** [查看示例 trace](https://smith.langchain.com/public/e103a798-8829-42a4-9a1a-9e0f8fc04c62/r)：打开此示例的公开 LangSmith 运行。

### PostgreSQL

#### Google

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### OpenAI

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### Anthropic

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### OpenRouter

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### Fireworks

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### Baseten

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

#### Ollama

```ts
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

await store.put(["users"], "user_123", {
  name: "John Smith",
  language: "English",
});

const getUserInfo = tool(
  async (_, runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    const userInfo = await runtime.store.get(["users"], userId);
    return userInfo?.value ? JSON.stringify(userInfo.value) : "Unknown user";
  },
  {
    name: "getUserInfo",
    description: "Look up user info by userId from the store.",
    schema: z.object({}),
  },
);

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [getUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "look up user information" }] },
  { context: { userId: "user_123" } },
);
```

> **【信息】** [查看示例 trace](https://smith.langchain.com/public/e0440c39-4561-4c13-90b3-5b2ff18037ef/r)：打开此示例的公开 LangSmith 运行。

<a id="write-long-term" />

## 从工具中写入长期记忆

### InMemoryStore

#### Google

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### OpenAI

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### Anthropic

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### OpenRouter

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### Fireworks

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### Baseten

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

#### Ollama

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { InMemoryStore } from "@langchain/langgraph";

// InMemoryStore saves data to an in-memory dictionary. Use a DB-backed store in production.
const store = new InMemoryStore();

const contextSchema = z.object({
  userId: z.string(),
});

// Schema defines the structure of user information for the LLM
const UserInfo = z.object({
  name: z.string(),
});

// Tool that allows agent to update user information (useful for chat applications)
const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) {
      throw new Error("userId is required");
    }
    // Store data in the store (namespace, key, data)
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  {
    name: "save_user_info",
    description: "Save user info",
    schema: UserInfo,
  },
);

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

// Run the agent
await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  // userId passed in context to identify whose information is being updated
  { context: { userId: "user_123" } },
);

// You can access the store directly to get the value
const result = await store.get(["users"], "user_123");
console.log(result?.value); // Output: { name: "John Smith" }
```

> **【信息】** [查看示例 trace](https://smith.langchain.com/public/b97eea49-0f7a-439c-b94d-2e5224c2fab4/r)：打开此示例的公开 LangSmith 运行。

### PostgreSQL

#### Google

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### OpenAI

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "openai:gpt-5.5",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### Anthropic

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### OpenRouter

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### Fireworks

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### Baseten

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

#### Ollama

```ts
import * as z from "zod";
import { tool, createAgent, type ToolRuntime } from "langchain";
import { PostgresStore } from "@langchain/langgraph-checkpoint-postgres/store";

const DB_URI =
  process.env.POSTGRES_URI ??
  "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const store = PostgresStore.fromConnString(DB_URI);
await store.setup();

const contextSchema = z.object({ userId: z.string() });

const UserInfo = z.object({ name: z.string() });

const saveUserInfo = tool(
  async (
    userInfo: z.infer<typeof UserInfo>,
    runtime: ToolRuntime<unknown, z.infer<typeof contextSchema>>,
  ) => {
    const userId = runtime.context.userId;
    if (!userId) throw new Error("userId is required");
    await runtime.store.put(["users"], userId, userInfo);
    return "Successfully saved user info.";
  },
  { name: "save_user_info", description: "Save user info", schema: UserInfo },
);

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [saveUserInfo],
  contextSchema,
  store,
});

await agent.invoke(
  { messages: [{ role: "user", content: "My name is John Smith" }] },
  { context: { userId: "user_123" } },
);

const result = await store.get(["users"], "user_123");
console.log(result?.value);
```

> **【信息】** [查看示例 trace](https://smith.langchain.com/public/564f79cc-a377-4016-a8d3-9f5752eab516/r)：打开此示例的公开 LangSmith 运行。

***

> **【信息】** [通过 MCP 将这些文档连接](/use-these-docs)到 Claude、VSCode 等，以获得实时答案。
>
> **【信息】** [在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/long-term-memory.mdx)或[提交问题](https://github.com/langchain-ai/docs/issues/new/choose)。
