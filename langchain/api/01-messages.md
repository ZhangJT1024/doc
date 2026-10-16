# LangChain JS API 参考 · 消息（Messages）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

## 目录

- [AIMessage](#AIMessage)
- [AIMessageChunk](#AIMessageChunk)
- [BaseMessage](#BaseMessage)
- [BaseMessageChunk](#BaseMessageChunk)
- [ContentBlock](#ContentBlock)
- [Citation](#Citation)
- [Base64ContentBlock](#Base64ContentBlock)
- [BaseDataContentBlock](#BaseDataContentBlock)
- [IDContentBlock](#IDContentBlock)
- [PlainTextContentBlock](#PlainTextContentBlock)
- [URLContentBlock](#URLContentBlock)
- [NonStandard](#NonStandard)
- [Reasoning](#Reasoning)
- [Text](#Text)
- [InvalidToolCall](#InvalidToolCall)
- [ServerToolCall](#ServerToolCall)
- [ServerToolCallChunk](#ServerToolCallChunk)
- [ServerToolCallResult](#ServerToolCallResult)
- [ToolCall](#ToolCall)
- [ToolCallChunk](#ToolCallChunk)
- [Document](#Document)
- [DocumentInput](#DocumentInput)
- [filterMessages](#filterMessages)
- [HumanMessage](#HumanMessage)
- [HumanMessageChunk](#HumanMessageChunk)
- [SystemMessage](#SystemMessage)
- [SystemMessageChunk](#SystemMessageChunk)
- [ToolMessage](#ToolMessage)
- [ToolMessageChunk](#ToolMessageChunk)
- [trimMessages](#trimMessages)

---

## 类

### `AIMessage`
<a id="AIMessage"></a>
import { AIMessage } from "langchain";

```javascript
class AIMessage
```

LangChain Messages

## 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | --- | --- |
| content | --- | --- |
| id | --- | --- |
| invalid_tool_calls | --- | --- |
| lc_kwargs | --- | --- |
| lc_namespace | --- | --- |
| lc_serializable | --- | --- |
| name | --- | --- |
| response_metadata | --- | --- |
| tool_calls | --- | --- |
| type | --- | --- |
| usage_metadata | --- | --- |
| _printableFields | --- | --- |
| [toStringTag] | --- | --- |
| contentBlocks | --- | --- |
| lc_aliases | --- | --- |
| lc_attributes | --- | --- |
| lc_id | --- | --- |
| lc_secrets | --- | --- |
| lc_serializable_keys | --- | --- |
| text | --- | --- |

## 方法
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | _getType ( ) : MessageType | |
| _updateId | _updateId ( value : string \| undefined ) | |
| getType | getType ( ) : MessageType | |
| toDict | toDict ( ) : StoredMessage | |
| toFormattedString | toFormattedString ( format : "pretty" ) : string | |
| toJSON | toJSON ( ) : Serialized | |
| toJSONNotImplemented | toJSONNotImplemented ( ) : SerializedNotImplemented | |
| [hasInstance] | [hasInstance]() | |
| isInstance | isInstance ( obj : BaseMessage < T > ) : obj is BaseMessage < T , MessageType > AIMessage < T > | |
| lc_name | lc_name ( ) : string | The name of the serializable. Override to provide an alias or to preserve the serialized module name in minified environments. Implemented as a static method to support loading logic. |

### `AIMessageChunk`
<a id="AIMessageChunk"></a>
import { AIMessageChunk } from "langchain";

```javascript
class AIMessageChunk
```

Represents a chunk of an AI message, which can be concatenated with other AI message chunks.

## 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | --- | --- |
| content | --- | --- |
| id | --- | --- |
| invalid_tool_calls | --- | --- |
| lc_kwargs | --- | --- |
| lc_namespace | --- | --- |
| lc_serializable | --- | --- |
| name | --- | --- |
| response_metadata | --- | --- |
| tool_call_chunks | --- | --- |
| tool_calls | --- | --- |
| type | --- | --- |
| usage_metadata | --- | --- |
| _printableFields | --- | --- |
| [toStringTag] | --- | --- |
| contentBlocks | --- | --- |
| lc_aliases | --- | --- |
| lc_attributes | --- | --- |
| lc_id | --- | --- |
| lc_secrets | --- | --- |
| lc_serializable_keys | --- | --- |
| text | --- | --- |

## 方法
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | _getType ( ) : MessageType | |
| _updateId | _updateId ( value : string \| undefined ) | |
| concat | concat ( chunk : AIMessageChunk < TStructure > ) : this | Used in Docs ChatOpenAI integration ChatOpenRouter integration |
| getType | getType ( ) : MessageType | |
| toDict | toDict ( ) : StoredMessage | |
| toFormattedString | toFormattedString ( format : "pretty" ) : string | |
| toJSON | toJSON ( ) : Serialized | |
| toJSONNotImplemented | toJSONNotImplemented ( ) : SerializedNotImplemented | |
| [hasInstance] | [hasInstance]() | |
| isInstance | isInstance ( obj : BaseMessage < T > ) : obj is BaseMessage < T , MessageType > AIMessageChunk < T > | |
| lc_name | lc_name ( ) : string | The name of the serializable. Override to provide an alias or to preserve the serialized module name in minified environments. Implemented as a static method to support loading logic. |

### `BaseMessage`
<a id="BaseMessage"></a>
import { BaseMessage } from "langchain";

```javascript
class BaseMessage
```

Base class for all types of messages in a conversation. It includes properties like `content`, `name`, and `additional_kwargs`. It also includes methods like `toDict()` and `_getType()`.

## 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | --- | --- |
| content | --- | --- |
| id | --- | --- |
| lc_kwargs | --- | --- |
| lc_namespace | --- | --- |
| lc_serializable | --- | --- |
| name | --- | --- |
| response_metadata | --- | --- |
| type | --- | --- |
| _printableFields | --- | --- |
| [toStringTag] | --- | --- |
| contentBlocks | --- | --- |
| lc_aliases | --- | --- |
| lc_attributes | --- | --- |
| lc_id | --- | --- |
| lc_secrets | --- | --- |
| lc_serializable_keys | --- | --- |
| text | --- | --- |

## 方法
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | _getType ( ) : MessageType | |
| _updateId | _updateId ( value : string \| undefined ) | |
| getType | getType ( ) : MessageType | |
| toDict | toDict ( ) : StoredMessage | |
| toFormattedString | toFormattedString ( format : "pretty" ) : string | |
| toJSON | toJSON ( ) : Serialized | |
| toJSONNotImplemented | toJSONNotImplemented ( ) : SerializedNotImplemented | |
| [hasInstance] | [hasInstance]() | |
| isInstance | isInstance ( obj : unknown ) : obj is BaseMessage < MessageStructure < MessageToolSet > , MessageType > | |
| lc_name | lc_name ( ) : string | The name of the serializable. Override to provide an alias or to preserve the serialized module name in minified environments. Implemented as a static method to support loading logic. |

### `BaseMessageChunk`
<a id="BaseMessageChunk"></a>
import { BaseMessageChunk } from "langchain";

```javascript
class BaseMessageChunk
```

Represents a chunk of a message, which can be concatenated with other message chunks. It includes a method `_merge_kwargs_dict()` for merging additional keyword arguments from another `BaseMessageChunk` into this one. It also overrides the `__add__()` method to support concatenation of `BaseMessageChunk` instances.

## 属性
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | --- | --- |
| content | --- | --- |
| id | --- | --- |
| lc_kwargs | --- | --- |
| lc_namespace | --- | --- |
| lc_serializable | --- | --- |
| name | --- | --- |
| response_metadata | --- | --- |
| type | --- | --- |
| _printableFields | --- | --- |
| [toStringTag] | --- | --- |
| contentBlocks | --- | --- |
| lc_aliases | --- | --- |
| lc_attributes | --- | --- |
| lc_id | --- | --- |
| lc_secrets | --- | --- |
| lc_serializable_keys | --- | --- |
| text | --- | --- |

## 方法
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | _getType ( ) : MessageType | |
| _updateId | _updateId ( value : string \| undefined ) | |
| concat | concat ( chunk : BaseMessageChunk ) : BaseMessageChunk < TStructure , TRole > | Used in Docs ChatOpenAI integration ChatOpenRouter integration |
| getType | getType ( ) : MessageType | |
| toDict | toDict ( ) : StoredMessage | |
| toFormattedString | toFormattedString ( format : "pretty" ) : string | |
| toJSON | toJSON ( ) : Serialized | |
| toJSONNotImplemented | toJSONNotImplemented ( ) : SerializedNotImplemented | |
| [hasInstance] | [hasInstance]() | |
| isInstance | isInstance ( obj : unknown ) : obj is BaseMessageChunk < MessageStructure < MessageToolSet > , MessageType > | |
| lc_name | lc_name ( ) : string | The name of the serializable. Override to provide an alias or to preserve the serialized module name in minified environments. Implemented as a static method to support loading logic. |

## 接口

### `ContentBlock` <a id="ContentBlock"></a>
import { ContentBlock } from "langchain";

```ts
interface ContentBlock {
  id: string;
  type: string;
}
```

内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| type | string | 内容块类型 |

### `Citation` <a id="Citation"></a>
import { Citation } from "langchain";

```ts
interface Citation {
  citedText: string;
  endIndex: number;
  source: string;
  startIndex: number;
  title: string;
  type: "citation";
  url: string;
}
```

用于引用文档中的数据的注解。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| citedText | string | 被引用的源文本摘录。 |
| endIndex | number | 响应文本中该注解适用的结束索引。 |
| source | string | 引用类型。 |
| startIndex | number | 响应文本中该注解适用的起始索引。 |
| title | string | 源文档标题。例如，网页的页面标题或论文的标题。 |
| type | "citation" | 内容块类型。 |
| url | string | 文档源 URL。 |

### `Base64ContentBlock` <a id="Base64ContentBlock"></a>
import { Base64ContentBlock } from "langchain";

```ts
interface Base64ContentBlock {
  id: string;
  metadata: Record<string, unknown>;
  mime_type: string;
  source_type: "base64";
  type: "image" | "audio" | "file";
  data: string;
}
```

已弃用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| metadata | Record<string, unknown> | |
| mime_type | string | |
| source_type | "base64" | |
| type | "image" | "audio" | "file" | 内容块类型。 |
| data | string | |

### `BaseDataContentBlock` <a id="BaseDataContentBlock"></a>
import { BaseDataContentBlock } from "langchain";

```ts
interface BaseDataContentBlock {
  id: string;
  metadata: Record<string, unknown>;
  mime_type: string;
  type: string;
}
```

已弃用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| metadata | Record<string, unknown> | |
| mime_type | string | |
| type | string | 内容块类型 |

### `IDContentBlock` <a id="IDContentBlock"></a>
import { IDContentBlock } from "langchain";

```ts
interface IDContentBlock {
  id: string;
  metadata: Record<string, unknown>;
  mime_type: string;
  source_type: "id";
  type: "image" | "audio" | "file";
}
```

已弃用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| metadata | Record<string, unknown> | |
| mime_type | string | |
| source_type | "id" | |
| type | "image" | "audio" | "file" | 内容块类型。 |

### `PlainTextContentBlock` <a id="PlainTextContentBlock"></a>
import { PlainTextContentBlock } from "langchain";

```ts
interface PlainTextContentBlock {
  id: string;
  metadata: Record<string, unknown>;
  mime_type: string;
  source_type: "text";
  text: string;
  type: "text" | "file";
}
```

已弃用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| metadata | Record<string, unknown> | |
| mime_type | string | |
| source_type | "text" | |
| text | string | |
| type | "text" | "file" | 内容块类型。 |

### `URLContentBlock` <a id="URLContentBlock"></a>
import { URLContentBlock } from "langchain";

```ts
interface URLContentBlock {
  id: string;
  metadata: Record<string, unknown>;
  mime_type: string;
  source_type: "url";
  type: "image" | "audio" | "file";
  url: string;
}
```

已弃用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| metadata | Record<string, unknown> | |
| mime_type | string | |
| source_type | "url" | |
| type | "image" | "audio" | "file" | 内容块类型。 |
| url | string | |

### `NonStandard` <a id="NonStandard"></a>
import { NonStandard } from "langchain";

```ts
interface NonStandard {
  id: string;
  type: string;
  value: TValue;
}
```

提供商特定的内容块。这用于表示不属于标准 LangChain 内容模型的内容块。如果提供商的非标准输出包含推理和工具调用，则应由适配器解析该有效负载并发出相应的标准推理和工具调用块。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| type | string | 内容块类型 |
| value | TValue | 提供商特定数据 |

### `Reasoning` <a id="Reasoning"></a>
import { Reasoning } from "langchain";

```ts
interface Reasoning {
  id: string;
  index: string | number;
  reasoning: string;
  type: "reasoning";
}
```

来自 LLM 的推理输出。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| index | string | number | 聚合响应中的块索引。用于流式传输。 |
| reasoning | string | 推理文本。通常是思想摘要或原始推理文本本身。这通常是从模型响应中的 `<think>` 标签中解析出来的。 |
| type | "reasoning" | 内容块类型 |

### `Text` <a id="Text"></a>
import { Text } from "langchain";

```ts
interface Text {
  annotations: BaseContentBlock | Citation[];
  id: string;
  index: string | number;
  text: string;
  type: "text";
}
```

来自 LLM 的文本输出。这通常表示消息的主文本内容，例如来自语言模型的响应或用户消息的文本。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| annotations | BaseContentBlock | Citation[] | 引用和其他注解。 |
| id | string | 内容块标识符，可以是提供商生成的（例如提供商特定的 ID），也可以是 LangChain 在创建时生成的（以 'lc-' 开头的 uuid）。 |
| index | string | number | 聚合响应中的块索引。用于流式传输。 |
| text | string | 块文本。 |
| type | "text" | 内容块类型 |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/HumanMessage
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖消息相关的基础类及其变体。

## 接口
### `InvalidToolCall`
<a id="InvalidToolCall"></a>
```ts
interface InvalidToolCall
```
代表无效的工具调用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | string | 传递给工具调用的参数 |
| error | string | 与工具调用相关的错误信息 |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| index | string \| number | 聚合响应中的块索引 |
| name | TName | 被调用的工具名称 |
| type | "invalid_tool_call" | 内容块类型 |

## 接口
### `ServerToolCall`
<a id="ServerToolCall"></a>
```ts
interface ServerToolCall
```
代表服务器工具调用。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | TArgs | 传递给工具调用的参数 |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| name | TName | 被调用的工具名称 |
| type | "server_tool_call" | 内容块类型 |

## 接口
### `ServerToolCallChunk`
<a id="ServerToolCallChunk"></a>
```ts
interface ServerToolCallChunk
```
代表工具调用的部分数据。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | string | 传递给工具调用的参数 |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| index | string \| number | 工具调用分块的索引 |
| name | TName | 被调用的工具名称 |
| type | "server_tool_call_chunk" | 内容块类型 |

## 接口
### `ServerToolCallResult`
<a id="ServerToolCallResult"></a>
```ts
interface ServerToolCallResult
```
代表服务器工具调用的结果。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| output | TOutput | 服务器工具调用的输出 |
| status | "success" \| "error" | 服务器工具调用的状态 |
| toolCallId | string | 此结果对应的工具调用的唯一标识符 |
| type | "server_tool_call_result" | 内容块类型 |

## 接口
### `ToolCall`
<a id="ToolCall"></a>
```ts
interface ToolCall
```
表示对工具调用的请求。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | TArgs | 传递给工具调用的参数 |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| name | TName | 被调用的工具名称 |
| type | "tool_call" | 内容块类型 |

## 接口
### `ToolCallChunk`
<a id="ToolCallChunk"></a>
```ts
interface ToolCallChunk
```
代表工具调用的部分数据。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| args | string | 传递给工具调用的参数 |
| id | string | 内容块标识符，可以由提供者生成（例如提供者特定的 ID），也可以由 LangChain 在创建时生成（以 'lc-' 开头的 uuid） |
| index | string \| number | 工具调用分块的索引 |
| name | TName | 被调用的工具名称 |
| type | "tool_call_chunk" | 内容块类型 |

## 类
### `Document`
<a id="Document"></a>
```ts
class Document
```
用于与文档交互的接口。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 文档的可选标识符。理想情况下，它在整个文档集合中应该是唯一的，并格式化为 UUID，但不会强制执行。 |
| metadata | Metadata | |
| pageContent | string | |

## 类
### `DocumentInput`
<a id="DocumentInput"></a>
```ts
interface DocumentInput
```
LangChain 文档。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 文档的可选标识符。理想情况下，它在整个文档集合中应该是唯一的，并格式化为 UUID，但不会强制执行。 |
| metadata | Metadata | |
| pageContent | string | |

## 函数
### `filterMessages`
<a id="filterMessages"></a>
```ts
filterMessages(options: FilterMessagesFields): Runnable<BaseMessage<MessageStructure<MessageToolSet>, MessageType>[], BaseMessage<MessageStructure<MessageToolSet>, MessageType>[]>
```
过滤消息。

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- |
| options | FilterMessagesFields | 否 | 可选过滤选项。仅当 messagesOrOptions 是 BaseMessage 对象数组时才应提供。 |

## 类
### `HumanMessage`
<a id="HumanMessage"></a>
```ts
class HumanMessage
```
在对话中表示人类消息。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | | |
| content | | |
| id | string | |
| lc_kwargs | | |
| lc_namespace | | |
| lc_serializable | | |
| name | | |
| response_metadata | | |
| type | | |
| _printableFields | | |
| [toStringTag] | | |
| contentBlocks | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_secrets | | |
| lc_serializable_keys | | |
| text | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| constructor | `constructor(fields: InferMessageContent<TStructure, "human"> \| HumanMessageFields<TStructure>): HumanMessage<TStructure>` | |
| _getType | `_getType(): MessageType` | ⚠️ 已弃用 |
| _updateId | `_updateId(value: string \| undefined): void` | |
| getType | `getType(): MessageType` | ⚠️ 已弃用 |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(obj: unknown): obj is HumanMessage<MessageStructure<MessageToolSet>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。用于在最小化环境中提供别名或保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

## 类
### `HumanMessageChunk`
<a id="HumanMessageChunk"></a>
```ts
class HumanMessageChunk
```
表示人类消息的块，可以与其他人类消息块拼接。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | | |
| content | | |
| id | string | |
| lc_kwargs | | |
| lc_namespace | | |
| lc_serializable | | |
| name | | |
| response_metadata | | |
| type | | |
| _printableFields | | |
| [toStringTag] | | |
| contentBlocks | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_secrets | | |
| lc_serializable_keys | | |
| text | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| constructor | `constructor(fields: InferMessageContent<TStructure, "human"> \| HumanMessageFields<TStructure>): HumanMessageChunk<TStructure>` | |
| _getType | `_getType(): MessageType` | ⚠️ 已弃用 |
| _updateId | `_updateId(value: string \| undefined): void` | |
| concat | `concat(chunk: HumanMessageChunk<TStructure>): this` | 用于 ChatOpenAI 集成、ChatOpenRouter 集成。 |
| getType | `getType(): MessageType` | ⚠️ 已弃用 |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(obj: unknown): obj is HumanMessageChunk<MessageStructure<MessageToolSet>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。用于在最小化环境中提供别名或保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

## 类
### `SystemMessage`
<a id="SystemMessage"></a>
```ts
class SystemMessage
```
在对话中表示系统消息。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | | |
| content | | |
| id | string | |
| lc_kwargs | | |
| lc_namespace | | |
| lc_serializable | | |
| name | | |
| response_metadata | | |
| type | | |
| _printableFields | | |
| [toStringTag] | | |
| contentBlocks | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_secrets | | |
| lc_serializable_keys | | |
| text | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| constructor | `constructor(fields: InferMessageContent<TStructure, "system"> \| SystemMessageFields<TStructure>): SystemMessage<TStructure>` | |
| _getType | `_getType(): MessageType` | ⚠️ 已弃用 |
| _updateId | `_updateId(value: string \| undefined): void` | |
| concat | `concat(chunk: string \| SystemMessage<MessageStructure<MessageToolSet>>): SystemMessage<TStructure>` | 将字符串或另一个系统消息与当前的系统消息拼接。 |
| getType | `getType(): MessageType` | ⚠️ 已弃用 |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(obj: unknown): obj is SystemMessage<MessageStructure<MessageToolSet>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。用于在最小化环境中提供别名或保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

## 类
### `SystemMessageChunk`
<a id="SystemMessageChunk"></a>
```ts
class SystemMessageChunk
```
表示系统消息的块，可以与其他系统消息块拼接。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | | |
| content | | |
| id | string | |
| lc_kwargs | | |
| lc_namespace | | |
| lc_serializable | | |
| name | | |
| response_metadata | | |
| type | | |
| _printableFields | | |
| [toStringTag] | | |
| contentBlocks | | |
| lc_aliases | | |
| lc_attributes | | |
| lc_id | | |
| lc_secrets | | |
| lc_serializable_keys | | |
| text | | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| constructor | `constructor(fields: InferMessageContent<TStructure, "system"> \| SystemMessageFields<TStructure>): SystemMessageChunk<TStructure>` | |
| _getType | `_getType(): MessageType` | ⚠️ 已弃用 |
| _updateId | `_updateId(value: string \| undefined): void` | |
| concat | `concat(chunk: SystemMessageChunk<TStructure>): this` | |
| getType | `getType(): MessageType` | ⚠️ 已弃用 |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(obj: unknown): obj is SystemMessageChunk<MessageStructure<MessageToolSet>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。用于在最小化环境中提供别名或保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/ToolMessage
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖消息类及其相关处理函数。

## 类
### `ToolMessage`
<a id="ToolMessage"></a>
import { ToolMessage } from "langchain";

```ts
class ToolMessage
```

代表对话中的工具消息。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | any | |
| artifact | any | |
| content | any | |
| id | string | |
| lc_direct_tool_output | any | |
| lc_kwargs | any | |
| lc_namespace | any | |
| lc_serializable | any | |
| metadata | any | |
| name | string | |
| response_metadata | any | |
| status | any | |
| tool_call_id | string | |
| type | any | |
| _printableFields | any | |
| [toStringTag] | any | |
| contentBlocks | any | |
| lc_aliases | any | |
| lc_attributes | any | |
| lc_id | any | |
| lc_secrets | any | |
| lc_serializable_keys | any | |
| text | any | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | `_getType(): MessageType` | |
| _updateId | `_updateId(value: string | undefined): void` | |
| getType | `getType(): MessageType` | |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(message: BaseMessage<T>): message is BaseMessage<T, MessageType>` | 用于检查一个对象是否为 ToolMessage。当与带有类型的 BaseMessage 调用时，保留 TStructure 类型参数。 |
| lc_name | `lc_name(): string` | 可序列化的名称。可覆盖以提供别名或在混淆环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

### `ToolMessageChunk`
<a id="ToolMessageChunk"></a>
import { ToolMessageChunk } from "langchain";

```ts
class ToolMessageChunk
```

表示工具消息的一个分片，可以与其他工具消息分片拼接。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| additional_kwargs | any | |
| artifact | any | |
| content | any | |
| id | string | |
| lc_kwargs | any | |
| lc_namespace | any | |
| lc_serializable | any | |
| name | string | |
| response_metadata | any | |
| status | any | |
| tool_call_id | string | |
| type | any | |
| _printableFields | any | |
| [toStringTag] | any | |
| contentBlocks | any | |
| lc_aliases | any | |
| lc_attributes | any | |
| lc_id | any | |
| lc_secrets | any | |
| lc_serializable_keys | any | |
| text | any | |

| 方法 | 签名 | 说明 |
| --- | --- | --- |
| _getType | `_getType(): MessageType` | |
| _updateId | `_updateId(value: string | undefined): void` | |
| concat | `concat(chunk: ToolMessageChunk<TStructure>): this` | 用于 Docs ChatOpenAI 集成和 ChatOpenRouter 集成。 |
| getType | `getType(): MessageType` | |
| toDict | `toDict(): StoredMessage` | |
| toFormattedString | `toFormattedString(format: "pretty"): string` | |
| toJSON | `toJSON(): Serialized` | |
| toJSONNotImplemented | `toJSONNotImplemented(): SerializedNotImplemented` | |
| isInstance | `isInstance(obj: unknown): obj is ToolMessageChunk<MessageStructure<MessageToolSet>>` | |
| lc_name | `lc_name(): string` | 可序列化的名称。可覆盖以提供别名或在混淆环境中保留序列化的模块名称。作为静态方法实现以支持加载逻辑。 |

## 函数
### `trimMessages`
<a id="trimMessages"></a>
import { trimMessages } from "langchain";

```ts
trimMessages(options: TrimMessagesFields): Runnable<BaseMessage<MessageStructure<MessageToolSet>, MessageType>[], BaseMessage<MessageStructure<MessageToolSet>, MessageType>[]>
```

用于截断消息。用于 Docs Memory 短期记忆。

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| options | TrimMessagesFields | * | 截断选项。 |

| 返回 |
| --- |
| `Runnable<BaseMessage<MessageStructure<MessageToolSet>, MessageType>[], BaseMessage<MessageStructure<MessageToolSet>, MessageType>[]>` |

---
写入完成。
写入路径：D:\code\langchain\api\_parts\G1-p3.md
条目数量：3
异常：无