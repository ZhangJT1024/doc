# Messages（消息）

> 原文：https://docs.langchain.com/oss/javascript/langchain/messages
> LangChain (TypeScript) 官方文档中文整理

消息（Messages）是 LangChain 中模型上下文的基本单位。它们表示模型的输入和输出，同时承载内容与元数据，用于在与 LLM 交互时表示对话的状态。

消息是包含以下内容的对象：

* [**角色（Role）**](#message-types) - 标识消息类型（例如 `system`、`user`）
* [**内容（Content）**](#message-content) - 表示消息的实际内容（如文本、图像、音频、文档等）
* [**元数据（Metadata）**](#message-metadata) - 可选字段，例如响应信息、消息 ID 和 token 用量

LangChain 提供了一种在所有模型 provider 上都通用的标准消息类型，确保无论调用哪个模型，行为都保持一致。

## 基础用法

使用消息最简单的方式是创建消息对象，并在[调用](/oss/javascript/langchain/models#invocation)时将它们传给模型。

```typescript
import { initChatModel, HumanMessage, SystemMessage } from "langchain";

const model = await initChatModel("gpt-5-nano");

const systemMsg = new SystemMessage("You are a helpful assistant.");
const humanMsg = new HumanMessage("Hello, how are you?");

const messages = [systemMsg, humanMsg];
const response = await model.invoke(messages);  // Returns AIMessage
```

> 【提示】多轮[智能体](/oss/javascript/langchain/agents)会累积很长的消息历史。[LangSmith](/langsmith/observability) 会记录每一轮、每个工具结果和模型响应，因此你可以检查完整的对话。按照[追踪快速入门](/langsmith/trace-with-langchain)启用追踪。
>
> 我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的 trace、检测问题并提出修复建议。

### 文本提示词

文本提示词就是字符串——非常适合那些不需要保留对话历史的直接生成任务。

```typescript
const response = await model.invoke("Write a haiku about spring");
```

**在以下情况使用文本提示词：**

* 你只有一个独立的请求
* 你不需要对话历史
* 你希望代码复杂度最低

### 消息提示词

或者，你可以通过提供一组消息对象，向模型传入一个消息列表。

```typescript
import { SystemMessage, HumanMessage, AIMessage } from "langchain";

const messages = [
  new SystemMessage("You are a poetry expert"),
  new HumanMessage("Write a haiku about spring"),
  new AIMessage("Cherry blossoms bloom..."),
];
const response = await model.invoke(messages);
```

**在以下情况使用消息提示词：**

* 管理多轮对话
* 处理多模态内容（图像、音频、文件）
* 包含系统指令

### 字典格式

你也可以直接用 OpenAI chat completions 格式来指定消息。

```typescript
const messages = [
  { role: "system", content: "You are a poetry expert" },
  { role: "user", content: "Write a haiku about spring" },
  { role: "assistant", content: "Cherry blossoms bloom..." },
];
const response = await model.invoke(messages);
```

## 消息类型

* [系统消息（System message）](#system-message) - 告诉模型如何行为，并为交互提供上下文
* [人类消息（Human message）](#human-message) - 表示用户输入以及与模型的交互
* [AI 消息（AI message）](#ai-message) - 由模型生成的响应，包括文本内容、工具调用和元数据
* [工具消息（Tool message）](#tool-message) - 表示[工具调用](/oss/javascript/langchain/models#tool-calling)的输出

### 系统消息

[`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) 表示一组初始指令，用于引导模型的行为。你可以使用系统消息来设定语气、定义模型的角色，并建立响应准则。

**基础指令**

```typescript
import { SystemMessage, HumanMessage, AIMessage } from "langchain";

const systemMsg = new SystemMessage("You are a helpful coding assistant.");

const messages = [
  systemMsg,
  new HumanMessage("How do I create a REST API?"),
];
const response = await model.invoke(messages);
```

**详细人设**

```typescript
import { SystemMessage, HumanMessage } from "langchain";

const systemMsg = new SystemMessage(`
You are a senior TypeScript developer with expertise in web frameworks.
Always provide code examples and explain your reasoning.
Be concise but thorough in your explanations.
`);

const messages = [
  systemMsg,
  new HumanMessage("How do I create a REST API?"),
];
const response = await model.invoke(messages);
```

***

### 人类消息

[`HumanMessage`](https://reference.langchain.com/javascript/langchain-core/messages/HumanMessage) 表示用户输入和交互。它们可以包含文本、图像、音频、文件，以及任何其他形式的多模态[内容](#message-content)。

#### 文本内容

**消息对象**

```typescript
const response = await model.invoke([
  new HumanMessage("What is machine learning?"),
]);
```

**字符串快捷方式**

```typescript
const response = await model.invoke("What is machine learning?");
```

#### 消息元数据

```typescript
const humanMsg = new HumanMessage({
  content: "Hello!",
  name: "alice",
  id: "msg_123",
});
```

> 【提示】`name` 字段的行为因 provider 而异——有些用它来标识用户，有些则忽略它。要确认这一点，请参阅该模型 provider 的[参考文档](https://reference.langchain.com/python/integrations/)。

***

### AI 消息

[`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 表示模型调用的输出。它们可以包含多模态数据、工具调用，以及你之后可以访问的 provider 特定元数据。

```typescript
const response = await model.invoke("Explain AI");
console.log(typeof response);  // AIMessage
```

调用模型时会返回 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 对象，其中包含响应中所有相关的元数据。

不同 provider 对消息类型的权衡/上下文化方式不同，这意味着有时手动创建一个新的 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 对象，并把它插入消息历史（就像它来自模型一样）会很有帮助。

```typescript
import { AIMessage, SystemMessage, HumanMessage } from "langchain";

const aiMsg = new AIMessage("I'd be happy to help you with that question!");

const messages = [
  new SystemMessage("You are a helpful assistant"),
  new HumanMessage("Can you help me?"),
  aiMsg,  // Insert as if it came from the model
  new HumanMessage("Great! What's 2+2?")
]

const response = await model.invoke(messages);
```

#### 属性

- `text`（string）：消息的文本内容。
- `content`（string | ContentBlock[]）：消息的原始内容。
- `content_blocks`（ContentBlock.Standard[]）：消息的标准化内容块。（参见[内容](#message-content)）
- `tool_calls`（ToolCall[] | None）：模型发起的工具调用。如果没有调用任何工具则为空。
- `id`（string）：消息的唯一标识符（由 LangChain 自动生成，或在 provider 响应中返回）
- `usage_metadata`（UsageMetadata | None）：消息的用量元数据，在可用时可能包含 token 数量。参见 [`UsageMetadata`](https://reference.langchain.com/javascript/langchain-core/messages/UsageMetadata)。
- `response_metadata`（ResponseMetadata | None）：消息的响应元数据。

#### 工具调用

当模型发起[工具调用](/oss/javascript/langchain/models#tool-calling)时，它们会包含在 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 中：

```typescript
const modelWithTools = model.bindTools([getWeather]);
const response = await modelWithTools.invoke("What's the weather in Paris?");

for (const toolCall of response.tool_calls) {
  console.log(`Tool: ${toolCall.name}`);
  console.log(`Args: ${toolCall.args}`);
  console.log(`ID: ${toolCall.id}`);
}
```

其他结构化数据，例如推理或引用，也可能出现在消息[内容](/oss/javascript/langchain/messages#message-content)中。

#### Token 用量

[`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 可以在其 [`usage_metadata`](https://reference.langchain.com/javascript/langchain-core/messages/UsageMetadata) 字段中保存 token 数量和其他用量元数据：

```typescript
import { initChatModel } from "langchain";

const model = await initChatModel("gpt-5-nano");

const response = await model.invoke("Hello!");
console.log(response.usage_metadata);
```

```json
{
  "output_tokens": 304,
  "input_tokens": 8,
  "total_tokens": 312,
  "input_token_details": {
    "cache_read": 0
  },
  "output_token_details": {
    "reasoning": 256
  }
}
```

详情请参阅 [`UsageMetadata`](https://reference.langchain.com/javascript/langchain-core/messages/UsageMetadata)。

#### 流式输出与分块

在流式传输期间，你会收到 [`AIMessageChunk`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessageChunk) 对象，它们可以组合成一条完整的消息对象：

```typescript
import { AIMessageChunk } from "langchain";

let finalChunk: AIMessageChunk | undefined;
for (const chunk of chunks) {
  finalChunk = finalChunk ? finalChunk.concat(chunk) : chunk;
}
```

> 【提示】了解更多：
>
> * [从聊天模型流式输出 token](/oss/javascript/langchain/models#stream)
> * [从智能体流式输出 token 和/或步骤](/oss/javascript/langchain/streaming)

***

### 工具消息

对于支持[工具调用](/oss/javascript/langchain/models#tool-calling)的模型，AI 消息可以包含工具调用。工具消息用于将单次工具执行的结果传回给模型。

[工具](/oss/javascript/langchain/tools)可以直接生成 [`ToolMessage`](https://reference.langchain.com/javascript/langchain-core/messages/ToolMessage) 对象。下面我们展示一个简单示例。更多内容请阅读[工具指南](/oss/javascript/langchain/tools)。

```typescript
import { AIMessage, ToolMessage } from "langchain";

const aiMessage = new AIMessage({
  content: [],
  tool_calls: [{
    name: "get_weather",
    args: { location: "San Francisco" },
    id: "call_123"
  }]
});

const toolMessage = new ToolMessage({
  content: "Sunny, 72°F",
  tool_call_id: "call_123"
});

const messages = [
  new HumanMessage("What's the weather in San Francisco?"),
  aiMessage,  // Model's tool call
  toolMessage,  // Tool execution result
];

const response = await model.invoke(messages);  // Model processes the result
```

#### 属性

- `content`（string，必填）：工具调用输出的字符串化结果。
- `tool_call_id`（string，必填）：此消息所响应的工具调用的 ID。必须与 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 中工具调用的 ID 匹配。
- `name`（string，必填）：被调用工具的名称。
- `artifact`（dict）：不发送给模型，但可以通过编程方式访问的附加数据。

> 【提示】`artifact` 字段存储不会发送给模型、但可以通过编程方式访问的补充数据。这对于存储原始结果、调试信息，或用于下游处理的数据而无需污染模型上下文非常有用。
>
> **示例：使用 artifact 存放检索元数据**
>
> 例如，一个[检索](/oss/javascript/deepagents/retrieval)工具可以检索文档中的一段文字供模型参考。消息的 `content` 包含模型将引用的文本，而 `artifact` 可以包含文档标识符或应用可使用的其他元数据（例如用于渲染页面）。参见下方示例：
>
> ```typescript
> import { ToolMessage } from "langchain";
>
> // Artifact available downstream
> const artifact = { document_id: "doc_123", page: 0 };
>
> const toolMessage = new ToolMessage({
>   content: "It was the best of times, it was the worst of times.",
>   tool_call_id: "call_123",
>   name: "search_books",
>   artifact
> });
> ```
>
> 有关使用 LangChain 构建检索[智能体](/oss/javascript/langchain/agents)的端到端示例，请参阅 [RAG 教程](/oss/javascript/deepagents/rag)。

***

## 消息内容

你可以把消息的内容看作发送给模型的数据载荷。消息有一个类型较宽松的 `content` 属性，支持字符串和无类型对象列表（例如字典）。这使得 LangChain 聊天模型可以直接支持 provider 原生的结构，例如[多模态](#multimodal)内容和其他数据。

另外，LangChain 为文本、推理、引用、多模态数据、服务端工具调用以及其他消息内容提供了专门的内容类型。参见下方的[内容块](#standard-content-blocks)。

LangChain 聊天模型在 `content` 属性中接受消息内容。

它可能包含以下任一项：

1. 一个字符串
2. provider 原生格式的内容块列表
3. [LangChain 标准内容块](#standard-content-blocks)列表

下面是一个使用[多模态](#multimodal)输入的示例：

```typescript
import { HumanMessage } from "langchain";

// String content
const humanMessage = new HumanMessage("Hello, how are you?");

// Provider-native format (e.g., OpenAI)
const humanMessage = new HumanMessage({
  content: [
    { type: "text", text: "Hello, how are you?" },
    {
      type: "image_url",
      image_url: { url: "https://example.com/image.jpg" },
    },
  ],
});

// List of standard content blocks
const humanMessage = new HumanMessage({
  contentBlocks: [
    { type: "text", text: "Hello, how are you?" },
    { type: "image", url: "https://example.com/image.jpg" },
  ],
});
```

### 标准内容块

LangChain 为消息内容提供了一种跨 provider 通用的标准表示。

消息对象实现了一个 `contentBlocks` 属性，它会惰性地将 `content` 属性解析为标准的、类型安全的表示。例如，由 [`ChatAnthropic`](/oss/javascript/integrations/chat/anthropic) 或 [`ChatOpenAI`](/oss/javascript/integrations/chat/openai) 生成的消息会以各自 provider 的格式包含 `thinking` 或 `reasoning` 块，但可以被惰性解析为一致的 [`ReasoningContentBlock`](#content-block-reference) 表示：

#### Anthropic

```typescript
import { AIMessage } from "@langchain/core/messages";

const message = new AIMessage({
  content: [
    {
      "type": "thinking",
      "thinking": "...",
      "signature": "WaUjzkyp...",
    },
    {
      "type":"text",
      "text": "...",
      "id": "msg_abc123",
    },
  ],
  response_metadata: { model_provider: "anthropic" },
});

console.log(message.contentBlocks);
```

#### OpenAI

```typescript
import { AIMessage } from "@langchain/core/messages";

const message = new AIMessage({
  content: [
    {
      "type": "reasoning",
      "id": "rs_abc123",
      "summary": [
        {"type": "summary_text", "text": "summary 1"},
        {"type": "summary_text", "text": "summary 2"},
      ],
    },
    {"type": "text", "text": "..."},
  ],
  response_metadata: { model_provider: "openai" },
});

console.log(message.contentBlocks);
```

要开始使用你选择的推理 provider，请参阅[集成指南](/oss/javascript/integrations/providers/overview)。

> 【提示】**序列化标准内容**
>
> 如果 LangChain 之外的应用需要访问标准内容块表示，你可以选择将内容块存储在消息内容中。
>
> 为此，你可以将 `LC_OUTPUT_VERSION` 环境变量设置为 `v1`。或者，用 `outputVersion: "v1"` 初始化任意聊天模型：
>
> ```typescript
> import { initChatModel } from "langchain";
>
> const model = await initChatModel(
>   "gpt-5-nano",
>   { outputVersion: "v1" }
> );
> ```

### 多模态

**多模态（Multimodality）** 指的是处理以不同形式出现的数据的能力，例如文本、音频、图像和视频。LangChain 为这些数据提供了可在各 provider 间通用的标准类型。

[聊天模型](/oss/javascript/langchain/models)可以接受多模态数据作为输入，并生成多模态数据作为输出。下面我们展示一些包含多模态数据的输入消息的简短示例。

> 【提示】额外的键可以放在内容块的顶层，也可以嵌套在 `"extras": {"key": value}` 中。
>
> 例如，[OpenAI](/oss/javascript/integrations/chat/openai) 要求为 PDF 提供文件名。有关你所选模型的具体细节，请参阅 [provider 页面](/oss/javascript/integrations/providers/overview)。

**图像输入**

```typescript
// From URL
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this image." },
    {
      type: "image",
      source_type: "url",
      url: "https://example.com/path/to/image.jpg"
    },
  ],
});

// From base64 data
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this image." },
    {
      type: "image",
      source_type: "base64",
      mime_type: "image/jpeg",
      data: "AAAAIGZ0eXBtcDQyAAAAAGlzb21tcDQyAAACAGlzb2...",
    },
  ],
});

// From provider-managed File ID
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this image." },
    { type: "image", source_type: "id", id: "file-abc123" },
  ],
});
```

**PDF 文档输入**

```typescript
// From URL
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this document." },
    { type: "file", source_type: "url", url: "https://example.com/path/to/document.pdf", mime_type: "application/pdf" },
  ],
});

// From base64 data
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this document." },
    {
      type: "file",
      source_type: "base64",
      data: "AAAAIGZ0eXBtcDQyAAAAAGlzb21tcDQyAAACAGlzb2...",
      mime_type: "application/pdf",
    },
  ],
});

// From provider-managed File ID
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this document." },
    { type: "file", source_type: "id", id: "file-abc123" },
  ],
});
```

**音频输入**

```typescript
// From base64 data
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this audio." },
    {
      type: "audio",
      source_type: "base64",
      data: "AAAAIGZ0eXBtcDQyAAAAAGlzb21tcDQyAAACAGlzb2...",
    },
  ],
});

// From provider-managed File ID
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this audio." },
    { type: "audio", source_type: "id", id: "file-abc123" },
  ],
});
```

**视频输入**

```typescript
// From base64 data
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this video." },
    {
      type: "video",
      source_type: "base64",
      data: "AAAAIGZ0eXBtcDQyAAAAAGlzb21tcDQyAAACAGlzb2...",
    },
  ],
});

// From provider-managed File ID
const message = new HumanMessage({
  content: [
    { type: "text", text: "Describe the content of this video." },
    { type: "video", source_type: "id", id: "file-abc123" },
  ],
});
```

> 【警告】并非所有模型都支持所有文件类型。请查看该模型 provider 的[参考文档](https://reference.langchain.com/python/integrations/)，了解支持的格式和大小限制。

### 内容块参考

内容块（无论是在创建消息时，还是访问 `contentBlocks` 字段时）表示为类型化对象列表。列表中的每一项都必须遵循以下块类型之一：

#### 核心（Core）

##### ContentBlock.Text

**用途：** 标准文本输出

- `type`（string，必填）：始终为 `"text"`
- `text`（string，必填）：文本内容
- `annotations`（Citation[]）：文本的注释列表

**示例：**

```typescript
{
    type: "text",
    text: "Hello world",
    annotations: []
}
```

##### ContentBlock.Reasoning

**用途：** 模型推理步骤

- `type`（string，必填）：始终为 `"reasoning"`
- `reasoning`（string，必填）：推理内容

**示例：**

```typescript
{
    type: "reasoning",
    reasoning: "The user is asking about..."
}
```

#### 多模态（Multimodal）

##### ContentBlock.Multimodal.Image

**用途：** 图像数据

- `type`（string，必填）：始终为 `"image"`
- `url`（string）：指向图像位置的 URL。
- `data`（string）：Base64 编码的图像数据。
- `fileId`（string）：对外部文件存储系统（例如 OpenAI 或 Anthropic 的 Files API）中图像的引用。
- `mimeType`（string）：图像 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#image)（例如 `image/jpeg`、`image/png`）。使用 base64 数据时必填。

##### ContentBlock.Multimodal.Audio

**用途：** 音频数据

- `type`（string，必填）：始终为 `"audio"`
- `url`（string）：指向音频位置的 URL。
- `data`（string）：Base64 编码的音频数据。
- `fileId`（string）：对外部文件存储系统（例如 OpenAI 或 Anthropic 的 Files API）中音频文件的引用。
- `mimeType`（string）：音频 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#audio)（例如 `audio/mpeg`、`audio/wav`）。使用 base64 数据时必填。

##### ContentBlock.Multimodal.Video

**用途：** 视频数据

- `type`（string，必填）：始终为 `"video"`
- `url`（string）：指向视频位置的 URL。
- `data`（string）：Base64 编码的视频数据。
- `fileId`（string）：对外部文件存储系统（例如 OpenAI 或 Anthropic 的 Files API）中视频文件的引用。
- `mimeType`（string）：视频 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#video)（例如 `video/mp4`、`video/webm`）。使用 base64 数据时必填。

##### ContentBlock.Multimodal.File

**用途：** 通用文件（PDF 等）

- `type`（string，必填）：始终为 `"file"`
- `url`（string）：指向文件位置的 URL。
- `data`（string）：Base64 编码的文件数据。
- `fileId`（string）：对外部文件存储系统（例如 OpenAI 或 Anthropic 的 Files API）中文件的引用。
- `mimeType`（string）：文件 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml)（例如 `application/pdf`）。使用 base64 数据时必填。

##### ContentBlock.Multimodal.PlainText

**用途：** 文档文本（`.txt`、`.md`）

- `type`（string，必填）：始终为 `"text-plain"`
- `text`（string，必填）：文本内容
- `title`（string）：文本内容的标题
- `mimeType`（string）：文本的 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml)（例如 `text/plain`、`text/markdown`）

#### 工具调用（Tool Calling）

##### ContentBlock.Tools.ToolCall

**用途：** 函数调用

- `type`（string，必填）：始终为 `"tool_call"`
- `name`（string，必填）：要调用的工具名称
- `args`（object，必填）：要传给工具的参数
- `id`（string，必填）：此工具调用的唯一标识符

**示例：**

```typescript
{
    type: "tool_call",
    name: "search",
    args: { query: "weather" },
    id: "call_123"
}
```

##### ContentBlock.Tools.ToolCallChunk

**用途：** 流式工具片段

- `type`（string，必填）：始终为 `"tool_call_chunk"`
- `name`（string）：被调用工具的名称
- `args`（string）：部分工具参数（可能是未完成的 JSON）
- `id`（string）：工具调用标识符
- `index`（number | string，必填）：此分块在流中的位置

##### ContentBlock.Tools.InvalidToolCall

**用途：** 格式错误的调用

- `type`（string，必填）：始终为 `"invalid_tool_call"`
- `name`（string）：调用失败的工具名称
- `args`（string）：解析失败的原始参数
- `error`（string，必填）：出错原因的描述

**常见错误：** 无效的 JSON、缺少必填字段

#### 服务端工具执行（Server-Side Tool Execution）

##### ContentBlock.Tools.ServerToolCall

**用途：** 在服务端执行的工具调用。

- `type`（string，必填）：始终为 `"server_tool_call"`
- `id`（string，必填）：与该工具调用关联的标识符。
- `name`（string，必填）：要调用的工具名称。
- `args`（string，必填）：部分工具参数（可能是未完成的 JSON）

##### ContentBlock.Tools.ServerToolCallChunk

**用途：** 流式服务端工具调用片段

- `type`（string，必填）：始终为 `"server_tool_call_chunk"`
- `id`（string）：与该工具调用关联的标识符。
- `name`（string）：被调用工具的名称
- `args`（string）：部分工具参数（可能是未完成的 JSON）
- `index`（number | string）：此分块在流中的位置

##### ContentBlock.Tools.ServerToolResult

**用途：** 搜索结果

- `type`（string，必填）：始终为 `"server_tool_result"`
- `tool_call_id`（string，必填）：对应服务端工具调用的标识符。
- `id`（string）：与服务端工具结果关联的标识符。
- `status`（string，必填）：服务端工具的执行状态。`"success"` 或 `"error"`。
- `output`：已执行工具的输出。

#### Provider 特定块（Provider-Specific Blocks）

##### ContentBlock.NonStandard

**用途：** provider 特定的逃生舱

- `type`（string，必填）：始终为 `"non_standard"`
- `value`（object，必填）：provider 特定的数据结构

**用法：** 用于实验性或 provider 独有的特性

更多 provider 特定的内容类型可在各模型 provider 的[参考文档](/oss/javascript/integrations/providers/overview)中找到。

上面提到的每个内容块在导入 [`ContentBlock`](https://reference.langchain.com/javascript/langchain-core/messages/ContentBlock) 类型时，都可以作为类型单独引用。

```typescript
import { ContentBlock } from "langchain";

// Text block
const textBlock: ContentBlock.Text = {
    type: "text",
    text: "Hello world",
}

// Image block
const imageBlock: ContentBlock.Multimodal.Image = {
    type: "image",
    url: "https://example.com/image.png",
    mimeType: "image/png",
}
```

> 【提示】在 [API 参考](https://reference.langchain.com/javascript/modules/_langchain_core.messages.html)中查看规范的类型定义。

> 【信息】内容块是在 LangChain v1 中作为消息的新属性引入的，用于在保持与现有代码向后兼容的同时，统一各 provider 的内容格式。
>
> 内容块并不是 [`content`](https://reference.langchain.com/javascript/langchain-core/messages/BaseMessage) 属性的替代品，而是一个新属性，可用于以标准化格式访问消息内容。

## 序列化

你可以将消息序列化为普通对象以便存储，并反序列化回消息实例。这对于持久化对话历史和恢复会话很有用。

```ts
import { HumanMessage } from "@langchain/core/messages";
import { load } from "@langchain/core/load";

const message = new HumanMessage("What is the capital of France?");

// Serialize to a plain object
const serialized = message.toJSON();

// Deserialize back to a message object
const restored = await load<HumanMessage>(JSON.stringify(serialized));
```

> 【警告】**`load()` 通过实例化类并调用构造函数来反序列化数据。切勿对不受信任或用户提供的输入调用 `load()`。** 只反序列化来自你可控来源的数据，例如你自己的数据库。

## 与聊天模型一起使用

[聊天模型](/oss/javascript/langchain/models)接受一组消息对象作为输入，并返回一条 [`AIMessage`](https://reference.langchain.com/javascript/langchain-core/messages/AIMessage) 作为输出。交互通常是无状态的，因此一个简单的对话循环会涉及用不断增长的消息列表调用模型。

请参阅以下指南以了解更多：

* 用于[持久化和管理对话历史](/oss/javascript/langchain/short-term-memory)的内置特性
* 管理上下文窗口的策略，包括[裁剪和摘要消息](/oss/javascript/langchain/short-term-memory#common-patterns)

---

> 【信息】将这些文档连接到 Claude、VSCode 等，通过 MCP 获得实时答案：[连接这些文档](/use-these-docs)。
>
> 在 GitHub 上[编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/messages.mdx)或[提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。

