# LangChain JS API 参考 · 中间件（Middleware）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 目录

- [ClearToolUsesEdit](#ClearToolUsesEdit)
- [context](#context)
- [ContextEdit](#ContextEdit)
- [contextEditingMiddleware](#contextEditingMiddleware)
- [ContextEditingMiddlewareConfig](#ContextEditingMiddlewareConfig)
- [countTokensApproximately](#countTokensApproximately)
- [detectCreditCard](#detectCreditCard)
- [detectEmail](#detectEmail)
- [detectIP](#detectIP)
- [detectMacAddress](#detectMacAddress)
- [detectUrl](#detectUrl)
- [dynamicSystemPromptMiddleware](#dynamicSystemPromptMiddleware)
- [humanInTheLoopMiddleware](#humanInTheLoopMiddleware)
- [llmToolSelectorMiddleware](#llmToolSelectorMiddleware)
- [modelCallLimitMiddleware](#modelCallLimitMiddleware)
- [modelFallbackMiddleware](#modelFallbackMiddleware)
- [modelRetryMiddleware](#modelRetryMiddleware)
- [openAIModerationMiddleware](#openAIModerationMiddleware)
- [OpenAIModerationMiddlewareOptions](#OpenAIModerationMiddlewareOptions)
- [PIIDetector](#PIIDetector)
- [PIIMatch](#PIIMatch)
- [piiMiddleware](#piiMiddleware)
- [PIIMiddlewareConfig](#PIIMiddlewareConfig)
- [piiRedactionMiddleware](#piiRedactionMiddleware)
- [PIIRedactionMiddlewareConfig](#PIIRedactionMiddlewareConfig)
- [PIIStrategy](#PIIStrategy)
- [PromptCachingMiddlewareConfig](#PromptCachingMiddlewareConfig)
- [providerToolSearchMiddleware](#providerToolSearchMiddleware)
- [ProviderToolSearchMiddlewareConfig](#ProviderToolSearchMiddlewareConfig)
- [RedactionRuleConfig](#RedactionRuleConfig)
- [ResolvedRedactionRule](#ResolvedRedactionRule)
- [resolveRedactionRule](#resolveRedactionRule)
- [summarizationMiddleware](#summarizationMiddleware)
- [SummarizationMiddlewareConfig](#SummarizationMiddlewareConfig)
- [Todo](#Todo)
- [todoListMiddleware](#todoListMiddleware)
- [TodoListMiddlewareOptions](#TodoListMiddlewareOptions)
- [ToolCallLimitConfig](#ToolCallLimitConfig)
- [toolCallLimitMiddleware](#toolCallLimitMiddleware)
- [toolEmulatorMiddleware](#toolEmulatorMiddleware)
- [toolErrorMiddleware](#toolErrorMiddleware)
- [toolRetryMiddleware](#toolRetryMiddleware)
- [ToolEmulatorOptions](#ToolEmulatorOptions)
- [ToolErrorHandler](#ToolErrorHandler)
- [ToolErrorMiddlewareConfig](#ToolErrorMiddlewareConfig)
- [ToolRetryMiddlewareConfig](#ToolRetryMiddlewareConfig)

---

## 函数
### `anthropicPromptCachingMiddleware`
导入：`import { anthropicPromptCachingMiddleware } from "langchain";`

```javascript
anthropicPromptCachingMiddleware(middlewareOptions?: Partial<__type>): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

为 Anthropic 模型创建提示词缓存中间件，以优化 API 使用。

该中间件在odor使用 Anthropic 模型时，会自动为最后的消息添加缓存控制头，从而启用其提示词缓存功能。对于具有重复提示词、长系统消息或广泛对话历史的应用程序，这可以显著降低成本。

**工作原理**：
中间件拦截模型请求，并添加缓存控制元数据，指示 Anthropic 的 API 缓存处理后的提示词前缀。在后续匹配前缀的请求中，缓存的表示将被重用，从而跳过冗余的标记处理。

**优势**：
- **成本降低**：避免重复处理相同的标记（缓存部分可节省高达 90%）
- **延迟更低**：缓存的提示词处理更快，因为嵌入已被预计算
- **更好的可扩展性**：减少计算负载可处理更多请求
- **一致的性能**：重复查询的响应时间稳定

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `middlewareOptions` | `Partial<__type>` | No | 缓存行为的配置选项 |

**返回**：
`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`
可传递给 `createAgent` 的中间件实例

**示例**：
### 基本用法（默认设置）
```typescript
import { createAgent } from "langchain";
import { anthropicPromptCachingMiddleware } from "langchain";

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  middleware: [
    anthropicPromptCachingMiddleware()
  ]
});
```

### 更长对话的自定义配置
```typescript
const cachingMiddleware = anthropicPromptCachingMiddleware({
  ttl: "1h",  // 缓存 1 小时，而非默认的 5 分钟
  minMessagesToCache: 5  // 仅在消息超过 5 条后缓存
});

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  systemPrompt: "You are a helpful assistant with deep knowledge of...", // 长系统提示词
  middleware: [cachingMiddleware]
});
```

### 基于运行时的条件缓存
```typescript
const agent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  middleware: [
    anthropicPromptCachingMiddleware({
      enableCaching: true,
      ttl: "5m"
    })
  ]
});

// 为特定请求禁用缓存
await agent.invoke(
  { messages: [new HumanMessage("Process this without caching")] },
  {
    configurable: {
      middleware_context: { enableCaching: false }
    }
  }
);
```

### 客户支持聊天机器人的最优设置
```typescript
const supportAgent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  systemPrompt: `You are a customer support agent for ACME Corp.

    Company policies:
    - Always be polite and professional
    - Refer to knowledge base for product information
    - Escalate billing issues to human agents
    ... (extensive policies and guidelines)
  `,
  tools: [searchKnowledgeBase, createTicket, checkOrderStatus],
  middleware: [
    anthropicPromptCachingMiddleware({
      ttl: "1h",  // 为稳定的系统提示词提供长 TTL
      minMessagesToCache: 1  // 由于巨大的系统提示词，立即缓存
    })
  ]
});
```

### `applyStrategy`
导入：`import { applyStrategy } from "langchain";`

```javascript
applyStrategy(
  content : string , 
  matches : PIIMatch [ ] , 
  strategy : PIIStrategy , 
  piiType : string 
) : string
```

根据匹配项对内容应用策略。

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `content` | `* string` | * | 内容 |
| `matches` | `* PIIMatch []` | * | 匹配项 |
| `strategy` | `* PIIStrategy` | * | 策略 |
| `piiType` | `* string` | * | PII 类型 |

## 类型
### `BedrockConversePromptCachingMiddlewareConfig`
导入：`import { BedrockConversePromptCachingMiddlewareConfig } from "langchain";`

```typescript
BedrockConversePromptCachingMiddlewareConfig : Partial < InferInteropZodInput < typeof contextSchema > >
```

BedrockConversePromptCachingMiddlewareConfig 类型。

### `BuiltInPIIType`
导入：`import { BuiltInPIIType } from "langchain";`

```typescript
BuiltInPIIType : "email" | "credit_card" | "ip" | "mac_address" | "url"
```

内置 PII 类型。

## 函数
### `bedrockPromptCachingMiddleware`
导入：`import { bedrockPromptCachingMiddleware } from "langchain";`

```javascript
bedrockPromptCachingMiddleware(middlewareOptions?: Partial<__type>): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

为 AWS Bedrock Converse 模型创建提示词缓存中间件，以优化 API 使用。

该中间件在odor使用 AWS Bedrock Converse 模型时，会自动启用 Bedrock 的提示词缓存。这可以显著降低具有重复提示词、长系统消息或广泛对话历史的应用程序的成本。

**工作原理**：
中间件拦截模型请求并设置缓存控制信号，`ChatBedrockConverse` 将其转换为 Bedrock `cachePoint` 断点。缓存点插入在系统提示词之后、工具定义之后以及最后一条消息之后，因此每个请求的稳定前缀都会被缓存。在后续匹配前缀的请求中，缓存的表示将被重用，从而跳过冗余的标记处理。精确位置因模型而异（例如，Amazon Nova 模型缓存的断点较少并忽略 `"1h"` TTL）。

**优势**：
- **成本降低**：避免重复处理相同的标记
- **延迟更低**：缓存的提示词处理更快，因为嵌入已被预计算
- **更好的可扩展性**：减少计算负载可处理更多请求
- **一致的性能**：重复查询的响应时间稳定

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `middlewareOptions` | `Partial<__type>` | No | 缓存行为的配置选项 |

**返回**：
`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`
可传递给 `createAgent` 的中间件实例

**示例**：
### 基本用法（默认设置）
```typescript
import { createAgent } from "langchain";
import { bedrockPromptCachingMiddleware } from "langchain";

const agent = createAgent({
  model: "bedrock:anthropic.claude-haiku-4-5-20251001-v1:0",
  middleware: [
    bedrockPromptCachingMiddleware()
  ]
});
```

### 更长对话的自定义配置
```typescript
const cachingMiddleware = bedrockPromptCachingMiddleware({
  ttl: "1h",  // 缓存 1 小时，而非默认的 5 分钟
  minMessagesToCache: 5  // 仅在消息超过 5 条后缓存
});

const agent = createAgent({
  model: "bedrock:anthropic.claude-haiku-4-5-20251001-v1:0",
  systemPrompt: "You are a helpful assistant with deep knowledge of...", // 长系统提示词
  middleware: [cachingMiddleware]
});
```

### 基于运行时的条件缓存
```typescript
const agent = createAgent({
  model: "bedrock:anthropic.claude-haiku-4-5-20251001-v1:0",
  middleware: [
    bedrockPromptCachingMiddleware({
      enableCaching: true,
      ttl: "5m"
    })
  ]
});

// 为特定请求禁用缓存
await agent.invoke(
  { messages: [new HumanMessage("Process this without caching")] },
  {
    configurable: {
      middleware_context: { enableCaching: false }
    }
  }
);
```

### 客户支持聊天机器人的最优设置
```typescript
const supportAgent = createAgent({
  model: "bedrock:anthropic.claude-haiku-4-5-20251001-v1:0",
  systemPrompt: `You are a customer support agent for ACME Corp.

    Company policies:
    - Always be polite and professional
    - Refer to knowledge base for product information
    - Escalate billing issues to human agents
    ... (extensive policies and guidelines)
  `,
  tools: [searchKnowledgeBase, createTicket, checkOrderStatus],
  middleware: [
    bedrockPromptCachingMiddleware({
      ttl: "1h",  // 为稳定的系统提示词提供长 TTL
      minMessagesToCache: 1  // 由于巨大的系统提示词，立即缓存
    })
  ]
});
```

## 类
### `ClearToolUsesEdit`
<a id="ClearToolUsesEdit"></a>

导入：
```ts
import { ClearToolUsesEdit } from "langchain";
```

签名：
```javascript
class ClearToolUsesEdit
```

说明：
用于当超出 Token 限制时清除工具输出的策略。该策略镜像了 Anthropic 的 `clear_tool_uses_20250919` 行为，当对话内容过大时，用占位符文本替换较旧的工具结果。它保留最近的工具结果，并且可以排除特定的工具不被清除。

属性：
| 属性 | 类型 | 说明 |
|------|------|-------------|
| `clearAtLeast` | number | (deprecated) |
| `clearToolInputs` | boolean | 是否在 AI 消息上清除原始工具调用参数。 |
| `excludeTools` | string [] | 排除不被清除的工具名称列表。 |
| `keep` | __ type | 编辑后应用的上下文保留策略。可使用 `messages`、`tokens` 或 `fraction` 指定保留多少个工具结果。 |
| `model` | __ type | |
| `placeholder` | string | 已清除工具输出的占位符文本。 |
| `trigger` | __ type | __ type [] | 上下文编辑的触发条件。可以是单个条件对象（所有属性必须满足）或条件数组（满足任一条件即可）。 |

方法：
| 方法 | 签名 | 说明 |
|------|------|-------------|
| `constructor` | `constructor ( config : ClearToolUsesEditConfig = { } ) : ClearToolUsesEdit` | |
| `apply` | `apply ( params : __type ) : Promise < void >` | 应用编辑操作，并返回新的 Token 计数。此方法应：根据参数中的 Token 判断是否需要编辑；原地修改消息数组（如果需要）；返回修改后的新 Token 计数。 |

示例：
```ts
import { ClearToolUsesEdit } from "langchain";

const edit = new ClearToolUsesEdit({
  trigger: { tokens: 100000 },  // Start clearing at 100K tokens
  keep: { messages: 3 },        // Keep 3 most recent tool results
  excludeTools: ["important"],   // Never clear "important" tool
  clearToolInputs: false,        // Keep tool call arguments
  placeholder: "[cleared]",      // Replacement text
});

// Multiple trigger conditions
const edit2 = new ClearToolUsesEdit({
  trigger: [
    { tokens: 100000, messages: 50 },
    { tokens: 50000, messages: 100 }
  ],
  keep: { messages: 3 },
});

// Fractional trigger with model profile
const edit3 = new ClearToolUsesEdit({
  trigger: { fraction: 0.8 },  // Trigger at 80% of model's max tokens
  keep: { fraction: 0.3 },     // Keep 30% of model's max tokens
});
```

## 函数
### `context`
<a id="context"></a>

导入：
```ts
import { context } from "langchain";
```

签名：
```javascript
context ( strings : TemplateStringsArray , values : unknown [ ] ) : string
```

说明：
LangChain 工具。

参数：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `strings` | `TemplateStringsArray` | * | |
| `values` | `unknown []` | * | |

## 接口
### `ContextEdit`
<a id="ContextEdit"></a>

导入：
```ts
import { ContextEdit } from "langchain";
```

签名：
```javascript
interface ContextEdit
```

说明：
描述上下文编辑策略的协议。实现此接口以创建管理对话上下文大小的自定义策略。`apply` 方法应原地修改消息数组并返回更新后的 Token 计数。

方法：
| 方法 | 签名 | 说明 |
|------|------|-------------|
| `apply` | `apply() ` | |

示例：
```ts
import { HumanMessage, type ContextEdit, type BaseMessage  } from "langchain";

class RemoveOldHumanMessages implements ContextEdit {
  constructor(private keepRecent: number = 10) {}

  async apply({ messages, countTokens }) {
    // Check current token count
    const tokens = await countTokens(messages);

    // Remove old human messages if over limit, keeping the most recent ones
    if (tokens > 50000) {
      const humanMessages: number[] = [];

      // Find all human message indices
      for (let i = 0; i < messages.length; i++) {
        if (HumanMessage.isInstance(messages[i])) {
          humanMessages.push(i);
        }
      }

      // Remove old human messages (keep only the most recent N)
      const toRemove = humanMessages.slice(0, -this.keepRecent);
      for (let i = toRemove.length - 1; i >= 0; i--) {
        messages.splice(toRemove[i]!, 1);
      }
    }
  }
}
```

## 函数
### `contextEditingMiddleware`
<a id="contextEditingMiddleware"></a>

导入：
```ts
import { contextEditingMiddleware } from "langchain";
```

签名：
```javascript
contextEditingMiddleware(config: ContextEditingMiddlewareConfig = {}): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```

说明：
中间件，自动修剪工具结果以管理上下文大小。该中间件在总输入 Token 计数超过配置的阈值时，按顺序应用一系列编辑。默认情况下，它使用 `ClearToolUsesEdit` 策略，镜像了 Anthropic 的 `clear_tool_uses_20250919` 行为，即在对话超过 100,000 Token 时清除较旧的工具结果。

参数：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `config` | `ContextEditingMiddlewareConfig` | No | 中间件的配置选项（默认：`{}`） |

返回：
`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`
一个可用于 `createAgent` 的中间件实例。

示例：
### Example 1
```ts
import { contextEditingMiddleware } from "langchain";
import { createAgent } from "langchain";

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  tools: [searchTool, calculatorTool],
  middleware: [
    contextEditingMiddleware(),
  ],
});
```

### Example 2
```ts
import { contextEditingMiddleware, ClearToolUsesEdit } from "langchain";

// Single condition: trigger if tokens >= 50000 AND messages >= 20
const agent1 = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  tools: [searchTool, calculatorTool],
  middleware: [
    contextEditingMiddleware({
      edits: [
        new ClearToolUsesEdit({
          trigger: { tokens: 50000, messages: 20 },
          keep: { messages: 5 },
          excludeTools: ["search"],
          clearToolInputs: true,
        }),
      ],
      tokenCountMethod: "approx",
    }),
  ],
});

// Multiple conditions: trigger if (tokens >= 50000 AND messages >= 20) OR (tokens >= 30000 AND messages >= 50)
const agent2 = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  tools: [searchTool, calculatorTool],
  middleware: [
    contextEditingMiddleware({
      edits: [
        new ClearToolUsesEdit({
          trigger: [
            { tokens: 50000, messages: 20 },
            { tokens: 30000, messages: 50 },
          ],
          keep: { messages: 5 },
        }),
      ],
    }),
  ],
});

// Fractional trigger with model profile
const agent3 = createAgent({
  model: chatModel,
  tools: [searchTool, calculatorTool],
  middleware: [
    contextEditingMiddleware({
      edits: [
        new ClearToolUsesEdit({
          trigger: { fraction: 0.8 },  // Trigger at 80% of model's max tokens
          keep: { fraction: 0.3 },     // Keep 30% of model's max tokens
          model: chatModel,
        }),
      ],
    }),
  ],
});
```

### Example 3
```ts
import { contextEditingMiddleware, type ContextEdit, type TokenCounter } from "langchain";
import type { BaseMessage } from "@langchain/core/messages";

class CustomEdit implements ContextEdit {
  async apply(params: {
    tokens: number;
    messages: BaseMessage[];
    countTokens: TokenCounter;
  }): Promise<number> {
    // Implement your custom editing logic here
    // and apply it to the messages array, then
    // return the new token count after edits
    return countTokens(messages);
  }
}
```

## 接口
### `ContextEditingMiddlewareConfig`
<a id="ContextEditingMiddlewareConfig"></a>

导入：
```ts
import { ContextEditingMiddlewareConfig } from "langchain";
```

签名：
```javascript
interface ContextEditingMiddlewareConfig
```

说明：
用于 Context Editing Middleware 的配置。

属性：
| 属性 | 类型 | 说明 |
|------|------|-------------|
| `edits` | `ContextEdit []` | 要应用的编辑策略序列。默认为镜像 Anthropic 默认值的单个 `ClearToolUsesEdit`。 |
| `tokenCountMethod` | `"model" | "approx"` | 是否使用近似 Token 计数（更快，但不太准确）或由聊天模型实现的精确计数（可能较慢，但更准确）。目前仅 OpenAI 模型支持精确计数。 |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/dynamicSystemPromptMiddleware
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖 LangChain 中的中间件相关功能，包括动态系统提示词以及人类介入（Human-in-the-Loop）机制。

## 函数
### `countTokensApproximately`
<a id="countTokensApproximately"></a>
import { countTokensApproximately } from "langchain";

```ts
countTokensApproximately(
  messages: BaseMessage<MessageStructure<MessageToolSet>, MessageType>[],
  tools: Record<string, any>[] | null
): number
```

默认基于字符数估算 token 数量。如果提供了工具，token 计数还会包含字符串化的工具架构。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| messages | `BaseMessage<MessageStructure<MessageToolSet>, MessageType>[]` | * | 用于计算 token 的消息。 |
| tools | `Record<string, any>[] | null` | 可选 | 要包含在 token 计数中的工具列表。每个工具可以是 LangChain 工具实例或代表工具架构的字典。LangChain 工具实例在计数前将转换为 OpenAI 工具格式。 |

### `detectCreditCard`
<a id="detectCreditCard"></a>
import { detectCreditCard } from "langchain";

```ts
detectCreditCard(content: string): PIIMatch[]
```

在内容中检测信用卡号码（使用 Luhn 算法验证）。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| content | string | * | 内容。 |

### `detectEmail`
<a id="detectEmail"></a>
import { detectEmail } from "langchain";

```ts
detectEmail(content: string): PIIMatch[]
```

在内容中检测电子邮件地址。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| content | string | * | 内容。 |

### `detectIP`
<a id="detectIP"></a>
import { detectIP } from "langchain";

```ts
detectIP(content: string): PIIMatch[]
```

在内容中检测 IP 地址（已验证）。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| content | string | * | 内容。 |

### `detectMacAddress`
<a id="detectMacAddress"></a>
import { detectMacAddress } from "langchain";

```ts
detectMacAddress(content: string): PIIMatch[]
```

在内容中检测 MAC 地址。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| content | string | * | 内容。 |

### `detectUrl`
<a id="detectUrl"></a>
import { detectUrl } from "langchain";

```ts
detectUrl(content: string): PIIMatch[]
```

在内容中检测 URL。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| content | string | * | 内容。 |

### `dynamicSystemPromptMiddleware`
<a id="dynamicSystemPromptMiddleware"></a>
import { dynamicSystemPromptMiddleware } from "langchain";

```javascript
dynamicSystemPromptMiddleware<TContextSchema = unknown>(fn: DynamicSystemPromptMiddlewareConfig<TContextSchema>): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```

动态系统提示词中间件。

允许在每次模型调用之前动态设置系统提示词。当提示词取决于当前代理状态或单次调用上下文时非常有用。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `fn` | `DynamicSystemPromptMiddlewareConfig<TContextSchema>` | 是 | 接收当前代理 `state` 和 `runtime`，并返回下一次模型调用的系统提示词字符串的函数。 |

`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`

一个为下一次模型调用设置 `systemPrompt` 的中间件实例。

```ts
import { z } from "zod";
import { dynamicSystemPrompt } from "langchain";
import { createAgent, SystemMessage } from "langchain";

const contextSchema = z.object({ region: z.string().optional() });

const middleware = dynamicSystemPrompt<z.infer<typeof contextSchema>>(
  (_state, runtime) => `You are a helpful assistant. Region: ${runtime.context.region ?? "n/a"}`
);

const agent = createAgent({
  model: "anthropic:claude-sonnet-4-5",
  contextSchema,
  middleware: [middleware],
});

await agent.invoke({ messages }, { context: { region: "EU" } });
```

### `humanInTheLoopMiddleware`
<a id="humanInTheLoopMiddleware"></a>
import { humanInTheLoopMiddleware } from "langchain";

```javascript
humanInTheLoopMiddleware(options: __type): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

创建用于工具审批和监督的人类介入（Human-in-the-Loop, HITL）中间件。

此中间件拦截 AI 代理发出的工具调用，并在执行前提供人类监督能力。它启用了选择性审批工作流，其中某些工具需要人类干预，而其他工具可以自动执行。被中间件中断的调用结果将具有 `__interrupt__` 属性，其中包含中断请求。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `options` | `__type` | 是 | 中间件的配置选项。 |

`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`

可传递给 `createAgent` 的中间件实例。

#### 基本用法（选择性工具审批）
```typescript
import { humanInTheLoopMiddleware } from "langchain";
import { createAgent } from "langchain";

const hitlMiddleware = humanInTheLoopMiddleware({
  interruptOn: {
    // 中断 write_file 工具并允许编辑或审批
    "write_file": {
      allowedDecisions: ["approve", "edit"],
      description: "File write operation requires approval"
    },
    // 自动审批 read_file 工具
    "read_file": false
  }
});

const agent = createAgent({
  model: "openai:gpt-4",
  tools: [writeFileTool, readFileTool],
  middleware: [hitlMiddleware]
});

await agent.invoke({ messages }, { context: { region: "EU" } });
```

#### 处理审批请求
```typescript
import { type HITLRequest, type HITLResponse, type Interrupt } from "langchain";
import { Command } from "@langchain/langgraph";

// 初始代理调用
const result = await agent.invoke({
  messages: [new HumanMessage("Write 'Hello' to output.txt")]
}, config);

// 检查代理是否因审批而暂停
if (result.__interrupt__) {
  const interruptRequest = result.__interrupt__?.[0] as Interrupt<HITLRequest>;

  // 向用户显示操作请求
  console.log("Actions:", interruptRequest.value.actionRequests);
  console.log("Review configs:", interruptRequest.value.reviewConfigs);

  // 携带决策恢复
  const resume: HITLResponse = {
    decisions: [{ type: "approve" }]
  };
  await agent.invoke(
    new Command({ resume }),
    config
  );
}
```

#### 不同决策类型
```typescript
import { type HITLResponse } from "langchain";

// 按原样批准工具调用
const resume: HITLResponse = {
  decisions: [{ type: "approve" }]
};

// 在执行前修改工具参数
const resume: HITLResponse = {
  decisions: [{
    type: "edit",
    editedAction: { name: "write_file", args: { filename: "safe.txt", content: "Modified" } }
  }]
};

// 提供反馈并拒绝
const resume: HITLResponse = {
  decisions: [{
    type: "reject",
    message: "File operation not allowed in demo mode"
  }]
};
```

#### 生产环境用例（数据库操作）
```typescript
const hitlMiddleware = humanInTheLoopMiddleware({
  interruptOn: {
    "execute_sql": {
      allowedDecisions: ["approve", "edit", "reject"],
      description: "SQL query requires DBA approval\nPlease review for safety and performance"
    },
    "read_schema": false,  // 读取元数据是安全的
    "delete_records": {
      allowedDecisions: ["approve", "reject"],
      description: "DESTRUCTIVE OPERATION - Requires manager approval"
    }
  },
  descriptionPrefix: "Database operation pending approval"
});
```

#### 使用动态可调用描述
```typescript
import { type DescriptionFactory, type ToolCall } from "langchain";
import type { AgentBuiltInState, Runtime } from "langchain/agents";

// 定义动态描述工厂
const formatToolDescription: DescriptionFactory = (
  toolCall: ToolCall,
  state: AgentBuiltInState,
  runtime: Runtime<unknown>
) => {
  return `Tool: ${toolCall.name}\nArguments:\n${JSON.stringify(toolCall.args, null, 2)}`;
};

const hitlMiddleware = humanInTheLoopMiddleware({
  interruptOn: {
    "write_file": {
      allowedDecisions: ["approve", "edit"],
      // 使用可访问工具调用、状态和运行时的动态描述
      description: formatToolDescription
    },
    // 或者使用内联函数
    "send_email": {
      allowedDecisions: ["approve", "reject"],
      description: (toolCall, state, runtime) => {
        const { to, subject } = toolCall.args;
        return `Email to ${to}\nSubject: ${subject}\n\nRequires approval before sending`;
      }
    }
  }
});
```

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/llmToolSelectorMiddleware
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖中间件模块的内容。

## 类型
### `InferAgentMiddleware`
类型 `InferAgentMiddleware` 是一个简写辅助工具，用于从 `AgentTypeConfig` 或 `ReactAgent` 中提取中间件类型。

### `LLMToolSelectorConfig`
类型 `LLMToolSelectorConfig` 是 `InferInteropZodInput < typeof LLMToolSelectorOptionsSchema >`。

## 函数
### `llmToolSelectorMiddleware`
<a id="llmToolSelectorMiddleware"></a>
用于选择使用基于 LLM 策略的工具的中间件。

当代理有许多可用工具时，此中间件会将其过滤为与用户查询最相关的工具。这可以减少 Token 使用量，并帮助主模型专注于正确的工具。

**导入**：
```ts
import { llmToolSelectorMiddleware } from "langchain/agents/middleware";
```

**签名**：
```javascript
llmToolSelectorMiddleware(options: __type): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `options` | `__type` | 是 | 中间件的配置选项 |

**返回**：
`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`

**示例**：
### 限制为 3 个工具：
```ts
import { llmToolSelectorMiddleware } from "langchain/agents/middleware";

const middleware = llmToolSelectorMiddleware({ maxTools: 3 });

const agent = createAgent({
  model: "openai:gpt-4o",
  tools: [tool1, tool2, tool3, tool4, tool5],
  middleware: [middleware],
});
```

### 使用较小的模型进行选择：
```ts
const middleware = llmToolSelectorMiddleware({
  model: "openai:gpt-4o-mini",
  maxTools: 2
});
```

### `modelCallLimitMiddleware`
<a id="modelCallLimitMiddleware"></a>
创建一个中间件来限制线程和运行层级的模型调用数量。

此中间件通过强制限制模型可调用的次数来防止过度的模型 API 调用。它支持两种类型的限制：

- **线程级限制**：限制整个会话线程中的模型总调用次数
- **运行级限制**：限制单个代理运行/调用的模型调用次数

**工作原理**：
该中间件在模型请求发送前拦截请求，并检查当前的调用计数是否符合配置的限制。如果任何限制都被超过，它将抛出 `ModelCallLimitMiddlewareError` 以停止执行并防止进一步的 API 调用。

**使用场景**：
- **成本控制**：防止生产环境中过度模型调用导致的成本失控
- **测试**：确保代理在开发/测试期间不会发起过多调用
- **安全**：限制潜在的无限循环或递归代理行为
- **速率限制**：强制执行组织关于每个会话模型使用的政策

**导入**：
```ts
import { createAgent, modelCallLimitMiddleware } from "langchain";
```

**签名**：
```javascript
modelCallLimitMiddleware(middlewareOptions?: Partial<__type>): AgentMiddleware<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `middlewareOptions` | `Partial<__type>` | 否 | 调用限制的配置选项 |

**返回**：
`AgentMiddleware<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`
一个可以传递给 `createAgent` 的中间件实例。

**示例**：
### 示例 1
```typescript
import { createAgent, modelCallLimitMiddleware } from "langchain";

// 限制每个线程 10 次调用，每个运行 3 次调用
const agent = createAgent({
  model: "openai:gpt-4o-mini",
  tools: [myTool],
  middleware: [
    modelCallLimitMiddleware({
      threadLimit: 10,
      runLimit: 3
    })
  ]
});
```

### 示例 2
```typescript
// 限制也可以在运行时通过上下文配置
const result = await agent.invoke(
  { messages: ["Hello"] },
  {
    configurable: {
      threadLimit: 5  // 为此运行覆盖默认限制
    }
  }
);
```

### `modelFallbackMiddleware`
<a id="modelFallbackMiddleware"></a>
提供错误时自动模型回退的中间件。

此中间件尝试按顺序使用备用模型重新尝试失败的模型调用。当模型调用失败时，它会尝试回退列表中的下一个模型，直到调用成功或所有模型耗尽。

**导入**：
```ts
import { createAgent, modelFallbackMiddleware } from "langchain";
```

**签名**：
```javascript
modelFallbackMiddleware(fallbackModels: string | AgentLanguageModelLike[]): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `fallbackModels` | `string \| AgentLanguageModelLike[]` | 是 | 按顺序尝试的备用模型 |

**返回**：
`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`
一个处理模型失败并提供回退机制的中间件实例。

**示例**：
```ts
import { createAgent, modelFallbackMiddleware } from "langchain";

// 创建带有备用模型（不包含主要模型）的中间件
const fallback = modelFallbackMiddleware(
  "openai:gpt-4o-mini",  // 第一个备用模型
  "anthropic:claude-sonnet-4-5-20250929",  // 第二个备用模型
);

const agent = createAgent({
  model: "openai:gpt-4o",  // 主要模型
  middleware: [fallback],
  tools: [],
});

// 如果 gpt-4o 失败，将自动尝试 gpt-4o-mini，然后是 claude
const result = await agent.invoke({
  messages: [{ role: "user", content: "Hello" }]
});
```

### `modelRetryMiddleware`
<a id="modelRetryMiddleware"></a>
自动带配置的可回退重试失败模型调用的中间件。

支持在特定异常上进行重试并进行指数级退避。

**导入**：
```ts
import { createAgent, modelRetryMiddleware } from "langchain";
```

**签名**：
```javascript
modelRetryMiddleware(config: __type = {}): AgentMiddleware<undefined, ZodObject<__type & __type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `config` | `__type` | 否 | 重试中间件的配置选项（默认：`{}`） |

**返回**：
`AgentMiddleware<undefined, ZodObject<__type & __type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`
一个处理模型失败并提供重试机制的中间件实例。

**示例**：
### 示例 1
```ts
import { createAgent, modelRetryMiddleware } from "langchain";

const agent = createAgent({
  model: "openai:gpt-4o",
  tools: [searchTool],
  middleware: [modelRetryMiddleware()],
});
```

### 示例 2
```ts
import { modelRetryMiddleware } from "langchain";

const retry = modelRetryMiddleware({
  maxRetries: 4,
  retryOn: [TimeoutError, NetworkError],
  backoffFactor: 1.5,
});
```

### 示例 3
```ts
function shouldRetry(error: Error): boolean {
  // 仅在速率限制错误时重试
  if (error.name === "RateLimitError") {
    return true;
  }
  // 或者检查特定的 HTTP 状态码
  if (error.name === "HTTPError" && "statusCode" in error) {
    const statusCode = (error as any).statusCode;
    return statusCode === 429 || statusCode === 503;
  }
  return false;
}

const retry = modelRetryMiddleware({
  maxRetries: 3,
  retryOn: shouldRetry,
});
```

### 示例 4
```ts
const retry = modelRetryMiddleware({
  maxRetries: 4,
  onFailure: "continue", // 失败时返回带错误信息的 AIMessage 而不是抛出异常
});
```

### 示例 5
```ts
const formatError = (error: Error) =>
  `Model call failed: ${error.message}. Please try again later.`;

const retry = modelRetryMiddleware({
  maxRetries: 4,
  onFailure: formatError,
});
```

### 示例 6
```ts
const retry = modelRetryMiddleware({
  maxRetries: 5,
  backoffFactor: 0.0, // 无指数增长
  initialDelayMs: 2000, // 始终等待 2 秒
});
```

### 示例 7
```ts
const retry = modelRetryMiddleware({
  maxRetries: 2,
  onFailure: "error", // 失败时重新抛出异常而不是返回消息
});
```

### `openAIModerationMiddleware`
<a id="openAIModerationMiddleware"></a>
提供器特定的中间件，用于 OpenAI 审核。

**导入**：
```ts
import { openAIModerationMiddleware } from "langchain";
```

**签名**：
```javascript
openAIModerationMiddleware(options: OpenAIModerationMiddlewareOptions): AgentMiddleware
```

**参数**：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `options` | `OpenAIModerationMiddlewareOptions` | 是 | 中间件的配置选项 |

**用于文档的 OpenAI 集成参数**
| 属性 | 类型 | 说明 |
|------|------|-------------|
| `options` | `OpenAIModerationMiddlewareOptions` | 中间件的配置选项 |

## 接口
### `OpenAIModerationMiddlewareOptions`
<a id="OpenAIModerationMiddlewareOptions"></a>
import { OpenAIModerationMiddlewareOptions } from "langchain";

用于配置 OpenAI Moderation 中间件的选项。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `checkInput` | `boolean` | 是否检查用户输入消息。 |
| `checkOutput` | `boolean` | 是否检查模型输出消息。 |
| `checkToolResults` | `boolean` | 是否检查工具结果消息。 |
| `exitBehavior` | `"end" | "error" | "replace"` | 违反规范时的处理方式。<br>"error"：当内容被标记时抛出错误<br>"end"：停止代理执行并返回违反消息<br>"replace"：用违反消息替换被标记的内容 |
| `model` | `string | BaseChatModel < BaseChatModelCallOptions , AIMessageChunk < MessageStructure < MessageToolSet >>>` | 用于审核的 OpenAI 模型。可以是模型名称或 `BaseChatModel` 实例。 |
| `moderationModel` | `ModerationModel` | 用于审核的审核模型。 |
| `violationMessage` | `string` | 违反规范消息的自定义模板。可用占位符：`{categories}`, `{category_scores}`, `{original_content}` |

## 类型
### `PIIDetector`
<a id="PIIDetector"></a>
处理内容并返回匹配项的自定义检测函数。

```ts
PIIDetector : ( content : string ) = > PIIMatch [ ]
```

## 类型
### `PIIMatch`
<a id="PIIMatch"></a>
表示内容中检测到的 PII 匹配项。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `end` | `number` | 匹配项的结束索引。 |
| `start` | `number` | 匹配项的起始索引。 |
| `text` | `string` | 匹配到的文本。 |

## 函数
### `piiMiddleware`
<a id="piiMiddleware"></a>
import { piiMiddleware } from "langchain";

创建一个用于检测和处理对话中个人身份信息 (PII) 的中间件。

该中间件可检测常见的 PII 类型，并应用可配置的策略来处理它们。
它可以检测电子邮件、信用卡、IP 地址、MAC 地址和 URL，无论是在用户输入还是代理输出中。

内置 PII 类型：
- `email`: 电子邮件地址
- `credit_card`: 信用卡号码（通过 Luhn 算法验证）
- `ip`: IP 地址（已验证）
- `mac_address`: MAC 地址
- `url`: URL（包括 `http`/`https` 和裸 URL）

策略：
- `block`: 检测到 PII 时抛出异常
- `redact`: 用 `[REDACTED_TYPE]` 占位符替换 PII
- `mask`: 部分掩盖 PII（例如，信用卡显示为 `****-****-****-1234`）
- `hash`: 用确定性哈希替换 PII（例如，`<email_hash:a1b2c3d4>`）

策略选择指南：
| 策略 | 保留身份？ | 适用场景 |
| --- | --- | --- |
| `block` | N/A | 完全避免 PII |
| `redact` | 否 | 常规合规、日志清理 |
| `mask` | 否 | 人类可读性、客服 UI |
| `hash` | 是 (伪匿名) | 分析、调试 |

```ts
piiMiddleware(piiType: string, options: __type = {}): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `piiType` | `string` | 是 | 要检测的 PII 类型。可以是内置类型（`email`, `credit_card`, `ip`, `mac_address`, `url`）或自定义类型名称。 |
| `options` | `__type` | 否 | 配置选项（默认：`{}`） |

`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`

用于与 `createAgent` 一起使用的中间件实例。

### Example 1
```typescript
import { piiMiddleware } from "langchain";
import { createAgent } from "langchain";

// Redact all emails in user input
const agent = createAgent({
  model: "openai:gpt-4",
  middleware: [
    piiMiddleware("email", { strategy: "redact" }),
  ],
});
```

### Example 2
```typescript
const agent = createAgent({
  model: "openai:gpt-4o",
  middleware: [
    piiMiddleware("credit_card", { strategy: "mask" }),
    piiMiddleware("url", { strategy: "redact" }),
    piiMiddleware("ip", { strategy: "hash" }),
  ],
});
```

### Example 3
```typescript
const agent = createAgent({
  model: "openai:gpt-4",
  middleware: [
    piiMiddleware("api_key", {
      detector: "sk-[a-zA-Z0-9]{32}",
      strategy: "block",
    }),
  ],
});
```

## 类型
### `PIIMiddlewareConfig`
<a id="PIIMiddlewareConfig"></a>
`PIIMiddlewareConfig : InferInteropZodInput < typeof contextSchema >`

## 函数
### `piiRedactionMiddleware`
<a id="piiRedactionMiddleware"></a>
import { piiRedactionMiddleware } from "langchain";

创建一个中间件，在消息发送到模型提供商之前检测并脱敏个人身份信息 (PII)，并在模型响应中恢复工具执行的原始值。

## 机制

该中间件在代理执行的两个点拦截：

### Request Phase (`wrapModelCall`)
- 对所有消息内容（HumanMessage, ToolMessage, SystemMessage, AIMessage）应用基于正则表达式的模式匹配。
- 处理消息文本和 AIMessage 工具调用参数。
- 每个匹配的模式生成：
  - 唯一标识符：`generateRedactionId()` 鈫?`"abc123"`
  - 脱敏标记：`[REDACTED_{RULE_NAME}_{ID}]` 鈫?`"[REDACTED_SSN_abc123]"`
  - 脱敏映射条目：`{ "abc123": "123-45-6789" }`
- 返回带有脱敏消息内容的修改后的请求。

### Response Phase (`afterModel`)
- 扫描 AIMessage 响应中符合模式 `/\[REDACTED_[A-Z_]+_(\w+)\]/g` 的脱敏标记。
- 用脱敏映射中的原始值替换标记。
- 处理标准响应和结构化输出（通过工具调用或 JSON 内容）。
- 对于结构化输出，在工具调用参数和 `structuredResponse` 状态字段中同时恢复值。
- 通过 RemoveMessage/AIMessage 返回新消息实例以更新状态。

## 数据流

```
User Input: "My SSN is 123-45-6789"
    鈫?[beforeModel]
Model Request: "My SSN is [REDACTED_SSN_abc123]"
    鈫?[model invocation]
Model Response: tool_call({ "ssn": "[REDACTED_SSN_abc123]" })
    鈫?[afterModel]
Tool Execution: tool({ "ssn": "123-45-6789" })
```

## 局限性

此中间件仅提供模型提供商隔离。PII 可能仍存在于：
- LangGraph 状态检查点（内存、数据库）
- 客户端与应用程序服务器之间的网络流量
- 应用程序日志和追踪数据
- 工具执行参数和响应
- 最终代理输出

对于全面的 PII 保护，请在应用程序、网络和存储层实施额外的控制。

```ts
piiRedactionMiddleware(options: __type = {}): AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `options` | `__type` | 否 | 配置选项（默认：`{}`） |

`AgentMiddleware<undefined, ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`

用于与 `createAgent` 一起使用的中间件实例。

### Example 1
```typescript
import { piiRedactionMiddleware } from "langchain";
import { createAgent } from "langchain";
import { tool } from "@langchain/core/tools";
import { z } from "zod/v3";

const PII_RULES = {
  ssn: /\b\d{3}-?\d{2}-?\d{4}\b/g,
  email: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/g,
  phone: /\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g,
};

const lookupUser = tool(async ({ ssn }) => {
  // Receives original value: "123-45-6789"
  return { name: "John Doe", account: "active" };
}, {
  name: "lookup_user",
  description: "Look up user by SSN",
  schema: z.object({ ssn: z.string() })
});

const agent = createAgent({
  model: new ChatOpenAI({ model: "gpt-4" }),
  tools: [lookupUser],
  middleware: [piiRedactionMiddleware({ rules: PII_RULES })]
});

const result = await agent.invoke({
  messages: [new HumanMessage("Look up SSN 123-45-6789")]
});
// Model request: "Look up SSN [REDACTED_SSN_abc123]"
// Model response: tool_call({ "ssn": "[REDACTED_SSN_abc123]" })
// Tool receives: { "ssn": "123-45-6789" }
```

### Example 2
```typescript
const agent = createAgent({
  model: new ChatOpenAI({ model: "gpt-4" }),
  tools: [someTool],
  middleware: [piiRedactionMiddleware()]
});

// Configure rules at runtime via middleware context
const result = await agent.invoke(
  { messages: [new HumanMessage("...")] },
  {
    configurable: {
      PIIRedactionMiddleware: {
        rules: {
          ssn: /\b\d{3}-?\d{2}-?\d{4}\b/g,
          email: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/g,
        }
      }
    }
  }
);
```

### Example 3
```typescript
const customRules = {
  employee_id: /EMP-\d{6}/g,
  api_key: /sk-[a-zA-Z0-9]{32}/g,
  credit_card: /\b\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}\b/g,
};

const middleware = piiRedactionMiddleware({ rules: customRules });
// Generates markers like: [REDACTED_EMPLOYEE_ID_xyz789]
```

## 类型
### `PIIRedactionMiddlewareConfig`
<a id="PIIRedactionMiddlewareConfig"></a>
`PIIRedactionMiddlewareConfig : InferInteropZodInput < typeof contextSchema >`

## 类型
### `PIIStrategy`
<a id="PIIStrategy"></a>
处理检测到的 PII 的策略。

`PIIStrategy : "block" | "redact" | "mask" | "hash"`

## 类型
### `PromptCachingMiddlewareConfig`
<a id="PromptCachingMiddlewareConfig"></a>
`PromptCachingMiddlewareConfig : Partial < InferInteropZodInput < typeof contextSchema > >`

## 函数

### `providerToolSearchMiddleware`
<a id="providerToolSearchMiddleware"></a>

**导入**
```ts
import { providerToolSearchMiddleware } from "langchain";
```

**签名**
```ts
providerToolSearchMiddleware(
  config: ProviderToolSearchMiddlewareConfig = {}
): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```

**说明**
提供商侧工具搜索中间件。

它利用服务端工具搜索：将完整的客户端工具目录转发给提供商，其中标记为 `defer_loading` 的延迟工具由提供商通过其自身的搜索按需披露。当某工具在 `searchableTools` 中被指定，或在构建时带有 `extras.defer_loading: true` 时，该工具即被视为延迟加载。

要求模型支持服务端工具搜索：OpenAI gpt-5.4+ 或 Anthropic Claude Sonnet 4+ / Opus 4+ / Haiku 4.5+。非 Anthropic/OpenAI 提供商将抛出错误；同系列中过旧的模型会直接显示提供商自身的 API 错误，而不会在此处被拦截。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `config` | `ProviderToolSearchMiddlewareConfig` | 否 | 中间件配置选项（默认 `{}`）。 |

**返回**
`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>` —— 可传给 `createAgent` 的中间件实例。

**示例**
```ts
import { createAgent, providerToolSearchMiddleware } from "langchain";
import { ChatAnthropic } from "@langchain/anthropic";

const agent = createAgent({
  model: new ChatAnthropic({ model: "claude-sonnet-4-5" }),
  tools: [getWeather, ...nicheTools],
  middleware: [
    // 将小众工具延迟到提供商的工具搜索中；模型将按需发现它们，
    // 而不是预先接收全部 schema。
    providerToolSearchMiddleware({ searchableTools: nicheTools }),
  ],
});
```

```ts
import { tool } from "@langchain/core/tools";
import { createAgent, providerToolSearchMiddleware } from "langchain";
import { ChatAnthropic } from "@langchain/anthropic";

// 在构建时标记为 `defer_loading` 的工具会自动延迟，
// 无需列入 `searchableTools`；中间件会遵循该标记。
const sendEmail = tool(sendEmailFn, {
  name: "send_email",
  description: "Send an email",
  schema: sendEmailSchema,
  extras: { defer_loading: true },
});

const agent = createAgent({
  model: new ChatAnthropic({ model: "claude-sonnet-4-5" }),
  tools: [getWeather, sendEmail],
  middleware: [providerToolSearchMiddleware()],
});
```

## 接口

### `ProviderToolSearchMiddlewareConfig`
<a id="ProviderToolSearchMiddlewareConfig"></a>

**导入**
```ts
import type { ProviderToolSearchMiddlewareConfig } from "langchain";
```

**说明**
`providerToolSearchMiddleware` 的配置。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `searchableTools` | `ToolIdentifier[]` | 被延迟的工具列表；在提供商的工具搜索披露它们之前，这些工具对模型隐藏。已在构建时通过 `extras.defer_loading === true` 标记的工具无论此选项如何都会被延迟；若省略 `searchableTools`，则只延迟预先标记的工具。 |

### `RedactionRuleConfig`
<a id="RedactionRuleConfig"></a>

**导入**
```ts
import type { RedactionRuleConfig } from "langchain";
```

**说明**
脱敏规则（redaction rule）的配置。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `detector` | `Detector` | 自定义检测函数或正则字符串。 |
| `piiType` | `string` | 要检测的 PII 类型（内置或自定义名称）。 |
| `strategy` | `PIIStrategy` | 处理检测到的 PII 的策略。 |

### `ResolvedRedactionRule`
<a id="ResolvedRedactionRule"></a>

**导入**
```ts
import type { ResolvedRedactionRule } from "langchain";
```

**说明**
带有具体检测函数的、已解析的脱敏规则。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `detector` | `PIIDetector` | 已解析出的具体检测函数。 |
| `piiType` | `string` | 要检测的 PII 类型。 |
| `strategy` | `PIIStrategy` | 处理检测到的 PII 的策略。 |

## 函数

### `resolveRedactionRule`
<a id="resolveRedactionRule"></a>

**导入**
```ts
import { resolveRedactionRule } from "langchain";
```

**签名**
```ts
resolveRedactionRule(config: RedactionRuleConfig): ResolvedRedactionRule
```

**说明**
将脱敏规则解析为具体的检测函数。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `config` | `RedactionRuleConfig` | 是 | 待解析的脱敏规则配置。 |

**返回**
`ResolvedRedactionRule` —— 带有具体检测函数的已解析规则。

### `summarizationMiddleware`
<a id="summarizationMiddleware"></a>

**导入**
```ts
import { summarizationMiddleware } from "langchain";
```

**签名**
```ts
summarizationMiddleware(
  options: SummarizationMiddlewareConfig
): AgentMiddleware<undefined, ZodObject<__type & __type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```

**说明**
摘要在 token 接近限制时自动总结对话历史的中间件。

该中间件监控消息的 token 数量，在达到阈值时自动总结较旧的消息，同时保留最近的消息，并确保 AI 消息与工具消息成对保留，以维持上下文连贯性。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `options` | `SummarizationMiddlewareConfig` | 是 | 摘要中间件的配置选项。 |

**返回**
`AgentMiddleware<...>` —— 一个中间件实例。

**示例**
```ts
import { summarizationMiddleware, createAgent } from "langchain";

// 单一条件：当 tokens >= 4000 且 messages >= 10 时触发
const agent1 = createAgent({
  llm: model,
  tools: [getWeather],
  middleware: [
    summarizationMiddleware({
      model: new ChatOpenAI({ model: "gpt-4o" }),
      trigger: { tokens: 4000, messages: 10 },
      keep: { messages: 20 },
    }),
  ],
});

// 多条件：(tokens >= 5000 且 messages >= 3) 或 (tokens >= 3000 且 messages >= 6)
const agent2 = createAgent({
  llm: model,
  tools: [getWeather],
  middleware: [
    summarizationMiddleware({
      model: new ChatOpenAI({ model: "gpt-4o" }),
      trigger: [
        { tokens: 5000, messages: 3 },
        { tokens: 3000, messages: 6 },
      ],
      keep: { messages: 20 },
    }),
  ],
});
```

## 类型

### `SummarizationMiddlewareConfig`
<a id="SummarizationMiddlewareConfig"></a>

**导入**
```ts
import type { SummarizationMiddlewareConfig } from "langchain";
```

**签名**
```ts
SummarizationMiddlewareConfig : InferInteropZodInput<typeof contextSchema>
```

**说明**
`summarizationMiddleware` 的配置类型。

### `Todo`
<a id="Todo"></a>

**导入**
```ts
import type { Todo } from "langchain";
```

**签名**
```ts
Todo : z.infer<typeof TodoSchema>
```

**说明**
表示一条待办事项的类型（由 `todoListMiddleware` 使用）。

## 函数

### `todoListMiddleware`
<a id="todoListMiddleware"></a>

**导入**
```ts
import { todoListMiddleware } from "langchain";
```

**签名**
```ts
todoListMiddleware(
  options?: TodoListMiddlewareOptions
): AgentMiddleware<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, undefined, unknown, readonly [DynamicStructuredTool<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, __type, __type, Command<unknown, __type, string>, unknown, "write_todos">], readonly []>
```

**说明**
创建一个为智能体提供待办事项（todo）列表管理能力的中间件。

该中间件新增了一个 `write_todos` 工具，让智能体能够为复杂的多步骤任务创建和管理结构化任务列表，帮助跟踪进度、组织任务，并向用户展示任务完成状态。

该中间件会自动注入系统提示词，引导智能体何时以及如何高效使用待办功能。它还强制 `write_todos` 在每个模型回合中最多被调用一次，因为该工具会整体替换待办列表，并行调用会导致优先顺序产生歧义。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `options` | `TodoListMiddlewareOptions` | 否 | 待办中间件的配置选项。 |

**返回**
`AgentMiddleware<...>` —— 提供待办管理能力的已配置中间件实例。

**示例**
```typescript
import { todoListMiddleware, createAgent } from "langchain";

const agent = createAgent({
  model: "openai:gpt-4o",
  middleware: [todoListMiddleware()],
});

// 智能体现在可以访问 write_todos 工具和待办状态跟踪
const result = await agent.invoke({
  messages: [new HumanMessage("Help me refactor my codebase")],
});

console.log(result.todos); // 带状态跟踪的待办事项数组
```

## 接口

### `TodoListMiddlewareOptions`
<a id="TodoListMiddlewareOptions"></a>

**导入**
```ts
import type { TodoListMiddlewareOptions } from "langchain";
```

**说明**
`todoListMiddleware` 的配置选项。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `systemPrompt` | `string` | 指导智能体使用待办工具的自定义系统提示词。未提供时使用默认的 `PLANNING_MIDDLEWARE_SYSTEM_PROMPT`。 |
| `toolDescription` | `string` | `writeTodos` 工具的自定义描述。未提供时使用默认的 `WRITE_TODOS_DESCRIPTION`。 |

## 类型

### `ToolCallLimitConfig`
<a id="ToolCallLimitConfig"></a>

**导入**
```ts
import type { ToolCallLimitConfig } from "langchain";
```

**签名**
```ts
ToolCallLimitConfig : InferInteropZodInput<typeof ToolCallLimitOptionsSchema>
```

**说明**
`toolCallLimitMiddleware` 的配置类型。

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/toolCallLimitMiddleware
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖中间件相关功能，包括工具调用限制、模拟以及错误处理和重试机制。

## 函数

### `toolCallLimitMiddleware`
<a id="toolCallLimitMiddleware"></a>
```javascript
toolCallLimitMiddleware(options: __type): AgentMiddleware<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```
监控代理执行期间的工具调用次数并强制执行限制。该中间件支持线程级和运行级的调用计数，并可配置退出行为。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `options` | `__type` | 是 | 中间件的配置选项 |

`AgentMiddleware<ZodObject<__type, "strip", ZodTypeAny, __type, __type>, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`

### 示例 1
```ts
import { toolCallLimitMiddleware } from "@langchain/langchain/agents/middleware";
import { createAgent } from "@langchain/langchain/agents";

// Block exceeded tools but let other tools and model continue
const limiter = toolCallLimitMiddleware({
  threadLimit: 20,
  runLimit: 10,
  exitBehavior: "continue", // default
});

const agent = createAgent({
  model: "openai:gpt-4o",
  middleware: [limiter]
});
```

### 示例 2
```ts
// End execution immediately with an AI message
const limiter = toolCallLimitMiddleware({
  runLimit: 5,
  exitBehavior: "end"
});

const agent = createAgent({
  model: "openai:gpt-4o",
  middleware: [limiter]
});
```

### 示例 3
```ts
// Strict limit with exception handling
const limiter = toolCallLimitMiddleware({
  toolName: "search",
  threadLimit: 5,
  exitBehavior: "error"
});

const agent = createAgent({
  model: "openai:gpt-4o",
  middleware: [limiter]
});

try {
  const result = await agent.invoke({ messages: [new HumanMessage("Task")] });
} catch (error) {
  if (error instanceof ToolCallLimitExceededError) {
    console.log(`Search limit exceeded: ${error}`);
  }
}
```

### 函数
### `toolEmulatorMiddleware`
<a id="toolEmulatorMiddleware"></a>
```javascript
toolEmulatorMiddleware(options: ToolEmulatorOptions = {}): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```
使用 LLM 代替执行指定的工具。该中间件允许出于测试目的选择性地模拟工具。默认情况下（当 `tools` 未定义时），所有工具都将被模拟。你可以通过传递工具名称或工具实例列表来指定要模拟的工具。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `options` | `ToolEmulatorOptions` | 否 | 中间件的配置选项（默认：`{}`） |

`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`

### 示例 1
```ts
import { toolEmulatorMiddleware } from "@langchain/langchain/agents/middleware";
import { createAgent } from "@langchain/langchain/agents";

const middleware = toolEmulatorMiddleware();

const agent = createAgent({
  model: "openai:gpt-4o",
  tools: [getWeather, getUserLocation, calculator],
  middleware: [middleware],
});
```

### 示例 2
```ts
const middleware = toolEmulatorMiddleware({
  tools: ["get_weather", "get_user_location"]
});
```

### 示例 3
```ts
const middleware = toolEmulatorMiddleware({
  tools: ["get_weather"],
  model: "anthropic:claude-sonnet-4-5-20250929"
});
```

### 示例 4
```ts
const middleware = toolEmulatorMiddleware({
  tools: [getWeather, getUserLocation]
});
```

### 函数
### `toolErrorMiddleware`
<a id="toolErrorMiddleware"></a>
```javascript
toolErrorMiddleware(config: ToolErrorMiddlewareConfig): AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>
```
将选定的工具执行错误转换为错误 `ToolMessage`。

处理是可选的：`ToolErrorMiddlewareConfig.onError` 必须返回内容，错误才会被发送给模型。如果返回为空，则传播原始错误。LangGraph 控制流信号始终传播且永远不会传递给 `onError`。

建议返回识别错误类型的内容，而不包含可能包含敏感或内部细节的原始错误消息。

此中间件不进行重试。若要在处理错误之前进行重试，请在此中间件后放置 `toolRetryMiddleware({ onFailure: "error" })`。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `config` | `ToolErrorMiddlewareConfig` | 是 |  |

`AgentMiddleware<undefined, undefined, unknown, readonly ClientTool | ServerTool[], readonly []>`

### 示例
```ts
import { createAgent, toolErrorMiddleware } from "langchain";

const agent = createAgent({
  model,
  tools: [searchTool],
  middleware: [
    toolErrorMiddleware({
      onError: (error, request) => {
        if (error instanceof TypeError) {
          return `Tool '${request.toolCall.name}' failed with TypeError.`;
        }
      },
    }),
  ],
});
```

### 函数
### `toolRetryMiddleware`
<a id="toolRetryMiddleware"></a>
```javascript
toolRetryMiddleware(config: __type = {}): AgentMiddleware<undefined, ZodObject<__type & __type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>
```
自动重试失败的工具调用，并支持可配置的回退机制。支持针对特定异常的重试和指数回退。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `config` | `__type` | 否 | 重试中间件的配置选项（默认：`{}`） |

`AgentMiddleware<undefined, ZodObject<__type & __type, "strip", ZodTypeAny, __type, __type>, __type, readonly ClientTool | ServerTool[], readonly []>`
一个处理工具故障重试的中间件实例。

### 示例 1
```ts
import { createAgent, toolRetryMiddleware } from "langchain";

const agent = createAgent({
  model: "openai:gpt-4o",
  tools: [searchTool],
  middleware: [toolRetryMiddleware()],
});
```

### 示例 2
```ts
import { toolRetryMiddleware } from "langchain";

const retry = toolRetryMiddleware({
  maxRetries: 4,
  retryOn: [TimeoutError, NetworkError],
  backoffFactor: 1.5,
});
```

### 示例 3
```ts
function shouldRetry(error: Error): boolean {
  // Only retry on 5xx errors
  if (error.name === "HTTPError" && "statusCode" in error) {
    const statusCode = (error as any).statusCode;
    return 500 <= statusCode && statusCode < 600;
  }
  return false;
}

const retry = toolRetryMiddleware({
  maxRetries: 3,
  retryOn: shouldRetry,
});
```

### 示例 4
```ts
const formatError = (error: Error) =>
  "Database temporarily unavailable. Please try again later.";

const retry = toolRetryMiddleware({
  maxRetries: 4,
  tools: ["search_database"],
  onFailure: formatError,
});
```

### 示例 5
```ts
import { tool } from "@langchain/core/tools";
import { z } from "zod";

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

const retry = toolRetryMiddleware({
  maxRetries: 4,
  tools: [searchDatabase], // Pass BaseTool instance
});
```

### 示例 6
```ts
const retry = toolRetryMiddleware({
  maxRetries: 5,
  backoffFactor: 0.0, // No exponential growth
  initialDelayMs: 2000, // Always wait 2 seconds
});
```

### 示例 7
```ts
const retry = toolRetryMiddleware({
  maxRetries: 2,
  onFailure: "raise", // Re-raise exception instead of returning message
});
```

## 接口

### `ToolEmulatorOptions`
<a id="ToolEmulatorOptions"></a>
用于配置工具模拟器（Tool Emulator）中间件的选项。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `model` | `string \| BaseChatModel` | 用于模拟的模型。可以是模型标识字符串（例如 "anthropic:claude-sonnet-4-5-20250929"），也可以是 `BaseChatModel` 实例。默认为代理模型。 |
| `tools` | `string \| ClientTool \| ServerTool[]` | 要模拟的工具名称（字符串）或工具实例列表。如果未定义（默认），将模拟所有工具。如果为空数组，则不模拟任何工具。如果包含工具名称/实例，则仅模拟这些工具。 |

## 类型

### `ToolErrorHandler`
<a id="ToolErrorHandler"></a>
当工具执行抛出异常时调用的处理程序。返回内容到模型作为错误 `ToolMessage` 展示。返回为空则传播原始错误。

`( error : unknown , request : ToolCallRequest ) = > MessageContent \| void \| Promise < MessageContent \| void >`

### `ToolErrorMiddlewareConfig`
<a id="ToolErrorMiddlewareConfig"></a>
`toolErrorMiddleware` 的配置。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `onError` | `ToolErrorHandler` | 当工具执行抛出异常时调用的处理程序。返回内容到模型作为错误 `ToolMessage` 展示，或者返回为空以传播原始错误。 |
| `tools` | `string \| ClientTool \| ServerTool []` | 可选的要处理错误的工具或工具名称。如果省略，则处理所有工具。 |

### `ToolRetryMiddlewareConfig`
<a id="ToolRetryMiddlewareConfig"></a>
`ToolRetryMiddlewareConfig` 类型。
`z . input < typeof ToolRetryMiddlewareOptionsSchema >`