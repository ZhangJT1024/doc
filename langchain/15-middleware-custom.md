# 自定义中间件

> 原文：https://docs.langchain.com/oss/javascript/langchain/middleware/custom
> LangChain (TypeScript) 官方文档中文整理

通过实现钩子（hook）来构建自定义中间件，这些钩子会在智能体执行流程的特定位置运行。

## 钩子

中间件提供两种风格的钩子来拦截智能体执行：

* **节点风格钩子（Node-style hooks）** —— 在特定执行点按顺序运行：`#node-style-hooks`
* **包裹风格钩子（Wrap-style hooks）** —— 围绕每次模型或工具调用运行：`#wrap-style-hooks`

### 节点风格钩子

在特定执行点按顺序运行。用于日志、验证和状态更新。

选择你的中间件所需的钩子。你可以在节点风格钩子和包裹风格钩子之间进行选择。

**节点风格钩子**在特定执行点运行：

| 钩子          | 运行时机                                    |
| ------------- | ------------------------------------------- |
| `beforeAgent` | 智能体开始之前（每次调用一次）              |
| `beforeModel` | 每次模型调用之前                            |
| `afterModel`  | 每次模型响应之后                            |
| `afterAgent`  | 智能体完成之后（每次调用一次）              |

**包裹风格钩子**围绕每次调用运行，让你可以控制执行：

| 钩子            | 运行时机           |
| --------------- | ------------------ |
| `wrapModelCall` | 围绕每次模型调用   |
| `wrapToolCall`  | 围绕每次工具调用   |

**示例：**

```typescript
import { createMiddleware, AIMessage } from "langchain";

const createMessageLimitMiddleware = (maxMessages: number = 50) => {
  return createMiddleware({
    name: "MessageLimitMiddleware",
    beforeModel: {
      canJumpTo: ["end"],
      hook: (state) => {
        if (state.messages.length === maxMessages) {
          return {
            messages: [new AIMessage("Conversation limit reached.")],
            jumpTo: "end",
          };
        }
        return;
      }
    },
    afterModel: (state) => {
      const lastMessage = state.messages[state.messages.length - 1];
      console.log(`Model returned: ${lastMessage.content}`);
      return;
    },
  });
};
```

### 包裹风格钩子

拦截执行并控制处理程序（handler）何时被调用。用于重试、缓存和转换。

由你决定处理程序被调用零次（短路）、一次（正常流程）还是多次（重试逻辑）。

**可用钩子：**

* `wrapModelCall` —— 围绕每次模型调用
* `wrapToolCall` —— 围绕每次工具调用

**示例：**

```typescript
import { createMiddleware } from "langchain";

const createRetryMiddleware = (maxRetries: number = 3) => {
  return createMiddleware({
    name: "RetryMiddleware",
    wrapModelCall: (request, handler) => {
      for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
          return handler(request);
        } catch (e) {
          if (attempt === maxRetries - 1) {
            throw e;
          }
          console.log(`Retry ${attempt + 1}/${maxRetries} after error: ${e}`);
        }
      }
      throw new Error("Unreachable");
    },
  });
};
```

## 状态更新

节点风格钩子和包裹风格钩子都可以更新智能体状态。但机制不同：

* **节点风格钩子**（`beforeAgent`、`beforeModel`、`afterModel`、`afterAgent`）：直接返回一个字典。该字典会使用图的 reducer 应用到智能体状态。
* **包裹风格钩子**（`wrapModelCall`、`wrapToolCall`）：对于模型调用，直接返回一个 [`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command)，以便在模型响应的同时注入状态更新。对于工具调用，直接返回一个 [`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command)。当你需要根据模型或工具调用期间运行的逻辑来跟踪或更新状态时使用它们，例如摘要触发点、使用量元数据，或根据请求或响应计算出的自定义字段。

### 节点风格钩子

从节点风格钩子返回一个字典，以将更新合并到智能体状态。字典的键映射到状态字段。

```typescript
import { createMiddleware } from "langchain";
import * as z from "zod";

const trackingStateSchema = z.object({
  modelCallCount: z.number().default(0),
});

const incrementAfterModel = createMiddleware({
  name: "incrementAfterModel",
  stateSchema: trackingStateSchema,
  afterModel: (state) => {
    return { modelCallCount: state.modelCallCount + 1 };
  },
});
```

### 包裹风格钩子

从 `wrapModelCall` 直接返回一个 [`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command)，以从模型调用层注入状态更新：

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";
import { Command } from "@langchain/langgraph";

const usageTrackingStateSchema = z.object({
  lastModelCallTokens: z.number().optional(),
});

const trackUsage = createMiddleware({
  name: "trackUsage",
  stateSchema: usageTrackingStateSchema,
  wrapModelCall: async (request, handler) => {
    const response = await handler(request);
    return new Command({ update: { lastModelCallTokens: 150 } });
  },
});
```

[`Command`](https://reference.langchain.com/javascript/langchain-langgraph/index/Command) 会流经图的 reducer，因此更新会被正确应用，并且消息是追加的，而不是替换现有状态。

#### 多个中间件的组合

当多个中间件层返回响应时，框架会传递最后产生的 `AIMessage`：

* **AIMessage 会向下流动：** 每个中间件的 `handler()` 会接收来自上一层的 `AIMessage`。当某个中间件返回 `AIMessage` 时，它就成为下一个中间件 handler 的输入。
* **不含消息更新的 Command 是直通的：** 如果某个中间件返回的 `Command` 的状态更新不涉及 `messages`，框架会将其视为对消息流的无操作。下一个中间件的 handler 会接收来自返回该 Command 的那个中间件**之前**的中间件的 `AIMessage`。
* **Reducer 行为与重试安全：** Command 仍会通过 reducer 应用（消息追加，冲突时外层优先）。重试逻辑会丢弃更早调用产生的 command。

```typescript
import * as z from "zod";
import { createMiddleware } from "langchain";
import { Command, StateSchema, ReducedValue } from "@langchain/langgraph";
import { AIMessage, SystemMessage } from "@langchain/core/messages";

/** Last-wins reducer: when both middleware write, outer overwrites inner. */
const customMiddlewareStateSchema = new StateSchema({
  traceLayer: new ReducedValue(
    z.string().optional(),
    { reducer: (a, b) => b },
  ),
});

const outerMiddleware = createMiddleware({
  name: "OuterMiddleware",
  stateSchema: customMiddlewareStateSchema,
  wrapModelCall: async (_request, handler) => {
    await handler(_request);
    return new Command({
      update: {
        traceLayer: "outer",
        messages: [new SystemMessage({ content: "[Outer ran]" })],
      },
    });
  },
});

const innerMiddleware = createMiddleware({
  name: "InnerMiddleware",
  stateSchema: customMiddlewareStateSchema,
  wrapModelCall: async (_request, handler) => {
    await handler(_request);
    return new Command({
      update: {
        traceLayer: "inner",
        messages: [new SystemMessage({ content: "[Inner ran]" })],
      },
    });
  },
});
```

## 创建中间件

### 使用类（Python）

`AgentMiddleware` 子类可以声明三个类属性，智能体工厂会在编译时读取它们：

* `state_schema` —— 使用自定义字段扩展智能体状态。参见[自定义状态 schema](#custom-state-schema)。
* `tools` —— 注册随中间件一起提供的额外工具（例如待办事项列表中间件上的 `write_todos`）。
* `transformers` —— 注册作用域感知的流转换器工厂。参见[自定义流转换器](#custom-stream-transformers)。

```python
from langchain.agents.middleware import (
    AgentMiddleware,
    AgentState,
    ModelRequest,
    ModelResponse,
)
from langgraph.runtime import Runtime
from typing import Any, Callable

class LoggingMiddleware(AgentMiddleware):
    def before_model(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        print(f"About to call model with {len(state['messages'])} messages")
        return None

    def after_model(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        print(f"Model returned: {state['messages'][-1].content}")
        return None

    async def abefore_model(
        self, state: AgentState, runtime: Runtime
    ) -> dict[str, Any] | None:
        # Async version of before_model
        return None

    async def aafter_model(
        self, state: AgentState, runtime: Runtime
    ) -> dict[str, Any] | None:
        # Async version of after_model
        print(f"Model returned: {state['messages'][-1].content}")
        return None


agent = create_agent(
    model="gpt-5.5",
    middleware=[LoggingMiddleware()],
    tools=[...],
)
```

**何时使用类：**

* 为同一个钩子同时定义同步和异步实现
* 单个中间件中需要多个钩子
* 需要复杂配置（例如可配置的阈值、自定义模型）
* 通过初始化时配置在多个项目之间复用

### 使用 createMiddleware（TypeScript）

`createMiddleware` 接受三个配置字段，智能体工厂会在编译时读取它们：

* `stateSchema` —— 使用自定义字段扩展智能体状态。参见[自定义状态 schema](#custom-state-schema)。
* `tools` —— 注册随中间件一起提供的额外工具。
* `streamTransformers` —— 注册作用域感知的流转换器工厂。参见[自定义流转换器](#custom-stream-transformers)。

**示例：**

```typescript
import { createMiddleware } from "langchain";

const loggingMiddleware = createMiddleware({
  name: "LoggingMiddleware",
  beforeModel: (state) => {
    console.log(`About to call model with ${state.messages.length} messages`);
    return;
  },
  afterModel: (state) => {
    const lastMessage = state.messages[state.messages.length - 1];
    console.log(`Model returned: ${lastMessage.content}`);
    return;
  },
});
```

## 自定义状态 schema

如果你的中间件需要跨钩子跟踪状态，中间件可以用自定义属性扩展智能体的状态。这使中间件能够：

* **跨执行跟踪状态**：维护在智能体整个执行生命周期中持续存在的计数器、标志或其他值

* **在钩子之间共享数据**：将信息从 `beforeModel` 传递到 `afterModel`，或在不同的中间件实例之间传递

* **实现横切关注点**：在不修改核心智能体逻辑的情况下添加速率限制、使用量跟踪、用户上下文或审计日志等功能

* **做出条件决策**：使用累积的状态来决定是否继续执行、跳转到不同节点，或动态修改行为

```typescript
import { createMiddleware, createAgent, HumanMessage } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const CustomState = new StateSchema({
  modelCallCount: z.number().default(0),
  userId: z.string().optional(),
});

const callCounterMiddleware = createMiddleware({
  name: "CallCounterMiddleware",
  stateSchema: CustomState,
  beforeModel: {
    canJumpTo: ["end"],
    hook: (state) => {
      if (state.modelCallCount > 10) {
        return { jumpTo: "end" };
      }

      return;
    },
  },
  afterModel: (state) => {
    return { modelCallCount: state.modelCallCount + 1 };
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [callCounterMiddleware],
});

const result = await agent.invoke({
  messages: [new HumanMessage("Hello")],
  modelCallCount: 0,
  userId: "user-123",
});
```

状态字段可以是公开的或私有的。以下划线（`_`）开头的字段被视为私有，不会包含在智能体的结果中。只有公开字段（不带前导下划线的字段）会被返回。

这对于存储不应暴露给调用方的内部中间件状态（例如临时跟踪变量或内部标志）很有用：

```typescript
import { StateSchema } from "@langchain/langgraph";
import * as z from "zod";

const PrivateState = new StateSchema({
  // Public field - included in invoke result
  publicCounter: z.number().default(0),
  // Private field - excluded from invoke result
  _internalFlag: z.boolean().default(false),
});

const middleware = createMiddleware({
  name: "ExampleMiddleware",
  stateSchema: PrivateState,
  afterModel: (state) => {
    // Both fields are accessible during execution
    if (state._internalFlag) {
      return { publicCounter: state.publicCounter + 1 };
    }
    return { _internalFlag: true };
  },
});

const result = await agent.invoke({
  messages: [new HumanMessage("Hello")],
  publicCounter: 0
});

// result only contains publicCounter, not _internalFlag
console.log(result.publicCounter); // 1
console.log(result._internalFlag); // undefined
```

## 自定义流转换器

> 【提示】中间件注册的转换器需要 `langchain@1.4.3` 或更高版本。

中间件可以注册流转换器工厂，将来自实时智能体流的事件投影到带类型的扩展通道上。这对于呈现计数器、侧信道产物、部分输出或线路级脱敏很有用，而无需与框架的内置投影耦合。

在编译时，中间件注册的工厂会与调用方直接传给智能体工厂的任何内容合并。[最终排序规则](/oss/javascript/langchain/event-streaming#register-transformers-on-middleware)会保持内置的 `ToolCallTransformer` 在最前面，并让调用方提供的条目排在最后。

将 `streamTransformers` 作为工厂元组传给 `createMiddleware`。每个工厂的形状为 `() => StreamTransformer<any>`（零参数），并在每个作用域中调用一次；每次调用返回一个新的转换器可以让每个子图保持隔离。

```typescript
import { createAgent, createMiddleware } from "langchain";

const toolActivityMiddleware = createMiddleware({
  name: "ToolActivityMiddleware",
  streamTransformers: [toolActivityTransformer],
});

const agent = createAgent({
  model: "gpt-5-nano",
  tools: [...],
  middleware: [toolActivityMiddleware],
});
```

完整的排序规则和 PII 脱敏示例请参见[在中间件上注册转换器](/oss/javascript/langchain/event-streaming#register-transformers-on-middleware)。

## 自定义上下文

中间件可以定义自定义的上下文 schema，以访问每次调用的元数据。与状态不同，上下文是只读的，并且不会在调用之间持久化。这使其非常适合：

* **用户信息**：传递在执行期间不会改变的用户 ID、角色或偏好
* **配置覆盖**：提供每次调用的设置，例如速率限制或功能标志
* **租户/工作区上下文**：为多租户应用包含特定组织的数据
* **请求元数据**：传递中间件所需的请求 ID、API key 或其他元数据

使用 Zod 定义上下文 schema，并通过中间件钩子中的 `runtime.context` 访问它。上下文 schema 中的必填字段会在 TypeScript 层面强制校验，确保你在调用 `agent.invoke()` 时必须提供它们。

```typescript
import { createAgent, createMiddleware, HumanMessage } from "langchain";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  tenantId: z.string(),
  apiKey: z.string().optional(),
});

const userContextMiddleware = createMiddleware({
  name: "UserContextMiddleware",
  contextSchema,
  wrapModelCall: (request, handler) => {
    // Access context from runtime
    const { userId, tenantId } = request.runtime.context;

    // Add user context to system message
    const contextText = `User ID: ${userId}, Tenant: ${tenantId}`;
    const newSystemMessage = request.systemMessage.concat(contextText);

    return handler({
      ...request,
      systemMessage: newSystemMessage,
    });
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  middleware: [userContextMiddleware],
  tools: [],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [new HumanMessage("Hello")] },
  // Required fields (userId, tenantId) must be provided
  {
    context: {
      userId: "user-123",
      tenantId: "acme-corp",
    },
  }
);
```

**必填上下文字段**：当你在 `contextSchema` 中定义必填字段（不带 `.optional()` 或 `.default()` 的字段）时，TypeScript 会强制要求这些字段必须在 `agent.invoke()` 调用期间提供。这确保了类型安全，并防止因缺少必需上下文而导致的运行时错误。

```typescript
// This will cause a TypeScript error if userId or tenantId are missing
const result = await agent.invoke(
  { messages: [new HumanMessage("Hello")] },
  { context: { userId: "user-123" } } // Error: tenantId is required
);
```

## 执行顺序

使用多个中间件时，请理解它们的执行方式：

```typescript
const agent = createAgent({
  model: "gpt-5.5",
  middleware: [middleware1, middleware2, middleware3],
  tools: [...],
});
```

#### 执行流程

**Before 钩子按顺序运行：**

1. `middleware1.before_agent()`
2. `middleware2.before_agent()`
3. `middleware3.before_agent()`

**智能体循环开始**

4. `middleware1.before_model()`
5. `middleware2.before_model()`
6. `middleware3.before_model()`

**Wrap 钩子像函数调用一样嵌套：**

7. `middleware1.wrap_model_call()` → `middleware2.wrap_model_call()` → `middleware3.wrap_model_call()` → model

**After 钩子按相反顺序运行：**

8. `middleware3.after_model()`
9. `middleware2.after_model()`
10. `middleware1.after_model()`

**智能体循环结束**

11. `middleware3.after_agent()`
12. `middleware2.after_agent()`
13. `middleware1.after_agent()`

**关键规则：**

* `before_*` 钩子：从第一个到最后一个
* `after_*` 钩子：从最后一个到第一个（相反顺序）
* `wrap_*` 钩子：嵌套（第一个中间件包裹所有其他中间件）

## 智能体跳转

要从中间件提前退出，返回一个包含 `jump_to` 的字典：

**可用的跳转目标：**

* `'end'`：跳转到智能体执行的末尾（或第一个 `after_agent` 钩子）
* `'tools'`：跳转到工具节点
* `'model'`：跳转到模型节点（或第一个 `before_model` 钩子）

```typescript
import { createAgent, createMiddleware, AIMessage } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  middleware: [
    createMiddleware({
      name: "BlockedContentMiddleware",
      beforeModel: {
        canJumpTo: ["end"],
        hook: (state) => {
          if (state.messages.at(-1)?.content.includes("BLOCKED")) {
            return {
              messages: [new AIMessage("I cannot respond to that request.")],
              jumpTo: "end" as const,
            };
          }
          return;
        },
      },
    }),
  ],
});

const result = await agent.invoke({
    messages: "Hello, world! BLOCKED"
});

/**
 * Expected output:
 * I cannot respond to that request.
 */
console.log(result.messages.at(-1)?.content);
```

## 最佳实践

1. 保持中间件专注 —— 每个中间件应只做好一件事
2. 优雅地处理错误 —— 不要让中间件错误导致智能体崩溃
3. **使用合适的钩子类型**：
   * 节点风格用于顺序逻辑（日志、验证）
   * 包裹风格用于控制流（重试、回退、缓存）
4. 清楚地记录任何自定义状态属性
5. 在集成之前独立地对中间件进行单元测试
6. 考虑执行顺序 —— 将关键中间件放在列表的最前面
7. 尽可能使用内置中间件

## 示例

### 动态提示词

在运行时动态修改系统提示词，以在每次模型调用之前注入上下文、用户特定指令或其他信息。这是最常见的中间件用例之一。

使用 `ModelRequest` 中的 `systemMessage` 字段来读取和修改系统提示词。它包含一个 [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) 对象（即使智能体是用字符串 [`systemPrompt`](https://reference.langchain.com/javascript/types/langchain.index.CreateAgentParams.html#systemprompt) 创建的）。

#### Google

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "google-genai:gemini-3.6-flash",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### OpenAI

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "openai:gpt-5.5",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### Anthropic

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-6",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### OpenRouter

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "openrouter:openrouter:z-ai/glm-5.2",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### Fireworks

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### Baseten

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

#### Ollama

```ts
import { createMiddleware, SystemMessage, createAgent } from "langchain";

const addContextMiddleware = createMiddleware({
  name: "AddContextMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

const agent = createAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt: "You are a helpful assistant.",
  middleware: [addContextMiddleware],
});
```

**查看示例 trace** —— 打开此示例的公开 LangSmith 运行记录：https://smith.langchain.com/public/4eaefcd5-90ad-4349-bb51-2cfc660e8af3/r

使用 [`SystemMessage.concat`](https://reference.langchain.com/javascript/langchain-core/utils/stream/concat) 来保留缓存控制元数据或其他中间件创建的结构化内容块。

### 动态模型选择

```ts
import { createMiddleware, initChatModel } from "langchain";

const models = {
  complex: await initChatModel("claude-sonnet-4-6"),
  simple: await initChatModel("claude-haiku-4-5-20251001"),
};

const dynamicModelMiddleware = createMiddleware({
  name: "DynamicModelMiddleware",
  wrapModelCall: (request, handler) => {
    const modifiedRequest = { ...request };
    if (request.messages.length > 10) {
      modifiedRequest.model = models.complex;
    } else {
      modifiedRequest.model = models.simple;
    }
    return handler(modifiedRequest);
  },
});
```

**查看示例 trace** —— 打开此示例的公开 LangSmith 运行记录：https://smith.langchain.com/public/dc39d44b-bef7-49cf-bef8-45e5f4881bf8/r

### 动态选择工具

在运行时选择相关工具以提高性能和准确性。本节介绍如何过滤预先注册的工具。关于注册在运行时发现的工具（例如来自 MCP 服务器），请参见[运行时工具注册](/oss/javascript/langchain/tools#dynamic-tool-selection)。

**好处：**

* **更短的提示词** —— 只暴露相关工具，降低复杂度
* **更好的准确性** —— 模型从更少的选项中做出正确选择
* **权限控制** —— 根据用户访问权限动态过滤工具

```typescript
import { createAgent, createMiddleware } from "langchain";

const toolSelectorMiddleware = createMiddleware({
  name: "ToolSelector",
  wrapModelCall: (request, handler) => {
    // Select a small, relevant subset of tools based on state/context
    const relevantTools = selectRelevantTools(request.state, request.runtime);
    const modifiedRequest = { ...request, tools: relevantTools };
    return handler(modifiedRequest);
  },
});

const agent = createAgent({
  model: "gpt-5.5",
  tools: allTools,
  middleware: [toolSelectorMiddleware],
});
```

### 工具调用监控

```ts
import { createMiddleware } from "langchain";

const toolMonitoringMiddleware = createMiddleware({
  name: "ToolMonitoringMiddleware",
  wrapToolCall: (request, handler) => {
    console.log(`Executing tool: ${request.toolCall.name}`);
    console.log(`Arguments: ${JSON.stringify(request.toolCall.args)}`);
    try {
      const result = handler(request);
      console.log("Tool completed successfully");
      return result;
    } catch (e) {
      console.log(`Tool failed: ${e}`);
      throw e;
    }
  },
});
```

### 提示词缓存（Anthropic）

在使用 Anthropic 模型时，使用带缓存控制指令的结构化内容块来缓存大型系统提示词：

#### 装饰器方式

```python
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from langchain.messages import SystemMessage
from typing import Callable


@wrap_model_call
def add_cached_context(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse],
) -> ModelResponse:
    # Always work with content blocks
    new_content = list(request.system_message.content_blocks) + [
        {
            "type": "text",
            "text": "Here is a large document to analyze:\n\n<document>...</document>",
            # content up until this point is cached
            "cache_control": {"type": "ephemeral"}
        }
    ]

    new_system_message = SystemMessage(content=new_content)
    return handler(request.override(system_message=new_system_message))
```

#### 类方式

```python
from langchain.agents.middleware import AgentMiddleware, ModelRequest, ModelResponse
from langchain.messages import SystemMessage
from typing import Callable


class CachedContextMiddleware(AgentMiddleware):
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse],
    ) -> ModelResponse:
        # Always work with content blocks
        new_content = list(request.system_message.content_blocks) + [
            {
                "type": "text",
                "text": "Here is a large document to analyze:\n\n<document>...</document>",
                "cache_control": {"type": "ephemeral"}  # This content will be cached
            }
        ]

        new_system_message = SystemMessage(content=new_content)
        return handler(request.override(system_message=new_system_message))
```

**说明：**

* `ModelRequest.system_message` 始终是一个 [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) 对象，即使智能体是用 `system_prompt="string"` 创建的
* 使用 `SystemMessage.content_blocks` 以块列表的形式访问内容，无论原始内容是字符串还是列表
* 修改系统消息时，使用 `content_blocks` 并追加新块，以保留现有结构
* 对于缓存控制等高级用例，你可以将 [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) 对象直接传给 `create_agent` 的 `system_prompt` 参数

使用 `ModelRequest` 中的 `systemMessage` 字段在中间件中修改系统消息。它包含一个 [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) 对象（即使智能体是用字符串 [`systemPrompt`](https://reference.langchain.com/javascript/types/langchain.index.CreateAgentParams.html#systemprompt) 创建的）。

**示例：链式中间件** —— 不同的中间件可以使用不同的方式：

```typescript
import { createMiddleware, SystemMessage, createAgent } from "langchain";

// Middleware 1: Uses systemMessage with simple concatenation
const myMiddleware = createMiddleware({
  name: "MyMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(`Additional context.`),
    });
  },
});

// Middleware 2: Uses systemMessage with structured content (preserves structure)
const myOtherMiddleware = createMiddleware({
  name: "MyOtherMiddleware",
  wrapModelCall: async (request, handler) => {
    return handler({
      ...request,
      systemMessage: request.systemMessage.concat(
        new SystemMessage({
          content: [
            {
              type: "text",
              text: " More additional context. This will be cached.",
              cache_control: { type: "ephemeral", ttl: "5m" },
            },
          ],
        })
      ),
    });
  },
});

const agent = createAgent({
  model: "google_genai:gemini-3.6-flash",
  systemPrompt: "You are a helpful assistant.",
  middleware: [myMiddleware, myOtherMiddleware],
});
```

生成的系统消息将是：

```typescript
new SystemMessage({
  content: [
    { type: "text", text: "You are a helpful assistant." },
    { type: "text", text: "Additional context." },
    {
        type: "text",
        text: " More additional context. This will be cached.",
        cache_control: { type: "ephemeral", ttl: "5m" },
    },
  ],
});
```

使用 [`SystemMessage.concat`](https://reference.langchain.com/javascript/langchain-core/utils/stream/concat) 来保留缓存控制元数据或其他中间件创建的结构化内容块。

## 其他资源

* [中间件 API 参考](https://reference.langchain.com/python/langchain/middleware/)
* [内置中间件](/oss/javascript/langchain/middleware/built-in)
* [测试智能体](/oss/javascript/langchain/test/)

---

> 【信息】[通过 MCP 将这些文档连接到](/use-these-docs) Claude、VSCode 等，以获得实时答案。

> 【信息】[在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/middleware/custom.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
