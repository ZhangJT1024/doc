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
