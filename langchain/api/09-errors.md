# LangChain JS API 参考 · 错误（Errors）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/index/
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章收录 `langchain` 导出的全部错误类。它们都直接或间接继承自 `Error`。

## 目录

- [MiddlewareError](#MiddlewareError)
- [MultipleStructuredOutputsError](#MultipleStructuredOutputsError)
- [MultipleToolsBoundError](#MultipleToolsBoundError)
- [PIIDetectionError](#PIIDetectionError)
- [StructuredOutputParsingError](#StructuredOutputParsingError)
- [ToolCallLimitExceededError](#ToolCallLimitExceededError)
- [ToolInvocationError](#ToolInvocationError)

---

## 类

### `MiddlewareError`
<a id="MiddlewareError"></a>

**导入**
```ts
import { MiddlewareError } from "langchain";
```

**签名**
```ts
class MiddlewareError
```

**继承**：`Error`

**说明**
中间件执行失败时抛出的错误。

请使用 `MiddlewareError.wrap()` 创建实例。其构造函数是私有的，以确保 `GraphBubbleUp` 类错误（如 `GraphInterrupt`）永远不会被包装。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `~brand` | `"MiddlewareError"` | 品牌标记。 |
| `cause` | `unknown` | 错误原因。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `stack` | `string` | 堆栈信息。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `isInstance` | `error is MiddlewareError` | 判断给定错误是否为 `MiddlewareError`。 |
| `wrap` | `(err: unknown) => Error` | 将错误包装成 `MiddlewareError`；若为 `GraphBubbleUp` 类错误（如 `GraphInterrupt`）则原样抛出。 |
| `[hasInstance]` | `(obj: unknown) => obj is MiddlewareError` | 实例判断。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `MultipleStructuredOutputsError`
<a id="MultipleStructuredOutputsError"></a>

**导入**
```ts
import { MultipleStructuredOutputsError } from "langchain";
```

**签名**
```ts
class MultipleStructuredOutputsError
```

**继承**：`Error`

**说明**
当模型返回了多个结构化输出工具调用、而预期只有一个时抛出。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `cause` | `unknown` | 错误原因。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `stack` | `string` | 堆栈信息。 |
| `toolNames` | `string[]` | 涉及的多个工具名称。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `constructor` | `new MultipleStructuredOutputsError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `MultipleToolsBoundError`
<a id="MultipleToolsBoundError"></a>

**导入**
```ts
import { MultipleToolsBoundError } from "langchain";
```

**签名**
```ts
class MultipleToolsBoundError
```

**继承**：`Error`

**说明**
当模型被绑定了多个工具、而此处只允许一个时抛出。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `cause` | `unknown` | 错误原因。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `stack` | `string` | 堆栈信息。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `constructor` | `new MultipleToolsBoundError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `PIIDetectionError`
<a id="PIIDetectionError"></a>

**导入**
```ts
import { PIIDetectionError } from "langchain";
```

**签名**
```ts
class PIIDetectionError
```

**继承**：`Error`

**说明**
当检测到 PII（个人身份信息）且处理策略为 `'block'` 时抛出。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `cause` | `unknown` | 错误原因。 |
| `matches` | `PIIMatch[]` | 检测到的 PII 匹配项。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `piiType` | `string` | 检测到的 PII 类型。 |
| `stack` | `string` | 堆栈信息。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `constructor` | `new PIIDetectionError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `StructuredOutputParsingError`
<a id="StructuredOutputParsingError"></a>

**导入**
```ts
import { StructuredOutputParsingError } from "langchain";
```

**签名**
```ts
class StructuredOutputParsingError
```

**继承**：`Error`

**说明**
当结构化输出工具调用的参数无法按给定 schema 解析时抛出。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `cause` | `unknown` | 错误原因。 |
| `errors` | `string[]` | 解析错误列表。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `stack` | `string` | 堆栈信息。 |
| `toolName` | `string` | 相关工具名称。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `constructor` | `new StructuredOutputParsingError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `ToolCallLimitExceededError`
<a id="ToolCallLimitExceededError"></a>

**导入**
```ts
import { ToolCallLimitExceededError } from "langchain";
```

**签名**
```ts
class ToolCallLimitExceededError
```

**继承**：`Error`

**说明**
工具调用次数超过限制时抛出。

当配置的退出行为为 `'error'`，且线程级或运行级的工具调用限制被超出时，会抛出该异常。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `cause` | `unknown` | 错误原因。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `runCount` | `number` | 当前运行级工具调用计数。 |
| `runLimit` | `number \| undefined` | 运行级工具调用限制（如已设置）。 |
| `stack` | `string` | 堆栈信息。 |
| `threadCount` | `number` | 当前线程级工具调用计数。 |
| `threadLimit` | `number \| undefined` | 线程级工具调用限制（如已设置）。 |
| `toolName` | `string \| undefined` | 被限制的工具名（若针对特定工具），对全部工具限制时为 `undefined`。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `constructor` | `new ToolCallLimitExceededError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |

### `ToolInvocationError`
<a id="ToolInvocationError"></a>

**导入**
```ts
import { ToolInvocationError } from "langchain";
```

**签名**
```ts
class ToolInvocationError
```

**继承**：`Error`

**说明**
工具调用抛出错误时抛出。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `~brand` | `"ToolInvocationError"` | 品牌标记。 |
| `cause` | `unknown` | 错误原因。 |
| `message` | `string` | 错误消息。 |
| `name` | `string` | 错误的名称/类型。 |
| `stack` | `string` | 堆栈信息。 |
| `toolCall` | `ToolCall` | 相关的工具调用。 |
| `toolError` | `Error` | 工具内部抛出的原始错误。 |
| `stackTraceLimit` | `number` | 堆栈跟踪深度限制。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `isInstance` | `error is ToolInvocationError` | 判断给定错误是否为 `ToolInvocationError`。 |
| `[hasInstance]` | `(obj: unknown) => obj is ToolInvocationError` | 实例判断。 |
| `constructor` | `new ToolInvocationError(...)` | 构造函数。 |
| `captureStackTrace` | — | 继承自 `Error`。 |
| `prepareStackTrace` | — | 继承自 `Error`。 |
