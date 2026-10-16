# LangChain JS API 参考 · 类型与工具（Types & Utilities）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章收录 `langchain` 导出的各类**类型别名**（大多为类型推断辅助工具）与 `ModelRequest` 接口。所有这些类型通常用于从 `createAgent` 的返回值、中间件或 `AgentTypeConfig` / `MiddlewareTypeConfig` 中**推断**出对应的状态、上下文、工具、响应等类型。

> 约定：`T` / `TMiddleware` / `TSchema` 等为泛型参数；这些"类型工具"本身不参与运行时，只在编辑器中提供类型提示。

## 目录

- [ModelRequest](#ModelRequest)
- [类型工具（Infer*/Resolve*/Combine*/Normalized*）](#Infer工具)

---

## 接口

### `ModelRequest`
<a id="ModelRequest"></a>

**导入**
```ts
import type { ModelRequest } from "langchain";
```

**说明**
在运行时修改一次模型调用的配置。所有字段都是可选的，只有提供的字段才会覆盖默认值。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `messages` | `BaseMessage<...>[]` | 发送给模型的消息。 |
| `model` | `AgentLanguageModelLike` | 本步骤使用的模型。 |
| `modelSettings` | `Record<string, unknown>` | 准备调用时绑定到模型的附加设置（例如 `headers`、`container` 等），通过 `bindTools()` 应用，每次请求都会重新绑定，因此可逐次变化。 |
| `responseFormat` | `TResponseFormat` | 本步骤的结构化输出配置。中间件可覆盖它以动态启用或改变结构化输出。 |
| `runtime` | `Runtime<TContext>` | 运行时上下文（元数据、signal、writer、interrupt 等）。 |
| `state` | `TState & AgentBuiltInState` | 当前智能体状态（含中间件状态与内置状态）。 |
| `systemMessage` | `SystemMessage` | 本步骤的系统消息。若初始化 `createAgent` 时未提供 `systemPrompt`，则使用空系统消息。 |
| `toolChoice` | `"required" \| "auto" \| "none" \| { type: "function", function: { name: string } }` | 工具选择配置（模型特定格式）。`"auto"` 由模型自行决定；`"none"` 不调用工具；`"required"` 必须调用；对象形式指定具体函数。 |
| `tools` | `ClientTool \| ServerTool[]` | 本步骤可用的工具。 |
| `systemPrompt` | `string` | ⚠ 已弃用 —— 本步骤的系统消息字符串。 |

**示例（`wrapModelCall` 中修改系统消息）**
```ts
wrapModelCall: async (request, handler) => {
  return handler({
    ...request,
    systemMessage: request.systemMessage.concat("something"),
  });
}
```

---

## 类型工具（Infer*/Resolve*/Combine*/Normalized*）
<a id="Infer工具"></a>

以下类型均为**类型级别的辅助工具**，用于从智能体/中间件推断出具体类型。除特别说明外，导入路径均为 `import type { X } from "langchain";`。

### `InferAgentType`
<a id="InferAgentType"></a>
```ts
InferAgentType : ResolveAgentTypeConfig<T>[K]
```
从 `AgentTypeConfig` 或 `ReactAgent` 中提取任意属性 `K`。

### `InferAgentState`
<a id="InferAgentState"></a>
```ts
InferAgentState : InferSchemaValue<InferAgentType<T, "State">> & InferMiddlewareStates<InferAgentType<T, "Middleware">>
```
推断智能体的**完整合并状态**（智能体自身状态 + 全部中间件状态），与 `invoke` 返回的状态类型一致。
示例：`type State = InferAgentState<typeof agent>;`

### `InferAgentStateSchema`
<a id="InferAgentStateSchema"></a>
```ts
InferAgentStateSchema : InferAgentType<T, "State">
```
提取传给 `createAgent` 的**原始** `stateSchema` 类型（不与中间件合并）。要取合并后的状态请用 `InferAgentState`。

### `InferAgentContext`
<a id="InferAgentContext"></a>
```ts
InferAgentContext : InferSchemaValue<InferAgentType<T, "Context">> & InferMiddlewareContexts<InferAgentType<T, "Middleware">>
```
推断智能体的**完整合并上下文**（智能体自身 `contextSchema` + 全部中间件上下文）。

### `InferAgentContextSchema`
<a id="InferAgentContextSchema"></a>
```ts
InferAgentContextSchema : InferAgentType<T, "Context">
```
提取传给 `createAgent` 的**原始** `contextSchema` 类型（不与中间件合并）。

### `InferAgentTools`
<a id="InferAgentTools"></a>
```ts
InferAgentTools : InferAgentType<T, "Tools">
```
提取 `Tools` 类型。示例：`type Tools = InferAgentTools<typeof agent>;`

### `InferAgentResponse`
<a id="InferAgentResponse"></a>
```ts
InferAgentResponse : InferAgentType<T, "Response">
```
提取结构化 `Response` 类型。示例：`type Response = InferAgentResponse<typeof agent>;`

### `InferAgentStreamTransformers`
<a id="InferAgentStreamTransformers"></a>
```ts
InferAgentStreamTransformers : InferAgentType<T, "StreamTransformers">
```
提取 `StreamTransformers`（转换器工厂元组）类型。

### `InferChannelType`
<a id="InferChannelType"></a>
```ts
InferChannelType : T extends AnyAnnotationRoot ? ToAnnotationRoot<T>["State"] : T extends InteropZodObject ? InferInteropZodInput<T> : __type
```
根据注解根或 Zod 对象推断通道类型。

### `InferContextInput`
<a id="InferContextInput"></a>
```ts
InferContextInput : ContextSchema extends InteropZodObject ? InferInteropZodInput<ContextSchema> : ContextSchema extends AnyAnnotationRoot ? ToAnnotationRoot<ContextSchema>["State"] : __type
```
从上下文 schema 推断输入类型（含可选默认值）。

### `InferMergedState`
<a id="InferMergedState"></a>
```ts
InferMergedState : InferMiddlewareStates<T> & AgentBuiltInState
```
从中间件数组推断合并状态（含内置状态）。

### `InferMergedInputState`
<a id="InferMergedInputState"></a>
```ts
InferMergedInputState : InferMiddlewareInputStates<T> & AgentBuiltInState
```
从中间件数组推断合并输入状态（含内置状态）。

### `InferMiddlewareType`
<a id="InferMiddlewareType"></a>
```ts
InferMiddlewareType : ResolveMiddlewareTypeConfig<T>[K]
```
从 `MiddlewareTypeConfig` 或 `AgentMiddleware` 中提取任意属性 `K`。

### `InferMiddlewareState`
<a id="InferMiddlewareState"></a>
```ts
InferMiddlewareState : T extends AgentMiddleware<TSchema, any, any, any, any>
  ? TSchema extends StateSchema<TFields> ? FilterPrivateProps<InferStateSchemaValue<TFields>>
  : TSchema extends InteropZodObject ? FilterPrivateProps<InferInteropZodOutput<TSchema>>
  : TSchema extends StateDefinitionInit ? FilterPrivateProps<InferSchemaValue<TSchema>> : __type
  : __type
```
从单个中间件推断其状态 schema 类型（过滤以 `_` 开头的私有属性，支持 Zod 与 LangGraph `StateSchema`）。

### `InferMiddlewareStates`
<a id="InferMiddlewareStates"></a>
```ts
InferMiddlewareStates : T extends readonly []
  ? __type
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? InferMiddlewareState<First> & InferMiddlewareStates<Rest> : InferMiddlewareState<First>
    : __type
```
从中间件数组推断合并状态（仅中间件状态）。

### `InferMiddlewareInputState`
<a id="InferMiddlewareInputState"></a>
```ts
InferMiddlewareInputState : T extends AgentMiddleware<TSchema, any, any, any, any>
  ? TSchema extends StateSchema<TFields> ? FilterPrivateProps<InferStateSchemaUpdate<TFields>>
  : TSchema extends InteropZodObject ? FilterPrivateProps<InferInteropZodInput<TSchema>>
  : TSchema extends StateDefinitionInit ? FilterPrivateProps<InferSchemaInput<TSchema>> : __type
  : __type
```
从单个中间件推断其输入状态 schema 类型（所有属性可选，过滤私有属性）。

### `InferMiddlewareInputStates`
<a id="InferMiddlewareInputStates"></a>
```ts
InferMiddlewareInputStates : T extends readonly [] ? __type
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? InferMiddlewareInputState<First> & InferMiddlewareInputStates<Rest> : InferMiddlewareInputState<First>
    : __type
```
从中间件数组推断合并输入状态（含默认值）。

### `InferMiddlewareContext`
<a id="InferMiddlewareContext"></a>
```ts
InferMiddlewareContext : T extends AgentMiddleware<any, TContextSchema, any, any, any>
  ? TContextSchema extends InteropZodObject ? InferInteropZodInput<TContextSchema> : __type
  : __type
```
从单个中间件推断其上下文 schema 类型。

### `InferMiddlewareContextInput`
<a id="InferMiddlewareContextInput"></a>
```ts
InferMiddlewareContextInput : T extends AgentMiddleware<any, TContextSchema, any, any, any>
  ? TContextSchema extends InteropZodOptional<Inner> ? InferInteropZodInput<Inner> | undefined
  : TContextSchema extends InteropZodObject ? InferInteropZodInput<TContextSchema> : __type
  : __type
```
从单个中间件推断输入上下文 schema 类型（含可选默认值）。

### `InferMiddlewareContexts`
<a id="InferMiddlewareContexts"></a>
```ts
InferMiddlewareContexts : T extends readonly [] ? __type
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? InferMiddlewareContext<First> & InferMiddlewareContexts<Rest> : InferMiddlewareContext<First>
    : __type
```
从中间件数组推断合并上下文。

### `InferMiddlewareContextInputs`
<a id="InferMiddlewareContextInputs"></a>
```ts
InferMiddlewareContextInputs : T extends readonly [] ? __type
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? MergeContextTypes<InferMiddlewareContextInput<First>, InferMiddlewareContextInputs<Rest>> : InferMiddlewareContextInput<First>
    : __type
```
从中间件数组推断合并输入上下文（含默认值）。

### `InferMiddlewareContextSchema`
<a id="InferMiddlewareContextSchema"></a>
```ts
InferMiddlewareContextSchema : InferMiddlewareType<T, "ContextSchema">
```
提取中间件的 `ContextSchema` 类型。

### `InferMiddlewareFullContext`
<a id="InferMiddlewareFullContext"></a>
```ts
InferMiddlewareFullContext : InferMiddlewareType<T, "FullContext">
```
提取中间件的 `FullContext` 类型。

### `InferMiddlewareSchema`
<a id="InferMiddlewareSchema"></a>
```ts
InferMiddlewareSchema : InferMiddlewareType<T, "Schema">
```
提取中间件的 `Schema` 类型。

### `InferMiddlewareTools`
<a id="InferMiddlewareTools"></a>
```ts
InferMiddlewareTools : T extends AgentMiddleware<any, any, any, TTools, any>
  ? TTools extends readonly (ClientTool | ServerTool)[] ? TTools : readonly []
  : readonly []
```
从单个中间件推断其工具（提取 `AgentMiddleware` 的 `TTools` 类型参数）。

### `InferMiddlewareToolsArray`
<a id="InferMiddlewareToolsArray"></a>
```ts
InferMiddlewareToolsArray : T extends readonly [] ? readonly []
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? readonly [...InferMiddlewareTools<First>, ...InferMiddlewareToolsArray<Rest>] : InferMiddlewareTools<First>
    : readonly []
```
从中间件数组递归提取并合并工具为单个元组。

### `InferMiddlewareToolsFromConfig`
<a id="InferMiddlewareToolsFromConfig"></a>
```ts
InferMiddlewareToolsFromConfig : InferMiddlewareType<T, "Tools">
```
从 `MiddlewareTypeConfig` 或 `AgentMiddleware` 提取 `Tools` 类型。

### `InferMiddlewareStreamTransformers`
<a id="InferMiddlewareStreamTransformers"></a>
```ts
InferMiddlewareStreamTransformers : T extends AgentMiddleware<any, any, any, any, TStreamTransformers> ? [TStreamTransformers] extends [readonly []] ? readonly [] : TStreamTransformers : readonly []
```
从单个中间件推断流转换器。

### `InferMiddlewareStreamTransformersArray`
<a id="InferMiddlewareStreamTransformersArray"></a>
```ts
InferMiddlewareStreamTransformersArray : T extends readonly [] ? readonly []
  : T extends readonly [First, ...Rest]
    ? First extends AgentMiddleware ? readonly [...InferMiddlewareStreamTransformers<First>, ...InferMiddlewareStreamTransformersArray<Rest>] : InferMiddlewareStreamTransformers<First>
    : readonly []
```
从中间件数组推断并合并流转换器。

### `InferMiddlewareStreamTransformersFromConfig`
<a id="InferMiddlewareStreamTransformersFromConfig"></a>
```ts
InferMiddlewareStreamTransformersFromConfig : InferMiddlewareType<T, "StreamTransformers">
```
从 `MiddlewareTypeConfig` 或 `AgentMiddleware` 提取 `StreamTransformers` 类型。

### `InferSchemaInput`
<a id="InferSchemaInput"></a>
```ts
InferSchemaInput : [A] extends [undefined] ? __type
  : A extends StateSchema<TFields> ? InferStateSchemaUpdate<TFields>
  : A extends InteropZodObject ? InferInteropZodInput<A>
  : A extends AnyAnnotationRoot ? A["Update"] : __type
```
根据 schema 推断其输入（更新）类型。

### `InferSchemaValue`
<a id="InferSchemaValue"></a>
```ts
InferSchemaValue : [A] extends [undefined] ? __type
  : A extends StateSchema<TFields> ? InferStateSchemaValue<TFields>
  : A extends InteropZodObject ? InferInteropZodOutput<A>
  : A extends AnyAnnotationRoot ? A["State"] : __type
```
根据 schema 推断其输出（值）类型。

### `InferSchemaValueType`
<a id="InferSchemaValueType"></a>
```ts
InferSchemaValueType : [TSchema] extends [never] ? AgentBuiltInState
  : TSchema extends StateSchema<TFields> ? InferStateSchemaValue<TFields> & AgentBuiltInState
  : TSchema extends InteropZodObject ? InferInteropZodOutput<TSchema> & AgentBuiltInState
  : TSchema extends StateDefinitionInit ? InferSchemaValue<TSchema> & AgentBuiltInState
  : AgentBuiltInState
```
推断 schema 的值类型，并与内置状态合并。

### `InferSchemaUpdateType`
<a id="InferSchemaUpdateType"></a>
```ts
InferSchemaUpdateType : [TSchema] extends [never] ? AgentBuiltInState
  : TSchema extends StateSchema<TFields> ? InferStateSchemaUpdate<TFields> & AgentBuiltInState
  : TSchema extends InteropZodObject ? InferInteropZodInput<TSchema> & AgentBuiltInState
  : TSchema extends StateDefinitionInit ? InferSchemaInput<TSchema> & AgentBuiltInState
  : AgentBuiltInState
```
推断 schema 的更新类型，并与内置状态合并。

### `NormalizedSchemaInput`
<a id="NormalizedSchemaInput"></a>
```ts
NormalizedSchemaInput : InferSchemaValueType<TSchema>
```
规范化后的 schema 输入类型。

### `NormalizedSchemaUpdate`
<a id="NormalizedSchemaUpdate"></a>
```ts
NormalizedSchemaUpdate : InferSchemaUpdateType<TSchema>
```
规范化后的 schema 更新类型。

### `ExtractZodArrayTypes`
<a id="ExtractZodArrayTypes"></a>
```ts
ExtractZodArrayTypes : T extends readonly [InteropZodType<A>, ...Rest]
  ? Rest extends readonly InteropZodType<any>[] ? A | ExtractZodArrayTypes<Rest> : A
  : never
```
从 Zod schema 数组中提取联合类型。

### `ToAnnotationRoot`
<a id="ToAnnotationRoot"></a>
```ts
ToAnnotationRoot : A extends AnyAnnotationRoot ? A
  : A extends InteropZodObject ? InteropZodToStateDefinition<A>
  : never
```
将注解根或 Zod 对象转换为注解根。

### `ResolveAgentTypeConfig`
<a id="ResolveAgentTypeConfig"></a>
```ts
ResolveAgentTypeConfig : T extends { Types: AgentTypeConfig } ? Types
  : T extends AgentTypeConfig ? T : never
```
将输入解析为 `AgentTypeConfig`：可直接传入 `AgentTypeConfig`，也可传入 `ReactAgent` 实例（`typeof agent`）。

### `ResolveMiddlewareTypeConfig`
<a id="ResolveMiddlewareTypeConfig"></a>
```ts
ResolveMiddlewareTypeConfig : T extends { Types: MiddlewareTypeConfig } ? Types
  : T extends MiddlewareTypeConfig ? T : never
```
将输入解析为 `MiddlewareTypeConfig`：可直接传入，也可传入 `AgentMiddleware` 实例。

### `CombineTools`
<a id="CombineTools"></a>
```ts
CombineTools : readonly [...TAgentTools, ...InferMiddlewareToolsArray<TMiddleware>]
```
将智能体工具与中间件工具合并为单个只读数组。

### `CombineStreamTransformers`
<a id="CombineStreamTransformers"></a>
```ts
CombineStreamTransformers : readonly [...TAgentStreamTransformers, ...InferMiddlewareStreamTransformersArray<TMiddleware>]
```
将智能体流转换器与中间件流转换器合并。

### `DefaultMiddlewareTypeConfig`
<a id="DefaultMiddlewareTypeConfig"></a>
```ts
DefaultMiddlewareTypeConfig : MiddlewareTypeConfig
```
中间件默认类型配置（未显式提供类型参数时使用）。

### `DynamicSystemPromptMiddlewareConfig`
<a id="DynamicSystemPromptMiddlewareConfig"></a>
```ts
DynamicSystemPromptMiddlewareConfig : (
  state: AgentBuiltInState,
  runtime: Runtime<TContextSchema>
) => string | SystemMessage | Promise<string | SystemMessage>
```
动态系统提示词中间件的配置类型。

### `HumanInTheLoopMiddlewareConfig`
<a id="HumanInTheLoopMiddlewareConfig"></a>
```ts
HumanInTheLoopMiddlewareConfig : InferInteropZodInput<typeof contextSchema>
```
人在环路中间件的配置类型。

### `HeadlessTool`
<a id="HeadlessTool"></a>
```ts
HeadlessTool : DynamicStructuredTool<SchemaT, InferInteropZodOutput<SchemaT>, InferInteropZodInput<SchemaT>, unknown, unknown, NameT> & __type
```
一种无头工具，在服务端总是中断智能体执行。其实现由客户端通过 `useStream({ tools: [...] })` 的 `.implement()` 单独提供。

### `HeadlessToolFields`
<a id="HeadlessToolFields"></a>
```ts
interface HeadlessToolFields {
  description: string;
  name: NameT;
  schema: SchemaT;
}
```
创建无头工具的配置字段：`description`（工具描述）、`name`（工具名，客户端据此匹配实现）、`schema`（定义输入的 Zod schema）。

### `HeadlessToolImplementation`
<a id="HeadlessToolImplementation"></a>
```ts
interface HeadlessToolImplementation {
  execute: (args: InferInteropZodOutput<SchemaT>) => Promise<OutputT>;
  tool: HeadlessTool<SchemaT, NameT>;
}
```
将无头工具与其执行函数配对。由 `HeadlessTool` 的 `.implement()` 创建，传给客户端的 `useStream({ tools: [...] })`。

### `ToolsToMessageToolSet`
<a id="ToolsToMessageToolSet"></a>
```ts
ToolsToMessageToolSet : { [K in T[number]]: ExtractToolDefinition<K> }
```
将工具数组 `(ClientTool | ServerTool)[]` 转换为 `MessageToolSet`（把每个工具名映射为其含输入/输出类型的 `MessageToolDefinition`）。
示例：`type ToolSet = ToolsToMessageToolSet<readonly [typeof myTool]>;`

### `UserInput`
<a id="UserInput"></a>
```ts
UserInput : InferSchemaInput<TStateSchema> & __type
```
`.invoke` 与 `.stream` 方法的基础输入类型。

### `TokenCounter`
<a id="TokenCounter"></a>
```ts
TokenCounter : (messages: BaseMessage[]) => number | Promise<number>
```
token 计数器函数类型。

### `ToolCallHandler`
<a id="ToolCallHandler"></a>
```ts
ToolCallHandler : (request: ToolCallRequest<TSchema, TContext>) => PromiseOrValue<ToolMessage | Command>
```
包装工具调用的处理函数类型：接收工具调用请求，返回工具结果或 `Command`。

### `JumpTo`
<a id="JumpTo"></a>
```ts
JumpTo : "model_request" | "tools" | typeof END
```
跳转目标（内部使用）。

### `JumpToTarget`
<a id="JumpToTarget"></a>
```ts
JumpToTarget : typeof JUMP_TO_TARGETS[number]
```
跳转目标类型。

### `N`
<a id="N"></a>
```ts
N : typeof START | "model_request" | "tools"
```
图节点名称联合类型。

### `WithStateGraphNodes`
<a id="WithStateGraphNodes"></a>
```ts
WithStateGraphNodes : Graph extends StateGraph<SD, S, U, N, I, O, C>
  ? StateGraph<SD, S, U, N | K, I, O, C>
  : never
```
在 `StateGraph` 上追加节点名 `K` 的辅助类型。

### `InferChannelType`
<a id="InferChannelType"></a>
```ts
InferChannelType : T extends AnyAnnotationRoot ? ToAnnotationRoot<T>["State"] : T extends InteropZodObject ? InferInteropZodInput<T> : __type
```
推断通道类型（同 `InferChannelType` 说明见上）。
