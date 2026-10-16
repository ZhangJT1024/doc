# 运行时（Runtime）

> 原文：https://docs.langchain.com/oss/javascript/langchain/runtime
> LangChain (TypeScript) 官方文档中文整理

# Runtime（运行时）

## 概述

LangChain 的 `createAgent` 底层运行在 LangGraph 的运行时之上。

LangGraph 暴露了一个 [`Runtime`](https://reference.langchain.com/javascript/langchain/index/Runtime) 对象，包含以下信息：

1. **Context（上下文）**：静态信息，如用户 id、数据库连接，或智能体调用的其他依赖项
2. **Store（存储）**：一个 [BaseStore](https://reference.langchain.com/javascript/langchain-core/stores/BaseStore) 实例，用于[长期记忆](/oss/javascript/langchain/long-term-memory)
3. **Stream writer（流写入器）**：用于通过 `"custom"` 流模式流式传输信息的对象
4. **Execution info（执行信息）**：当前执行的身份与重试信息（线程 ID、运行 ID、尝试次数）
5. **Server info（服务器信息）**：在 LangGraph Server 上运行时的服务器专属元数据（助手 ID、图 ID、已认证用户）

> **【提示】** 运行时上下文是你在智能体中传递数据的方式。与其把东西存在全局状态里，你可以把值——例如数据库连接、用户会话或配置——附加到上下文上，并在工具和中间件内部访问它们。这样能保持无状态、可测试且可复用。

你可以在[工具](#inside-tools)和[中间件](#inside-middleware)中访问运行时信息。

## 访问

使用 `createAgent` 创建智能体时，你可以指定 `contextSchema` 来定义存储在智能体 [`Runtime`](https://reference.langchain.com/javascript/langchain/index/Runtime) 中的 `context` 结构。

调用智能体时，传入 `context` 参数并提供该次运行的相关配置：

```ts
import * as z from "zod";
import { createAgent } from "langchain";

const contextSchema = z.object({
  userName: z.string(),
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [
    /* ... */
  ],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  { context: { userName: "John Smith" } }
);
```

### 在工具内部

你可以在工具内部访问运行时信息以：

* 访问上下文
* 读取或写入长期记忆
* 写入[自定义流](/oss/javascript/langchain/streaming#custom-updates)（例如，工具进度 / 更新）

使用 `runtime` 参数在工具内部访问 [`Runtime`](https://reference.langchain.com/javascript/langchain/index/Runtime) 对象。

```ts
import * as z from "zod";
import { tool } from "langchain";
import { type ToolRuntime } from "@langchain/core/tools";

const contextSchema = z.object({
  userName: z.string(),
});

const fetchUserEmailPreferences = tool(
  async (_, runtime: ToolRuntime<any, typeof contextSchema>) => {
    const userName = runtime.context?.userName;
    if (!userName) {
      throw new Error("userName is required");
    }

    let preferences = "The user prefers you to write a brief and polite email.";
    if (runtime.store) {
      const memory = await runtime.store?.get(["users"], userName);
      if (memory) {
        preferences = memory.value.preferences;
      }
    }
    return preferences;
  },
  {
    name: "fetch_user_email_preferences",
    description: "Fetch the user's email preferences.",
    schema: z.object({}),
  }
);
```

### 工具内部的执行信息与服务器信息

通过 `runtime.executionInfo` 访问执行身份（线程 ID、运行 ID），并在 LangGraph Server 上运行时通过 `runtime.serverInfo` 访问服务器专属元数据（助手 ID、已认证用户）：

```ts
import { tool } from "langchain";
import * as z from "zod";

const contextAwareTool = tool(
  async (_input, runtime) => {
    // Access thread and run IDs
    const info = runtime.executionInfo;
    console.log(`Thread: ${info.threadId}, Run: ${info.runId}`);

    // Access server info (only available on LangGraph Server)
    const server = runtime.serverInfo;
    if (server != null) {
      console.log(`Assistant: ${server.assistantId}`);
      if (server.user != null) {
        console.log(`User: ${server.user.identity}`);
      }
    }

    return "done";
  },
  {
    name: "context_aware_tool",
    description: "A tool that uses execution and server info.",
    schema: z.object({}),
  }
);
```

当不在 LangGraph Server 上运行时（例如在本地开发期间），`serverInfo` 为 `null`。

> **【注意】** `runtime.executionInfo` 和 `runtime.serverInfo` 需要 `deepagents>=1.9.0`（或 `@langchain/langgraph>=1.2.8`）。

### 在中间件内部

你可以在中间件中访问运行时信息，以创建动态提示、修改消息，或根据用户上下文控制智能体行为。

使用 `runtime` 参数在中间件内部访问 [`Runtime`](https://reference.langchain.com/javascript/langchain/index/Runtime) 对象。

```ts
import * as z from "zod";
import { createAgent, createMiddleware, SystemMessage } from "langchain";

const contextSchema = z.object({
  userName: z.string(),
});

// Dynamic prompt middleware
const dynamicPromptMiddleware = createMiddleware({
  name: "DynamicPrompt",
  contextSchema,
  beforeModel: (state, runtime) => {
    const userName = runtime.context?.userName;
    if (!userName) {
      throw new Error("userName is required");
    }

    const systemMsg = `You are a helpful assistant. Address the user as ${userName}.`;
    return {
      messages: [new SystemMessage(systemMsg), ...state.messages],
    };
  },
});

// Logging middleware
const loggingMiddleware = createMiddleware({
  name: "Logging",
  contextSchema,
  beforeModel: (state, runtime) => {
    console.log(`Processing request for user: ${runtime.context?.userName}`);
    return;
  },
  afterModel: (state, runtime) => {
    console.log(`Completed request for user: ${runtime.context?.userName}`);
    return;
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [
    /* ... */
  ],
  middleware: [dynamicPromptMiddleware, loggingMiddleware],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "What's my name?" }] },
  { context: { userName: "John Smith" } }
);
```

### 中间件内部的执行信息与服务器信息

中间件钩子也可以访问 `runtime.executionInfo` 和 `runtime.serverInfo`：

```ts
import { createMiddleware } from "langchain";

const authGate = createMiddleware({
  name: "AuthGate",
  beforeModel: (state, runtime) => {
    const server = runtime.serverInfo;
    if (server != null && server.user == null) {
      throw new Error("Authentication required");
    }
    console.log(`Thread: ${runtime.executionInfo.threadId}`);
    return;
  },
});
```

> **【注意】** 需要 `deepagents>=1.9.0`（或 `@langchain/langgraph>=1.2.8`）。
