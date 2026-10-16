# 模型上下文协议（MCP）

> 原文：https://docs.langchain.com/oss/javascript/langchain/mcp
> LangChain (TypeScript) 官方文档中文整理

> **文档索引**
> 获取完整文档索引：https://docs.langchain.com/llms.txt
> 使用该文件可在进一步探索前发现所有可用页面。

> 针对 `langchain-mcp-adapters` 包的旧版 MCP 文档。

[模型上下文协议（MCP）](https://modelcontextprotocol.io/introduction) 是一种开放协议，它标准化了应用程序如何向 LLM 提供工具和上下文。LangChain 智能体可以使用 [`@langchain/mcp-adapters`](https://github.com/langchain-ai/langchainjs/tree/main/libs/langchain-mcp-adapters) 库来使用定义在 MCP 服务器上的工具。

## 快速开始

安装 `@langchain/mcp-adapters` 库：

#### npm

```bash
npm install @langchain/mcp-adapters
```

#### pnpm

```bash
pnpm add @langchain/mcp-adapters
```

#### yarn

```bash
yarn add @langchain/mcp-adapters
```

#### bun

```bash
bun add @langchain/mcp-adapters
```

`@langchain/mcp-adapters` 使智能体能够使用跨一个或多个 MCP 服务器定义的工具。

> **【提示】** `MultiServerMCPClient` **默认是无状态的**。每次工具调用都会创建一个全新的 MCP `ClientSession`，执行工具，然后进行清理。

#### 访问多个 MCP 服务器

```ts
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { ChatAnthropic } from "@langchain/anthropic";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({
    math: {
        transport: "stdio",  // Local subprocess communication
        command: "node",
        // Replace with absolute path to your math_server.js file
        args: ["/path/to/math_server.js"],
    },
    weather: {
        transport: "http",  // HTTP-based remote server
        // Ensure you start your weather server on port 8000
        url: "http://localhost:8000/mcp",
    },
});

const tools = await client.getTools();
const agent = createAgent({
    model: "claude-sonnet-5",
    tools,
});

const mathResponse = await agent.invoke({
    messages: [{ role: "user", content: "what's (3 + 5) x 12?" }],
});

const weatherResponse = await agent.invoke({
    messages: [{ role: "user", content: "what is the weather in nyc?" }],
});
```

> **【提示】** 使用 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-javascript-langchain-mcp) 将 MCP 工具调用与智能体的推理步骤一起追踪。按照[追踪快速开始](/langsmith/trace-with-langchain)进行设置。

#### 示例：查询 LangChain 文档

[LangChain 文档 MCP 服务器](/use-these-docs) 是一个位于 `https://docs.langchain.com/mcp` 的公共 HTTP 端点。将智能体连接到它即可搜索和阅读文档，而无需编写自定义工具。

> **【提示】** 文档 MCP 服务器是公开的，不需要 API 密钥。有关 IDE 和编码智能体的设置（Claude Code、Cursor 等），请参阅[以编程方式使用文档](/use-these-docs)。

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({
    docs: {
        transport: "http",
        url: "https://docs.langchain.com/mcp",
    },
});

const tools = await client.getTools();
const agent = createAgent({
    model: "claude-sonnet-5",
    tools,
});

const response = await agent.invoke({
    messages: [
        {
            role: "user",
            content: "How do I add short-term memory to a LangChain agent?",
        },
    ],
});
```

该服务器公开以下工具：

| 工具                                       | 描述                                                                                   |
| ------------------------------------------ | --------------------------------------------------------------------------------------------- |
| `search_docs_by_lang_chain`                | 搜索文档以查找相关的指南、操作说明和示例。                                       |
| `query_docs_filesystem_docs_by_lang_chain` | 通过虚拟文件系统（`rg`、`head`、`cat` 及相关命令）读取或搜索文档。 |
| `submit_feedback`                          | 报告文档页面的问题。                                                   |

## 自定义服务器

要创建自己的 MCP 服务器，可以使用 `@modelcontextprotocol/sdk` 库。该库提供了一种简单的方式来定义[工具](https://modelcontextprotocol.io/docs/learn/server-concepts#tools-ai-actions)并将其作为服务器运行。

#### npm

```bash
npm install @modelcontextprotocol/sdk
```

#### pnpm

```bash
pnpm add @modelcontextprotocol/sdk
```

#### yarn

```bash
yarn add @modelcontextprotocol/sdk
```

#### bun

```bash
bun add @modelcontextprotocol/sdk
```

要使用 MCP 工具服务器测试你的智能体，请使用以下示例：

#### 数学服务器（stdio 传输）

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
    CallToolRequestSchema,
    ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
    {
        name: "math-server",
        version: "0.1.0",
    },
    {
        capabilities: {
            tools: {},
        },
    }
);

server.setRequestHandler(ListToolsRequestSchema, async () => {
    return {
        tools: [
        {
            name: "add",
            description: "Add two numbers",
            inputSchema: {
                type: "object",
                properties: {
                    a: {
                        type: "number",
                        description: "First number",
                    },
                    b: {
                        type: "number",
                        description: "Second number",
                    },
                },
                required: ["a", "b"],
            },
        },
        {
            name: "multiply",
            description: "Multiply two numbers",
            inputSchema: {
                type: "object",
                properties: {
                    a: {
                        type: "number",
                        description: "First number",
                    },
                    b: {
                        type: "number",
                        description: "Second number",
                    },
                },
                required: ["a", "b"],
            },
        },
        ],
    };
});

server.setRequestHandler(CallToolRequestSchema, async (request) => {
    switch (request.params.name) {
        case "add": {
            const { a, b } = request.params.arguments as { a: number; b: number };
            return {
                content: [
                {
                    type: "text",
                    text: String(a + b),
                },
                ],
            };
        }
        case "multiply": {
            const { a, b } = request.params.arguments as { a: number; b: number };
            return {
                content: [
                {
                    type: "text",
                    text: String(a * b),
                },
                ],
            };
        }
        default:
            throw new Error(`Unknown tool: ${request.params.name}`);
    }
});

async function main() {
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.error("Math MCP server running on stdio");
}

main();
```

#### 天气服务器（SSE 传输）

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import {
    CallToolRequestSchema,
    ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import express from "express";

const app = express();
app.use(express.json());

const server = new Server(
    {
        name: "weather-server",
        version: "0.1.0",
    },
    {
        capabilities: {
            tools: {},
        },
    }
);

server.setRequestHandler(ListToolsRequestSchema, async () => {
    return {
        tools: [
        {
            name: "get_weather",
            description: "Get weather for location",
            inputSchema: {
            type: "object",
            properties: {
                location: {
                type: "string",
                description: "Location to get weather for",
                },
            },
            required: ["location"],
            },
        },
        ],
    };
});

server.setRequestHandler(CallToolRequestSchema, async (request) => {
    switch (request.params.name) {
        case "get_weather": {
            const { location } = request.params.arguments as { location: string };
            return {
                content: [
                    {
                        type: "text",
                        text: `It's always sunny in ${location}`,
                    },
                ],
            };
        }
        default:
            throw new Error(`Unknown tool: ${request.params.name}`);
    }
});

app.post("/mcp", async (req, res) => {
    const transport = new SSEServerTransport("/mcp", res);
    await server.connect(transport);
});

const PORT = process.env.PORT || 8000;
app.listen(PORT, () => {
    console.log(`Weather MCP server running on port ${PORT}`);
});
```

## 传输方式

MCP 支持不同的客户端-服务器通信传输机制。

### HTTP

`http` 传输（也称为 `streamable-http`）使用 HTTP 请求进行客户端-服务器通信。有关更多详细信息，请参阅 [MCP HTTP 传输规范](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#streamable-http)。

对于你自己运行的服务器，使用本地 URL；或使用托管的 URL，例如 [LangChain 文档 MCP 服务器](/use-these-docs)（`https://docs.langchain.com/mcp`），它是公开的，不需要 API 密钥。

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({
    mcp: {
        transport: "http",
        // url: "http://localhost:8000/mcp", // Local server
        url: "https://docs.langchain.com/mcp", // Hosted server
    },
});

const tools = await client.getTools();
const agent = createAgent({ model: "openai:gpt-5.4", tools });
const response = await agent.invoke({
    messages: [
        {
            role: "user",
            content: "How do I connect LangChain to an MCP server over HTTP?",
        },
    ],
});
```

#### 传递请求头

通过 HTTP 连接 MCP 服务器时，你可以在连接配置中使用 `headers` 字段来包含自定义请求头（例如用于身份验证或追踪）。本示例使用 [LangChain 文档 MCP 服务器](/use-these-docs)；请为需要身份验证的服务器替换请求头值：

#### 使用 MultiServerMCPClient 传递请求头

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({
    mcp: {
        transport: "http",
        url: "https://docs.langchain.com/mcp",
        headers: {
            Authorization: "Bearer YOUR_TOKEN",
            "X-Custom-Header": "custom-value",
        },
    },
});

const tools = await client.getTools();
const agent = createAgent({ model: "openai:gpt-5.4", tools });
const response = await agent.invoke({
    messages: [
        {
            role: "user",
            content: "How do I connect LangChain to an MCP server over HTTP?",
        },
    ],
});
```

#### 身份验证

`@langchain/mcp-adapters` 库底层使用官方的 [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)，它允许你通过实现 `OAuthClientProvider` 接口来提供自定义身份验证机制。

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";

const client = new MultiServerMCPClient({
    weather: {
        transport: "http",
        url: "http://localhost:8000/mcp",
        authProvider: authProvider,
    },
});
```

* [OAuth 客户端文档](https://github.com/modelcontextprotocol/typescript-sdk/blob/main/docs/clients/oauth.md)
* [OAuth 示例](https://github.com/modelcontextprotocol/typescript-sdk/blob/main/examples/oauth/README.md)

### stdio

客户端将服务器作为子进程启动，并通过标准输入/输出进行通信。最适合本地工具和简单设置。

```typescript
const client = new MultiServerMCPClient({
    math: {
        transport: "stdio",
        command: "node",
        args: ["/path/to/math_server.js"],
    },
});
```

## 核心功能

### 工具

[工具](https://modelcontextprotocol.io/docs/concepts/tools) 允许 MCP 服务器公开可执行函数，LLM 可以调用这些函数来执行操作——例如查询数据库、调用 API 或与外部系统交互。LangChain 将 MCP 工具转换为 LangChain [工具](/oss/javascript/langchain/tools)，使其可直接用于任何 LangChain 智能体或工作流。

#### 加载工具

使用 `client.getTools()` 从 MCP 服务器检索工具并将其传递给你的智能体：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({...});
const tools = await client.getTools();
const agent = createAgent({ model: "claude-sonnet-5", tools });
```

当 MCP 工具执行失败（`CallToolResult` 的 `isError: true`）时，`@langchain/mcp-adapters` 会抛出 `ToolException`。请将工具调用包裹在 try/catch 中以处理这些错误。与 Python 适配器不同，TypeScript 适配器不会将错误作为失败的工具消息返回给模型。

#### 结构化内容

MCP 工具可以在人类可读的文本响应之外返回[结构化内容](https://modelcontextprotocol.io/specification/2025-03-26/server/tools#structured-content)。当工具需要在向模型展示的文本之外返回机器可解析的数据（如 JSON）时，这很有用。

当 MCP 工具返回 `structuredContent` 时，适配器会在工具消息的 `artifact` 数组中添加一个 `mcp_structured_content` 条目。你也可以使用[工具拦截器](#tool-interceptors)来自动处理或转换结构化内容。

**从 artifact 中提取结构化内容**

调用智能体后，你可以从响应中的工具消息访问结构化内容：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";
import { createAgent } from "langchain";

const client = new MultiServerMCPClient({...});
const tools = await client.getTools();
const agent = createAgent({ model: "claude-sonnet-5", tools });

const result = await agent.invoke({
    messages: [{ role: "user", content: "Get data from the server" }],
});

for (const message of result.messages) {
    if (message.type !== "tool" || !message.artifact) continue;
    for (const item of message.artifact) {
        if (item?.type === "mcp_structured_content") {
            console.log(item.data);
        }
    }
}
```

#### 多模态工具内容

MCP 工具可以在其响应中返回[多模态内容](https://modelcontextprotocol.io/specification/2025-03-26/server/tools#tool-result)（图像、文本等）。当 MCP 服务器返回包含多个部分的内容（例如文本和图像）时，适配器会将它们转换为 LangChain 的[标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)。你可以通过 `ToolMessage` 上的 `contentBlocks` 属性访问标准化表示：

#### Google

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "google-genai:gemini-3.6-flash", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### OpenAI

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "openai:gpt-5.5", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### Anthropic

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "anthropic:claude-sonnet-4-6", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### OpenRouter

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "openrouter:openrouter:z-ai/glm-5.2", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### Fireworks

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "fireworks:accounts/fireworks/models/glm-5p2", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### Baseten

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "baseten:zai-org/GLM-5.2", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

#### Ollama

```ts
import { createAgent } from "langchain";

async function accessMultimodalToolContent(): Promise<void> {
  const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");
  const client = new MultiServerMCPClient({});
  const tools = await client.getTools();
  const agent = createAgent({ model: "ollama:north-mini-code-1.0", tools });

  const result = await agent.invoke({
    messages: [
      { role: "user", content: "Take a screenshot of the current page" },
    ],
  });

  // Access multimodal content from tool messages
  for (const message of result.messages) {
    if (message.type === "tool") {
      // Raw content in provider-native format
      console.log(`Raw content: ${message.content}`);

      // Standardized content blocks
      for (const block of message.contentBlocks) {
        if (block.type === "text") {
          console.log(`Text: ${block.text}`);
        } else if (block.type === "image") {
          console.log(`Image URL: ${block.url}`);
          console.log(`Image base64: ${block.base64?.slice(0, 50)}...`);
        }
      }
    }
  }
}
```

这使你能够以与提供商无关的方式处理多模态工具响应，无论底层 MCP 服务器如何格式化其内容。

### 资源

[资源](https://modelcontextprotocol.io/docs/concepts/resources) 允许 MCP 服务器公开客户端可以读取的数据——例如文件、数据库记录或 API 响应。

#### 加载资源

使用 `client.listResources()` 发现资源，使用 `client.readResource()` 读取其内容：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";

const client = new MultiServerMCPClient({...});

// List resources from a server
const resourcesByServer = await client.listResources("server_name");
for (const resource of resourcesByServer["server_name"] ?? []) {
    console.log(`URI: ${resource.uri}, MIME type: ${resource.mimeType}`);
}

// Read a specific resource by URI
const contents = await client.readResource(
    "server_name",
    "file:///path/to/file.txt",
);
for (const content of contents) {
    console.log(`URI: ${content.uri}, MIME type: ${content.mimeType}`);
    if (content.text) console.log(content.text);
}
```

## 高级功能

### 工具拦截器

MCP 服务器作为独立进程运行——它们无法访问 LangGraph 运行时信息，如[存储](/oss/javascript/langgraph/stores)、[上下文](/oss/javascript/langchain/context-engineering)或智能体状态。使用 `MultiServerMCPClient` 上的 `beforeToolCall` 和 `afterToolCall` 钩子来修改工具参数、请求头或结果：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";

const client = new MultiServerMCPClient({
    mcpServers: {
        math: {
            transport: "stdio",
            command: "npx",
            args: ["-y", "@modelcontextprotocol/server-math"],
        },
    },
    beforeToolCall: ({ serverName, name, args }) => {
        const nextArgs = { ...(args as Record<string, unknown>), injected: true };
        return {
            args: nextArgs,
            headers: { "X-Request-ID": crypto.randomUUID() },
        };
    },
    afterToolCall: (res) => {
        if (res.name === "someTool") {
            return { result: ["modified-output", []] };
        }
        return { result: res.result };
    },
});

const tools = await client.getTools();
```

* **beforeToolCall**：可以返回 `{ args?, headers? }`。HTTP 和 SSE 传输支持请求头。Stdio 连接不支持自定义请求头。
* **afterToolCall**：可以返回 `{ result }`，其中 `result` 是一个 `[content, artifact]` 元组、一个 `ToolMessage`、一个 LangGraph `Command`，或原始结果。

### 进度通知

使用 `onProgress` 订阅长时间运行的工具执行的进度更新：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";

const client = new MultiServerMCPClient({
    mcpServers: {
        everything: {
            transport: "stdio",
            command: "npx",
            args: ["-y", "@modelcontextprotocol/server-everything"],
        },
    },
    onProgress: (progress, source) => {
        const pct =
            progress.percentage ??
            (progress.progress != null && progress.total
                ? Math.round((progress.progress / progress.total) * 100)
                : undefined);
        if (pct == null) return;
        const origin =
            source.type === "tool" ? `${source.server}/${source.name}` : "unknown";
        console.log(`[progress:${origin}] ${pct}%`);
    },
});

const tools = await client.getTools();
```

### 日志

MCP 协议支持来自服务器的[日志](https://modelcontextprotocol.io/specification/2025-03-26/server/utilities/logging#log-levels)通知。使用 `onMessage` 订阅：

```typescript
import { MultiServerMCPClient } from "@langchain/mcp-adapters";

const client = new MultiServerMCPClient({
    mcpServers: {
        everything: {
            transport: "stdio",
            command: "npx",
            args: ["-y", "@modelcontextprotocol/server-everything"],
        },
    },
    onMessage: (log, source) => {
        console.log(`[${source.server}] ${log.level}: ${log.data}`);
    },
});

const tools = await client.getTools();
```

你还可以使用 `client.setLoggingLevel("debug")` 或 `client.setLoggingLevel("server_name", "debug")` 设置服务器日志级别。

## 其他资源

* [MCP 文档](https://modelcontextprotocol.io/introduction)

* [MCP 传输文档](https://modelcontextprotocol.io/docs/concepts/transports)

* [`@langchain/mcp-adapters`](https://github.com/langchain-ai/langchainjs/tree/main/libs/langchain-mcp-adapters/)

***

> **【信息】** [通过 MCP 将这些文档连接](/use-these-docs)到 Claude、VSCode 等，以获得实时答案。
>
> **【信息】** [在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/mcp.mdx)或[提交问题](https://github.com/langchain-ai/docs/issues/new/choose)。
