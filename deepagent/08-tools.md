# 工具（Tools）

> 原文：Tools
> 来源：https://docs.langchain.com/oss/javascript/deepagents/tools

---

> 将 Deep Agents 连接到自定义函数、API、数据库以及任何 MCP server

Deep Agents 可以调用你定义的任何 tool、任何 [LangChain tool](https://python.langchain.com/docs/concepts/tools/)，以及来自任何 [MCP server](#mcp-tools) 的 tools。
通过 `tools=` 参数将它们与用于文件管理和 subagent 派生的[内置 harness tools](/oss/javascript/deepagents/overview#execution-environment)一起传给 `create_deep_agent`。

#### Google

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [search, fetchUrl, runQuery],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  tools: [search, fetchUrl, runQuery],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools: [search, fetchUrl, runQuery],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools: [search, fetchUrl, runQuery],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [search, fetchUrl, runQuery],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [search, fetchUrl, runQuery],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";


const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [search, fetchUrl, runQuery],
});
```

## 自定义 tools

将任何可调用对象直接传给 `tools=`，例如普通函数、LangChain `@tool` 装饰的函数，或 tool dicts。
Deep Agents 会从函数签名和 docstring 推断 tool schema，因此在大多数情况下你无需定义单独的 schema。

#### Google

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [internetSearch],
});
```

#### OpenAI

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  tools: [internetSearch],
});
```

#### Anthropic

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools: [internetSearch],
});
```

#### OpenRouter

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools: [internetSearch],
});
```

#### Fireworks

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [internetSearch],
});
```

#### Baseten

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [internetSearch],
});
```

#### Ollama

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const internetSearch = tool(
  async ({
    query,
    maxResults = 5,
    topic = "general",
    includeRawContent = false,
  }: {
    query: string;
    maxResults?: number;
    topic?: "general" | "news" | "finance";
    includeRawContent?: boolean;
  }) => {
    const tavilySearch = new TavilySearch({
      maxResults,
      tavilyApiKey: process.env.TAVILY_API_KEY,
      includeRawContent,
      topic,
    });
    return await tavilySearch._call({ query });
  },
  {
    name: "internet_search",
    description: "Run a web search",
    schema: z.object({
      query: z.string().describe("The search query"),
      maxResults: z.number().optional().default(5),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general"),
      includeRawContent: z.boolean().optional().default(false),
    }),
  },
);

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [internetSearch],
});
```

关于定义和使用 LangChain tools 的完整细节（tool dicts、`StructuredTool`、返回类型、错误处理等），请参阅 [Tools](/oss/javascript/langchain/tools)。

## MCP tools

> **注意**：Deep Agents 完全支持 [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp)，这是将 agents 连接到外部服务的开放标准。从任何 MCP server 加载 tools 并直接传给 `create_deep_agent`。

MCP 是一种开放协议，让 agents 通过标准接口连接到一个不断增长的 server 生态——数据库、API、文件系统、浏览器等等。无需为每个服务编写自定义集成代码，你只需将 Deep Agents 指向一个 MCP server，它就会获得该 server 暴露的所有 tools。

安装 `@langchain/mcp-adapters` 以连接到 MCP servers：

```bash
npm install @langchain/mcp-adapters
```

#### Google

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const { MultiServerMCPClient } = await import("@langchain/mcp-adapters");

const client = new MultiServerMCPClient({
  my_server: {
    transport: "http",
    url: "http://localhost:8000/mcp",
  },
});

const tools = await client.getTools();

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools,
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "Use the MCP server to help me." }],
});
```

关于详细的配置选项——包括 stdio servers、OAuth authentication、tool filtering 和有状态会话——请参阅完整的 [MCP guide](/oss/javascript/langchain/mcp)。

## 内置 harness tools

除了你提供的 tools 之外，每个 Deep Agent 都随附来自 harness 的一组内置 tools：

| Tool         | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `ls`         | 列出目录中的文件。                                   |
| `read_file`  | 读取文件内容（支持分页和多模态）。 |
| `write_file` | 创建新文件。                                            |
| `edit_file`  | 在文件中执行精确的字符串替换。                  |
| `glob`       | 查找匹配 glob 模式的文件。                          |
| `grep`       | 搜索文件内容。                                        |
| `execute`    | 运行 shell 命令（仅限 sandbox backends）。                 |
| `task`       | 派生一个 subagent 来处理被委派的任务。                 |

要添加基于 `write_todos` 的结构化任务规划，请通过 [`TodoListMiddleware`](https://reference.langchain.com/javascript/langchain/index/todoListMiddleware) 启用。参阅 [Task planning](/oss/javascript/deepagents/overview#task-planning)。

关于每个内置 tool 具体做什么的完整拆解，请参阅 [Harness overview](/oss/javascript/deepagents/overview#execution-environment)。

## 多模态 tool 输出

当所选模型支持多模态 tool 结果时，自定义 tools 可以返回纯文本或[标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)（文本、图像、音频、视频和文件）。内置的 `read_file` tool 对受支持的非文本文件类型也会返回多模态块。

对于纯文本结果，返回一个字符串；对于文本加媒体或交错的多模态输出，返回一个有序的内容块列表。示例和上下文压缩注意事项，请参阅 [Multimodal](/oss/javascript/deepagents/multimodal) 和 [Tool return values](/oss/javascript/langchain/tools#return-multimodal-content)。

***
