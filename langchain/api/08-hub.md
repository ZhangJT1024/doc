# LangChain Hub API 参考

本章节涵盖 `langchain/hub` 与 `langchain/hub/node` 模块中提供的函数，主要用于与 LangChain Hub 交互，包括从 Hub 拉取（pull）、推送（push）提示词，以及根据命名空间推断模型提供商。

## 目录
- [pull (langchain/hub)](#pull)
- [push (langchain/hub)](#push)
- [pull (langchain/hub/node)](#pull-node)
- [push (langchain/hub/node)](#push-node)
- [inferModelProviderFromNamespace (langchain/hub/node)](#inferModelProviderFromNamespace)

## pull
<a id="pull"></a>
### `pull`

**导入**
```ts
import { pull } from "langchain/hub";
```

**签名**
```ts
pull<T extends Runnable<any, any, RunnableConfig<Record<string, any>>>>(
  ownerRepoCommit: string,
  options?: __type
): Promise<T>
```

**说明**
从 Hub 中拉取提示词。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `ownerRepoCommit` | `string` | 是 | 包含提示词的仓库名称，以及可选的 commit 哈希，用斜杠 `/` 分隔。 |
| `options` | `__type` | 否 | 附加选项。 |

## push
<a id="push"></a>
### `push`

**导入**
```ts
import { push } from "langchain/hub";
```

**签名**
```ts
push(
  repoFullName: string,
  runnable: Runnable,
  options?: __type
): Promise<string>
```

**说明**
将提示词推送到 Hub。如果指定的仓库不存在，则会自动创建。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `repoFullName` | `string` | 是 | 仓库的完整名称。 |
| `runnable` | `Runnable` | 是 | 要推送的提示词。 |
| `options` | `__type` | 否 | 附加选项。 |

## pull (langchain/hub/node)
<a id="pull-node"></a>
### `pull`

**导入**
```ts
import { pull } from "langchain/hub/node";
```

**签名**
```ts
pull<T extends Runnable<any, any, RunnableConfig<Record<string, any>>>>(
  ownerRepoCommit: string,
  options?: __type
): Promise<T>
```

**说明**
从 Hub 中拉取提示词。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `ownerRepoCommit` | `string` | 是 | 包含提示词的仓库名称，以及可选的 commit 哈希，用斜杠 `/` 分隔。 |
| `options` | `__type` | 否 | 附加选项。 |

## push (langchain/hub/node)
<a id="push-node"></a>
### `push`

**导入**
```ts
import { push } from "langchain/hub/node";
```

**签名**
```ts
push(
  repoFullName: string,
  runnable: Runnable,
  options?: __type
): Promise<string>
```

**说明**
将提示词推送到 Hub。如果指定的仓库不存在，则会自动创建。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `repoFullName` | `string` | 是 | 仓库的完整名称。 |
| `runnable` | `Runnable` | 是 | 要推送的提示词。 |
| `options` | `__type` | 否 | 附加选项。 |

## inferModelProviderFromNamespace (langchain/hub/node)
<a id="inferModelProviderFromNamespace"></a>
### `inferModelProviderFromNamespace`

**导入**
```ts
import { inferModelProviderFromNamespace } from "langchain/hub/node";
```

**签名**
```ts
inferModelProviderFromNamespace(
  idArray: string[]
): string | undefined
```

**说明**
从 ID 命名空间中推断模型提供商，以避免类名冲突。对于非 LangChain 包，从命名空间中提取提供商名称。
例如：`["langchain", "chat_models", "vertexai", "ChatVertexAI"]` -> `"google-vertexai"`
例如：`["langchain_deepseek", "chat_models", "ChatDeepSeek"]` -> `"deepseek"`

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `idArray` | `string[]` | 是 | 来自清单（manifest）的完整 ID 数组。 |
