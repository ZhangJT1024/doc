# LangChain JS API 参考 · 工具（Tools）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 目录

- [Action](#Action)
- [ActionRequest](#ActionRequest)
- [DynamicStructuredTool](#DynamicStructuredTool)
- [DynamicTool](#DynamicTool)
- [ExecutedToolCall](#ExecutedToolCall)
- [providerStrategy](#providerStrategy)
- [ResponseFormat](#ResponseFormat)
- [ResponseFormatUndefined](#ResponseFormatUndefined)
- [StructuredTool](#StructuredTool)
- [Tool](#Tool)
- [ToolCall](#ToolCall)
- [ToolCallRequest](#ToolCallRequest)
- [ToolResult](#ToolResult)
- [toolStrategy](#toolStrategy)
- [ToolStrategy](#ToolStrategy)
- [TypedToolStrategy](#TypedToolStrategy)
- [ToolRuntime](#ToolRuntime)
- [HeadlessTool](#HeadlessTool)
- [HeadlessToolFields](#HeadlessToolFields)
- [HeadlessToolImplementation](#HeadlessToolImplementation)

---

## 类
### `Action`
<a id="Action"></a>
import { Action } from "langchain";

Action 代表一个具有名称和参数的动作。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | `Record < string , any >` | 动作所需的参数键值对（例如，`{"a": 1, "b": 2}`）。 |
| name | `string` | 请求的动作的类型或名称（例如，`"add_numbers"`）。 |

## 类
### `ActionRequest`
<a id="ActionRequest"></a>
import { ActionRequest } from "langchain";

ActionRequest 代表一个具有名称、参数和描述的动作请求。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | `Record < string , any >` | 动作所需的参数键值对（例如，`{"a": 1, "b": 2}`）。 |
| description | `string` | 将要审查的动作的描述。 |
| name | `string` | 请求的动作的名称。 |

## 类
### `DynamicStructuredTool`
<a id="DynamicStructuredTool"></a>
import { DynamicStructuredTool } from "langchain";

一个可以从函数、名称和描述动态创建的工具，旨在与结构化数据一起工作。它扩展了 `StructuredTool` 类，并覆盖了 `_call` 方法，以便在调用工具时执行提供的函数。

Schema 可以作为 Zod 或 JSON schema 传递。如果传递了 JSON schema，工具将不会验证输入。

```javascript
class DynamicStructuredTool
```

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| callbacks | | |
| defaultConfig | | |
| description | | |
| extras | | |
| func | | |
| lc_kwargs | | |
| lc_runnable | | |
| lc_serializable | | |
| metadata | | |
| name | | |
| responseFormat | | |
| returnDirect | | |
| schema | | |
| tags | | |
| verbose | | |
| verboseParsingErrors | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_namespace | | |
| lc_secrets | | |
| lc_serializable_keys | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `_addVersion()` | `_addVersion ( pkg : string , version : string )` | |
| `_batchWithConfig()` | `_batchWithConfig ( func : ( inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > > [ ] , runManagers : CallbackManagerForChainRun | undefined [ ] , batchOptions : RunnableBatchOptions ) = > Promise < Error | ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] > , inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > __type > | Partial < RunnableConfig < Record < string , any > > __type > [ ] , batchOptions : RunnableBatchOptions ) : Promise < Error | ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] >` | 内部方法，用于处理可运行对象的批处理和配置。它接收一个函数、输入值和可选配置，并返回解析到输出值的 Promise。 |
| `_call()` | `_call ( arg : SchemaOutputT , runManager : CallbackManagerForToolRun , parentConfig : RunnableConfig < Record < string , any > > ) : Promise < ToolOutputT > | AsyncGenerator < ToolEventT , ToolOutputT , any >` | |
| `_callWithConfig()` | `_callWithConfig < T > ( func : ( input : T ) = > Promise < ToolOutputT | ToolMessage View source on GitHub < MessageStructure < MessageToolSet > > > | ( input : T , config : Partial < RunnableConfig < Record < string , any > > > , runManager : CallbackManagerForChainRun ) = > Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > > , input : T , options : Partial < RunnableConfig < Record < string , any > > > __type ) : Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | |
| `_getOptionsList()` | `_getOptionsList < O extends RunnableConfig < Record < string , any > > __type > ( options : Partial < O > | Partial < O > [ ] , length : number ) : Partial < O > [ ]` | |
| `_separateRunnableConfigFromCallOptions()` | `_separateRunnableConfigFromCallOptions ( options : Partial < RunnableConfig < Record < string , any > > > ) : [ RunnableConfig < Record < string , any > > , Omit < Partial < RunnableConfig < Record < string , any > > > , keyof RunnableConfig < Record < string , any > > > ]` | |
| `_streamIterator()` | `_streamIterator ( input : StructuredToolCallInput , options : Partial < RunnableConfig < Record < string , any > > > ) : AsyncGenerator < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | 默认流式实现。子类如果支持流式输出，应覆盖此方法。 |
| `_streamLog()` | `_streamLog ( input : StructuredToolCallInput , logStreamCallbackHandler : LogStreamCallbackHandler , config : Partial < CallOptions > ) : AsyncGenerator < RunLogPatch >` | |
| `_transformStreamWithConfig()` | `_transformStreamWithConfig < I , O > ( inputGenerator : AsyncGenerator < I > , transformer : ( generator : AsyncGenerator < I > , runManager : CallbackManagerForChainRun , options : Partial < RunnableConfig < Record < string , any > > > ) = > AsyncGenerator < O > , options : Partial < RunnableConfig < Record < string , any > > > __type ) : AsyncGenerator < O >` | 辅助方法，用于将输入值的迭代器转换为带有回调的输出值的迭代器。用于在 Runnable 子类中实现 `stream()` 或 `transform()`。 |
| `assign()` | `assign ( mapping : RunnableMapLike < Record < string , unknown > , Record < string , unknown > > ) : Runnable` | 将新字段赋值到此可运行对象的字典输出中。返回一个新的可运行对象。 |
| `asTool()` | `asTool < T = StructuredToolCallInput < SchemaT , SchemaInputT > > ( fields : __type ) : RunnableToolLike < InteropZodType < ToolCall < string , Record < string , any > > | T > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | 将可运行对象转换为工具。返回一个新的 `RunnableToolLike` 实例，其中包含可运行对象、名称、描述和 Schema。 |
| `batch()` | `batch ( inputs : StructuredToolCallInput < SchemaT , SchemaInputT > [ ] , options : Partial < RunnableConfig < Record < string , any > > > | Partial < RunnableConfig < Record < string , any > > > [ ] , batchOptions : RunnableBatchOptions __type ) : Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] >` | `batch` 的默认实现，它调用 `invoke` N 次。子类如果可以更高效地批处理，应覆盖此方法。 |
| `call()` | `call < TArg , TConfig extends ToolRunnableConfig | undefined > ( arg : TArg , configArg : TConfig , tags : string [ ] ) : Promise < ToolReturnType < NonNullable < TArg > , TConfig , ToolOutputT > >` | ⚠ 已弃用 — 调用 |
| `getGraph()` | `getGraph ( _ : RunnableConfig < Record < string , any > > ) : Graph` | |
| `getName()` | `getName ( suffix : string ) : string` | |
| `invoke()` | `invoke < TInput , TConfig extends ToolRunnableConfig | undefined > ( input : TInput , config : TConfig ) : Promise < ToolReturnType < TInput , TConfig , ToolOutputT > >` | 使用提供的输入和配置调用工具。 |
| `pick()` | `pick ( keys : string | string [ ] ) : Runnable` | 从此可运行对象的字典输出中挑选键。返回一个新的可运行对象。 |
| `pipe()` | `pipe < NewRunOutput > ( coerceable : RunnableLike < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , NewRunOutput > ) : Runnable < StructuredToolCallInput < SchemaT , SchemaInputT > , Exclude < NewRunOutput , Error > >` | 创建一个新的可运行序列，按顺序运行每个独立的可运行对象，将一个可运行对象或类可运行对象的输出管道传输到另一个可运行对象或类可运行对象中。 |
| `stream()` | `stream ( input : StructuredToolCallInput , options : Partial < RunnableConfig < Record < string , any > > > ) : Promise < IterableReadableStream < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > > >` | 分块流式输出。 |
| `streamEvents()` | `streamEvents ( input : StructuredToolCallInput , options : Partial < RunnableConfig < Record < string , any > > > __type , streamOptions : Omit < EventStreamCallbackHandlerInput , "autoClose" > ) : IterableReadableStream < StreamEvent >` | 生成由可运行对象内部步骤产生的事件流。用于创建一个迭代器，提供有关可运行对象进度实时信息，包括来自中间结果的 `StreamEvents`。一个 `StreamEvent` 是一个具有以下 Schema 的字典： `event : string` - 事件名称的格式为：`on_[runnable_type]_(start|stream|end)`。 `name : string` - 生成事件的可运行对象的名称。 `run_id : string` - 与生成事件的给定执行相关联的随机生成的 ID。如果父可运行对象作为执行的一部分被调用，则分配其唯一的 ID。 `tags : string[]` - 生成事件的可运行对象的标签。 `metadata : Record<string, any>` - 生成事件的可运行对象的元数据。 `data : Record<string, any>` |
| `streamLog()` | `streamLog ( input : StructuredToolCallInput , logStreamCallbackHandler : LogStreamCallbackHandler , config : Partial < CallOptions > ) : AsyncGenerator < RunLogPatch >` | ⚠ 已弃用 — 流式传输所有从可运行对象报告到回调系统的输出，包括 LLM、检索器、工具等的全部内部运行。输出作为 `Log` 对象流式传输，其中包含描述每次步骤中运行状态如何变化的 `jsonpatch` 操作列表，以及运行的最终状态。 |
| `toJSON()` | `toJSON ( ) : Serialized` | |
| `toJSONNotImplemented()` | `toJSONNotImplemented ( ) : SerializedNotImplemented` | |
| `transform()` | `transform ( generator : AsyncGenerator < StructuredToolCallInput < SchemaT , SchemaInputT > > , options : Partial < CallOptions > ) : AsyncGenerator < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | `transform` 的默认实现，它缓冲输入然后调用 `stream`。子类如果可以在输入仍在生成时开始产生输出，应覆盖此方法。 |
| `withConfig()` | `withConfig ( config : Partial < CallOptions > ) : Runnable < StructuredToolCallInput < SchemaT , SchemaInputT > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 将配置绑定到可运行对象，返回一个新的可运行对象。 |
| `withFallbacks()` | `withFallbacks ( fields : __type | Runnable < StructuredToolCallInput < SchemaT , SchemaInputT > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > > [ ] ) : RunnableWithFallbacks < StructuredToolCallInput < SchemaT , SchemaInputT > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 从当前可运行对象创建一个新的可运行对象，如果初始调用失败，它将尝试调用传入的其他回退可运行对象。 |
| `withListeners()` | `withListeners ( params : __type ) : Runnable < StructuredToolCallInput < SchemaT , SchemaInputT > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 将生命周期监听器绑定到可运行对象，返回一个新的可运行对象。`Run` 对象包含有关运行的信息，包括其 ID、类型、输入、输出、错误、开始时间、结束时间以及添加到运行中的任何标签或元数据。 |
| `withRetry()` | `withRetry ( fields : __type ) : RunnableRetry < StructuredToolCallInput < SchemaT , SchemaInputT > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 为现有可运行对象添加重试逻辑。 |
| `isRunnable()` | `isRunnable ( thing : any ) : thing is Runnable < any , any , RunnableConfig < Record < string , any > > >` | |
| `lc_name()` | `lc_name ( ) : string` | 可序列化的名称。覆盖以提供别名或在最小化环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/...
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖工具相关的 API，包括动态工具、执行后的工具调用信息以及提供者策略。

## 类
### `DynamicTool`
<a id="DynamicTool"></a>

导入：
```ts
import { DynamicTool } from "langchain";
```

签名：
```ts
class DynamicTool
```

一个可以从函数、名称和描述动态创建的工具。

属性：
| 属性 | 类型 | 说明 |
|------|------|-------------|
| `callbacks` | | |
| `defaultConfig` | | |
| `description` | | |
| `extras` | | |
| `func` | | |
| `lc_kwargs` | | |
| `lc_runnable` | | |
| `lc_serializable` | | |
| `metadata` | | |
| `name` | | |
| `responseFormat` | | |
| `returnDirect` | | |
| `schema` | | |
| `tags` | | |
| `verbose` | | |
| `verboseParsingErrors` | | |
| `lc_aliases` | | |
| `lc_attributes` | | |
| `lc_id` | | |
| `lc_namespace` | | |
| `lc_secrets` | | |
| `lc_serializable_keys` | | |

方法：
| 方法 | 签名 | 说明 |
|------|------|-------------|
| `_addVersion` | `_addVersion ( pkg : string , version : string )` | |
| `_batchWithConfig` | `_batchWithConfig < T extends any > ( func : ( inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > > [ ] , runManagers : CallbackManagerForChainRun | undefined [ ] , batchOptions : RunnableBatchOptions ) = > Promise < Error | ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] > , inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > __type > | Partial < RunnableConfig < Record < string , any > > __type > [ ] , batchOptions : RunnableBatchOptions ) : Promise < Error | ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] >` | 内部方法，用于处理可运行对象的批处理和配置。它接收一个函数、输入值和可选配置，并返回一个解析到输出值的 Promise。|
| `_callWithConfig` | `_callWithConfig < T extends any > ( func : ( input : T ) = > Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > > | ( input : T , config : Partial < RunnableConfig < Record < string , any > > > , runManager : CallbackManagerForChainRun ) = > Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > > , input : T , options : Partial < RunnableConfig < Record < string , any > > > __type ) : Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | |
| `_getOptionsList` | `_getOptionsList < O extends RunnableConfig < Record < string , any > > __type > ( options : Partial < O > | Partial < O > [ ] , length : number ) : Partial < O > [ ]` | |
| `_separateRunnableConfigFromCallOptions` | `_separateRunnableConfigFromCallOptions ( options : Partial < RunnableConfig < Record < string , any > > > ) : [ RunnableConfig < Record < string , any > > , Omit < Partial < RunnableConfig < Record < string , any > > > , keyof RunnableConfig < Record < string , any > > > ]` | |
| `_streamIterator` | `_streamIterator ( input : any , options : Partial < RunnableConfig < Record < string , any > > > ) : AsyncGenerator < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | 默认流式实现。子类应在支持流式输出时重写此方法。|
| `_streamLog` | `_streamLog ( input : any , logStreamCallbackHandler : LogStreamCallbackHandler , config : Partial < CallOptions > ) : AsyncGenerator < RunLogPatch >` | |
| `_transformStreamWithConfig` | `_transformStreamWithConfig < I extends any , O > ( inputGenerator : AsyncGenerator < I > , transformer : ( generator : AsyncGenerator < I > , runManager : CallbackManagerForChainRun , options : Partial < RunnableConfig < Record < string , any > > > ) = > AsyncGenerator < O > , options : Partial < RunnableConfig < Record < string , any > > > __type ) : AsyncGenerator < O >` | 转换输入值迭代器的辅助方法，用于将输出值迭代器转换为带回调函数的迭代器。用于在 Runnable 子类中实现 `stream()` 或 `transform()`。|
| `assign` | `assign ( mapping : RunnableMapLike < Record < string , unknown > , Record < string , unknown > > ) : Runnable` | 将新字段分配到此可运行对象的 dict 输出中。返回一个新的可运行对象。|
| `asTool` | `asTool < T extends any = any > ( fields : __type ) : RunnableToolLike < InteropZodType < ToolCall < string , Record < string , any > > | T > , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | 将可运行对象转换为工具。返回包含可运行对象、名称、描述和模式的新 RunnableToolLike 实例。|
| `batch` | `batch ( inputs : any [ ] , options : Partial < RunnableConfig < Record < string , any > > > | Partial < RunnableConfig < Record < string , any > > > [ ] , batchOptions : RunnableBatchOptions __type ) : Promise < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > [ ] >` | `batch` 的默认实现，它调用 N 次 `invoke`。子类如果可以更高效地进行批处理，应重写此方法。|
| `call` | `call < TArg extends string | ToolCall < string , Record < string , any > > | __type | undefined , TConfig extends ToolRunnableConfig | undefined > ( arg : TArg , configArg : TConfig ) : Promise < ToolReturnType < NonNullable < TArg > , TConfig , ToolOutputT > >` | ⚠ 已弃用 — `call`|
| `getGraph` | `getGraph ( _ : RunnableConfig < Record < string , any > > ) : Graph` | |
| `getName` | `getName ( suffix : string ) : string` | |
| `invoke` | `invoke < TInput extends any , TConfig extends ToolRunnableConfig | undefined > ( input : TInput , config : TConfig ) : Promise < ToolReturnType < TInput , TConfig , ToolOutputT > >` | 使用提供的输入和配置调用工具。|
| `pick` | `pick ( keys : string | string [ ] ) : Runnable` | 从此可运行对象的 dict 输出中挑选键。返回一个新的可运行对象。|
| `pipe` | `pipe < NewRunOutput > ( coerceable : RunnableLike < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , NewRunOutput > ) : Runnable < any , Exclude < NewRunOutput , Error > >` | 创建一个新的可运行序列，按顺序运行每个单独的可运行对象，并将一个可运行对象或类可运行对象的输出管道传输到另一个可运行对象或类可运行对象中。|
| `stream` | `stream ( input : any , options : Partial < RunnableConfig < Record < string , any > > > ) : Promise < IterableReadableStream < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > > >` | 分块流式传输输出。|
| `streamEvents` | `streamEvents ( input : any , options : Partial < RunnableConfig < Record < string , any > > > __type , streamOptions : Omit < EventStreamCallbackHandlerInput , "autoClose" > ) : IterableReadableStream < StreamEvent >` | 生成由可运行对象的内部步骤发出的事件流。用于创建一个迭代器，以提供关于可运行对象进度的实时信息，包括来自中间结果的 StreamEvents。|
| `streamLog` | `streamLog ( input : any , options : Partial < RunnableConfig < Record < string , any > > > , streamOptions : Omit < LogStreamCallbackHandlerInput , "autoClose" > ) : AsyncGenerator < RunLogPatch >` | ⚠ 已弃用 — `streamLog` 报告来自可运行对象的全部输出，如由回调系统报告的。这包括 LLM、检索器、工具等的全部内部运行。输出作为 Log 对象流式传输，其中包括描述运行状态在每一步如何变化的 jsonpatch 操作列表，以及运行的最终状态。|
| `toJSON` | `toJSON ( ) : Serialized` | |
| `toJSONNotImplemented` | `toJSONNotImplemented ( ) : SerializedNotImplemented` | |
| `transform` | `transform ( generator : AsyncGenerator < any > , options : Partial < CallOptions > ) : AsyncGenerator < ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > >` | `transform` 的默认实现，它缓冲输入然后调用 `stream`。子类如果可以在输入仍在生成时开始产生输出，应重写此方法。|
| `withConfig` | `withConfig ( config : Partial < CallOptions > ) : Runnable < any , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 将配置绑定到可运行对象，返回一个新的可运行对象。|
| `withFallbacks` | `withFallbacks ( fields : __type | Runnable < any , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > > [ ] ) : RunnableWithFallbacks < any , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 创建一个新的可运行对象，如果初始调用失败，它将尝试调用传递的其他备用可运行对象。|
| `withListeners` | `withListeners ( params : __type ) : Runnable < any , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 将生命周期监听器绑定到可运行对象，返回一个新的可运行对象。|
| `withRetry` | `withRetry ( fields : __type ) : RunnableRetry < any , ToolOutputT | ToolMessage < MessageStructure < MessageToolSet > > , RunnableConfig < Record < string , any > > >` | 为现有的可运行对象添加重试逻辑。|
| `isRunnable` | `isRunnable ( thing : any ) : thing is Runnable < any , any , RunnableConfig < Record < string , any > > >` | |
| `lc_name` | `lc_name ( ) : string` | 可序列化对象的名称。可重写以提供别名或在压缩环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。|

## 接口
### `ExecutedToolCall`
<a id="ExecutedToolCall"></a>

导入：
```ts
import { ExecutedToolCall } from "langchain";
```

已执行的工具调用的信息。

属性：
| 属性 | 类型 | 说明 |
|------|------|-------------|
| `args` | `Record < string , unknown >` | 传递给工具的参数。|
| `name` | `string` | 被调用的工具名称。|
| `result` | `unknown` | 工具调用的结果（如果可用）。|
| `tool_id` | `string` | 工具调用的 ID。|

## 函数
### `providerStrategy`
<a id="providerStrategy"></a>

导入：
```ts
import { providerStrategy } from "langchain";
```

为结构化输出创建提供者策略，使用原生 JSON Schema 支持。

此函数用于配置智能体在底层模型支持原生 JSON Schema 输出时的结构化输出（例如 OpenAI 的 `gpt-4o`、`gpt-4o-mini` 以及更新的模型）。

与使用函数调用来提取结构化输出的 `toolStrategy` 不同，`providerStrategy` 利用提供者的原生结构化输出能力，从而实现更高效且更可靠的架构强制执行。

当与支持 JSON Schema 输出的模型一起使用时，模型将直接返回符合提供的架构的响应，而无需调用工具。这是当您的模型支持时，用于结构化输出的推荐方法。

签名：
```ts
providerStrategy<T extends InteropZodType<unknown>>(responseFormat: T | __type): ProviderStrategy<T extends InteropZodType<U> ? U : never>
```

参数：
| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `responseFormat` | `T \| __type` | Yes | 要强制执行的架构，可以是 Zod 架构、标准架构（例如 Valibot、ArkType、TypeBox）、JSON 架构对象或包含 `schema` 和可选 `strict` 标志的选项对象 |

返回：
`ProviderStrategy<T extends InteropZodType<U> ? U : never>`

一个可以作为 `createAgent` 中的 `responseFormat` 使用的 `ProviderStrategy` 实例。

示例：
```ts
import { providerStrategy, createAgent } from "langchain";
import { z } from "zod";

const agent = createAgent({
  model: "claude-haiku-4-5",
  responseFormat: providerStrategy(
    z.object({
      answer: z.string().describe("The answer to the question"),
      confidence: z.number().min(0).max(1),
    })
  ),
});
```

```ts
// Using strict mode for stricter schema enforcement
const agent = createAgent({
  model: "claude-haiku-4-5",
  responseFormat: providerStrategy({
    schema: z.object({
      name: z.string(),
      age: z.number(),
    }),
    strict: true
  }),
});
```

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/providerStrategy
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 函数
### `providerStrategy`
<a id="providerStrategy"></a>
import { providerStrategy } from "langchain";

```javascript
providerStrategy<T extends InteropZodType<unknown>>(responseFormat: T | __type): ProviderStrategy<T extends InteropZodType<U> ? U : never>
```

使用原生 JSON schema 支持为结构化输出创建提供商策略。

此函数用于配置支持原生 JSON schema 输出的模型（例如 OpenAI 的 `gpt-4o`、`gpt-4o-mini` 以及更新的模型）的代理结构化输出。
与使用函数调用提取结构化输出的 `toolStrategy` 不同，`providerStrategy` 利用提供商的原生结构化输出能力，从而实现更高效、更可靠的架构强制执行。

当与支持 JSON schema 输出的模型配合使用时，模型将直接返回符合提供架构的响应，而无需调用工具。这是当模型支持时用于结构化输出的推荐方法。

| 名称 | 类型 | 必填 | 说明 |
|------|------|----------|-------------|
| `responseFormat` | `T \| __type` | 是 | 强制执行的架构，可以是 Zod 架构、标准架构（例如 Valibot、ArkType、TypeBox）、JSON 架构对象，或者包含 `schema` 属性和可选 `strict` 标志的选项对象 |

`ProviderStrategy<T extends InteropZodType<U> ? U : never>`

可用于 `createAgent` 中作为 `responseFormat` 的 `ProviderStrategy` 实例。

### 示例 1
```ts
import { providerStrategy, createAgent } from "langchain";
import { z } from "zod";

const agent = createAgent({
  model: "claude-haiku-4-5",
  responseFormat: providerStrategy(
    z.object({
      answer: z.string().describe("The answer to the question"),
      confidence: z.number().min(0).max(1),
    })
  ),
});
```

### 示例 2
```ts
// Using strict mode for stricter schema enforcement
const agent = createAgent({
  model: "claude-haiku-4-5",
  responseFormat: providerStrategy({
    schema: z.object({
      name: z.string(),
      age: z.number(),
    }),
    strict: true
  }),
});
```

## 类型
### `ResponseFormat`
<a id="ResponseFormat"></a>
import { ResponseFormat } from "langchain";

`ToolStrategy < any > | ProviderStrategy < any >`

## 类型
### `ResponseFormatUndefined`
<a id="ResponseFormatUndefined"></a>
import { ResponseFormatUndefined } from "langchain";

表示未提供响应格式的特殊类型。

当使用此类型时，结果中不应包含 `structuredResponse` 属性。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `__responseFormatUndefined` | `boolean` | `true` |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/StructuredTool
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 类

### `StructuredTool`
<a id="StructuredTool"></a>
```typescript
import { StructuredTool } from "langchain";
```

接受 Zod 模式定义的任意形状输入的工具基类。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| callbacks | | |
| defaultConfig | | |
| description | | |
| extras | | |
| lc_kwargs | | |
| lc_runnable | | |
| lc_serializable | | |
| metadata | | |
| name | | |
| responseFormat | | |
| returnDirect | | |
| schema | | |
| tags | | |
| verbose | | |
| verboseParsingErrors | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_namespace | | |
| lc_secrets | | |
| lc_serializable_keys | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _addVersion | `_addVersion(pkg: string, version: string)` | |
| _batchWithConfig | `_batchWithConfig(func: (inputs: T[], options?: Partial<RunnableConfig<Record<string, any>>>[], runManagers?: CallbackManagerForChainRun | undefined[], batchOptions: RunnableBatchOptions) => Promise<Error | ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT[], inputs: T[], options?: Partial<RunnableConfig<Record<string, any>>> & __type | Partial<RunnableConfig<Record<string, any>>> & __type[], batchOptions: RunnableBatchOptions): Promise<Error | ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT[]>` | 处理可运行对象的批处理和配置。它接受一个函数、输入值和可选配置，并返回一个解析到输出值的 promise。 |
| _call | `_call(arg: SchemaOutputT, runManager: CallbackManagerForToolRun, parentConfig: ToolRunnableConfig): Promise<ToolOutputT> | AsyncGenerator<ToolEventT, ToolOutputT, any>` | |
| _callWithConfig | `_callWithConfig(func: (input: T) => Promise<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT | (input: T, config?: Partial<RunnableConfig<Record<string, any>>>>, runManager?: CallbackManagerForChainRun) => Promise<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, input: T, options?: Partial<RunnableConfig<Record<string, any>>> & __type): Promise<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | |
| _getOptionsList | `_getOptionsList(options: Partial<O> | Partial<O>[], length: number): Partial<O>[]` | |
| _separateRunnableConfigFromCallOptions | `_separateRunnableConfigFromCallOptions(options: Partial<RunnableConfig<Record<string, any>>>): [RunnableConfig<Record<string, any>>, Omit<Partial<RunnableConfig<Record<string, any>>>, keyof RunnableConfig<Record<string, any>>>]` | |
| _streamIterator | `_streamIterator(input: StructuredToolCallInput, options: Partial<RunnableConfig<Record<string, any>>>): AsyncGenerator<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | 默认流式实现。子类如果支持流式输出应覆盖此方法。 |
| _streamLog | `_streamLog(input: StructuredToolCallInput, logStreamCallbackHandler: LogStreamCallbackHandler, config: Partial<CallOptions>): AsyncGenerator<RunLogPatch>` | |
| _transformStreamWithConfig | `_transformStreamWithConfig(inputGenerator: AsyncGenerator<I>, transformer: (generator: AsyncGenerator<I>, runManager: CallbackManagerForChainRun, options: Partial<RunnableConfig<Record<string, any>>>) => AsyncGenerator<O>, options: Partial<RunnableConfig<Record<string, any>>> & __type): AsyncGenerator<O>` | 辅助方法将输入值迭代器转换为输出值迭代器，带回调。用于在 Runnable 子类中实现 stream() 或 transform()。 |
| assign | `assign(mapping: RunnableMapLike<Record<string, unknown>, Record<string, unknown>>): Runnable` | 将新字段分配到此可运行对象的 dict 输出中。返回一个新的可运行对象。 |
| asTool | `asTool(fields: __type): RunnableToolLike<InteropZodType<ToolCall<string, Record<string, any>>> | T, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | 将可运行对象转换为工具。返回包含可运行对象、名称、描述和模式的新 RunnableToolLike 实例。 |
| batch | `batch(inputs: StructuredToolCallInput<SchemaT, SchemaInputT>[], options?: Partial<RunnableConfig<Record<string, any>>> | Partial<RunnableConfig<Record<string, any>>>[], batchOptions: RunnableBatchOptions): Promise<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT[]>` | 批处理的默认实现，通过 N 次调用 invoke 实现。子类如果可以更高效地进行批处理，应覆盖此方法。 |
| call | `call(arg: TArg, configArg: TConfig, tags: string[]): Promise<ToolReturnType<TArg, TConfig, ToolOutputT>>` | ⚠ 已弃用 — call |
| getGraph | `getGraph(_: RunnableConfig<Record<string, any>>): Graph` | |
| getName | `getName(suffix: string): string` | |
| invoke | `invoke(input: TInput, config: TConfig): Promise<ToolReturnType<TInput, TConfig, ToolOutputT>>` | 使用提供的输入和配置调用工具。 |
| pick | `pick(keys: string | string[]): Runnable` | 从此可运行对象的 dict 输出中提取键。返回一个新的可运行对象。 |
| pipe | `pipe(coerceable: RunnableLike<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, NewRunOutput): Runnable<StructuredToolCallInput<SchemaT, SchemaInputT>, Exclude<NewRunOutput, Error>>` | 创建一个新的可运行序列，按顺序运行每个单独的可运行对象，并将一个可运行对象或类可运行对象的输出管道传给另一个。 |
| stream | `stream(input: StructuredToolCallInput, options?: Partial<RunnableConfig<Record<string, any>>>): Promise<IterableReadableStream<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | 以块形式流式输出。 |
| streamEvents | `streamEvents(input: StructuredToolCallInput, options?: Partial<RunnableConfig<Record<string, any>>> & __type, streamOptions?: Omit<EventStreamCallbackHandlerInput, "autoClose">): IterableReadableStream<StreamEvent>` | 生成由可运行对象内部步骤产生的事件流。用于创建一个迭代器，提供关于可运行对象 tiến độ 的实时信息，包括来自中间结果的 StreamEvents。 |
| streamLog | `streamLog(input: StructuredToolCallInput, options?: Partial<RunnableConfig<Record<string, any>>>, streamOptions?: Omit<LogStreamCallbackHandlerInput, "autoClose">): AsyncGenerator<RunLogPatch>` | ⚠ 已弃用 — streamLog 从可运行对象流式传输所有输出，如回调系统报告的。这包括 LLM、检索器、工具等的全部内部运行。输出作为 Log 对象流式传输，其中包含描述运行状态在每一步如何变化的 jsonpatch 操作列表，以及运行的最终状态。 |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| transform | `transform(generator: AsyncGenerator<StructuredToolCallInput<SchemaT, SchemaInputT>>, options?: Partial<CallOptions>): AsyncGenerator<ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | transform 的默认实现，先缓冲输入然后调用 stream。子类如果可以在输入仍在生成时开始产生输出，应覆盖此方法。 |
| withConfig | `withConfig(config: Partial<CallOptions>): Runnable<StructuredToolCallInput<SchemaT, SchemaInputT>, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, RunnableConfig<Record<string, any>>>` | 将配置绑定到可运行对象，返回一个新的可运行对象。 |
| withFallbacks | `withFallbacks(fields: __type | Runnable<StructuredToolCallInput<SchemaT, SchemaInputT>, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, RunnableConfig<Record<string, any>>>[]): RunnableWithFallbacks<StructuredToolCallInput<SchemaT, SchemaInputT>, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT>` | 从当前可运行对象创建一个新的可运行对象，如果初始调用失败，它将尝试调用传递的备选可运行对象。 |
| withListeners | `withListeners(params: __type): Runnable<StructuredToolCallInput<SchemaT, SchemaInputT>, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, RunnableConfig<Record<string, any>>>` | 将生命周期监听器绑定到可运行对象，返回一个新的可运行对象。Run 对象包含关于运行的信息，包括其 id、类型、输入、输出、错误、开始时间、结束时间，以及在运行中添加的任何标签或元数据。 |
| withRetry | `withRetry(fields: __type): RunnableRetry<StructuredToolCallInput<SchemaT, SchemaInputT>, ToolMessage<MessageStructure<MessageToolSet>>> | ToolOutputT, RunnableConfig<Record<string, any>>>` | 为现有可运行对象添加重试逻辑。 |
| isRunnable | `isRunnable(thing: any): thing is Runnable<any, any, RunnableConfig<Record<string, any>>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。可覆盖以提供别名或在精简环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |
```

## 类
### `Tool`
<a id="Tool"></a>
import { Tool } from "langchain";

```javascript
class Tool
```

接受字符串作为输入的工具基类。

#### 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `callbacks` | | |
| `defaultConfig` | | |
| `description` | | |
| `extras` | | |
| `lc_kwargs` | | |
| `lc_runnable` | | |
| `lc_serializable` | | |
| `metadata` | | |
| `name` | | |
| `responseFormat` | | |
| `returnDirect` | | |
| `schema` | | |
| `tags` | | |
| `verbose` | | |
| `verboseParsingErrors` | | |
| `lc_aliases` | | |
| `lc_attributes` | | |
| `lc_id` | | |
| `lc_namespace` | | |
| `lc_secrets` | | |
| `lc_serializable_keys` | | |

#### 方法
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `_addVersion` | `_addVersion ( pkg : string , version : string )` | |
| `_batchWithConfig` | `_batchWithConfig < T extends any > ( func : ( inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > > [ ] , runManagers : CallbackManagerForChainRun | undefined [ ] , batchOptions : RunnableBatchOptions ) = > Promise < Error | ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT [ ] > , inputs : T [ ] , options : Partial < RunnableConfig < Record < string , any > > __type > | Partial < RunnableConfig < Record < string , any > > __type > [ ] , batchOptions : RunnableBatchOptions ) : Promise < Error | ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT [ ] >` | 内部方法，用于处理可运行对象的批处理和配置。它接收一个函数、输入值和可选配置，并返回一个解析到输出值的 Promise。 |
| `_call` | `_call ( arg : string | undefined , runManager : CallbackManagerForToolRun , parentConfig : ToolRunnableConfig ) : Promise < ToolOutputT > | AsyncGenerator < ToolEventT , ToolOutputT , any >` | |
| `_callWithConfig` | `_callWithConfig < T extends any > ( func : ( input : T ) = > Promise < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT > | ( input : T , config : Partial < RunnableConfig < Record < string , any > > > , runManager : CallbackManagerForChainRun ) = > Promise < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT > , input : T , options : Partial < RunnableConfig < Record < string , any > > > __type ) : Promise < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT >` | |
| `_getOptionsList` | `_getOptionsList < O extends RunnableConfig < Record < string , any > > __type > ( options : Partial < O > | Partial < O > [ ] , length : number ) : Partial < O > [ ]` | |
| `_separateRunnableConfigFromCallOptions` | `_separateRunnableConfigFromCallOptions ( options : Partial < RunnableConfig < Record < string , any > > > ) : [ RunnableConfig < Record < string , any > > , Omit < Partial < RunnableConfig < Record < string , any > > > , keyof RunnableConfig < Record < string , any > > > ]` | |
| `_streamIterator` | `_streamIterator ( input : any , options : Partial < RunnableConfig < Record < string , any > > > ) : AsyncGenerator < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT >` | 默认流式实现。子类如果支持流式输出，应覆盖此方法。 |
| `_streamLog` | `_streamLog ( input : any , logStreamCallbackHandler : LogStreamCallbackHandler , config : Partial < CallOptions > ) : AsyncGenerator < RunLogPatch >` | |
| `_transformStreamWithConfig` | `_transformStreamWithConfig < I extends any , O > ( inputGenerator : AsyncGenerator < I > , transformer : ( generator : AsyncGenerator < I > , runManager : CallbackManagerForChainRun , options : Partial < RunnableConfig < Record < string , any > > > ) = > AsyncGenerator < O > , options : Partial < RunnableConfig < Record < string , any > > > __type ) : AsyncGenerator < O >` | 辅助方法用于将输入值的迭代器转换为输出值的迭代器，带有回调。用于在 Runnable 子类中实现 `stream()` 或 `transform()`。 |
| `assign` | `assign ( mapping : RunnableMapLike < Record < string , unknown > , Record < string , unknown > > ) : Runnable` | 将新字段分配到此可运行对象的字典输出中。返回一个新的可运行对象。 |
| `asTool` | `asTool < T extends any = any > ( fields : __type ) : RunnableToolLike < InteropZodType < ToolCall < string , Record < string , any > > | T > , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT >` | 将可运行对象转换为工具。返回包含可运行对象、名称、描述和模式的新 RunnableToolLike 实例。 |
| `batch` | `batch ( inputs : any [ ] , options : Partial < RunnableConfig < Record < string , any > > > | Partial < RunnableConfig < Record < string , any > > > [ ] , batchOptions : RunnableBatchOptions & __type ) : Promise < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT [ ] >` | 批处理的默认实现，它调用 N 次 `invoke`。子类如果能更高效地进行批处理，应覆盖此方法。 |
| `call` | `call < TArg extends string | ToolCall < string , Record < string , any > > | __type | undefined , TConfig extends ToolRunnableConfig | undefined > ( arg : TArg , callbacks : TConfig ) : Promise < ToolReturnType < NonNullable < TArg > , TConfig , ToolOutputT > >` | ⚠️ 已弃用 — 调用工具。 |
| `getGraph` | `getGraph ( _ : RunnableConfig < Record < string , any > > ) : Graph` | |
| `getName` | `getName ( suffix : string ) : string` | |
| `invoke` | `invoke < TInput extends any , TConfig extends ToolRunnableConfig | undefined > ( input : TInput , config : TConfig ) : Promise < ToolReturnType < TInput , TConfig , ToolOutputT > >` | 使用提供的输入和配置调用工具。 |
| `pick` | `pick ( keys : string | string [ ] ) : Runnable` | 从此可运行对象的字典输出中提取键。返回一个新的可运行对象。 |
| `pipe` | `pipe < NewRunOutput > ( coerceable : RunnableLike < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT , NewRunOutput > ) : Runnable < any , Exclude < NewRunOutput , Error > >` | 创建一个新的可运行对象序列，按顺序运行每个单独的可运行对象，将一个可运行对象或类似对象的输出管道传递到另一个可运行对象或类似对象。 |
| `stream` | `stream ( input : any , options : Partial < RunnableConfig < Record < string , any > > > ) : Promise < IterableReadableStream < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT > >` | 分块流式输出。 |
| `streamEvents` | `streamEvents ( input : any , options : Partial < RunnableConfig < Record < string , any > > > __type , streamOptions : Omit < EventStreamCallbackHandlerInput , "autoClose" > ) : IterableReadableStream < StreamEvent >` | 生成由可运行对象内部步骤发出的事件流。用于创建一个迭代器，提供关于可运行对象进度的实时信息，包括来自中间结果的 StreamEvents。 |
| `streamLog` | `streamLog ( input : any , options : Partial < RunnableConfig < Record < string , any > > > , streamOptions : Omit < LogStreamCallbackHandlerInput , "autoClose" > ) : AsyncGenerator < RunLogPatch >` | ⚠️ 已弃用 — 流式传输由回调系统报告的可运行对象的所有输出。这包括 LLM、检索器、工具等的内部所有运行。输出作为包含描述运行状态如何变化的 JSONPatch 操作列表的 Log 对象流式传输。 |
| `toJSON` | `toJSON ( ) : Serialized` | |
| `toJSONNotImplemented` | `toJSONNotImplemented ( ) : SerializedNotImplemented` | |
| `transform` | `transform ( generator : AsyncGenerator < any > , options : Partial < CallOptions > ) : AsyncGenerator < ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT >` | `transform` 的默认实现，它缓冲输入然后调用 `stream`。子类如果能在输入仍在生成时就开始产生输出，应覆盖此方法。 |
| `withConfig` | `withConfig ( config : Partial < CallOptions > ) : Runnable < any , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT , RunnableConfig < Record < string , any > > >` | 将配置绑定到可运行对象，返回一个新的可运行对象。 |
| `withFallbacks` | `withFallbacks ( fields : __type | Runnable < any , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT , RunnableConfig < Record < string , any > > > [ ] ) : RunnableWithFallbacks < any , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT >` | 从当前可运行对象创建一个新的可运行对象，如果初始调用失败，它将尝试调用传递的其他回退可运行对象。 |
| `withListeners` | `withListeners ( params : __type ) : Runnable < any , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT , RunnableConfig < Record < string , any > > >` | 将生命周期监听器绑定到可运行对象，返回一个新的可运行对象。 |
| `withRetry` | `withRetry ( fields : __type ) : RunnableRetry < any , ToolMessage < MessageStructure < MessageToolSet > > | ToolOutputT , RunnableConfig < Record < string , any > > >` | 为现有的可运行对象添加重试逻辑。 |
| `isRunnable` | `isRunnable ( thing : any ) : thing is Runnable < any , any , RunnableConfig < Record < string , any > > >` | |
| `lc_name` | `lc_name ( ) : string` | 可序列化对象的名称。可覆盖以提供别名或在压缩环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

## 接口
### `ToolCall`
<a id="ToolCall"></a>
import { ToolCall } from "langchain";

ToolCall 包含已执行的工具调用信息。

#### 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `args` | `Record < string , any >` | 传递给工具的参数。 |
| `error` | `string` | 如果工具调用失败时的可选错误消息。 |
| `id` | `string` | 工具调用的 ID。 |
| `name` | `string` | 被调用的工具的名称。 |
| `result` | `unknown` | 工具调用的结果。 |

## 接口
### `ToolCallRequest`
<a id="ToolCallRequest"></a>
import { ToolCallRequest } from "langchain";

ToolCallRequest 代表 wrapToolCall 钩子的工具调用请求。包含工具调用信息以及代理当前状态和运行时。

#### 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `runtime` | `Runtime < TContext >` | 包含元数据、信号、写入器、中断等的运行时上下文。 |
| `state` | `TState & AgentBuiltInState` | 当前代理状态（包括中间件状态和内置状态）。 |
| `tool` | `ClientTool | ServerTool | undefined` | 被调用的 BaseTool 实例。提供对工具元数据（如名称、描述、模式等）的访问。对于动态注册且在创建代理时未预先声明的工具，此项将为 `undefined`。在这种情况下，中间件应通过将请求与工具实现合并来提供工具属性。 |
| `toolCall` | `ToolCall` | 要执行的工具调用。 |

## 接口
### `ToolResult`
<a id="ToolResult"></a>
import { ToolResult } from "langchain";

ToolResult 包含工具执行的结果信息。

#### 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `error` | `string` | 如果工具调用失败时的可选错误消息。 |
| `id` | `string` | 工具调用的 ID。 |
| `result` | `any` | 工具调用的结果。 |

## 函数
### `toolStrategy`
<a id="toolStrategy"></a>
```typescript
import { toolStrategy, createAgent } from "langchain";
import { z } from "zod";
```
```javascript
toolStrategy<T extends InteropZodType<any>>(responseFormat: T, options?: ToolStrategyOptions): TypedToolStrategy<T extends InteropZodType<U> ? U : never>
```
使用函数调用创建结构化输出的工具策略。

此函数通过将架构转换为模型调用的函数工具来配置结构化输出。与使用原生 JSON 架构支持的 `providerStrategy` 不同，`toolStrategy` 支持任何支持函数调用的模型，使其在不同提供商和模型版本之间具有更广泛的兼容性。

模型将调用一个具有与您的架构匹配参数的函数，代理将从工具调用中提取并验证结构化的输出。当您的模型不支持原生 JSON 架构输出时，此方法会自动使用。

## 类
### `ToolStrategy`
<a id="ToolStrategy"></a>
用于跟踪结构化输出工具元数据的工具策略。

包含处理通过工具调用生成的结构化响应的所有必要信息，包括原始架构、其类型分类以及工具策略中使用的相应工具实现。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `options` | `ToolStrategyOptions` | 用于工具输出的选项 |
| `schema` | `Record < string , unknown >` | 用于结构化输出的原始 JSON Schema |
| `tool` | `__ type` | 将用于解析工具调用参数的工具 |
| `name` | `string` | |

| 方法 | 签名 | 说明 |
|------|------|-------------|
| `parse` | `Record < string , unknown >` | 根据架构解析工具参数 |
| `fromSchema` | `ToolStrategy < S extends InteropZodType < U > ? U : unknown >` | |

## 接口
### `TypedToolStrategy`
<a id="TypedToolStrategy"></a>
```javascript
interface TypedToolStrategy
```
保留类型信息的 `ToolStrategy` 数组的品牌类型。

| 属性 | 类型 | 说明 |
|------|------|-------------|
| `_schemaType` | | |
| `[unscopables]` | | |
| `length` | | |

## 类型
### `ToolRuntime`
<a id="ToolRuntime"></a>
```typescript
import { tool, type ToolRuntime } from "@langchain/core/tools";
```
```javascript
ToolRuntime: RunnableConfig & __type
```
Runtime 上下文会自动注入到工具中。

当工具函数包含名为 `tool_runtime` 且类型提示为 `ToolRuntime` 的参数时，工具执行系统将自动注入包含以下内容的实例：

- `state`: 当前图状态
- `toolCallId`: 当前工具调用的 ID
- `config`: 当前执行的 `RunnableConfig`
- `context`: Runtime 上下文
- `store`: 用于持久化存储的 `BaseStore` 实例
- `writer`: 用于流式输出的 Stream writer

不需要 `Annotated` 包装器 - 只需将 `runtime: ToolRuntime` 作为参数使用。

### `HeadlessTool`
<a id="HeadlessTool"></a>
```javascript
HeadlessTool: DynamicStructuredTool < SchemaT , InferInteropZodOutput < SchemaT > , InferInteropZodInput < SchemaT > , unknown , unknown , NameT > __type
```
在服务器上始终中断代理执行的无头工具。

实现通过客户端提供的 `useStream({ tools: [...] })` 配合 `.implement()` 分别提供。

### `HeadlessToolFields`
<a id="HeadlessToolFields"></a>
```javascript
HeadlessToolFields: DynamicStructuredTool < SchemaT , InferInteropZodOutput < SchemaT > , InferInteropZodInput < SchemaT > , unknown , unknown , NameT > __type
```
创建无头工具的配置字段。

### `HeadlessToolImplementation`
<a id="HeadlessToolImplementation"></a>
```javascript
HeadlessToolImplementation: DynamicStructuredTool < SchemaT , InferInteropZodOutput < SchemaT > , InferInteropZodInput < SchemaT > , unknown , unknown , NameT > __type
```
将无头工具与其执行函数配对的工具实现。

通过对 `HeadlessTool` 调用 `.implement()` 创建。在客户端侧传递给 `useStream({ tools: [...] })`。