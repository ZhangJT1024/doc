# 预置中间件

> 原文：https://docs.langchain.com/oss/javascript/langchain/middleware/built-in
> LangChain (TypeScript) 官方文档中文整理

> 面向常见智能体用例的预置中间件

LangChain 和 [Deep Agents](/oss/javascript/deepagents/overview) 为常见用例提供了预置中间件。每个中间件都是生产可用的，并且可以针对你的具体需求进行配置。

## 与提供方无关的中间件

以下中间件可与任何 LLM 提供方配合使用：

| 中间件                                        | 说明                                                                             |
| --------------------------------------------- | -------------------------------------------------------------------------------- |
| [工具重试](#tool-retry)                       | 以指数退避自动重试失败的工具调用。                                               |
| [模型重试](#model-retry)                      | 以指数退避自动重试失败的模型调用。                                               |
| [模型回退](#model-fallback)                   | 当主模型失败时自动回退到替代模型。                                               |
| [摘要](#summarization)                        | 当接近 token 上限时自动总结对话历史。                                            |
| [人在环路（Human-in-the-loop）](#human-in-the-loop) | 暂停执行，等待人工批准工具调用。                                          |
| [模型调用上限](#model-call-limit)             | 限制模型调用次数以防止成本过高。                                                 |
| [工具调用上限](#tool-call-limit)              | 通过限制调用次数控制工具执行。                                                   |
| [PII 检测](#pii-detection)                    | 检测并处理个人身份信息（PII）。                                                  |
| [待办事项列表](#to-do-list)                   | 为智能体配备任务规划与跟踪能力。                                                 |
| [LLM 工具选择器](#llm-tool-selector)          | 在调用主模型之前使用 LLM 选择相关工具。                                          |
| [提供方工具搜索](#provider-tool-search)       | 将工具推迟到提供方的服务端工具搜索之后，按需呈现。                               |
| [文件系统](#filesystem-middleware)            | 为智能体提供用于存储上下文和长期记忆的文件系统。                                 |
| [子智能体中间件](#subagent)                   | 添加生成子智能体的能力。                                                         |
| [上下文编辑](#context-editing)                | 通过裁剪或清除工具使用来管理对话上下文。                                         |
| [LLM 工具模拟器](#llm-tool-emulator)          | 出于测试目的，使用 LLM 模拟工具执行。                                            |

### 工具错误

### 工具重试

以可配置的指数退避自动重试失败的工具调用。工具重试适用于以下场景：

* 处理外部 API 调用中的瞬时故障。
* 提高依赖网络的工具的可靠性。
* 构建能够优雅处理临时错误的弹性智能体。

**API 参考：** [`toolRetryMiddleware`](https://reference.langchain.com/javascript/langchain/index/toolRetryMiddleware)

```typescript
import { createAgent, toolRetryMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, databaseTool],
  middleware: [
    toolRetryMiddleware({
      maxRetries: 3,
      backoffFactor: 2.0,
      initialDelayMs: 1000,
    }),
  ],
});
```

#### 配置选项

- **`maxRetries`** `number`，默认 `2`：初始调用之后的最大重试次数（默认情况下共 3 次尝试）。必须 >= 0。
- **`tools`** `(ClientTool | ServerTool | string)[]`：可选，指定要应用重试逻辑的工具或工具名数组。可以是 `BaseTool` 实例列表，也可以是工具名字符串列表。若为 `undefined`，则应用于所有工具。
- **`retryOn`** `((error: Error) => boolean) | (new (...args: any[]) => Error)[]`，默认 `(error) => getRetryable(error) ?? true`：要么是要对其重试的错误构造函数数组，要么是一个接收错误并在应当重试时返回 `true` 的函数。使用 `langchain>=1.5.9` 时，默认不再重试被标记为不可重试的错误。其他所有错误都会被重试。
- **`onFailure`** `'error' | 'continue' | ((error: Error) => string)`，默认 `continue`：当所有重试都耗尽时的行为。选项：
  * `'continue'`（默认）—— 返回带有错误详情的 `ToolMessage`，允许 LLM 处理该失败并可能从中恢复
  * `'error'` —— 重新抛出异常，停止智能体执行
  * 自定义函数 —— 接收异常并返回用于 `ToolMessage` 内容的字符串，允许自定义错误格式

  **已弃用的值：** `'raise'`（请改用 `'error'`）和 `'return_message'`（请改用 `'continue'`）。这些已弃用的值仍然可用，但会显示警告。
- **`backoffFactor`** `number`，默认 `2.0`：指数退避的乘数。每次重试等待 `initialDelayMs * (backoffFactor ** retryNumber)` 毫秒。设为 `0.0` 表示固定延迟。必须 >= 0。
- **`initialDelayMs`** `number`，默认 `1000`：第一次重试之前的初始延迟（毫秒）。必须 >= 0。
- **`maxDelayMs`** `number`，默认 `60000`：重试之间的最大延迟（毫秒），用于限制指数退避的增长。必须 >= 0。
- **`jitter`** `boolean`，默认 `true`：是否在延迟上添加随机抖动（`±25%`）以避免惊群效应。

#### 完整示例

该中间件会以指数退避自动重试失败的工具调用。

**关键配置：**

* `maxRetries` —— 重试次数（默认：2）
* `backoffFactor` —— 指数退避的乘数（默认：2.0）
* `initialDelayMs` —— 起始延迟（毫秒）（默认：1000ms）
* `maxDelayMs` —— 延迟增长的上限（默认：60000ms）
* `jitter` —— 添加随机变化（默认：true）

**失败处理：**

* `onFailure: "continue"`（默认）—— 返回错误消息
* `onFailure: "error"` —— 重新抛出异常
* 自定义函数 —— 返回错误消息的函数

```typescript
import { createAgent, toolRetryMiddleware } from "langchain";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

// Basic usage with default settings (2 retries, exponential backoff)
const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, databaseTool],
  middleware: [toolRetryMiddleware()],
});

// Retry specific exceptions only
const retry = toolRetryMiddleware({
  maxRetries: 4,
  retryOn: [TimeoutError, NetworkError],
  backoffFactor: 1.5,
});

// Custom exception filtering
function shouldRetry(error: Error): boolean {
  // Only retry on 5xx errors
  if (error.name === "HTTPError" && "statusCode" in error) {
    const statusCode = (error as any).statusCode;
    return 500 <= statusCode && statusCode < 600;
  }
  return false;
}

const retryWithFilter = toolRetryMiddleware({
  maxRetries: 3,
  retryOn: shouldRetry,
});

// Apply to specific tools with custom error handling
const formatError = (error: Error) =>
  "Database temporarily unavailable. Please try again later.";

const retrySpecificTools = toolRetryMiddleware({
  maxRetries: 4,
  tools: ["search_database"],
  onFailure: formatError,
});

// Apply to specific tools using BaseTool instances
const searchDatabase = tool(
  async ({ query }) => {
    // Search implementation
    return results;
  },
  {
    name: "search_database",
    description: "Search the database",
    schema: z.object({ query: z.string() }),
  }
);

const retryWithToolInstance = toolRetryMiddleware({
  maxRetries: 4,
  tools: [searchDatabase], // Pass BaseTool instance
});

// Constant backoff (no exponential growth)
const constantBackoff = toolRetryMiddleware({
  maxRetries: 5,
  backoffFactor: 0.0, // No exponential growth
  initialDelayMs: 2000, // Always wait 2 seconds
});

// Raise exception on failure
const strictRetry = toolRetryMiddleware({
  maxRetries: 2,
  onFailure: "error", // Re-raise exception instead of returning message
});
```

#### 将错误标记为可重试

> 【提示】需要 `@langchain/core>=1.2.8` 和 `langchain>=1.5.9`。

在抛出错误之前对其标记，重试中间件就会遵循该判定。这样可以避免把重试浪费在会以相同方式重复出现的失败上，例如无效输入或被拒绝的凭证。

使用 `@langchain/core/errors` 中的 `stampRetryable`：

```typescript
import { stampRetryable } from "@langchain/core/errors";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const lookupRecord = tool(
  async ({ id }) => {
    const response = await fetch(`https://api.example.com/records/${id}`);

    if (response.status === 422) {
      // Retrying the same identifier fails the same way
      throw stampRetryable(new Error("Malformed record identifier"), false);
    }
    if (response.status === 503) {
      throw stampRetryable(new Error("Record service unavailable"), true);
    }

    return response.json();
  },
  {
    name: "lookup_record",
    description: "Look up a record by identifier",
    schema: z.object({ id: z.string() }),
  }
);
```

对于再次尝试也不会成功的失败，传入 `false`；对于瞬时故障，传入 `true`。你没有标记的错误会被重试，因此现有工具会保持其当前行为。

标记错误不会改变其类型或结构，因此针对它的 `instanceof` 检查仍然有效。要读回标记，调用 `getRetryable(error)`，它会返回 `true`、`false`，或在该错误从未被标记时返回 `undefined`。

聊天模型集成已经对它们识别出的失败进行了标记，因此你无需自己标记这些失败。以下失败会在第一次尝试时就失败，而不会消耗重试次数：

* **认证失败**：API key 缺失、无效或已过期。
* **权限被拒绝**：凭证有效但无权访问所请求的资源。
* **未知模型**：提供方无法识别的模型标识符。
* **上下文窗口溢出**：输入长度超过模型可接受的范围。
* **无效的工具结果**：格式错误的工具结果块。
* **载荷过大**：提供方因大小而拒绝的请求体。
* **配额耗尽**：计费或使用量限制，与速率限制不同。
* **调用被取消**：通过中止信号停止的请求。

速率限制、超时和服务器错误仍可重试。提供方未识别的任何内容都保持未标记，并像以前一样重试。

### 模型重试

以可配置的指数退避自动重试失败的模型调用。模型重试适用于以下场景：

* 处理模型 API 调用中的瞬时故障。
* 提高依赖网络的模型请求的可靠性。
* 构建能够优雅处理临时模型错误的弹性智能体。

**API 参考：** [`modelRetryMiddleware`](https://reference.langchain.com/javascript/langchain/index/modelRetryMiddleware)

```typescript
import { createAgent, modelRetryMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, databaseTool],
  middleware: [
    modelRetryMiddleware({
      maxRetries: 3,
      backoffFactor: 2.0,
      initialDelayMs: 1000,
    }),
  ],
});
```

#### 配置选项

- **`maxRetries`** `number`，默认 `2`：初始调用之后的最大重试次数（默认情况下共 3 次尝试）。必须 >= 0。
- **`retryOn`** `((error: Error) => boolean) | (new (...args: any[]) => Error)[]`，默认 `(error) => getRetryable(error) ?? true`：要么是要对其重试的错误构造函数数组，要么是一个接收错误并在应当重试时返回 `true` 的函数。使用 `langchain>=1.5.9` 时，默认不再重试被标记为不可重试的错误。其他所有错误都会被重试。
- **`onFailure`** `'error' | 'continue' | ((error: Error) => string)`，默认 `continue`：当所有重试都耗尽时的行为。选项：
  * `'continue'`（默认）—— 返回带有错误详情的 `AIMessage`，允许智能体优雅地处理该失败
  * `'error'` —— 重新抛出异常，停止智能体执行
  * 自定义函数 —— 接收异常并返回用于 `AIMessage` 内容的字符串，允许自定义错误格式
- **`backoffFactor`** `number`，默认 `2.0`：指数退避的乘数。每次重试等待 `initialDelayMs * (backoffFactor ** retryNumber)` 毫秒。设为 `0.0` 表示固定延迟。必须 >= 0。
- **`initialDelayMs`** `number`，默认 `1000`：第一次重试之前的初始延迟（毫秒）。必须 >= 0。
- **`maxDelayMs`** `number`，默认 `60000`：重试之间的最大延迟（毫秒），用于限制指数退避的增长。必须 >= 0。
- **`jitter`** `boolean`，默认 `true`：是否在延迟上添加随机抖动（`±25%`）以避免惊群效应。

#### 完整示例

该中间件会以指数退避自动重试失败的模型调用。

```typescript
import { createAgent, modelRetryMiddleware } from "langchain";

// Basic usage with default settings (2 retries, exponential backoff)
const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool],
  middleware: [modelRetryMiddleware()],
});

class TimeoutError extends Error {
    // ...
}
class NetworkError extends Error {
    // ...
}

// Retry specific exceptions only
const retry = modelRetryMiddleware({
  maxRetries: 4,
  retryOn: [TimeoutError, NetworkError],
  backoffFactor: 1.5,
});

// Custom exception filtering
function shouldRetry(error: Error): boolean {
  // Only retry on rate limit errors
  if (error.name === "RateLimitError") {
    return true;
  }
  // Or check for specific HTTP status codes
  if (error.name === "HTTPError" && "statusCode" in error) {
    const statusCode = (error as any).statusCode;
    return statusCode === 429 || statusCode === 503;
  }
  return false;
}

const retryWithFilter = modelRetryMiddleware({
  maxRetries: 3,
  retryOn: shouldRetry,
});

// Return error message instead of raising
const retryContinue = modelRetryMiddleware({
  maxRetries: 4,
  onFailure: "continue", // Return AIMessage with error instead of throwing
});

// Custom error message formatting
const formatError = (error: Error) =>
  `Model call failed: ${error.message}. Please try again later.`;

const retryWithFormatter = modelRetryMiddleware({
  maxRetries: 4,
  onFailure: formatError,
});

// Constant backoff (no exponential growth)
const constantBackoff = modelRetryMiddleware({
  maxRetries: 5,
  backoffFactor: 0.0, // No exponential growth
  initialDelayMs: 2000, // Always wait 2 seconds
});

// Raise exception on failure
const strictRetry = modelRetryMiddleware({
  maxRetries: 2,
  onFailure: "error", // Re-raise exception instead of returning message
});
```

### 模型回退

当主模型失败时自动回退到替代模型。模型回退适用于以下场景：

* 构建能够处理模型故障的弹性智能体。
* 通过回退到更便宜的模型来优化成本。
* 在 OpenAI、Anthropic 等提供方之间实现冗余。

```typescript
import { createAgent, modelFallbackMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    modelFallbackMiddleware(
      "gpt-5.4-mini",
      "claude-3-5-sonnet-20241022"
    ),
  ],
});
```

#### 配置选项

该中间件接受可变数量的字符串参数，按顺序表示回退模型：

- **`...models`** `string[]`，必填：当主模型失败时要按顺序尝试的一个或多个回退模型字符串。

```typescript
modelFallbackMiddleware(
  "first-fallback-model",
  "second-fallback-model",
  // ... more models
)
```

### 摘要

当接近 token 上限时自动总结对话历史，在压缩较旧上下文的同时保留最近的消息。摘要适用于以下场景：

* 超出上下文窗口的长时间运行的对话。
* 拥有大量历史记录的多轮对话。
* 需要保留完整对话上下文的应用。

> 【提示】摘要是一种面向文本的上下文压缩。它不会对图像/音频/视频载荷进行缩放、降采样或其他方式的压缩。由 `keep` 保留的最近消息仍会包含其原始多模态块，而被摘要的较旧多模态消息只会以生成的文本摘要形式表示。对于图像密集型应用，请将媒体存储在文件系统或对象存储中，并通过消息历史传递 URL 或文件引用。

```typescript
import { createAgent, summarizationMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [weatherTool, calculatorTool],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: { tokens: 4000 },
      keep: { messages: 20 },
    }),
  ],
});
```

#### 配置选项

> 【提示】如果使用 `langchain@1.1.0`，`trigger` 和 `keep` 的 `fraction` 条件（如下所示）依赖聊天模型的[配置文件数据](/oss/javascript/langchain/models#model-profiles)。如果数据不可用，请使用其他条件或手动指定：
>
> ```typescript
> const customProfile: ModelProfile = {
>     maxInputTokens: 100_000,
>     // ...
> }
> model = await initChatModel("...", {
>     profile: customProfile,
> });
> ```

- **`model`** `string | BaseChatModel`，必填：用于生成摘要的模型。可以是模型标识符字符串（例如 `'openai:gpt-5.4-mini'`）或 `BaseChatModel` 实例。
- **`trigger`** `object | object[]`：触发摘要的条件。可以是：
  * **单一条件（AND 逻辑）：** 单个对象，其中所有属性必须同时满足。

    ```typescript
    // Trigger when tokens >= 4000 AND messages >= 10
    trigger: { tokens: 4000, messages: 10 }
    ```

  * **多个条件（OR 逻辑）：** 条件对象数组。当任一对象的条件被满足时触发摘要。

    ```typescript
    // Trigger when tokens >= 3000 OR messages >= 6
    trigger: [{ tokens: 3000 }, { messages: 6 }]
    ```

  每个条件可以包含：

  * `fraction`（number）：模型上下文大小的比例（0-1）
  * `tokens`（number）：绝对 token 数量
  * `messages`（number）：消息数量

  每个条件至少必须指定一个属性。如果未提供，摘要将不会自动触发。
- **`keep`** `object`，默认 `{messages: 20}`：摘要后要保留多少上下文。请精确指定以下之一：
  * `fraction`（number）：要保留的模型上下文大小的比例（0-1）
  * `tokens`（number）：要保留的绝对 token 数量
  * `messages`（number）：要保留的最近消息数量
- **`tokenCounter`** `function`：自定义 token 计数函数。默认为基于字符的计数。
- **`summaryPrompt`** `string`：用于摘要的自定义提示词模板。如果未指定，则使用内置模板。该模板应包含 `{messages}` 占位符，对话历史将被插入其中。
- **`trimTokensToSummarize`** `number`，默认 `4000`：生成摘要时要包含的最大 token 数。消息会在摘要之前被裁剪以符合该限制。
- **`summaryPrefix`** `string`：添加到摘要消息的前缀。如果未提供，则使用默认前缀。
- **`maxTokensBeforeSummary`** `number`（已弃用）：**已弃用：** 请改用 `trigger: { tokens: value }`。触发摘要的 token 阈值。
- **`messagesToKeep`** `number`（已弃用）：**已弃用：** 请改用 `keep: { messages: value }`。要保留的最近消息。

#### 完整示例

摘要中间件会监控消息的 token 计数，并在达到阈值时自动总结较旧的消息。

**触发条件**控制摘要何时运行：

* 单个阈值在该阈值被满足时触发
* 带多个阈值的触发子句只有在所有阈值都被满足时才触发（AND 逻辑）
* 触发条件列表在任一项被满足时触发（OR 逻辑）
* 每个阈值可以使用 `fraction`（模型上下文大小的比例）、`tokens`（绝对数量）或 `messages`（消息数量）

**保留条件**控制要保留多少上下文（请精确指定其中之一）：

* `fraction` —— 要保留的模型上下文大小的比例
* `tokens` —— 要保留的绝对 token 数量
* `messages` —— 要保留的最近消息数量

```typescript
import { createAgent, summarizationMiddleware } from "langchain";

// Single condition (AND logic): trigger if tokens >= 4000 AND messages >= 10
const agent = createAgent({
  model: "gpt-5.5",
  tools: [weatherTool, calculatorTool],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: { tokens: 4000, messages: 10 },
      keep: { messages: 20 },
    }),
  ],
});

// Multiple conditions (OR logic): trigger if tokens >= 3000 OR messages >= 6
const agent2 = createAgent({
  model: "gpt-5.5",
  tools: [weatherTool, calculatorTool],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: [
        { tokens: 3000 },
        { messages: 6 },
      ],
      keep: { messages: 20 },
    }),
  ],
});

// Using fractional limits
const agent3 = createAgent({
  model: "gpt-5.5",
  tools: [weatherTool, calculatorTool],
  middleware: [
    summarizationMiddleware({
      model: "gpt-5.4-mini",
      trigger: { fraction: 0.8 },
      keep: { fraction: 0.3 },
    }),
  ],
});
```

### 人在环路（Human-in-the-loop）

在工具调用执行之前暂停智能体执行，等待人工批准、编辑或拒绝。[人在环路](/oss/javascript/langchain/human-in-the-loop)适用于以下场景：

* 需要人工批准的高风险操作（例如数据库写入、金融交易）。
* 人工监督为强制要求的合规工作流。
* 由人工反馈引导智能体的长时间运行的对话。

> 【警告】人在环路中间件需要[检查点保存器（checkpointer）](/oss/javascript/langgraph/checkpointers#checkpoints)，以便在中断之间维护状态。

```typescript
import { createAgent, humanInTheLoopMiddleware } from "langchain";

function readEmailTool(emailId: string): string {
  /** Mock function to read an email by its ID. */
  return `Email content for ID: ${emailId}`;
}

function sendEmailTool(recipient: string, subject: string, body: string): string {
  /** Mock function to send an email. */
  return `Email sent to ${recipient} with subject '${subject}'`;
}

const agent = createAgent({
  model: "gpt-5.5",
  tools: [readEmailTool, sendEmailTool],
  middleware: [
    humanInTheLoopMiddleware({
      interruptOn: {
        sendEmailTool: {
          allowedDecisions: ["approve", "edit", "reject"],
        },
        readEmailTool: false,
      }
    })
  ]
});
```

> 【提示】完整的示例、配置选项和集成模式请参见[人在环路文档](/oss/javascript/langchain/human-in-the-loop)。

> 【信息】观看这段[视频指南](https://www.youtube.com/watch?v=tdOeUVERukA)，演示人在环路中间件的行为。

### 模型调用上限

限制模型调用次数，以防止无限循环或成本过高。模型调用上限适用于以下场景：

* 防止失控的智能体发出过多 API 调用。
* 对生产部署实施成本控制。
* 在特定的调用预算内测试智能体行为。

```typescript
import { createAgent, modelCallLimitMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "gpt-5.5",
  checkpointer: new MemorySaver(), // Required for thread limiting
  tools: [],
  middleware: [
    modelCallLimitMiddleware({
      threadLimit: 10,
      runLimit: 5,
      exitBehavior: "end",
    }),
  ],
});
```

> 【信息】观看这段[视频指南](https://www.youtube.com/watch?v=x5jLQTFXR0Y)，演示模型调用上限中间件的行为。

#### 配置选项

- **`threadLimit`** `number`：一个线程中所有运行的最大模型调用次数。默认为无限制。
- **`runLimit`** `number`：单次调用的最大模型调用次数。默认为无限制。
- **`exitBehavior`** `string`，默认 `end`：达到上限时的行为。选项：`'end'`（优雅终止）或 `'error'`（抛出异常）。

### 工具调用上限

通过限制工具调用次数来控制智能体执行，可以全局应用于所有工具，也可以针对特定工具。工具调用上限适用于以下场景：

* 防止对昂贵的外部 API 进行过多调用。
* 限制网络搜索或数据库查询。
* 对特定工具的使用实施速率限制。
* 防止失控的智能体循环。

```typescript
import { createAgent, toolCallLimitMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, databaseTool],
  middleware: [
    toolCallLimitMiddleware({ threadLimit: 20, runLimit: 10 }),
    toolCallLimitMiddleware({
      toolName: "search",
      threadLimit: 5,
      runLimit: 3,
    }),
  ],
});
```

> 【信息】观看这段[视频指南](https://www.youtube.com/watch?v=oL6am5UqODY)，演示工具调用上限中间件的行为。

#### 配置选项

- **`toolName`** `string`：要限制的特定工具名称。如果未提供，限制将**全局应用于所有工具**。
- **`threadLimit`** `number`：一个线程（对话）中所有运行的最大工具调用次数。在具有相同线程 ID 的多次调用之间持续存在。需要检查点保存器来维护状态。`undefined` 表示无线程上限。
- **`runLimit`** `number`：单次调用（一条用户消息 → 响应循环）的最大工具调用次数。每条新的用户消息都会重置。`undefined` 表示无运行上限。

  **注意：** `threadLimit` 或 `runLimit` 至少必须指定一个。
- **`exitBehavior`** `string`，默认 `continue`：达到上限时的行为：
  * `'continue'`（默认）—— 用错误消息阻止超出的工具调用，让其他工具和模型继续运行。模型根据错误消息决定何时结束。
  * `'error'` —— 抛出 `ToolCallLimitExceededError` 异常，立即停止执行
  * `'end'` —— 立即停止执行，为超出的工具调用返回一条 ToolMessage 和一条 AI 消息。仅在限制单个工具时有效；如果其他工具有待处理的调用则会抛出错误。

#### 完整示例

使用以下方式指定限制：

* **线程上限（Thread limit）** —— 一个对话中所有运行的最大调用次数（需要检查点保存器）
* **运行上限（Run limit）** —— 单次调用的最大调用次数（每轮重置）

退出行为：

* `'continue'`（默认）—— 用错误消息阻止超出的调用，智能体继续
* `'error'` —— 立即抛出异常
* `'end'` —— 用 ToolMessage + AI 消息停止（仅限单工具场景）

```typescript
import { createAgent, toolCallLimitMiddleware } from "langchain";

const globalLimiter = toolCallLimitMiddleware({ threadLimit: 20, runLimit: 10 });
const searchLimiter = toolCallLimitMiddleware({ toolName: "search", threadLimit: 5, runLimit: 3 });
const databaseLimiter = toolCallLimitMiddleware({ toolName: "query_database", threadLimit: 10 });
const strictLimiter = toolCallLimitMiddleware({ toolName: "scrape_webpage", runLimit: 2, exitBehavior: "error" });

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, databaseTool, scraperTool],
  middleware: [globalLimiter, searchLimiter, databaseLimiter, strictLimiter],
});
```

### PII 检测

使用可配置的策略检测并处理对话中的个人身份信息（PII）。PII 检测适用于以下场景：

* 具有合规要求的医疗和金融应用。
* 需要清理日志的客服智能体。
* 任何处理敏感用户数据的应用。

```typescript
import { createAgent, piiMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    piiMiddleware("email", { strategy: "redact", applyToInput: true }),
    piiMiddleware("credit_card", { strategy: "mask", applyToInput: true }),
  ],
});
```

#### 自定义 PII 类型

你可以通过提供 `detector` 参数来创建自定义 PII 类型。这允许你检测内置类型之外、针对你具体用例的模式。

**创建自定义检测器的三种方式：**

1. **正则表达式模式字符串** —— 简单的模式匹配

2. **RegExp 对象** —— 对正则标志有更多控制

3. **自定义函数** —— 带验证的复杂检测逻辑

```typescript
import { createAgent, piiMiddleware, type PIIMatch } from "langchain";

// Method 1: Regex pattern string
const agent1 = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    piiMiddleware("api_key", {
      detector: "sk-[a-zA-Z0-9]{32}",
      strategy: "block",
    }),
  ],
});

// Method 2: RegExp object
const agent2 = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    piiMiddleware("phone_number", {
      detector: /\+?\d{1,3}[\s.-]?\d{3,4}[\s.-]?\d{4}/,
      strategy: "mask",
    }),
  ],
});

// Method 3: Custom detector function
function detectSSN(content: string): PIIMatch[] {
  const matches: PIIMatch[] = [];
  const pattern = /\d{3}-\d{2}-\d{4}/g;
  let match: RegExpExecArray | null;

  while ((match = pattern.exec(content)) !== null) {
    const ssn = match[0];
    // Validate: first 3 digits shouldn't be 000, 666, or 900-999
    const firstThree = parseInt(ssn.substring(0, 3), 10);
    if (firstThree !== 0 && firstThree !== 666 && !(firstThree >= 900 && firstThree <= 999)) {
      matches.push({
        text: ssn,
        start: match.index ?? 0,
        end: (match.index ?? 0) + ssn.length,
      });
    }
  }
  return matches;
}

const agent3 = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    piiMiddleware("ssn", {
      detector: detectSSN,
      strategy: "hash",
    }),
  ],
});
```

**自定义检测器函数签名：**

检测器函数必须接受一个字符串（内容）并返回匹配项：

返回一个 `PIIMatch` 对象数组：

```typescript
interface PIIMatch {
  text: string;    // The matched text
  start: number;   // Start index in content
  end: number;      // End index in content
}

function detector(content: string): PIIMatch[] {
  return [
    { text: "matched_text", start: 0, end: 12 },
    // ... more matches
  ];
}
```

> 【提示】对于自定义检测器：
>
> * 简单模式使用正则字符串
> * 需要标志（例如不区分大小写匹配）时使用 RegExp 对象
> * 需要超出模式匹配的验证逻辑时使用自定义函数
> * 自定义函数让你完全控制检测逻辑，并可以实现复杂的验证规则

#### 配置选项

- **`piiType`** `string`，必填：要检测的 PII 类型。可以是内置类型（`email`、`credit_card`、`ip`、`mac_address`、`url`）或自定义类型名称。
- **`strategy`** `string`，默认 `redact`：如何处理检测到的 PII。选项：
  * `'block'` —— 检测到时抛出错误
  * `'redact'` —— 替换为 `[REDACTED_TYPE]`
  * `'mask'` —— 部分掩码（例如 `****-****-****-1234`）
  * `'hash'` —— 替换为确定性哈希（例如 `<email_hash:a1b2c3d4>`）
- **`detector`** `RegExp | string | function`：自定义检测器。可以是：
  * `RegExp` —— 用于匹配的正则表达式模式
  * `string` —— 正则表达式模式字符串（例如 `"sk-[a-zA-Z0-9]{32}"`）
  * `function` —— 自定义检测器函数 `(content: string) => PIIMatch[]`

  如果未提供，则使用该 PII 类型的内置检测器。
- **`applyToInput`** `boolean`，默认 `true`：在模型调用之前检查用户消息
- **`applyToOutput`** `boolean`，默认 `false`：在模型调用之后检查 AI 消息
- **`applyToToolResults`** `boolean`，默认 `false`：在执行之后检查工具结果消息

### 待办事项列表

为智能体配备面向复杂多步骤任务的任务规划与跟踪能力。待办事项列表适用于以下场景：

* 需要跨多个工具协调的复杂多步骤任务。
* 进度可见性很重要的长时间运行操作。

> 【提示】该中间件会自动为智能体提供一个 `write_todos` 工具和系统提示词，以引导有效的任务规划。

```typescript
import { createAgent, todoListMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [readFile, writeFile, runTests],
  middleware: [todoListMiddleware()],
});
```

> 【信息】观看这段[视频指南](https://www.youtube.com/watch?v=dwvhZ1z_Pas)，演示待办事项列表中间件的行为。

#### 配置选项

没有可用的配置选项（使用默认值）。

### LLM 工具选择器

在调用主模型之前，使用 LLM 智能地选择相关工具。LLM 工具选择器适用于以下场景：

* 拥有许多工具（10+）且大多数工具与每次查询无关的智能体。
* 通过过滤不相关的工具来减少 token 使用量。
* 提高模型的专注度和准确性。

该中间件使用结构化输出询问 LLM 哪些工具与当前查询最相关。结构化输出 schema 定义了可用的工具名称和描述。模型提供方通常会在幕后将这些结构化输出信息添加到系统提示词中。

```typescript
import { createAgent, llmToolSelectorMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [tool1, tool2, tool3, tool4, tool5, ...],
  middleware: [
    llmToolSelectorMiddleware({
      model: "gpt-5.4-mini",
      maxTools: 3,
      alwaysInclude: ["search"],
    }),
  ],
});
```

#### 配置选项

- **`model`** `string | BaseChatModel`：用于工具选择的模型。可以是模型标识符字符串（例如 `'openai:gpt-5.4-mini'`）或 `BaseChatModel` 实例。默认为智能体的主模型。
- **`systemPrompt`** `string`：给选择模型的指令。如果未指定，则使用内置提示词。
- **`maxTools`** `number`：要选择的最大工具数量。如果模型选择了更多，只会使用前 maxTools 个。如果未指定则无限制。
- **`alwaysInclude`** `string[]`：无论选择结果如何都始终包含的工具名称。这些不计入 maxTools 限制。

### 提供方工具搜索

将选定的工具推迟到模型提供方的服务端工具搜索之后，这样模型会按需发现它们，而不是预先接收所有工具 schema。提供方工具搜索适用于：

* 使用许多工具时减少上下文膨胀。
* 通过只呈现相关工具来提高工具选择准确性。

> 【提示】需要支持服务端工具搜索的模型：Anthropic（Claude Sonnet 4+/Opus 4+/Haiku 4.5+）或 OpenAI（gpt-5.5+）。其他提供方会抛出错误。

**API 参考：** [`providerToolSearchMiddleware`](https://reference.langchain.com/javascript/langchain/index/providerToolSearchMiddleware)

```typescript
import { createAgent, providerToolSearchMiddleware } from "langchain";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const getWeather = tool(async () => "Sunny, 22C", {
  name: "get_weather",
  description: "Get the current weather for a city",
  schema: z.object({ city: z.string() }),
});

const lookupOrderStatus = tool(async () => "OUT_FOR_DELIVERY", {
  name: "lookup_order_status",
  description: "Look up the current delivery status of a customer order by ID",
  schema: z.object({ orderId: z.string() }),
});

const nicheTools = [lookupOrderStatus];

const agent = createAgent({
  model: "anthropic:claude-opus-4-8",
  tools: [getWeather, ...nicheTools],
  middleware: [
    providerToolSearchMiddleware({ searchableTools: nicheTools }),
  ],
});
```

#### 配置选项

- **`searchableTools`** `(string | StructuredToolInterface)[]`：要推迟到提供方工具搜索之后的工具，按名称或实例给出。被推迟的工具在搜索将其呈现之前不会提供给模型。使用 `extras.defer_loading: true` 构造的工具无论如何都会被推迟；如果省略 `searchableTools`，则只会推迟那些预先标记的工具。

#### 完整示例

该中间件会将 `searchableTools` 中包含的所有工具选择加入推迟和搜索。工具也可以在构造时通过设置 `extras.defer_loading: true` 来选择加入推迟。

```typescript
import { createAgent, providerToolSearchMiddleware } from "langchain";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

// Marked `defer_loading` at construction, so it's deferred on its own —
// no need to list it in `searchableTools`.
const sendEmail = tool(async () => "sent", {
  name: "send_email",
  description: "Send an email",
  schema: z.object({ to: z.string() }),
  extras: { defer_loading: true },
});

const agent = createAgent({
  model: "anthropic:claude-opus-4-8",
  tools: [sendEmail],
  middleware: [providerToolSearchMiddleware()],
});
```

### 文件系统中间件

上下文工程是构建有效智能体的主要挑战之一。当使用会返回可变长度结果的工具（例如 `web_search` 和 RAG）时，这尤其困难，因为过长的工具结果会很快填满你的上下文窗口。

来自 [Deep Agents](/oss/javascript/deepagents/overview) 的 `FilesystemMiddleware` 提供了四个用于与短期和长期记忆交互的工具：

* `ls`：列出文件系统中的文件
* `read_file`：读取整个文件或文件中的一定行数
* `write_file`：向文件系统写入一个新文件
* `edit_file`：编辑文件系统中已有的文件

```typescript
import { createAgent } from "langchain";
import { createFilesystemMiddleware } from "deepagents";

// FilesystemMiddleware is included by default in createDeepAgent
// You can customize it if building a custom agent
const agent = createAgent({
  model: "claude-sonnet-4-6",
  middleware: [
    createFilesystemMiddleware({
      backend: undefined,  // Optional: custom backend (defaults to StateBackend)
      systemPrompt: "Write to the filesystem when...",  // Optional custom system prompt override
      customToolDescriptions: {
        ls: "Use the ls tool when...",
        read_file: "Use the read_file tool to...",
      },  // Optional: Custom descriptions for filesystem tools
    }),
  ],
});
```

#### 短期与长期文件系统

默认情况下，这些工具会写入你图状态中的本地“文件系统”。要启用跨线程的持久化存储，请配置一个 `CompositeBackend`，将特定路径（例如 `/memories/`）路由到 `StoreBackend`。

```typescript
import { createAgent } from "langchain";
import { createFilesystemMiddleware, CompositeBackend, StateBackend, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph-checkpoint";

const store = new InMemoryStore();

const agent = createAgent({
  model: "claude-sonnet-4-6",
  store,
  middleware: [
    createFilesystemMiddleware({
      backend: new CompositeBackend(
        new StateBackend(),
        { "/memories/": new StoreBackend() }
      ),
      systemPrompt: "Write to the filesystem when...", // Optional custom system prompt override
      customToolDescriptions: {
        ls: "Use the ls tool when...",
        read_file: "Use the read_file tool to...",
      }, // Optional: Custom descriptions for filesystem tools
    }),
  ],
});
```

当你为 `/memories/` 配置了带 `StoreBackend` 的 `CompositeBackend` 时，任何以 **/memories/** 为前缀的文件都会保存到持久化存储中，并在不同线程之间保留。没有该前缀的文件则保留在临时的状态存储中。

### 子智能体

将任务交给子智能体可以隔离上下文，在深入处理任务的同时保持主（监督者）智能体的上下文窗口干净。

来自 [Deep Agents](/oss/javascript/deepagents/overview) 的子智能体中间件允许你通过 `task` 工具提供子智能体。

```typescript
import { tool } from "langchain";
import { createAgent } from "langchain";
import { createSubAgentMiddleware } from "deepagents";
import { z } from "zod";

const getWeather = tool(
  async ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const agent = createAgent({
  model: "claude-sonnet-4-6",
  middleware: [
    createSubAgentMiddleware({
      defaultModel: "claude-sonnet-4-6",
      defaultTools: [],
      subagents: [
        {
          name: "weather",
          description: "This subagent can get weather in cities.",
          systemPrompt: "Use the get_weather tool to get the weather in a city.",
          tools: [getWeather],
          model: "gpt-5.5",
          middleware: [],
        },
      ],
    }),
  ],
});
```

子智能体由 **name**、**description**、**system prompt** 和 **tools** 定义。你还可以为子智能体提供自定义的 **model** 或额外的 **middleware**。当你想给子智能体一个额外的状态键以便与主智能体共享时，这尤其有用。

对于更复杂的用例，你也可以提供自己预构建的 LangGraph 图作为子智能体。

```typescript
import { tool, createAgent } from "langchain";
import { createSubAgentMiddleware, type SubAgent } from "deepagents";
import { z } from "zod";

const getWeather = tool(
  async ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

const weatherSubagent: SubAgent = {
  name: "weather",
  description: "This subagent can get weather in cities.",
  systemPrompt: "Use the get_weather tool to get the weather in a city.",
  tools: [getWeather],
  model: "gpt-5.5",
  middleware: [],
};

const agent = createAgent({
  model: "claude-sonnet-4-6",
  middleware: [
    createSubAgentMiddleware({
      defaultModel: "claude-sonnet-4-6",
      defaultTools: [],
      subagents: [weatherSubagent],
    }),
  ],
});
```

除了任何用户定义的子智能体之外，主智能体始终可以访问一个 `general-purpose` 子智能体。该子智能体拥有与主智能体相同的指令，以及主智能体可访问的所有工具。`general-purpose` 子智能体的主要目的是上下文隔离——主智能体可以把复杂任务委托给这个子智能体，并在不因中间工具调用而膨胀的情况下拿回简洁的答案。

### 上下文编辑

在达到 token 上限时，通过清除较旧的工具调用输出来管理对话上下文，同时保留最近的结果。这有助于在包含大量工具调用的长对话中保持上下文窗口可控。上下文编辑适用于以下场景：

* 包含大量工具调用、超出 token 上限的长对话
* 通过移除不再相关的较旧工具输出来降低 token 成本
* 只在上下文中保留最近的 N 个工具结果

```typescript
import { createAgent, contextEditingMiddleware, ClearToolUsesEdit } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [],
  middleware: [
    contextEditingMiddleware({
      edits: [
        new ClearToolUsesEdit({
          triggerTokens: 100000,
          keep: 3,
        }),
      ],
    }),
  ],
});
```

#### 配置选项

- **`edits`** `ContextEdit[]`，默认 `[new ClearToolUsesEdit()]`：要应用的 [`ContextEdit`](https://reference.langchain.com/javascript/langchain/index/ContextEdit) 策略数组

**[`ClearToolUsesEdit`](https://reference.langchain.com/javascript/langchain/index/ClearToolUsesEdit) 选项：**

- **`triggerTokens`** `number`，默认 `100000`：触发该编辑的 token 数量。当对话超过该 token 数量时，较旧的工具输出将被清除。
- **`clearAtLeast`** `number`，默认 `0`：编辑运行时至少要回收的 token 数量。如果设为 0，则按需尽可能多地清除。
- **`keep`** `number`，默认 `3`：必须保留的最近工具结果数量。这些永远不会被清除。
- **`clearToolInputs`** `boolean`，默认 `false`：是否清除 AI 消息上发起调用的工具调用参数。当为 `true` 时，工具调用参数会被替换为空对象。
- **`excludeTools`** `string[]`，默认 `[]`：要排除在清除之外的工具有名列表。这些工具的输出永远不会被清除。
- **`placeholder`** `string`，默认 `[cleared]`：为已清除的工具输出插入的占位文本。它会替换原始工具消息内容。

#### 完整示例

该中间件会在达到 token 上限时应用上下文编辑策略。最常见的策略是 `ClearToolUsesEdit`，它会在保留最近工具结果的同时清除较旧的工具结果。

**工作原理：**

1. 监控对话中的 token 数量
2. 达到阈值时，清除较旧的工具输出
3. 保留最近的 N 个工具结果
4. 可选地为上下文保留工具调用参数

```typescript
import { createAgent, contextEditingMiddleware, ClearToolUsesEdit } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, calculatorTool, databaseTool],
  middleware: [
    contextEditingMiddleware({
      edits: [
        new ClearToolUsesEdit({
          triggerTokens: 2000,
          keep: 3,
          clearToolInputs: false,
          excludeTools: [],
          placeholder: "[cleared]",
        }),
      ],
    }),
  ],
});
```

### LLM 工具模拟器

出于测试目的，使用 LLM 模拟工具执行，用 AI 生成的响应替代实际的工具调用。LLM 工具模拟器适用于以下场景：

* 在不执行真实工具的情况下测试智能体行为。
* 在外部工具不可用或成本高昂时开发智能体。
* 在实现实际工具之前对智能体工作流进行原型设计。

```typescript
import { createAgent, toolEmulatorMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [getWeather, searchDatabase, sendEmail],
  middleware: [
    toolEmulatorMiddleware(), // Emulate all tools
  ],
});
```

#### 配置选项

- **`tools`** `(string | ClientTool | ServerTool)[]`：要模拟的工具名称（字符串）或工具实例列表。如果为 `undefined`（默认），则**所有**工具都会被模拟。如果为空数组 `[]`，则不模拟任何工具。如果是带工具名称/实例的数组，则只模拟这些工具。
- **`model`** `string | BaseChatModel`：用于生成模拟工具响应的模型。可以是模型标识符字符串（例如 `'google_genai:gemini-3.6-flash'`）或 `BaseChatModel` 实例。如果未指定，则默认为智能体的模型。

#### 完整示例

该中间件使用 LLM 为工具调用生成看似合理的响应，而不是执行实际工具。

```typescript
import { createAgent, toolEmulatorMiddleware, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  async ({ location }) => `Weather in ${location}`,
  {
    name: "get_weather",
    description: "Get the current weather for a location",
    schema: z.object({ location: z.string() }),
  }
);

const sendEmail = tool(
  async ({ to, subject, body }) => "Email sent",
  {
    name: "send_email",
    description: "Send an email",
    schema: z.object({
      to: z.string(),
      subject: z.string(),
      body: z.string(),
    }),
  }
);

// Emulate all tools (default behavior)
const agent = createAgent({
  model: "gpt-5.5",
  tools: [getWeather, sendEmail],
  middleware: [toolEmulatorMiddleware()],
});

// Emulate specific tools by name
const agent2 = createAgent({
  model: "gpt-5.5",
  tools: [getWeather, sendEmail],
  middleware: [
    toolEmulatorMiddleware({
      tools: ["get_weather"],
    }),
  ],
});

// Emulate specific tools by passing tool instances
const agent3 = createAgent({
  model: "gpt-5.5",
  tools: [getWeather, sendEmail],
  middleware: [
    toolEmulatorMiddleware({
      tools: [getWeather],
    }),
  ],
});

// Use custom model for emulation
const agent5 = createAgent({
  model: "gpt-5.5",
  tools: [getWeather, sendEmail],
  middleware: [
    toolEmulatorMiddleware({
      model: "claude-sonnet-4-6",
    }),
  ],
});
```

## 提供方专用中间件

这些中间件针对特定的 LLM 提供方进行了优化。完整细节和示例请参见各提供方的文档。

* **Anthropic** —— 面向 Claude 模型的提示词缓存、bash 工具、文本编辑器、记忆和文件搜索中间件：`/oss/javascript/integrations/middleware/anthropic`
* **AWS** —— 面向 Amazon Bedrock 模型的提示词缓存中间件：`/oss/javascript/integrations/middleware/aws`

---

> 【信息】[通过 MCP 将这些文档连接到](/use-these-docs) Claude、VSCode 等，以获得实时答案。

> 【信息】[在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/middleware/built-in.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
