# LangChain JS API 参考 · 存储与序列化（Storage & Serialization）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/storage/in_memory 等
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章覆盖与键值存储（`langchain/storage/*`）、可序列化基类（`langchain/load/serializable`）以及反序列化加载（`langchain/load`）相关的类、函数与接口。

## 目录

- [InMemoryStore](#InMemoryStore)
- [EncoderBackedStore](#EncoderBackedStore)
- [LocalFileStore](#LocalFileStore)
- [Serializable](#Serializable)
- [createDocumentStoreFromByteStore](#createDocumentStoreFromByteStore)
- [load](#load)
- [get_lc_unique_name](#get_lc_unique_name)
- [BaseSerialized](#BaseSerialized)
- [SerializableInterface](#SerializableInterface)
- [SerializableLike](#SerializableLike)
- [SerializedConstructor](#SerializedConstructor)
- [SerializedNotImplemented](#SerializedNotImplemented)
- [SerializedSecret](#SerializedSecret)
- [Serialized](#Serialized)

---

## 类

### `InMemoryStore`
<a id="InMemoryStore"></a>

**导入**
```ts
import { InMemoryStore } from "langchain/storage/in_memory";
// 也可从主入口导入：import { InMemoryStore } from "langchain";
```

**签名**
```ts
class InMemoryStore
```

**继承**：`BaseStore<string, T>`

**说明**
基于字典的 `BaseStore` 内存实现，用于在内存中存储键值对。

**构造**
```ts
new InMemoryStore<T = any>(kwargs: SerializedFields, _args: never[]): InMemoryStore<T>
```

**属性**：`lc_kwargs`、`lc_namespace`、`lc_serializable`、`store`、`lc_aliases`、`lc_attributes`、`lc_id`、`lc_secrets`、`lc_serializable_keys`

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `mdelete` | `mdelete(keys: string[]): Promise<void>` | 删除给定键及其对应的值。 |
| `mget` | `mget(keys: string[]): Promise<T[]>` | 获取给定键对应的值。 |
| `mset` | `mset(keyValuePairs: [string, T][]): Promise<void>` | 设置给定键的值。 |
| `yieldKeys` | `yieldKeys(prefix?: string): AsyncGenerator<string>` | 异步生成器，逐个产出存储中的键；若提供前缀，则只产出以该前缀开头的键。 |
| `toJSON` | `toJSON(): Serialized` | 序列化。 |
| `toJSONNotImplemented` | `toJSONNotImplemented(): SerializedNotImplemented` | 未实现的序列化占位。 |
| `lc_name` | `lc_name(): string` | 可序列化对象的名称（静态方法，支持加载逻辑）。 |

**示例**
```typescript
const store = new InMemoryStore<BaseMessage>();
await store.mset(
  Array.from({ length: 5 }).map((_, index) => [
    `message:id:${index}`,
    index % 2 === 0
      ? new AIMessage("ai stuff...")
      : new HumanMessage("human stuff..."),
  ]),
);

const retrievedMessages = await store.mget(["message:id:0", "message:id:1"]);
await store.mdelete(await store.yieldKeys("message:id:").toArray());
```

### `EncoderBackedStore`
<a id="EncoderBackedStore"></a>

**导入**
```ts
import { EncoderBackedStore } from "langchain/storage/encoder_backed";
```

**签名**
```ts
class EncoderBackedStore
```

**继承**：`BaseStore<K, V>`

**说明**
在底层存储之上提供一层抽象，允许对键和值进行编码与解码。

**属性**：`keyEncoder`、`valueSerializer`、`valueDeserializer`、`store`、`lc_kwargs`、`lc_namespace`、`lc_serializable`、`lc_aliases`、`lc_attributes`、`lc_id`、`lc_secrets`、`lc_serializable_keys`

**方法**：`constructor`、`mdelete`、`mget`、`mset`、`yieldKeys`、`toJSON`、`toJSONNotImplemented`、`lc_name`（签名同 `InMemoryStore` 的 `BaseStore` 接口）。

### `LocalFileStore`
<a id="LocalFileStore"></a>

**导入**
```ts
import { LocalFileStore } from "langchain/storage/file_system";
```

**签名**
```ts
class LocalFileStore
```

**继承**：`BaseStore<string, Uint8Array>`

**说明**
基于文件系统的 `BaseStore` 实现，用于在文件系统中存储键值对。

**属性**：`rootPath`、`store`、`lc_kwargs`、`lc_namespace`、`lc_serializable`、`lc_aliases`、`lc_attributes`、`lc_id`、`lc_secrets`、`lc_serializable_keys`

**方法**：`constructor`、`fromPath`（静态，从路径创建）、`mdelete`、`mget`、`mset`、`yieldKeys`、`toJSON`、`toJSONNotImplemented`、`lc_name`

**示例**
```typescript
const store = await LocalFileStore.fromPath("./messages");
await store.mset(
  Array.from({ length: 5 }).map((_, index) => [
    `message:id:${index}`,
    new TextEncoder().encode(
      JSON.stringify(
        index % 2 === 0
          ? new AIMessage("ai stuff...")
          : new HumanMessage("human stuff..."),
      ),
    ),
  ]),
);
const retrievedMessages = await store.mget(["message:id:0", "message:id:1"]);
console.log(retrievedMessages.map((v) => new TextDecoder().decode(v)));
for await (const key of store.yieldKeys("message:id:")) {
  await store.mdelete([key]);
}
```

### `Serializable`
<a id="Serializable"></a>

**导入**
```ts
import { Serializable } from "langchain/load/serializable";
```

**签名**
```ts
class Serializable
```

**说明**
LangChain 可序列化对象的基类。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `lc_kwargs` | `SerializedFields` | 序列化的字段。 |
| `lc_namespace` | `string[]` | 包含该类的模块路径，例如 `["langchain", "llms"]`。通常与导出该类的入口点相同。 |
| `lc_serializable` | `boolean` | 是否可序列化。 |
| `lc_aliases` | `对象 \| undefined` | 别名映射。 |
| `lc_attributes` | `SerializedFields \| undefined` | 附加属性。 |
| `lc_id` | `string[]` | 序列化 ID。 |
| `lc_secrets` | `对象 \| undefined` | 密钥声明。 |
| `lc_serializable_keys` | `string[] \| undefined` | 需要序列化的键。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `toJSON` | `toJSON(): Serialized` | 序列化为 `Serialized`。 |
| `toJSONNotImplemented` | `toJSONNotImplemented(): SerializedNotImplemented` | 未实现的序列化占位。 |
| `lc_name` | `lc_name(): string` | 可序列化对象的名称；可覆盖以提供别名或在压缩环境中保留序列化模块名。作为静态方法实现以支持加载逻辑。 |

## 函数

### `createDocumentStoreFromByteStore`
<a id="createDocumentStoreFromByteStore"></a>

**导入**
```ts
import { createDocumentStoreFromByteStore } from "langchain/storage/encoder_backed";
```

**签名**
```ts
createDocumentStoreFromByteStore(
  store: BaseStore<string, Uint8Array<ArrayBufferLike>>
): EncoderBackedStore<string, Document<Record<string, any>>, Uint8Array<ArrayBufferLike>>
```

**说明**
基于一个字节存储（`BaseStore<string, Uint8Array>`）创建一个存放 `Document` 的 `EncoderBackedStore`。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `store` | `BaseStore<string, Uint8Array<ArrayBufferLike>>` | 是 | 底层字节存储。 |

**返回**
`EncoderBackedStore<string, Document<Record<string, any>>, Uint8Array<ArrayBufferLike>>`

### `load`
<a id="load"></a>

**导入**
```ts
import { load } from "langchain/load";
```

**签名**
```ts
load<T>(
  text: string,
  secretsMap: Record<string, any> = {},
  optionalImportsMap: OptionalImportMap & Record<string, any> = {},
  additionalImportsMap: Record<string, any> = {},
  secretsFromEnv?: boolean
): Promise<T>
```

**说明**
从序列化文本表示中加载一个 LangChain 模块。**注意**：该功能目前处于 beta 阶段，被加载的类可能独立于 semver 发生变化。

> **警告——不安全的反序列化风险。** 该函数会根据 `text` 的内容实例化类并调用构造函数。**切勿**对不可信或用户提供的输入调用它。只反序列化来自你控制的、可信来源的数据。
> 关于 `secretsFromEnv`、`secretsMap` 和 import maps 的详细安全指引，请参见 `@langchain/core/load` 的 `LoadOptions`。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `text` | `string` | 是 | 模块的序列化文本表示。 |
| `secretsMap` | `Record<string, any>` | 否 | 密钥映射（默认 `{}`）。 |
| `optionalImportsMap` | `OptionalImportMap & Record<string, any>` | 否 | 可选导入映射（默认 `{}`）。 |
| `additionalImportsMap` | `Record<string, any>` | 否 | 附加导入映射（默认 `{}`）。 |
| `secretsFromEnv` | `boolean` | 否 | 是否从环境变量读取密钥。 |

**返回**
`Promise<T>` —— 已加载的 LangChain 模块实例。

### `get_lc_unique_name`
<a id="get_lc_unique_name"></a>

**导入**
```ts
import { get_lc_unique_name } from "langchain/load/serializable";
```

**签名**
```ts
get_lc_unique_name(serializableClass: typeof Serializable): string
```

**说明**
获取模块的唯一名称，而不是使用父类实现。不应被子类化；如需别名请在子类中覆盖 `lc_name`。

**参数**
| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `serializableClass` | `typeof Serializable` | 是 | 可序列化类。 |

**返回**
`string`

## 接口

### `BaseSerialized`
<a id="BaseSerialized"></a>

**导入**
```ts
import type { BaseSerialized } from "langchain/load/serializable";
```

**说明**
所有序列化形式的基础接口。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `graph` | `Record<string, any>` | 图信息。 |
| `id` | `string[]` | 序列化 ID 数组。 |
| `lc` | `number` | LangChain 序列化版本号。 |
| `name` | `string` | 名称。 |
| `type` | `T` | 序列化类型。 |

### `SerializableInterface`
<a id="SerializableInterface"></a>

**导入**
```ts
import type { SerializableInterface } from "langchain/load/serializable";
```

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `lc_id` | `string[]` | 序列化 ID。 |

### `SerializableLike`
<a id="SerializableLike"></a>

**导入**
```ts
import type { SerializableLike } from "langchain/load/serializable";
```

**说明**
可被序列化的对象的接口。这是一个鸭子类型（duck-typed）接口，用于避免循环导入。

**属性**
| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `lc_secrets` | `Record<string, string>` | 密钥声明。 |
| `lc_serializable` | `boolean` | 是否可序列化。 |

**方法**
| 方法 | 签名 | 说明 |
| --- | --- | --- |
| `toJSON` | `toJSON(): Serialized` | 序列化。 |

### `SerializedConstructor`
<a id="SerializedConstructor"></a>

**导入**
```ts
import type { SerializedConstructor } from "langchain/load/serializable";
```

**继承**：`BaseSerialized<"constructor">`

**属性**：`graph`、`id`、`kwargs`（`SerializedFields`）、`lc`、`name`、`type`（`"constructor"`）

### `SerializedNotImplemented`
<a id="SerializedNotImplemented"></a>

**导入**
```ts
import type { SerializedNotImplemented } from "langchain/load/serializable";
```

**继承**：`BaseSerialized<"not_implemented">`

**属性**：`graph`、`id`、`lc`、`name`、`type`（`"not_implemented"`）

### `SerializedSecret`
<a id="SerializedSecret"></a>

**导入**
```ts
import type { SerializedSecret } from "langchain/load/serializable";
```

**继承**：`BaseSerialized<"secret">`

**属性**：`graph`、`id`、`lc`、`name`、`type`（`"secret"`）

## 类型

### `Serialized`
<a id="Serialized"></a>

**导入**
```ts
import type { Serialized } from "langchain/load/serializable";
```

**签名**
```ts
Serialized : SerializedConstructor | SerializedSecret | SerializedNotImplemented
```

**说明**
所有序列化形式的联合类型。
