# 智能体中的上下文工程（Context engineering）

> 原文：https://docs.langchain.com/oss/javascript/langchain/context-engineering
> LangChain (TypeScript) 官方文档中文整理

# 智能体中的上下文工程

## 概述

构建智能体（或任何 LLM 应用）最难的部分，是让它们足够可靠。虽然它们可能在原型阶段能正常工作，但在真实用例中往往会失败。

### 智能体为什么会失败？

当智能体失败时，通常是因为智能体内部的 LLM 调用采取了错误的行动 / 没有做我们期望它做的事。LLM 失败的原因有两个：

1. 底层 LLM 能力不足
2. 没有把“正确的”上下文传递给 LLM

更多时候——实际上是第二个原因导致智能体不可靠。

**上下文工程（Context engineering）** 是以正确的格式提供正确的信息和工具，让 LLM 能够完成任务。这是 AI 工程师的首要职责。这种“正确”上下文的缺失是构建更可靠智能体的头号障碍，而 LangChain 的智能体抽象正是为促进上下文工程而专门设计的。

> **【提示】** 刚接触上下文工程？先从[概念概述](/oss/javascript/concepts/context)开始，了解不同类型的上下文以及何时使用它们。

### 智能体循环

一个典型的智能体循环包含两个主要步骤：

1. **模型调用** - 使用提示词和可用工具调用 LLM，返回一个响应或一个执行工具的请求
2. **工具执行** - 执行 LLM 请求的工具，返回工具结果

![Core agent loop diagram](https://mintcdn.com/langchain-5e9cc07a/Tazq8zGc0yYUYrDl/oss/images/core_agent_loop.png?fit=max&auto=format&n=Tazq8zGc0yYUYrDl&q=85&s=ac72e48317a9ced68fd1be64e89ec063)

这个循环会持续进行，直到 LLM 决定结束。

### 你可以控制什么

要构建可靠的智能体，你需要控制智能体循环每一步中发生的事情，以及步骤之间发生的事情。

| 上下文类型                                     | 你控制的内容                                                                   | 瞬态或持久 |
| ---------------------------------------------- | ------------------------------------------------------------------------------ | ---------- |
| **[模型上下文](#model-context)**               | 进入模型调用的内容（指令、消息历史、工具、响应格式）                           | 瞬态       |
| **[工具上下文](#tool-context)**                | 工具可以访问和产生的内容（对状态、存储、运行时上下文的读/写）                  | 持久       |
| **[生命周期上下文](#life-cycle-context)**      | 模型调用与工具调用之间发生的事情（摘要、护栏、日志等）                         | 持久       |

* **瞬态上下文**：LLM 单次调用所看到的内容。你可以修改消息、工具或提示词，而不改变状态中保存的内容。
* **持久上下文**：跨轮次保存在状态中的内容。生命周期钩子和工具写入会永久地修改它。

### 数据源

在整个过程中，你的智能体会访问（读 / 写）不同的数据源：

| 数据源                | 又称           | 作用域         | 示例                                                             |
| --------------------- | -------------- | -------------- | ---------------------------------------------------------------- |
| **运行时上下文**      | 静态配置       | 会话级         | 用户 ID、API 密钥、数据库连接、权限、环境设置                    |
| **状态（State）**     | 短期记忆       | 会话级         | 当前消息、上传的文件、认证状态、工具结果                         |
| **存储（Store）**     | 长期记忆       | 跨会话         | 用户偏好、提取的洞察、记忆、历史数据                             |

### 工作原理

LangChain 的[中间件](/oss/javascript/langchain/middleware)是让上下文工程对 LangChain 开发者变得实用的底层机制。

中间件允许你挂钩智能体生命周期中的任意步骤，并：

* 更新上下文
* 跳转到智能体生命周期中的不同步骤

在本指南中，你会频繁看到使用中间件 API 作为实现上下文工程目标的手段。

## 模型上下文

控制每次模型调用的输入内容——指令、可用工具、使用哪个模型以及输出格式。这些决策直接影响可靠性与成本。

* **[系统提示词（System Prompt）](#system-prompt)**：开发者给 LLM 的基础指令。
* **[消息（Messages）](#messages)**：发送给 LLM 的完整消息列表（对话历史）。
* **[工具（Tools）](#tools)**：智能体可用于采取行动的实用工具。
* **[模型（Model）](#model)**：要调用的实际模型（包括配置）。
* **[响应格式（Response Format）](#response-format)**：模型最终响应的模式（schema）规范。

所有这些模型上下文类型都可以从**状态**（短期记忆）、**存储**（长期记忆）或**运行时上下文**（静态配置）中获取。

### 系统提示词

系统提示词设定 LLM 的行为和能力。不同的用户、上下文或对话阶段需要不同的指令。成功的智能体会利用记忆、偏好和配置，为对话的当前状态提供正确的指令。

#### State（状态）

从状态访问消息数量或对话上下文：

```typescript
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [
    dynamicSystemPromptMiddleware((state) => {
      // Read from State: check conversation length
      const messageCount = state.messages.length;

      let base = "You are a helpful assistant.";

      if (messageCount > 10) {
        base += "\nThis is a long conversation - be extra concise.";
      }

      return base;
    }),
  ],
});
```

#### Store（存储）

从长期记忆访问用户偏好：

```typescript
import * as z from "zod";
import { createAgent, dynamicSystemPromptMiddleware } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

type Context = z.infer<typeof contextSchema>;

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  contextSchema,
  middleware: [
    dynamicSystemPromptMiddleware<Context>(async (state, runtime) => {
      const userId = runtime.context.userId;

      // Read from Store: get user preferences
      const store = runtime.store;
      const userPrefs = await store.get(["preferences"], userId);

      let base = "You are a helpful assistant.";

      if (userPrefs) {
        const style = userPrefs.value?.communicationStyle || "balanced";
        base += `\nUser prefers ${style} responses.`;
      }

      return base;
    }),
  ],
});
```

#### Runtime Context（运行时上下文）

从运行时上下文访问用户 ID 或配置：

```typescript
import * as z from "zod";
import { createAgent, dynamicSystemPromptMiddleware } from "langchain";

const contextSchema = z.object({
  userRole: z.string(),
  deploymentEnv: z.string(),
});

type Context = z.infer<typeof contextSchema>;

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  contextSchema,
  middleware: [
    dynamicSystemPromptMiddleware<Context>((state, runtime) => {
      // Read from Runtime Context: user role and environment
      const userRole = runtime.context.userRole;
      const env = runtime.context.deploymentEnv;

      let base = "You are a helpful assistant.";

      if (userRole === "admin") {
        base += "\nYou have admin access. You can perform all operations.";
      } else if (userRole === "viewer") {
        base += "\nYou have read-only access. Guide users to read operations only.";
      }

      if (env === "production") {
        base += "\nBe extra careful with any data modifications.";
      }

      return base;
    }),
  ],
});
```

### 消息

消息构成了发送给 LLM 的提示词。
管理消息内容至关重要，以确保 LLM 拥有正确的信息来良好地响应。

#### State（状态）

在当前查询相关时，从状态注入已上传文件的上下文：

```typescript
import { createMiddleware } from "langchain";

const injectFileContext = createMiddleware({
  name: "InjectFileContext",
  wrapModelCall: (request, handler) => {
    // request.state is a shortcut for request.state.messages
    const uploadedFiles = request.state.uploadedFiles || [];

    if (uploadedFiles.length > 0) {
      // Build context about available files
      const fileDescriptions = uploadedFiles.map(file =>
        `- ${file.name} (${file.type}): ${file.summary}`
      );

      const fileContext = `Files you have access to in this conversation:
${fileDescriptions.join("\n")}

Reference these files when answering questions.`;

      // Inject file context before recent messages
      const messages = [
        ...request.messages,  // Rest of conversation
        { role: "user", content: fileContext }
      ];
      request = request.override({ messages });
    }

    return handler(request);
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [injectFileContext],
});
```

#### Store（存储）

从存储注入用户的邮件写作风格以指导起草：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

const injectWritingStyle = createMiddleware({
  name: "InjectWritingStyle",
  contextSchema,
  wrapModelCall: async (request, handler) => {
    const userId = request.runtime.context.userId;

    // Read from Store: get user's writing style examples
    const store = request.runtime.store;
    const writingStyle = await store.get(["writing_style"], userId);

    if (writingStyle) {
      const style = writingStyle.value;
      // Build style guide from stored examples
      const styleContext = `Your writing style:
- Tone: ${style.tone || 'professional'}
- Typical greeting: "${style.greeting || 'Hi'}"
- Typical sign-off: "${style.signOff || 'Best'}"
- Example email you've written:
${style.exampleEmail || ''}`;

      // Append at end - models pay more attention to final messages
      const messages = [
        ...request.messages,
        { role: "user", content: styleContext }
      ];
      request = request.override({ messages });
    }

    return handler(request);
  },
});
```

#### Runtime Context（运行时上下文）

根据用户所在司法辖区，从运行时上下文注入合规规则：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

const contextSchema = z.object({
  userJurisdiction: z.string(),
  industry: z.string(),
  complianceFrameworks: z.array(z.string()),
});

type Context = z.infer<typeof contextSchema>;

const injectComplianceRules = createMiddleware<Context>({
  name: "InjectComplianceRules",
  contextSchema,
  wrapModelCall: (request, handler) => {
    // Read from Runtime Context: get compliance requirements
    const { userJurisdiction, industry, complianceFrameworks } = request.runtime.context;

    // Build compliance constraints
    const rules = [];
    if (complianceFrameworks.includes("GDPR")) {
      rules.push("- Must obtain explicit consent before processing personal data");
      rules.push("- Users have right to data deletion");
    }
    if (complianceFrameworks.includes("HIPAA")) {
      rules.push("- Cannot share patient health information without authorization");
      rules.push("- Must use secure, encrypted communication");
    }
    if (industry === "finance") {
      rules.push("- Cannot provide financial advice without proper disclaimers");
    }

    if (rules.length > 0) {
      const complianceContext = `Compliance requirements for ${userJurisdiction}:
${rules.join("\n")}`;

      // Append at end - models pay more attention to final messages
      const messages = [
        ...request.messages,
        { role: "user", content: complianceContext }
      ];
      request = request.override({ messages });
    }

    return handler(request);
  },
});
```

> **【注意】** **瞬态与持久消息更新：**
>
> 上面的示例使用 `wrap_model_call` 进行**瞬态**更新——仅为单次调用修改发送给模型的消息，而不改变状态中保存的内容。
>
> 对于修改状态的**持久**更新，你可以：
>
> * 直接从 `wrapModelCall` 返回一个 [`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command)，从模型调用层注入状态更新。
> * 使用生命周期钩子，如 `beforeModel`、`afterModel` 或 `wrapToolCall`（用于工具返回）来更新对话历史。更多细节请参阅[中间件文档](/oss/javascript/langchain/middleware)。
>
> 更多信息请参阅[状态更新](/oss/javascript/langchain/middleware/custom#state-updates)。

### 工具

工具让模型能够与数据库、API 和外部系统交互。你如何定义和选择工具，直接影响模型能否有效完成任务。

#### 定义工具

每个工具都需要清晰的名称、描述、参数名和参数描述。这些不只是元数据——它们引导模型对何时以及如何使用该工具进行推理。

```typescript
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const searchOrders = tool(
  async ({ userId, status, limit }) => {
    // Implementation here
  },
  {
    name: "search_orders",
    description: `Search for user orders by status.

    Use this when the user asks about order history or wants to check
    order status. Always filter by the provided status.`,
    schema: z.object({
      userId: z.string().describe("Unique identifier for the user"),
      status: z.enum(["pending", "shipped", "delivered"]).describe("Order status to filter by"),
      limit: z.number().default(10).describe("Maximum number of results to return"),
    }),
  }
);
```

#### 选择工具

并非每个工具都适合每种情况。工具太多可能会让模型不堪重负（上下文过载）并增加错误；工具太少则限制能力。动态工具选择会根据认证状态、用户权限、功能开关或对话阶段来调整可用工具集。

##### State（状态）

仅在达到特定对话里程碑后启用高级工具：

```typescript
import { createMiddleware } from "langchain";

const stateBasedTools = createMiddleware({
  name: "StateBasedTools",
  wrapModelCall: (request, handler) => {
    // Read from State: check authentication and conversation length
    const state = request.state;
    const isAuthenticated = state.authenticated || false;
    const messageCount = state.messages.length;

    let filteredTools = request.tools;

    // Only enable sensitive tools after authentication
    if (!isAuthenticated) {
      filteredTools = request.tools.filter(t => t.name.startsWith("public_"));
    } else if (messageCount < 5) {
      filteredTools = request.tools.filter(t => t.name !== "advanced_search");
    }

    return handler({ ...request, tools: filteredTools });
  },
});
```

##### Store（存储）

根据存储中的用户偏好或功能开关过滤工具：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

const storeBasedTools = createMiddleware({
  name: "StoreBasedTools",
  contextSchema,
  wrapModelCall: async (request, handler) => {
    const userId = request.runtime.context.userId;

    // Read from Store: get user's enabled features
    const store = request.runtime.store;
    const featureFlags = await store.get(["features"], userId);

    let filteredTools = request.tools;

    if (featureFlags) {
      const enabledFeatures = featureFlags.value?.enabledTools || [];
      filteredTools = request.tools.filter(t => enabledFeatures.includes(t.name));
    }

    return handler({ ...request, tools: filteredTools });
  },
});
```

##### Runtime Context（运行时上下文）

根据运行时上下文中的用户权限过滤工具：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

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
      filteredTools = request.tools.filter(t => t.name !== "delete_data");
    } else {
      filteredTools = request.tools.filter(t => t.name.startsWith("read_"));
    }

    return handler({ ...request, tools: filteredTools });
  },
});
```

关于既过滤已预注册工具、又在运行时注册工具（例如来自 MCP 服务器），请参阅[动态工具](/oss/javascript/langchain/tools#dynamic-tool-selection)。

### 模型

不同的模型有不同的优势、成本和上下文窗口。为手头的任务选择合适的模型，而这在智能体运行期间可能会发生变化。

#### State（状态）

根据状态中的对话长度使用不同的模型：

```typescript
import { createMiddleware, initChatModel } from "langchain";

// Initialize models once outside the middleware
const largeModel = initChatModel("claude-sonnet-4-6");
const standardModel = initChatModel("gpt-5.5");
const efficientModel = initChatModel("gpt-5.4-mini");

const stateBasedModel = createMiddleware({
  name: "StateBasedModel",
  wrapModelCall: (request, handler) => {
    // request.messages is a shortcut for request.state.messages
    const messageCount = request.messages.length;
    let model;

    if (messageCount > 20) {
      model = largeModel;
    } else if (messageCount > 10) {
      model = standardModel;
    } else {
      model = efficientModel;
    }

    return handler({ ...request, model });
  },
});
```

#### Store（存储）

使用存储中用户偏好的模型：

```typescript
import * as z from "zod";
import { createMiddleware, initChatModel } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

// Initialize available models once
const MODEL_MAP = {
  "gpt-5.5": initChatModel("gpt-5.5"),
  "gpt-5.4-mini": initChatModel("gpt-5.4-mini"),
  "claude-sonnet": initChatModel("claude-sonnet-4-6"),
};

const storeBasedModel = createMiddleware({
  name: "StoreBasedModel",
  contextSchema,
  wrapModelCall: async (request, handler) => {
    const userId = request.runtime.context.userId;

    // Read from Store: get user's preferred model
    const store = request.runtime.store;
    const userPrefs = await store.get(["preferences"], userId);

    let model = request.model;

    if (userPrefs) {
      const preferredModel = userPrefs.value?.preferredModel;
      if (preferredModel && MODEL_MAP[preferredModel]) {
        model = MODEL_MAP[preferredModel];
      }
    }

    return handler({ ...request, model });
  },
});
```

#### Runtime Context（运行时上下文）

根据运行时上下文中的成本限制或环境选择模型：

```typescript
import * as z from "zod";
import { createMiddleware, initChatModel } from "langchain";

const contextSchema = z.object({
  costTier: z.string(),
  environment: z.string(),
});

// Initialize models once outside the middleware
const premiumModel = initChatModel("claude-sonnet-4-6");
const standardModel = initChatModel("gpt-5.5");
const budgetModel = initChatModel("gpt-5.4-mini");

const contextBasedModel = createMiddleware({
  name: "ContextBasedModel",
  contextSchema,
  wrapModelCall: (request, handler) => {
    // Read from Runtime Context: cost tier and environment
    const costTier = request.runtime.context.costTier;
    const environment = request.runtime.context.environment;

    let model;

    if (environment === "production" && costTier === "premium") {
      model = premiumModel;
    } else if (costTier === "budget") {
      model = budgetModel;
    } else {
      model = standardModel;
    }

    return handler({ ...request, model });
  },
});
```

更多示例请参阅[动态模型](/oss/javascript/langchain/models#dynamic-model-selection)。

### 响应格式

结构化输出将非结构化文本转换为经过验证的结构化数据。当提取特定字段或为下游系统返回数据时，自由形式的文本是不够的。

**工作原理：** 当你提供 schema 作为响应格式时，模型的最终响应保证符合该 schema。智能体会运行模型 / 工具调用循环，直到模型不再调用工具，然后最终响应会被强制转换为所提供的格式。

#### 定义格式

schema 定义引导模型。字段名、类型和描述精确指定了输出应遵循的格式。

```typescript
import { z } from "zod";

const customerSupportTicket = z.object({
  category: z.enum(["billing", "technical", "account", "product"]).describe(
    "Issue category"
  ),
  priority: z.enum(["low", "medium", "high", "critical"]).describe(
    "Urgency level"
  ),
  summary: z.string().describe(
    "One-sentence summary of the customer's issue"
  ),
  customerSentiment: z.enum(["frustrated", "neutral", "satisfied"]).describe(
    "Customer's emotional tone"
  ),
}).describe("Structured ticket information extracted from customer message");
```

#### 选择格式

动态响应格式选择会根据用户偏好、对话阶段或角色调整 schema——在早期返回简单格式，随着复杂度增加返回详细格式。

##### State（状态）

根据对话状态配置结构化输出：

```typescript
import { createMiddleware } from "langchain";
import { z } from "zod";

const simpleResponse = z.object({
  answer: z.string().describe("A brief answer"),
});

const detailedResponse = z.object({
  answer: z.string().describe("A detailed answer"),
  reasoning: z.string().describe("Explanation of reasoning"),
  confidence: z.number().describe("Confidence score 0-1"),
});

const stateBasedOutput = createMiddleware({
  name: "StateBasedOutput",
  wrapModelCall: (request, handler) => {
    // request.state is a shortcut for request.state.messages
    const messageCount = request.messages.length;

    let responseFormat;
    if (messageCount < 3) {
      // Early conversation - use simple format
      responseFormat = simpleResponse;
    } else {
      // Established conversation - use detailed format
      responseFormat = detailedResponse;
    }

    return handler({ ...request, responseFormat });
  },
});
```

##### Store（存储）

根据存储中的用户偏好配置输出格式：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

const verboseResponse = z.object({
  answer: z.string().describe("Detailed answer"),
  sources: z.array(z.string()).describe("Sources used"),
});

const conciseResponse = z.object({
  answer: z.string().describe("Brief answer"),
});

const storeBasedOutput = createMiddleware({
  name: "StoreBasedOutput",
  wrapModelCall: async (request, handler) => {
    const userId = request.runtime.context.userId;

    // Read from Store: get user's preferred response style
    const store = request.runtime.store;
    const userPrefs = await store.get(["preferences"], userId);

    const style = userPrefs?.value?.responseStyle || "concise";
    const responseFormat =
      style === "verbose" ? verboseResponse : conciseResponse;

    return handler({
      ...request,
      responseFormat,
    });
  },
});
```

##### Runtime Context（运行时上下文）

根据运行时上下文（如用户角色或环境）配置输出格式：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";

const contextSchema = z.object({
  userRole: z.string(),
  environment: z.string(),
});

const adminResponse = z.object({
  answer: z.string().describe("Answer"),
  debugInfo: z.record(z.any()).describe("Debug information"),
  systemStatus: z.string().describe("System status"),
});

const userResponse = z.object({
  answer: z.string().describe("Answer"),
});

const contextBasedOutput = createMiddleware({
  name: "ContextBasedOutput",
  wrapModelCall: (request, handler) => {
    // Read from Runtime Context: user role and environment
    const userRole = request.runtime.context.userRole;
    const environment = request.runtime.context.environment;

    let responseFormat;
    if (userRole === "admin" && environment === "production") {
      responseFormat = adminResponse;
    } else {
      responseFormat = userResponse;
    }

    return handler({ ...request, responseFormat });
  },
});
```

## 工具上下文

工具的特殊之处在于它们既读取上下文又写入上下文。

在最基本的情况下，当工具执行时，它接收 LLM 的请求参数，并返回一条工具消息。工具完成自己的工作并产生结果。

工具还可以为模型获取重要信息，使其能够执行并完成任务。

### 读取

大多数真实世界的工具需要的不仅仅是 LLM 的参数。它们需要用户 ID 用于数据库查询、API 密钥用于外部服务，或当前会话状态用于做决策。工具从状态、存储和运行时上下文读取来访问这些信息。

#### State（状态）

从状态读取以检查当前会话信息：

```typescript
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";

const checkAuthentication = tool(
  async (_, runtime: ToolRuntime) => {
    // Read from State: check current auth status
    const currentState = runtime.state;
    const isAuthenticated = currentState.authenticated || false;

    if (isAuthenticated) {
      return "User is authenticated";
    } else {
      return "User is not authenticated";
    }
  },
  {
    name: "check_authentication",
    description: "Check if user is authenticated",
    schema: z.object({}),
  }
);
```

#### Store（存储）

从存储读取以访问持久化的用户偏好：

```typescript
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
});

const getPreference = tool(
  async ({ preferenceKey }, runtime: ToolRuntime) => {
    const userId = runtime.context.userId;

    // Read from Store: get existing preferences
    const store = runtime.store;
    const existingPrefs = await store.get(["preferences"], userId);

    if (existingPrefs) {
      const value = existingPrefs.value?.[preferenceKey];
      return value ? `${preferenceKey}: ${value}` : `No preference set for ${preferenceKey}`;
    } else {
      return "No preferences found";
    }
  },
  {
    name: "get_preference",
    description: "Get user preference from Store",
    schema: z.object({
      preferenceKey: z.string(),
    }),
  }
);
```

#### Runtime Context（运行时上下文）

从运行时上下文读取配置，如 API 密钥和用户 ID：

```typescript
import * as z from "zod";
import { tool } from "@langchain/core/tools";
import { createAgent } from "langchain";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
  dbConnection: z.string(),
});

const fetchUserData = tool(
  async ({ query }, runtime: ToolRuntime<any, typeof contextSchema>) => {
    // Read from Runtime Context: get API key and DB connection
    const { userId, apiKey, dbConnection } = runtime.context;

    // Use configuration to fetch data
    const results = await performDatabaseQuery(dbConnection, query, apiKey);

    return `Found ${results.length} results for user ${userId}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data using Runtime Context configuration",
    schema: z.object({
      query: z.string(),
    }),
  }
);

const agent = createAgent({
  model: "gpt-5.5",
  tools: [fetchUserData],
  contextSchema,
});
```

### 写入

工具结果可用于帮助智能体完成给定任务。工具既可以直接向模型返回结果，也可以更新智能体的记忆，使重要上下文可用于后续步骤。

#### State（状态）

使用 Command 写入状态以跟踪会话专属信息：

```typescript
import * as z from "zod";
import { tool } from "@langchain/core/tools";
import { createAgent } from "langchain";
import { Command } from "@langchain/langgraph";

const authenticateUser = tool(
  async ({ password }) => {
    // Perform authentication
    if (password === "correct") {
      // Write to State: mark as authenticated using Command
      return new Command({
        update: { authenticated: true },
      });
    } else {
      return new Command({ update: { authenticated: false } });
    }
  },
  {
    name: "authenticate_user",
    description: "Authenticate user and update State",
    schema: z.object({
      password: z.string(),
    }),
  }
);
```

#### Store（存储）

写入存储以跨会话持久化数据：

```typescript
import * as z from "zod";
import { createAgent, tool, type ToolRuntime } from "langchain";

const savePreference = tool(
  async ({ preferenceKey, preferenceValue }, runtime: ToolRuntime<any, typeof contextSchema>) => {
    const userId = runtime.context.userId;

    // Read existing preferences
    const store = runtime.store;
    const existingPrefs = await store.get(["preferences"], userId);

    // Merge with new preference
    const prefs = existingPrefs?.value || {};
    prefs[preferenceKey] = preferenceValue;

    // Write to Store: save updated preferences
    await store.put(["preferences"], userId, prefs);

    return `Saved preference: ${preferenceKey} = ${preferenceValue}`;
  },
  {
    name: "save_preference",
    description: "Save user preference to Store",
    schema: z.object({
      preferenceKey: z.string(),
      preferenceValue: z.string(),
    }),
  }
);
```

关于在工具中访问状态、存储和运行时上下文的全面示例，请参阅[工具](/oss/javascript/langchain/tools)。

## 生命周期上下文

控制**核心智能体步骤之间**发生的事情——拦截数据流以实现横切关注点，如摘要、护栏和日志。

正如你在[模型上下文](#model-context)和[工具上下文](#tool-context)中所看到的，[中间件](/oss/javascript/langchain/middleware)是让上下文工程变得实用的机制。中间件允许你挂钩智能体生命周期中的任意步骤，并可以：

1. **更新上下文** - 修改状态和存储以持久化变更、更新对话历史，或保存洞察
2. **在生命周期中跳转** - 根据上下文移动到智能体循环中的不同步骤（例如，若满足某条件则跳过工具执行，用修改后的上下文重复模型调用）

![Middleware hooks in the agent loop](https://mintcdn.com/langchain-5e9cc07a/RAP6mjwE5G00xYsA/oss/images/middleware_final.png?fit=max&auto=format&n=RAP6mjwE5G00xYsA&q=85&s=eb4404b137edec6f6f0c8ccb8323eaf1)

### 示例：摘要

最常见的生命周期模式之一是：当对话历史变得过长时自动压缩它。与[模型上下文](#messages)中展示的瞬态消息裁剪不同，摘要会**持久地更新状态**——永久地用摘要替换旧消息，并保存以供所有未来轮次使用。

LangChain 为此提供了内置中间件：

```typescript
import { createAgent, summarizationMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: { tokens: 4000 },
      keep: { messages: 20 },
    }),
  ],
});
```

当对话超过 token 限制时，`SummarizationMiddleware` 会自动：

1. 使用单独的 LLM 调用对较早的消息进行摘要
2. 在状态中（永久地）用一条摘要消息替换它们
3. 保持近期消息完整以提供上下文

被摘要的对话历史会被永久更新——未来的轮次将看到摘要而不是原始消息。

> **【注意】** 关于内置中间件的完整列表、可用钩子以及如何创建自定义中间件，请参阅[中间件文档](/oss/javascript/langchain/middleware)。

## 最佳实践

1. **从简单开始** - 从静态提示词和工具开始，仅在需要时添加动态特性
2. **增量测试** - 一次添加一个上下文工程特性
3. **监控性能** - 跟踪模型调用、token 使用量和延迟
4. **使用内置中间件** - 善用 [`SummarizationMiddleware`](/oss/javascript/langchain/middleware#summarization)、[`LLMToolSelectorMiddleware`](/oss/javascript/langchain/middleware#llm-tool-selector) 等
5. **记录你的上下文策略** - 清楚说明正在传递什么上下文以及为什么
6. **理解瞬态与持久**：模型上下文变更是瞬态的（每次调用），而生命周期上下文变更会持久化到状态

## 相关资源

* [上下文概念概述](/oss/javascript/concepts/context) - 了解上下文类型以及何时使用它们
* [中间件](/oss/javascript/langchain/middleware) - 完整的中间件指南
* [工具](/oss/javascript/langchain/tools) - 工具创建与上下文访问
* [记忆](/oss/javascript/concepts/memory) - 短期和长期记忆模式
* [智能体](/oss/javascript/langchain/agents) - 核心智能体概念
