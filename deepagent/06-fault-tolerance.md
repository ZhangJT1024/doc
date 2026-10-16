# 容错

> 原文：Fault tolerance
> 来源：https://docs.langchain.com/oss/javascript/deepagents/fault-tolerance

---

容错 middleware 在出错时保持你的 deep agent 继续运行。并非所有错误都应以相同方式处理：瞬时故障（网络超时、速率限制）应被自动重试，LLM 能够恢复的错误（错误的工具输出、解析失败）应被反馈给模型，而需要人工介入的错误应暂停 agent。

## 错误处理策略

不同的错误需要不同的处理策略：

| 错误类型                                                      | 由谁修复       | 策略                          | Middleware 或特性                                                                                                                                                                                                                                                           |
| --------------------------------------------------------------- | ------------------ | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 瞬时错误（网络问题、速率限制）                  | 系统（自动） | 采用指数退避重试    | [ModelRetryMiddleware](https://reference.langchain.com/javascript/langchain/index/modelRetryMiddleware), [ToolRetryMiddleware](https://reference.langchain.com/javascript/langchain/index/toolRetryMiddleware)                                                                  |
| 用户可修复的错误（缺少信息、指令不清） | 人工               | 用 `interrupt()` 暂停          | [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)                                                                                                                                                                                                               |
| 提供商宕机                                                 | 系统（自动） | 回退到备用模型 | [ModelFallbackMiddleware](https://reference.langchain.com/python/langchain/agents/middleware/model_fallback/ModelFallbackMiddleware)                                                                                                                                            |
| 调用过多（失控循环）                                 | 系统（自动） | 限制每次运行的模型和工具调用  | [ModelCallLimitMiddleware](https://reference.langchain.com/python/langchain/agents/middleware/model_call_limit/ModelCallLimitMiddleware), [ToolCallLimitMiddleware](https://reference.langchain.com/python/langchain/agents/middleware/tool_call_limit/ToolCallLimitMiddleware) |
| 意外错误                                               | 开发者             | 让它们冒泡出来                | 无 middleware；让异常向外传播                                                                                                                                                                                                                                      |

下面各节通过代码示例介绍每种策略。

#### Transient errors

添加 retry middleware，以自动重试网络问题和速率限制。模型调用和工具调用各自拥有自己的 retry middleware，采用指数退避：

```typescript
import { createAgent, modelRetryMiddleware, toolRetryMiddleware } from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  tools: [searchTool, fetchUrlTool],
  middleware: [
    modelRetryMiddleware({ maxRetries: 3, backoffFactor: 2.0, initialDelayMs: 1000 }),
    toolRetryMiddleware({
      maxRetries: 2,
      tools: ["search", "fetch_url"],
      retryOn: [TimeoutError, TypeError],
    }),
  ],
});
```

#### User-fixable

在需要时暂停并向用户收集信息（例如账号 ID、订单号或澄清说明）。使用 `interrupt_on` 在特定工具调用前暂停 agent：

```typescript
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [sendEmailTool, deleteRecordTool],
  interruptOn: {
    send_email: true,
    delete_record: true,
  },
});
```

关于完整的 human-in-the-loop 指南，请参阅 [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)。

#### Provider outage

如果你的主模型提供商完全宕机，使用 [ModelFallbackMiddleware](https://reference.langchain.com/javascript/langchain/index/modelFallbackMiddleware) 切换到备用模型：

```typescript
import { createAgent, modelFallbackMiddleware } from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  tools: [searchTool],
  middleware: [
    modelFallbackMiddleware("gpt-5.5"),
  ],
});
```

#### Excessive calls

如果没有限制，一个困惑的 agent 可能会通过循环调用同一个工具或在几分钟内进行数百次模型调用来耗尽你的 LLM API 预算。为每次运行的模型调用和工具执行都设置上限：

```typescript
import { createAgent, modelCallLimitMiddleware, toolCallLimitMiddleware } from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  tools: [searchTool],
  middleware: [
    modelCallLimitMiddleware({ runLimit: 50 }),
    toolCallLimitMiddleware({ runLimit: 200 }),
  ],
});
```

#### Unexpected

让它们冒泡出来以便调试。不要捕获你无法处理的错误。此模式同样适用于 JavaScript SDK 中的自定义 middleware。

## 速率限制

有两种互补的方式来限制资源使用：控制对模型提供商的请求速率，以及限制每次运行的调用总数。

### 提供商速率限制

Chat model 提供商会限制在给定时间段内可进行的调用次数。要控制发出请求的速率，请使用 `rate_limiter` 初始化你的模型：

### 调用限制

如果没有限制，一个困惑的 agent 可能会通过循环调用同一个工具或在几分钟内进行数百次模型调用来耗尽你的 LLM API 预算。为每次运行的模型调用和工具执行都设置上限：

```typescript
import { createAgent, modelCallLimitMiddleware, toolCallLimitMiddleware } from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  middleware: [
    modelCallLimitMiddleware({ runLimit: 50 }),
    toolCallLimitMiddleware({ runLimit: 200 }),
  ],
});
```

使用 `run_limit` 限制单次调用内的调用次数（每一轮重置）。使用 `thread_limit` 限制整个对话中的调用次数（需要 checkpointer）。关于完整配置，请参阅 [ModelCallLimitMiddleware](https://reference.langchain.com/javascript/langchain/index/modelCallLimitMiddleware) 和 [ToolCallLimitMiddleware](https://reference.langchain.com/javascript/langchain/index/toolCallLimitMiddleware)。

## 重试

瞬时故障（网络超时、速率限制）应被自动重试。模型调用和工具调用各自拥有自己的 retry middleware，采用指数退避：

```typescript
import {
  createAgent,
  modelRetryMiddleware,
  toolRetryMiddleware,
} from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  middleware: [
    // Retry model calls on rate limits, timeouts, and 5xx errors
    modelRetryMiddleware({ maxRetries: 3, backoffFactor: 2.0, initialDelayMs: 1000 }),
    // Retry specific tools that hit external APIs (not all tools)
    toolRetryMiddleware({
      maxRetries: 2,
      tools: ["search", "fetch_url"],
      retryOn: [TimeoutError, TypeError],
    }),
  ],
});
```

应将 [ToolRetryMiddleware](https://reference.langchain.com/javascript/langchain/index/toolRetryMiddleware) 限定到特定工具，而不是重试所有内容。一个失败的文件系统 `read_file` 不会从重试中获益，但一个超时的 web search 很可能会。关于完整配置，请参阅 [ModelRetryMiddleware](https://reference.langchain.com/javascript/langchain/index/modelRetryMiddleware)。

## 回退

如果你的主模型提供商完全宕机，fallback middleware 会切换到备用模型：

```typescript
import {
  createAgent,
  modelFallbackMiddleware,
} from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  middleware: [
    // If the primary model is fully down, fall back to an alternative
    modelFallbackMiddleware("gpt-5.5"),
  ],
});
```

关于完整配置，请参阅 [ModelFallbackMiddleware](https://reference.langchain.com/javascript/langchain/index/modelFallbackMiddleware)。

## 错误处理

当工具在执行期间抛出异常时，agent 运行默认会中止。`ToolErrorMiddleware` 在 JavaScript SDK 中尚不可用。
