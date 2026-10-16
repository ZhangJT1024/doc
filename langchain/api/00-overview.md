# LangChain JS API 参考 · 总览

> 本目录是 [LangChain JS 官方 API 参考](https://reference.langchain.com/javascript/langchain) 的系统化中文整理，
> 覆盖 `langchain` npm 包 **全部导出**（类、函数、接口、类型）及其属性和方法。
>
> - 版本快照：**v1.5.15**（官方参考站）。
> - 签名、类型、代码块一律保持官方原文（英文），**只有说明性文字译为简体中文**。
> - 每个条目都给出官方原文链接，便于对照查证。

## 与 `01`～`27` 教程的关系

- `01`～`27` 是**概念/教程**（来自 docs.langchain.com），回答"是什么、怎么用"。
- `api/` 是**API 参考**（来自 reference.langchain.com），回答"有哪些导出、参数、返回值、方法、属性"。
- 两者配合使用：先在教程里建立认知，再在 API 参考里查具体签名。

## 模块结构

`langchain` 包按子路径（subpath）导出不同的模块：

| 模块路径 | 导入方式 | 内容 | 对应文档 |
| --- | --- | --- | --- |
| `langchain`（`index`） | `import { ... } from "langchain"` | 智能体、工具、消息、中间件等主入口 | `01`～`10` |
| `langchain/tools` | `import { ... } from "langchain/tools"` | 工具相关（HeadlessTool 等） | [02-tools.md](./02-tools.md) |
| `langchain/chat_models/universal` | `import { ... } from "langchain/chat_models/universal"` | 通用 `initChatModel` 等 | [06-models.md](./06-models.md) |
| `langchain/hub`、`langchain/hub/node` | `import { ... } from "langchain/hub"` | LangChain Hub 的 `pull`/`push` | [08-hub.md](./08-hub.md) |
| `langchain/load` | `import { ... } from "langchain/load"` | 反序列化 `load` | [07-storage-and-serialization.md](./07-storage-and-serialization.md) |
| `langchain/load/serializable` | `import { ... } from "langchain/load/serializable"` | `Serializable` 基类及序列化接口 | [07-storage-and-serialization.md](./07-storage-and-serialization.md) |
| `langchain/storage/encoder_backed` | `import { ... } from "langchain/storage/encoder_backed"` | `EncoderBackedStore` 等 | [07-storage-and-serialization.md](./07-storage-and-serialization.md) |
| `langchain/storage/file_system` | `import { ... } from "langchain/storage/file_system"` | `LocalFileStore` | [07-storage-and-serialization.md](./07-storage-and-serialization.md) |
| `langchain/storage/in_memory` | `import { ... } from "langchain/storage/in_memory"` | `InMemoryStore` | [07-storage-and-serialization.md](./07-storage-and-serialization.md) |
| `langchain/browser` | `import { ... } from "langchain/browser"` | 与 `index` 相同的浏览器构建（内容重复，本整理不单列） | — |

> 提示：`index` 与 `browser` 的导出内容一致，本整理只收录 `index`（以及其它非重复子模块）。

## 文档索引

| 文档 | 主题 | 主要内容 |
| --- | --- | --- |
| [01-messages.md](./01-messages.md) | 消息 | `BaseMessage` 及各类消息、`ContentBlock` 内容块、`filterMessages`/`trimMessages` |
| [02-tools.md](./02-tools.md) | 工具 | `Tool`/`StructuredTool`/`DynamicTool`/`HeadlessTool`、结构化输出策略、工具相关接口 |
| [03-agents.md](./03-agents.md) | 智能体 | `createAgent`、`ReactAgent`、`createMiddleware`、状态/上下文/流式类型、生命周期钩子 |
| [04-middleware.md](./04-middleware.md) | 中间件 | 全部内置中间件工厂函数、PII 检测、配置类型 |
| [05-human-in-the-loop.md](./05-human-in-the-loop.md) | 人在环路 | `HITLRequest`/`HITLResponse`、`*Decision`、`Interrupt`、`WhenPredicate` |
| [06-models.md](./06-models.md) | 模型 | `initChatModel`、模型工具函数、测试用 `FakeToolCallingModel`/`fakeModel` |
| [07-storage-and-serialization.md](./07-storage-and-serialization.md) | 存储与序列化 | `InMemoryStore`/`EncoderBackedStore`/`LocalFileStore`、`Serializable` 系列、`load` |
| [08-hub.md](./08-hub.md) | Hub | `pull`/`push`、`inferModelProviderFromNamespace` |
| [09-errors.md](./09-errors.md) | 错误 | 所有导出的错误类 |
| [10-types-and-utilities.md](./10-types-and-utilities.md) | 类型与工具 | `Infer*`/`Combine*`/`Resolve*` 等类型辅助、运行时与包装器类型 |

## 全部导出索引

### 类（Classes）

**消息**（[01-messages.md](./01-messages.md)）：
`BaseMessage`、`BaseMessageChunk`、`AIMessage`、`AIMessageChunk`、`HumanMessage`、`HumanMessageChunk`、`SystemMessage`、`SystemMessageChunk`、`ToolMessage`、`ToolMessageChunk`、`Document`

**工具与结构化输出**（[02-tools.md](./02-tools.md)）：
`Tool`、`StructuredTool`、`DynamicTool`、`DynamicStructuredTool`、`ToolStrategy`、`ProviderStrategy`

**中间件/上下文**（[04-middleware.md](./04-middleware.md)）：
`ClearToolUsesEdit`

**模型（测试）**（[06-models.md](./06-models.md)）：
`FakeToolCallingModel`

**存储与序列化**（[07-storage-and-serialization.md](./07-storage-and-serialization.md)）：
`InMemoryStore`、`EncoderBackedStore`、`LocalFileStore`、`Serializable`

**错误**（[09-errors.md](./09-errors.md)）：
`MiddlewareError`、`MultipleStructuredOutputsError`、`MultipleToolsBoundError`、`PIIDetectionError`、`StructuredOutputParsingError`、`ToolCallLimitExceededError`、`ToolInvocationError`

### 函数（Functions）

**智能体**（[03-agents.md](./03-agents.md)）：
`createAgent`、`createMiddleware`、`createSubagentTransformer`、`createToolCallTransformer`

**中间件**（[04-middleware.md](./04-middleware.md)）：
`anthropicPromptCachingMiddleware`、`bedrockPromptCachingMiddleware`、`contextEditingMiddleware`、`dynamicSystemPromptMiddleware`、`humanInTheLoopMiddleware`、`llmToolSelectorMiddleware`、`modelCallLimitMiddleware`、`modelFallbackMiddleware`、`modelRetryMiddleware`、`openAIModerationMiddleware`、`piiMiddleware`、`piiRedactionMiddleware`（deprecated）、`providerToolSearchMiddleware`、`summarizationMiddleware`、`todoListMiddleware`、`toolCallLimitMiddleware`、`toolEmulatorMiddleware`、`toolErrorMiddleware`、`toolRetryMiddleware`

**PII / 内容工具**（[04-middleware.md](./04-middleware.md)）：
`detectCreditCard`、`detectEmail`、`detectIP`、`detectMacAddress`、`detectUrl`、`resolveRedactionRule`、`applyStrategy`、`context`、`countTokensApproximately`

**消息工具**（[01-messages.md](./01-messages.md)）：
`filterMessages`、`trimMessages`

**模型**（[06-models.md](./06-models.md)）：
`initChatModel`、`getChatModelByClassName`、`_inferModelProvider`、`fakeModel`

**结构化输出**（[02-tools.md](./02-tools.md)）：
`toolStrategy`、`providerStrategy`

**存储 / 序列化 / Hub / 加载**：
`createDocumentStoreFromByteStore`、`get_lc_unique_name`（[07](./07-storage-and-serialization.md)）、`pull`、`push`、`inferModelProviderFromNamespace`（[08](./08-hub.md)）、`load`（[07](./07-storage-and-serialization.md)）

### 接口（Interfaces）

**智能体**（[03-agents.md](./03-agents.md)）：
`ReactAgent`、`AgentMiddleware`、`AgentTypeConfig`、`DefaultAgentTypeConfig`、`BuiltInState`、`MiddlewareTypeConfig`、`ModelRequest`、`SubagentRunStream`

**工具**（[02-tools.md](./02-tools.md)）：
`ToolCall`、`ToolCallRequest`、`ToolResult`、`ExecutedToolCall`、`Action`、`ActionRequest`、`TypedToolStrategy`、`ToolErrorMiddlewareConfig`

**消息/内容块**（[01-messages.md](./01-messages.md)）：
`ContentBlock`、`Text`、`Reasoning`、`Citation`、`NonStandard`、`ToolCall`、`ToolCallChunk`、`InvalidToolCall`、`ServerToolCall`、`ServerToolCallChunk`、`ServerToolCallResult`、`DocumentInput`

**中间件与护栏**（[04](./04-middleware.md) / [05](./05-human-in-the-loop.md)）：
`ContextEdit`、`ContextEditingMiddlewareConfig`、`ClearToolUsesEditConfig`、`PIIMatch`、`OpenAIModerationMiddlewareOptions`、`RedactionRuleConfig`、`ResolvedRedactionRule`、`TodoListMiddlewareOptions`、`ToolEmulatorOptions`

**人在环路**（[05-human-in-the-loop.md](./05-human-in-the-loop.md)）：
`HITLRequest`、`HITLResponse`、`ApproveDecision`、`EditDecision`、`RejectDecision`、`ReviewConfig`、`Interrupt`

**模型**（[06-models.md](./06-models.md)）：
`ConfigurableChatModelCallOptions`、`InitChatModelFields`

**测试**（[06-models.md](./06-models.md)）：
`LangChainMatchers`

**序列化**（[07-storage-and-serialization.md](./07-storage-and-serialization.md)）：
`BaseSerialized`、`SerializableInterface`、`SerializableLike`、`SerializedConstructor`、`SerializedNotImplemented`、`SerializedSecret`

### 类型（Types）

类型众多，按主题分散在各文档中，主要包括：

- **智能体推断辅助**：`InferAgentState`、`InferAgentContext`、`InferAgentTools`、`InferAgentResponse`、`InferAgentMiddleware`、`InferAgentType`、`InferAgentStateSchema`、`InferAgentContextSchema`、`InferAgentStreamTransformers` 等（[10](./10-types-and-utilities.md)）
- **中间件推断辅助**：`InferMiddlewareState`、`InferMiddlewareContext`、`InferMiddlewareTools`、`InferMiddleware*` 系列（[10](./10-types-and-utilities.md)）
- **模式推断**：`InferSchemaInput`、`InferSchemaValue`、`InferSchemaValueType`、`InferSchemaUpdateType`、`NormalizedSchemaInput`、`NormalizedSchemaUpdate`、`ExtractZodArrayTypes`（[10](./10-types-and-utilities.md)）
- **组合与解析**：`CombineTools`、`CombineStreamTransformers`、`ResolveAgentTypeConfig`、`ResolveMiddlewareTypeConfig`、`ToolsToMessageToolSet`、`ToAnnotationRoot`（[10](./10-types-and-utilities.md)）
- **运行时/包装器**：`Runtime`、`ToolRuntime`、`WrapModelCallHook`、`WrapModelCallHandler`、`WrapToolCallHook`、`ToolCallHandler`、`MiddlewareResult`、`TokenCounter`（[10](./10-types-and-utilities.md)）
- **生命周期钩子**：`BeforeAgentHook`、`AfterAgentHook`、`BeforeModelHook`、`AfterModelHook`、`JumpTo`、`JumpToTarget`（[03](./03-agents.md) / [10](./10-types-and-utilities.md)）
- **配置类型**：`ModelRetryMiddlewareConfig`、`ModelCallLimitMiddlewareConfig`、`ToolRetryMiddlewareConfig`、`ToolCallLimitConfig`、`SummarizationMiddlewareConfig`、`PromptCachingMiddlewareConfig`、`BedrockConversePromptCachingMiddlewareConfig`、`ProviderToolSearchMiddlewareConfig`、`HumanInTheLoopMiddlewareConfig`、`DynamicSystemPromptMiddlewareConfig`、`LLMToolSelectorConfig`、`PIIMiddlewareConfig`、`PIIStrategy`、`PIIDetector`、`BuiltInPIIType`、`PIIRedactionMiddlewareConfig`（[04](./04-middleware.md)）
- **结构化输出相关**：`ResponseFormat`、`ResponseFormatUndefined`、`ToolCallStreamUnion`（[02](./02-tools.md)）
- **决策/中断**：`Decision`、`DecisionType`、`DescriptionFactory`、`WhenPredicate`、`InterruptOnConfig`（[05](./05-human-in-the-loop.md)）
- **哈希/其它**：`Serialized`、`ChatModelProvider`、`ConfigurableFields`、`AgentRunStream`、`AnyAgentMiddleware`、`AnyAnnotationRoot`、`N`、`UserInput`、`WithStateGraphNodes`

## 阅读约定

- `签名`：官方 TypeScript/JavaScript 签名，保持英文，直接可用。
- `参数` / `返回`：来自官方的参数表与返回值说明。
- `属性` / `方法`：类与接口的成员清单；类的方法另附其**签名**（来自官方方法页）。
- `⚠ Deprecated`：官方标注为已弃用的条目。
- 所有 `import` 路径以官方为准，通常为 `"langchain"`。

> 官方文档会持续更新，本目录为整理时的快照（v1.5.15）；如有出入，以官方参考站为准。
