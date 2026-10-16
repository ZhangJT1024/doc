# LangChain JS API 参考 · 智能体（Agents）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 目录

- [AfterAgentHook](#AfterAgentHook)
- [AfterModelHook](#AfterModelHook)
- [AgentMiddleware](#AgentMiddleware)
- [AgentRunStream](#AgentRunStream)
- [AgentTypeConfig](#AgentTypeConfig)
- [AnyAgentMiddleware](#AnyAgentMiddleware)
- [AnyAnnotationRoot](#AnyAnnotationRoot)
- [BeforeAgentHook](#BeforeAgentHook)
- [BeforeModelHook](#BeforeModelHook)
- [BuiltInState](#BuiltInState)
- [createAgent](#createAgent)
- [CreateAgentParams](#CreateAgentParams)
- [createMiddleware](#createMiddleware)
- [createSubagentTransformer](#createSubagentTransformer)
- [createToolCallTransformer](#createToolCallTransformer)
- [DefaultAgentTypeConfig](#DefaultAgentTypeConfig)
- [MiddlewareTypeConfig](#MiddlewareTypeConfig)
- [SubagentRunStream](#SubagentRunStream)
- [MiddlewareResult](#MiddlewareResult)
- [Runtime](#Runtime)
- [ToolCallStreamUnion](#ToolCallStreamUnion)
- [WrapModelCallHandler](#WrapModelCallHandler)
- [WrapModelCallHook](#WrapModelCallHook)
- [WrapToolCallHook](#WrapToolCallHook)
- [ReactAgent](#ReactAgent)

---

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/...
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖智能体（Agents）相关的核心类型、中间件、钩子以及 `createAgent` 核心函数。

## 类型

### `AfterAgentHook`
<a id="AfterAgentHook"></a>
导入 `import { AfterAgentHook } from "langchain";`

```ts
AfterAgentHook < TSchema , TContext > | __type
```

智能体生命周期 `afterAgent` 事件的钩子类型。可以是一个处理器函数，或者包含处理器和可选跳转目标的对象。该钩子在智能体调用结束时调用一次。

## 类型

### `AfterModelHook`
<a id="AfterModelHook"></a>
导入 `import { AfterModelHook } from "langchain";`

```ts
AfterModelHook < TSchema , TContext > | __type
```

智能体生命周期 `afterModel` 事件的钩子类型。可以是一个处理器函数，或者包含处理器和可选跳转目标的对象。该钩子在每次模型调用之后调用。

## 接口

### `AgentMiddleware`
<a id="AgentMiddleware"></a>
导入 `import { AgentMiddleware } from "langchain";`

```ts
AgentMiddleware
```

Agent 中间件基础接口。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `afterAgent` | `AfterAgentHook < TSchema , TFullContext >` | 在智能体执行完成后运行的函数。该函数在智能体调用结束时调用一次。它允许在所有模型调用和工具执行完成后修改智能体的最终状态。 |
| `afterModel` | `AfterModelHook < TSchema , TFullContext >` | 模型调用后的函数。该函数在模型被调用且在任何工具被调用之前调用。它允许在模型被调用后修改智能体状态，例如更新工具调用参数。 |
| `beforeAgent` | `BeforeAgentHook < TSchema , TFullContext >` | 在智能体执行开始前运行的函数。该函数在智能体调用开始时调用一次。它允许在任何模型调用或工具执行之前修改智能体状态。 |
| `beforeModel` | `BeforeModelHook < TSchema , TFullContext >` | 模型调用前的函数。该函数在模型被调用之前以及在 `wrapModelCall` 钩子之前调用。它允许修改智能体状态。 |
| `contextSchema` | `TContextSchema` | 中间件上下文的架构。中间件上下文是只读的，且不会在多次调用之间持久化。它可以是：<br>A Zod 对象<br>A Zod 可选对象<br>A Zod 默认对象<br>Undefined |
| `name` | `string` | 中间件的名称。 |
| `stateSchema` | `TSchema` | 中间件状态的架构。中间件状态在多次调用之间是持久化的。它可以是：<br>A Zod 对象 (InteropZodObject)<br>来自 LangGraph 的 `StateSchema` (支持 `ReducedValue`, `UntrackedValue`)<br>一个 `AnnotationRoot`<br>Undefined |
| `streamTransformers` | `TStreamTransformers` | 中间件注册的流转换器工厂。这些将与 `createAgent({ streamTransformers })` 在智能体编译时合并。 |
| `tools` | `TTools` | 中间件注册的附加工具。 |
| `wrapModelCall` | `WrapModelCallHook < TSchema , TFullContext >` | 用自定义逻辑包装模型调用。这允许你：<br>在调用模型之前修改请求<br>处理错误并使用不同参数重试<br>对响应进行后处理<br>实现自定义缓存、日志或其他跨切面关注点 |
| `wrapToolCall` | `WrapToolCallHook < TSchema , TFullContext >` | 用自定义逻辑包装工具执行。这允许你：<br>在执行前修改工具调用参数<br>处理错误并使用不同参数重试<br>对工具结果进行后处理<br>实现缓存、日志、身份验证或其他跨切面关注点<br>为高级控制流返回 `Command` 对象 |

## 类型

### `AgentRunStream`
<a id="AgentRunStream"></a>
导入 `import { AgentRunStream } from "langchain";`

```ts
GraphRunStream < TValues , TExtensions > __type
```

一个带有原生智能体级别投影的 `GraphRunStream`，这些投影由 `createGraphRunStream`（通过 `__native` 转换器）直接分配到实例上。这是一个纯类型覆盖——不存在运行时子类。当你需要描述 `streamEvents(..., { version: "v3" })` 的返回类型时使用 `AgentRunStream` 类型。

## 接口

### `AgentTypeConfig`
<a id="AgentTypeConfig"></a>
导入 `import { AgentTypeConfig } from "langchain";`

```ts
AgentTypeConfig
```

类型包，封装了所有智能体类型参数。此接口将整个智能系统中使用的所有泛型类型参数捆绑到单个配置对象中。这种模式简化了类型签名，并使在不更改多个函数签名的情况下添加新类型参数变得更加容易。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `Context` | `TContext` | 上下文架构类型 |
| `Middleware` | `TMiddleware` | 中间件数组类型 |
| `Response` | `TResponse` | 使用 `responseFormat` 时的结构化响应类型 |
| `State` | `TState` | 自定义状态架构类型 |
| `StreamTransformers` | `TStreamTransformers` | 在 `createAgent({ streamTransformers })` 中注册的流转换器工厂元组。用于推断 `streamEvents(..., { version: "v3" })` 返回的 `run.extensions` 的形状。 |
| `Tools` | `TTools` | 来自智能体和中间件的合并工具类型 |

## 类型

### `AnyAgentMiddleware`
<a id="AnyAgentMiddleware"></a>
导入 `import { AnyAgentMiddleware } from "langchain";`

```ts
AgentMiddleware < StateDefinitionInit | undefined , InteropZodObject | InteropZodDefault < InteropZodObject > | InteropZodOptional < InteropZodObject > | undefined , any , readonly ClientTool | ServerTool [ ] , ReadonlyArray < ( ) = > StreamTransformer < any > >
```

用于智能体配置数组的放宽的中间件类型。它接受具有任何注册工具或流转换器的中间件实例，同时在传递 `const` 中间件数组时保留精确的推断。

## 类型

### `AnyAnnotationRoot`
<a id="AnyAnnotationRoot"></a>
导入 `import { AnyAnnotationRoot } from "langchain";`

```ts
AnnotationRoot < any >
```

## 类型

### `BeforeAgentHook`
<a id="BeforeAgentHook"></a>
导入 `import { BeforeAgentHook } from "langchain";`

```ts
BeforeAgentHook < TSchema , TContext > | __type
```

智能体生命周期 `beforeAgent` 事件的钩子类型。可以是一个处理器函数，或者包含处理器和可选跳转目标的对象。该钩子在智能体调用开始时调用一次。

## 类型

### `BeforeModelHook`
<a id="BeforeModelHook"></a>
导入 `import { BeforeModelHook } from "langchain";`

```ts
BeforeModelHook < TSchema , TContext > | __type
```

智能体生命周期 `beforeModel` 事件的钩子类型。可以是一个处理器函数，或者包含处理器和可选跳转目标的对象。该钩子在每次模型调用之前调用。

## 接口

### `BuiltInState`
<a id="BuiltInState"></a>
导入 `import { BuiltInState } from "langchain";`

```ts
BuiltInState
```

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `__interrupt__` | `Interrupt < unknown >[]` | |
| `jumpTo` | `"tools" | "model" | "end"` | 可选属性。当由中间件设置时，智能体将跳转到指定的节点，而不是遵循正常的路由逻辑。该属性在使用后会自动清除。<br>`model_request`: 跳转回模型进行另一次 LLM 调用<br>`tools`: 跳转到工具执行（需要有可用工具） |
| `messages` | `BaseMessage < TMessageStructure , MessageType >[]` | |

## 函数

### `createAgent`
<a id="createAgent"></a>
导入 `import { createAgent } from "langchain";`

```javascript
createAgent<StructuredResponseFormat extends Record<string, any> = Record<string, any>, TStateSchema extends StateDefinitionInit | undefined = undefined, ContextSchema extends InteropZodObject | AnyAnnotationRoot = AnyAnnotationRoot, TMiddleware extends readonly AnyAgentMiddleware[] = readonly AnyAgentMiddleware[], TTools extends readonly ClientTool | ServerTool[] = readonly ClientTool | ServerTool[], TStreamTransformers extends readonly () => StreamTransformer<any>[] = readonly []>(params: CreateAgentParams<StructuredResponseFormat, TStateSchema, ContextSchema, InteropZodType<StructuredResponseFormat>> & __type): ReactAgent<AgentTypeConfig<StructuredResponseFormat, TStateSchema, ContextSchema, TMiddleware, readonly [TTools, InferMiddlewareToolsArray<TMiddleware>], readonly [TStreamTransformers, InferMiddlewareStreamTransformersArray<TMiddleware>]>>
```

创建一个生产就绪的 ReAct（Reasoning + Acting）智能体，它将语言模型与工具和中间件结合，创建能够对任务进行推理、决定使用哪些工具并迭代地朝着解决方案工作的系统。

该智能体遵循 ReAct 模式，将推理步骤与工具调用交替进行，以迭代地朝着解决方案工作。它可以处理序列或并行的多个工具调用，在交互过程中保持状态，并提供可审计的决策过程。

### 核心组件

#### 模型
推理引擎可以指定为：
- **字符串标识符**：如 `"openai:gpt-4o"` 用于简单设置
- **模型实例**：配置好的模型对象以获得完全控制
- **动态函数**：根据状态在运行时选择模型

#### 工具
工具赋予智能体采取行动的能力：
- 传递由 `tool` 函数创建的工具数组
- 或者提供一个配置好的 `ToolNode` 用于自定义错误处理

#### 提示词 (Prompt)
塑造你的智能体处理任务的方式：
- 字符串用于简单指令
- `SystemMessage` 用于结构化提示
- 函数用于基于状态的动态提示

#### 中间件
中间件允许你扩展智能体的行为：
- 添加模型前/后处理以进行上下文注入或验证
- 添加动态控制流，例如终止调用或重试
- 添加人类在环 (Human-in-the-loop) 能力
- 向智能体添加工具调用
- 向智能体添加工具结果

### 高级功能

- **结构化输出**：使用 `responseFormat` 配合 Zod 架构来获取类型化的响应
- **记忆**：扩展状态架构以跨交互记忆信息
- **流式传输**：在智能体处理时获取实时更新

### 参数

| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `params` | `CreateAgentParams<StructuredResponseFormat, TStateSchema, ContextSchema, InteropZodType<StructuredResponseFormat>> & __type` | 是 | |

### 返回

`ReactAgent<AgentTypeConfig<StructuredResponseFormat, TStateSchema, ContextSchema, TMiddleware, readonly [TTools, InferMiddlewareToolsArray<TMiddleware>], readonly [TStreamTransformers, InferMiddlewareStreamTransformersArray<TMiddleware>]>>`

一个包含 `invoke` 和 `stream` 方法的 `ReactAgent` 实例

### 示例

#### 示例 1
```ts
import { createAgent, tool } from "langchain";
import { z } from "zod";

const search = tool(
  ({ query }) => `Results for: ${query}`,
  {
    name: "search",
    description: "Search for information",
    schema: z.object({
      query: z.string().describe("The search query"),
    })
  }
);

const agent = createAgent({
  llm: "openai:gpt-4o",
  tools: [search],
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Search for ReAct agents" }],
});
```

#### 示例 2
```ts
import { createAgent } from "langchain";
import { z } from "zod";

const ContactInfo = z.object({
  name: z.string(),
  email: z.string(),
  phone: z.string(),
});

const agent = createAgent({
  llm: "openai:gpt-4o",
  tools: [],
  responseFormat: ContactInfo,
});

const result = await agent.invoke({
  messages: [{
    role: "user",
    content: "Extract: John Doe, john@example.com, (555) 123-4567"
  }],
});

console.log(result.structuredResponse);
// { name: 'John Doe', email: 'john@example.com', phone: '(555) 123-4567' }
```

#### 示例 3
```ts
const stream = await agent.stream(
  { messages: [{ role: "user", content: "What's the weather?" }] },
  { streamMode: "values" }
);

for await (const chunk of stream) {
  // ...
}
```

#### 示例 4
```ts
import { createAgent } from "langchain";
import { StateSchema, ReducedValue } from "@langchain/langgraph";
import { z } from "zod";

const AgentState = new StateSchema({
  userId: z.string(),
  count: z.number().default(0),
  history: new ReducedValue(
    z.array(z.string()).default(() => []),
    { inputSchema: z.string(), reducer: (c, n) => [...c, n] }
  ),
});

const agent = createAgent({
  model: "openai:gpt-4o",
  tools: [searchTool],
  stateSchema: AgentState,
});
```

## 类型

### `CreateAgentParams`
<a id="CreateAgentParams"></a>
导入 `import { CreateAgentParams } from "langchain";`

```ts
CreateAgentParams Type
```

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `checkpointer` | `BaseCheckpointSaver | boolean` | 可选的检查点保存器，用于持久化智能体状态。 |
| `contextSchema` | `ContextSchema` | 可选的上下文架构。它允许在智能体调用中传递一个类型化的上下文对象，并在 `prompt` 和 `middleware` 等钩子中访问。与定义的 `stateSchema` 不同，上下文不会在智能体调用之间持久化。 |
| `description` | `string` | 可选的智能体描述。可用于向底层的监督 LLM 描述智能体。 |
| `includeAgentName` | `"inline"` | 用于指定如何向底层的监督 LLM 暴露智能体名称。<br>undefined：依赖于 LLM 提供商的 `AIMessage#name`。目前仅 OpenAI 支持。<br>\"inline\"：使用 XML 风格的标签将智能体名称直接添加到 `AIMessage` 的内容字段中。例如：\"How can I help you\" -> \"\<name\>agent_name\</name>\<content\>How can I help you?\</content>\" |
| `middleware` | `readonly AnyAgentMiddleware []` | 智能体执行期间运行的中间件实例。每个中间件都可以定义自己的状态架构并钩入智能体生命周期。 |
| `model` | `string | AgentLanguageModelLike` | 定义智能体使用的模型。你可以传递一个 LangChain 聊天模型实例，或者传递一个字符串。如果提供字符串，智能体会基于提供的模型名称和提供商初始化一个 `ChatModel`。它支持多种模型提供商并允许对模型参数进行运行时配置。 |
| `name` | `string` | 可选的智能体名称。 |
| `responseFormat` | `ResponseFormatType` | 可选的最终智能体输出架构。如果提供，输出将格式化以匹配给定的架构并返回在 `structuredResponse` 状态键中。如果不提供，输出状态中将不会存在 `structuredResponse`。可以作为以下形式传递：<br>Zod 架构<br>JSON 架构<br>Create React Agent ResponseFormat |
| `signal` | `AbortSignal` | 可选的中止信号，表示应中止整个操作。 |
| `stateSchema` | `TStateSchema` | 可选的智能体状态架构。它允许你定义持久化跨智能体调用的自定义状态属性，并可在钩子、中间件和整个智能体执行过程中访问。使用检查点时状态会被持久化，并且可以由中间件或在执行期间更新。与 `contextSchema` 定义的上下文不同，状态在有检查点时在智能体调用之间是持久化的，适用于维护对话历史、用户偏好或其他需要在多次交互中持久化的数据。 |
| `store` | `BaseStore` | 可选的存储器，用于持久化智能体状态。 |
| `streamTransformers` | `ReadonlyArray <() => StreamTransformer < any >>` | 内置到编译图中的流转换器工厂。它们会自动为每次 `streamEvents(..., { version: "v3" })` 调用运行，在内置智能体转换器（工具调用、中间件）之后以及传递给 `streamEvents(input, { version: "v3", transformers })` 的调用端转换器之前运行。你可以用它来添加始终应在智能体运行流中可用的领域特定流投影。投影值可以通过 `run.extensions` 访问。 |
| `systemPrompt` | `string | SystemMessage` | 可选的模型的系统提示。你可以使用字符串来提供简单的静态系统提示。这是最常用的用例，并且与模板字符串结合使用很好。当你需要结构化内容所需的特殊功能时，请使用 `SystemMessage`：<br>Anthropic 缓存控制：使用 `SystemMessage` 配合数组内容来启用每块的缓存控制设置（例如，`cache_control: { type: "ephemeral" }`）。这允许你在系统提示的不同部分具有不同的缓存设置。<br>多个内容块：当你需要具有不同元数据或格式要求的多个文本块时。<br>与现有代码集成：当你在处理已经产生 `SystemMessage` 实例的代码时。 |
| `tools` | `ServerTool | ClientTool []` | 工具或 `ToolNode` 列表。 |
| `version` | `"v1" | "v2"` | 确定要创建的图的版本。可以是：<br>`v1`: 工具节点处理包含所有工具调用的完整 `AIMessage`。所有工具调用通过 `Promise.all` 在单个图节点内并发执行。当你的工具调用子图或其他长运行的异步任务，并且你需要真正的并行时，选择 `v1` —— 因为 `Promise.all` 方法不受 LangGraph 每任务检查点序列化的影响。<br>`v2`: 每个工具调用使用 `Send` API 作为独立的图任务分发。任务由 LangGraph 并行调度，但当工具调用子图时，底层的检查点写入可能会导致有效的序列化，使得并发工具调用顺序执行。当你需要每项工具调用的检查点、独立的故障隔离或在单个工具调用中支持 `interrupt()` 时，`v2` 是更好的选择。 |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖 LangChain JS 中关于智能体（Agents）的核心组件，包括中间件、转换器、类型定义以及生产就绪的 ReActAgent。

## 函数
### `createMiddleware`
<a id="createMiddleware"></a>
```ts
createMiddleware<TSchema extends StateDefinitionInit | undefined = undefined, TContextSchema extends InteropZodObject | undefined = undefined, TTools extends readonly ClientTool | ServerTool[] = readonly ClientTool | ServerTool[], TStreamTransformers extends readonly () => StreamTransformer<any>[] = readonly []>(config: __type): AgentMiddleware<TSchema, TContextSchema, NormalizeContextSchema<TContextSchema>, TTools, TStreamTransformers>
```
创建具有自动架构推断的中间件实例。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `config` | `__type` | 是 | 中间件配置 |

返回：
`AgentMiddleware<TSchema, TContextSchema, NormalizeContextSchema<TContextSchema>, TTools, TStreamTransformers>`
一个中间件实例。

### 示例
```ts
const authMiddleware = createMiddleware({
  name: "AuthMiddleware",
  stateSchema: z.object({
    isAuthenticated: z.boolean().default(false),
  }),
  contextSchema: z.object({
    userId: z.string(),
  }),
  beforeModel: async (state, runtime) => {
    if (!state.isAuthenticated) {
      throw new Error("Not authenticated");
    }
  },
});
```

```ts
import { StateSchema, ReducedValue } from "@langchain/langgraph";

const historyMiddleware = createMiddleware({
  name: "HistoryMiddleware",
  stateSchema: new StateSchema({
    count: z.number().default(0),
    history: new ReducedValue(
      z.array(z.string()).default(() => []),
      { inputSchema: z.string(), reducer: (current, next) => [...current, next] }
    ),
  }),
  beforeModel: async (state, runtime) => {
    return { count: state.count + 1 };
  },
});
```

### `createSubagentTransformer`
<a id="createSubagentTransformer"></a>
```ts
createSubagentTransformer(scope: Namespace = []): () => NativeStreamTransformer<SubagentProjection>
```
创建一个原生转换器，用于在 `run.subagents` 上呈现嵌套的命名智能体。

它监控 `tasks` 事件以记录每个命名空间的 `lc_agent_name`（由 `createAgent({ name })` 设置）以及触发的工具调用，然后为任何一个级别低于作用域且带有 `lc_agent_name` 的嵌套运行发送带类型的 `SubagentRunStream` 处理程序。

每个处理程序都由其专有的每个子代理转换器实例（`createMessagesTransformer`、`createToolCallTransformer` 和嵌套的 `createSubagentTransformer`）支持，并作用于子代理的命名空间。

子树中的每个事件都直接进入这些转换器，它们通过命名空间进行自我过滤；子代理的最终 `output` 在其 `lifecycle` 完成时从其最后的 `values` 快照中解析。

标记为 `__native: true` —— `subagents` 投影直接出现在 `GraphRunStream` 实例上作为 `run.subagents`。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `scope` | `Namespace` | 否 | 此转换器所属的命名空间前缀。根智能体使用 `[]`；嵌套处理程序使用其子智能体的命名空间，因此孙子智能体会递归发现。(默认值: `[]`) |

返回：
`() => NativeStreamTransformer<SubagentProjection>`

### `createToolCallTransformer`
<a id="createToolCallTransformer"></a>
```ts
createToolCallTransformer ( path : Namespace ) : ( ) = > NativeStreamTransformer < ToolCallProjection >
```
创建一个原生转换器，将工具通道事件相关联到每个调用的 `ToolCallStream` 对象中。

标记为 `__native: true` —— 投影键直接出现在 `GraphRunStream` 实例上作为 `run.toolCalls`。

| 名称 | 类型 | 说明 |
|------|------|-------------|
| `path` | `*` | `Namespace` |

## 接口
### `DefaultAgentTypeConfig`
<a id="DefaultAgentTypeConfig"></a>
```ts
interface DefaultAgentTypeConfig
```
智能体的默认类型配置。当未提供显式类型参数时使用。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `Context` | `AnyAnnotationRoot` | 上下文架构类型属性 |
| `Middleware` | `readonly AnyAgentMiddleware []` | 中间件数组类型属性 |
| `Response` | `Record < string , any >` | 使用 `responseFormat` 属性时的结构化响应类型 |
| `State` | `undefined` | 自定义状态架构类型属性 |
| `StreamTransformers` | `readonly []` | 在 `createAgent({ streamTransformers })` 中注册的流转换器元组。用于推断 `streamEvents(..., { version: "v3" })` 返回流上的 `run.extensions` 形状 |
| `Tools` | `readonly ClientTool | ServerTool []` | 智能体和中间件的组合工具类型，继承自 `AgentTypeConfig` |
| `P Context` | `TContext` | 上下文架构类型属性 |
| `P Middleware` | `TMiddleware` | 中间件数组类型属性 |
| `P Response` | `TResponse` | 使用 `responseFormat` 属性时的结构化响应类型 |
| `P State` | `TState` | 自定义状态架构类型属性 |
| `P StreamTransformers` | `TStreamTransformers` | 在 `createAgent({ streamTransformers })` 中注册的流转换器元组 |
| `P Tools` | `TTools` | 智能体和中间件的组合工具类型 |

### `MiddlewareTypeConfig`
<a id="MiddlewareTypeConfig"></a>
```ts
interface MiddlewareTypeConfig
```
封装了所有中间件类型参数的类型包。

此接口将中间件系统中使用的所有泛型类型参数捆绑到单个配置对象中。这种模式简化了类型签名，并使无需更改多个函数签名即可轻松添加新的类型参数。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `ContextSchema` | `TContextSchema` | 中间件上下文架构类型 |
| `FullContext` | `TFullContext` | 完整上下文类型 |
| `Schema` | `TSchema` | 中间件状态架构类型 |
| `StreamTransformers` | `TStreamTransformers` | 中间件注册的流转换器 |
| `Tools` | `TTools` | 工具数组类型 |

### `SubagentRunStream`
<a id="SubagentRunStream"></a>
```ts
interface SubagentRunStream
```
在 `run.subagents` 上呈现的嵌套命名智能体执行。

一个子智能体是嵌套的 `createAgent` 运行，其 `lc_agent_name` 与其父智能体不同，例如在工具主体内调用的 `createAgent({ name })`。

该处理程序暴露了作用域投影（`messages`、`toolCalls`、嵌套的 `subagents`）以及解析出的名称、触发它的原因以及子智能体的最终输出状态。

这是一个自包含的处理程序（不是 `SubgraphRunStream` 的子类）：其投影是由直接由 `createSubagentTransformer` 驱动的每个子代理转换器实例产生的。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `cause` | `LifecycleCause | undefined` | 派发此子智能体的工具调用，如 `{ type: "toolCall", tool_call_id }`，或者当无法恢复原始工具调用时为 `undefined`（例如它不是从工具触发的）。 |
| `messages` | `AsyncIterable < Omit < ChatModelStream $1, "then" >>` | 作用于此子代理的每条消息聊天模型令牌流。 |
| `name` | `string` | 子智能体的 `lc_agent_name`（由 `createAgent({ name })` 设置）。 |
| `output` | `Promise < TValues >` | 在完成时解析出子智能体的最终状态。 |
| `subagents` | `AsyncIterable < SubagentRunStream < Record < string , unknown >, readonly ClientTool | ServerTool []>>` | 此子智能体从其自身的工具中派发的嵌套子智能体。 |
| `toolCalls` | `AsyncIterable < ToolCallStreamUnion < TTools >>` | 子智能体自身的工具调用流。 |

## 类型
### `MiddlewareResult`
<a id="MiddlewareResult"></a>
```ts
MiddlewareResult : TState __type | void
```
中间件函数的返回类型。

### `Runtime`
<a id="Runtime"></a>
```ts
Runtime : Partial < Omit < LangGraphRuntime < TContext > , "context" | "configurable" > > WithMaybeContext < TContext > __type
```
可供中间件使用的运行时信息（只读）。

### `ToolCallStreamUnion`
<a id="ToolCallStreamUnion"></a>
```ts
ToolCallStreamUnion : { [ K in keyof TTools ] : ToolCallStream < ToolNameOf < TTools [ K ] > , ToolInputOf < TTools [ K ] > , ToolOutputOf < TTools [ K ] > > } [ number ]
```
工具变体（variants）的判别联合类型，每个工具对应一个。它使 TypeScript 在消费者检查 `call.name === "someToolName"` 时能够缩小 `.input` 和 `.output` 的类型。

当工具元组是没有任何字面量名称类型的普通 `(ClientTool | ServerTool)[]` 时，回退到 `ToolCallStream`（无类型）。

### `WrapModelCallHandler`
<a id="WrapModelCallHandler"></a>
```ts
WrapModelCallHandler : ( request : Omit < ModelRequest < NormalizedSchemaInput < TSchema > , TContext > , "systemPrompt" | "systemMessage" > __type ) = > PromiseOrValue < AIMessage >
```
用于包装模型调用的处理函数类型。

它接收一个模型请求并返回 AI 消息响应。

### `WrapModelCallHook`
<a id="WrapModelCallHook"></a>
```ts
WrapModelCallHook : ( request : ModelRequest < NormalizedSchemaInput < TSchema > , TContext > , handler : WrapModelCallHandler < TSchema , TContext > ) = > PromiseOrValue < AIMessage | Command >
```
用于 `wrapModelCall` 钩子的包装函数类型。

它允许中间件拦截并修改模型执行。

这使您可以：
- 在调用模型之前修改请求（例如，更改系统提示词，添加/移除工具）
- 处理错误并使用不同参数重试
- 后处理响应
- 实现自定义缓存、日志或其他横切关注点

### `WrapToolCallHook`
<a id="WrapToolCallHook"></a>
```ts
WrapToolCallHook : ( request : ToolCallRequest < NormalizedSchemaInput < TSchema > , TContext > , handler : ToolCallHandler < NormalizedSchemaInput < TSchema > , TContext > ) = > PromiseOrValue < ToolMessage | Command >
```
用于 `wrapToolCall` 钩子的包装函数类型。

它允许中间件拦截并修改工具执行。

## 类
### `ReactAgent`
<a id="ReactAgent"></a>
```ts
interface ReactAgent
```
ReactAgent 是一个生产就绪的 ReAct（Reasoning + Acting）智能体，它结合了语言模型与工具和中间件。

该智能体由单个类型包 `Types` 参数化，该包封装了所有类型信息：

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `options` | `any` | |
| `checkpointer` | `any` | |
| `graph` | `any` | |
| `store` | `any` | |

| 方法 | 签名 | 说明 |
|------|------|-------------|
| `drawMermaid()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/drawMermaid` | |
| `drawMermaidPng()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/drawMermaidPng` | |
| `invoke()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/invoke` | |
| `stream()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/stream` | |
| `streamEvents()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/streamEvents` | |
| `withConfig()` | `https://reference.langchain.com/javascript/langchain/index/ReactAgent/withConfig` | |

### 示例
```typescript
// Using the type bag pattern
type MyTypes = AgentTypeConfig<
  { name: string },  // Response
  typeof myState,    // State
  typeof myContext,  // Context
  typeof middleware, // Middleware
  typeof tools       // Tools
>;

const agent: ReactAgent<MyTypes> = createAgent({ ... });
```