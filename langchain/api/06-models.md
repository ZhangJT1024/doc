# LangChain JS API 参考 · 模型（Models）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/chat_models/universal
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖通用聊天模型初始化（`langchain/chat_models/universal`）、模型工具函数，以及测试用的 `FakeToolCallingModel` / `fakeModel` / `LangChainMatchers`。

## 目录

- [initChatModel](#initChatModel)
- [getChatModelByClassName](#getChatModelByClassName)
- [_inferModelProvider](#_inferModelProvider)
- [fakeModel](#fakeModel)
- [FakeToolCallingModel](#FakeToolCallingModel)
- [ConfigurableChatModelCallOptions](#ConfigurableChatModelCallOptions)
- [InitChatModelFields](#InitChatModelFields)
- [LangChainMatchers](#LangChainMatchers)
- [ChatModelProvider](#ChatModelProvider)
- [ConfigurableFields](#ConfigurableFields)

---

## 函数

### `initChatModel`
<a id="initChatModel"></a>

**导入**
```ts
import { initChatModel } from "langchain/chat_models/universal";
// 也可从主入口导入：import { initChatModel } from "langchain";
```

**签名**
```ts
initChatModel<
  RunInput extends BaseLanguageModelInput = BaseLanguageModelInput,
  CallOptions extends ConfigurableChatModelCallOptions = ConfigurableChatModelCallOptions
>(
  model: string,
  fields?: Partial<Record<string, any>> & __type
): Promise<ConfigurableModel<RunInput, CallOptions>>
```

**说明**
根据模型名和提供商初始化一个 ChatModel。必须已安装与该模型提供商对应的集成包。

该函数会根据模型名推断提供商（若未指定）。常见前缀对应关系：

- `gpt-3...` 或 `gpt-4...` → `openai`
- `claude...` → `anthropic`
- `amazon....` → `bedrock`
- `gemini...` → `google-vertexai`
- `command...` → `cohere`
- `accounts/fireworks...` → `fireworks`

> **安全提示**：将 `configurableFields` 设为 `"any"` 意味着像 `apiKey`、`baseUrl` 等字段可在运行时被修改，可能将模型请求重定向到其他服务/用户。若你会接受不可信配置，请显式枚举 `configurableFields`。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `model` | `string` | 是 | 模型名（可带 `provider:` 前缀）。 |
| `fields` | `Partial<Record<string, any>> & InitChatModelFields` | 否 | 其他字段（如 `temperature`、`modelProvider`、`configurableFields`、`configPrefix` 等）。 |

**返回**
`Promise<ConfigurableModel<RunInput, CallOptions>>`

**示例**
```typescript
import { initChatModel } from "langchain/chat_models/universal";

const gpt4 = await initChatModel("openai:gpt-4", { temperature: 0.25 });
const gpt4Result = await gpt4.invoke("what's your name");

const claude = await initChatModel("anthropic:claude-3-opus-20240229", { temperature: 0.25 });
const claudeResult = await claude.invoke("what's your name");

const gemini = await initChatModel("gemini-1.5-pro", {
  modelProvider: "google-vertexai",
  temperature: 0.25,
});
const geminiResult = await gemini.invoke("what's your name");
```

```typescript
// 运行时可选模型
const configurableModel = await initChatModel(undefined, {
  temperature: 0,
  configurableFields: ["model", "apiKey"],
});

const gpt4Result = await configurableModel.invoke("what's your name", {
  configurable: { model: "gpt-4" },
});
const claudeResult = await configurableModel.invoke("what's your name", {
  configurable: { model: "claude-sonnet-4-5-20250929" },
});
```

### `getChatModelByClassName`
<a id="getChatModelByClassName"></a>

**导入**
```ts
import { getChatModelByClassName } from "langchain/chat_models/universal";
```

**签名**
```ts
getChatModelByClassName(className: string, modelProvider: string): Promise<any>
```

**说明**
根据类名或模型提供商获取聊天模型类的辅助函数。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `className` | `string` | 是 | 类名（如 `"ChatOpenAI"`、`"ChatAnthropic"`）。 |
| `modelProvider` | `string` | 否 | 可选的模型提供商键，用于直接查找（如 `"google-vertexai-web"`）。提供时会使用直接查找以避免类名冲突问题。 |

**返回**
`Promise<any>`

### `_inferModelProvider`
<a id="_inferModelProvider"></a>

**导入**
```ts
import { _inferModelProvider } from "langchain/chat_models/universal";
```

**签名**
```ts
_inferModelProvider(modelName: string): string | undefined
```

**说明**
尝试根据给定的模型名推断提供商。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `modelName` | `string` | 是 | 要推断提供商的模型名。 |

**示例**
```typescript
_inferModelProvider("gpt-4"); // 返回 "openai"
_inferModelProvider("claude-2"); // 返回 "anthropic"
_inferModelProvider("unknown-model"); // 返回 undefined
```

### `fakeModel`
<a id="fakeModel"></a>

**导入**
```ts
import { fakeModel } from "langchain";
```

**签名**
```ts
fakeModel(): FakeBuiltModel
```

**说明**
LangChain 测试工具，用于构建一个假的（fake）模型。

**返回**
`FakeBuiltModel`

## 类

### `FakeToolCallingModel`
<a id="FakeToolCallingModel"></a>

**导入**
```ts
import { FakeToolCallingModel } from "langchain";
```

**签名**
```ts
class FakeToolCallingModel
```

**继承**：`BaseChatModel`

**说明**
用于测试工具调用功能的假聊天模型。

**构造**
```ts
new FakeToolCallingModel(__namedParameters: FakeToolCallingModelFields & __type = {}): FakeToolCallingModel
```

**属性**：`cache`、`callbacks`、`caller`、`disableStreaming`、`lc_kwargs`、`lc_namespace`、`lc_runnable`、`lc_serializable`、`metadata`、`name`、`outputVersion`、`ParsedCallOptions`、`structuredResponse`、`tags`、`toolCalls`、`toolStyle`、`verbose`、`callKeys`、`index`、`lc_aliases`、`lc_attributes`、`lc_id`、`lc_secrets`、`lc_serializable_keys`、`profile`

**方法**
| 方法 | 签名 |
| --- | --- |
| `_generate` | `(messages, _options, _runManager): Promise<ChatResult>` |
| `_streamResponseChunks` | `(_messages, _options, _runManager): AsyncGenerator<ChatGenerationChunk>` |
| `_streamChatModelEvents` | `(messages, options, runManager): AsyncGenerator<ChatModelStreamEvent>` |
| `bindTools` | `(tools: StructuredTool<...>[]): FakeToolCallingModel \| RunnableBinding<any, any, any>` |
| `invoke` | `(input, options): Promise<AIMessageChunk<...>>` |
| `stream` | `(input, options): Promise<IterableReadableStream<AIMessageChunk<...>>>` |
| `batch` | `(inputs, options, batchOptions): Promise<AIMessageChunk<...>[]>` |
| `generate` | `(messages, options, callbacks): Promise<LLMResult>` |
| `generatePrompt` | `(promptValues, options, callbacks): Promise<LLMResult>` |
| `getNumTokens` | `(content: MessageContent): Promise<number>` |
| `withStructuredOutput` | `(_schema: any): RunnableLambda<unknown, any, ...>` |
| `getName` | `(suffix: string): string` |
| `lc_name` | `lc_name(): string` |

> 其余大量方法（如 `assign`、`asTool`、`pipe`、`pick`、`withConfig`、`withRetry`、`withFallbacks`、`withListeners`、`streamEvents`、`streamLog`、`transform`、`toJSON` 等）继承自 `BaseChatModel` / `Runnable`，此处不再逐一展开。

## 接口

### `ConfigurableChatModelCallOptions`
<a id="ConfigurableChatModelCallOptions"></a>

**导入**
```ts
import type { ConfigurableChatModelCallOptions } from "langchain/chat_models/universal";
```

**继承**：`BaseChatModelCallOptions`

**说明**
可配置聊天模型的调用选项。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `callbacks` | `Callbacks` | 本次调用及任何子调用（例如 Chain 调用 LLM）的回调。标签会传给所有回调，元数据会传给 `handle*Start` 回调。 |
| `configurable` | `Record<string, any>` | 此前在该 Runnable（或子 Runnable）上配置为可配置项的运行时值。 |
| `ls_structured_output_format` | 对象 | 描述结构化输出的格式。若某输出被视为结构化输出，应提供此项。 |
| `maxConcurrency` | `number` | 最大并行调用数。 |
| `maxRetries` | `number` | 仅对本次调用覆盖模型配置的 `maxRetries`。 |
| `metadata` | `Record<string, unknown>` | 本次调用及子调用的元数据。键应为字符串，值应可 JSON 序列化。 |
| `outputVersion` | `MessageOutputVersion` | 存入消息内容的 `AIMessage` 输出格式版本。`"v0"` 为提供商特定格式（可用 `.contentBlocks` 惰性解析）；`"v1"` 为标准格式。也可通过环境变量 `LC_OUTPUT_VERSION=v1` 默认开启。 |
| `recursionLimit` | `number` | 调用可递归的最大次数，默认 25。 |
| `runId` | `string` | 本次调用追踪运行的唯一标识，未提供时自动生成 UUID。 |
| `runName` | `string` | 本次调用追踪运行的名称，默认取类名。 |
| `signal` | `AbortSignal` | 本次调用的中止信号。 |
| `stop` | `string[]` | 本次调用的停止词。 |
| `tags` | `string[]` | 本次调用及子调用的标签，用于过滤。 |
| `timeout` | `number` | 本次调用超时时间（毫秒）。 |
| `tool_choice` | `ToolChoice` | 指定聊天模型如何使用工具。 |
| `tools` | `StructuredToolInterface<...> \| Record<string, unknown> \| ToolDefinition \| RunnableToolLike<...>[]` | 模型可用的工具。 |

### `InitChatModelFields`
<a id="InitChatModelFields"></a>

**导入**
```ts
import type { InitChatModelFields } from "langchain/chat_models/universal";
```

**继承**：`Partial<Record<string, any>>`

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `configPrefix` | `string` | 运行时配置键的前缀。 |
| `configurableFields` | `"any" \| string[]` | 允许在运行时配置的字段。 |
| `modelProvider` | `string` | 模型提供商。 |

### `LangChainMatchers`
<a id="LangChainMatchers"></a>

**导入**
```ts
import type { LangChainMatchers } from "langchain";
```

**继承**：`StreamMatchers<R>`

**说明**
LangChain 测试工具的匹配器（matchers）接口。

**方法**：`toBeAIMessage`、`toBeHumanMessage`、`toBeSystemMessage`、`toBeToolMessage`、`toContainToolCall`、`toHaveBeenInterrupted`、`toHaveStreamOutput`、`toHaveStreamReasoning`、`toHaveStreamText`、`toHaveStreamToolCalls`、`toHaveStreamUsage`、`toHaveStructuredResponse`、`toHaveToolCallCount`、`toHaveToolCalls`、`toHaveToolMessages`

## 类型

### `ChatModelProvider`
<a id="ChatModelProvider"></a>

**导入**
```ts
import type { ChatModelProvider } from "langchain/chat_models/universal";
```

**签名**
```ts
ChatModelProvider : keyof typeof MODEL_PROVIDER_CONFIG
```

**说明**
聊天模型提供商的键类型。

### `ConfigurableFields`
<a id="ConfigurableFields"></a>

**导入**
```ts
import type { ConfigurableFields } from "langchain/chat_models/universal";
```

**签名**
```ts
ConfigurableFields : "any" | string[]
```

**说明**
可配置字段的类型：`"any"` 或字段名数组。
