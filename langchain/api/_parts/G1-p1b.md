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
