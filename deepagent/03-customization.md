# 自定义（Customization）

> 原文：Customize Deep Agents
> 来源：https://docs.langchain.com/oss/javascript/deepagents/customization

---

> 了解如何使用 system prompts、tools、subagents 等来定制 Deep Agents

围绕你的目标来构建 harness。`create_deep_agent` 为你提供一个生产就绪的基础：把它连接到你的数据、塑造它的行为，并按你的用例需求添加能力。

`createDeepAgent` 自带一套预先组装好的 harness：默认包含文件系统、摘要（summarization）、subagents 和 prompt caching。下面的参数让你定义 agent 的人设、把它连接到你的数据和工具，并通过额外的 middleware 扩展 [Deep Agents stack](#deep-agents-stack)。

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt: "You are a helpful assistant.",
  tools: [search, fetchUrl],
  memory: ["./AGENTS.md"],
  skills: ["./skills/"],
});
```

| 参数                                                                              | 作用                                                                     |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `model`                                                                           | 使用哪个模型                                                             |
| `systemPrompt`                                                                    | 给 agent 的自定义指令                                                    |
| `tools`                                                                           | agent 可以调用的领域工具                                                 |
| `memory`                                                                          | 启动时加载的 AGENTS.md 文件                                              |
| `skills`                                                                          | 用于按需获取知识的 skills 目录                                           |
| `backend`                                                                         | 文件系统 backend（默认是 StateBackend）                                  |
| `permissions`                                                                     | 文件系统的路径级访问控制                                                 |
| `subagents`                                                                       | 用于委派任务的自定义 subagents                                           |
| `middleware`                                                                      | 追加到 [Deep Agents stack](#deep-agents-stack) 的额外 middleware         |
| `interruptOn`                                                                     | 在工具调用前暂停以等待人工批准                                           |
| `responseFormat`                                                                  | 结构化输出 schema                                                        |
| [`contextSchema`](/oss/javascript/deepagents/context-engineering#runtime-context) | 每次运行的运行时上下文 schema（用户 IDs、API keys、feature flags）      |

完整参数列表请参见 [`createDeepAgent`](https://reference.langchain.com/javascript/deepagents/types/CreateDeepAgentParams) API 参考。若要从零开始组合一个完全自定义的 harness，请参见 [Configure the harness](/oss/javascript/langchain/agents#configure-the-harness)。

> **提示**：当你添加 tools、subagents 和 backends 时，使用 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-deepagents-customization) 来追踪每一部分协同工作时的表现。按照 [observability quickstart](/langsmith/observability-quickstart) 完成设置，并查看 [Going to production](/oss/javascript/deepagents/going-to-production) 了解在 LangSmith 上的部署方式。
>
> 我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的 traces、检测问题并提出修复建议。

## Model

传入 `provider:model` 格式的 `model` 字符串，或一个已初始化的 model 实例。所有 provider 请参见 [supported models](/oss/javascript/deepagents/models#supported-models)，经过测试的建议请参见 [suggested models](/oss/javascript/deepagents/models#suggested-models)。

> **提示**：使用 `provider:model` 格式（例如 `openai:gpt-5.5`）可以在模型之间快速切换。

#### OpenAI

👉 阅读 [OpenAI chat model integration docs](/oss/javascript/integrations/chat/openai/)

##### npm

```bash
npm install @langchain/openai deepagents
```

##### pnpm

```bash
pnpm install @langchain/openai deepagents
```

##### yarn

```bash
yarn add @langchain/openai deepagents
```

##### bun

```bash
bun add @langchain/openai deepagents
```

##### default parameters

```typescript
import { createDeepAgent } from "deepagents";

process.env.OPENAI_API_KEY = "your-api-key";

const agent = createDeepAgent({ model: "gpt-5.5" });
// this calls initChatModel for the specified model with default parameters
// to use specific model parameters, use initChatModel directly
```

##### initChatModel

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

process.env.OPENAI_API_KEY = "your-api-key";

const model = await initChatModel("gpt-5.5");
const agent = createDeepAgent({
  model,
  temperature: 0,
});
```

##### Model Class

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: new ChatOpenAI({
    model: "gpt-5.5",
    apiKey: "your-api-key",
    temperature: 0,
  }),
});
```

#### Anthropic

👉 阅读 [Anthropic chat model integration docs](/oss/javascript/integrations/chat/anthropic/)

##### npm

```bash
npm install @langchain/anthropic deepagents
```

##### pnpm

```bash
pnpm install @langchain/anthropic deepagents
```

##### yarn

```bash
yarn add @langchain/anthropic deepagents
```

##### bun

```bash
bun add @langchain/anthropic deepagents
```

##### default parameters

```typescript
import { createDeepAgent } from "deepagents";

process.env.ANTHROPIC_API_KEY = "your-api-key";

const agent = createDeepAgent({ model: "anthropic:claude-sonnet-4-6" });
// this calls initChatModel for the specified model with default parameters
// to use specific model parameters, use initChatModel directly
```

##### initChatModel

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

process.env.ANTHROPIC_API_KEY = "your-api-key";

const model = await initChatModel("claude-sonnet-4-6");
const agent = createDeepAgent({
  model,
  temperature: 0,
});
```

##### Model Class

```typescript
import { ChatAnthropic } from "@langchain/anthropic";
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: new ChatAnthropic({
    model: "claude-sonnet-4-6",
    apiKey: "your-api-key",
    temperature: 0,
  }),
});
```

#### Azure

👉 阅读 [Azure chat model integration docs](/oss/javascript/integrations/chat/azure/)

##### npm

```bash
npm install @langchain/azure deepagents
```

##### pnpm

```bash
pnpm install @langchain/azure deepagents
```

##### yarn

```bash
yarn add @langchain/azure deepagents
```

##### bun

```bash
bun add @langchain/azure deepagents
```

##### default parameters

```typescript
import { createDeepAgent } from "deepagents";

process.env.AZURE_OPENAI_API_KEY = "your-api-key";
process.env.AZURE_OPENAI_ENDPOINT = "your-endpoint";
process.env.OPENAI_API_VERSION = "your-api-version";

const agent = createDeepAgent({ model: "azure_openai:gpt-5.5" });
// this calls initChatModel for the specified model with default parameters
// to use specific model parameters, use initChatModel directly
```

##### initChatModel

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

process.env.AZURE_OPENAI_API_KEY = "your-api-key";
process.env.AZURE_OPENAI_ENDPOINT = "your-endpoint";
process.env.OPENAI_API_VERSION = "your-api-version";

const model = await initChatModel("azure_openai:gpt-5.5");
const agent = createDeepAgent({
  model,
  temperature: 0,
});
```

##### Model Class

```typescript
import { AzureChatOpenAI } from "@langchain/openai";
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: new AzureChatOpenAI({
    model: "gpt-5.5",
    azureOpenAIApiKey: "your-api-key",
    azureOpenAIApiEndpoint: "your-endpoint",
    azureOpenAIApiVersion: "your-api-version",
    temperature: 0,
  }),
});
```

#### Google Gemini

👉 阅读 [ChatGoogle chat model integration docs](/oss/javascript/integrations/chat/google)

##### npm

```bash
npm install @langchain/google deepagents
```

##### pnpm

```bash
pnpm install @langchain/google deepagents
```

##### yarn

```bash
yarn add @langchain/google deepagents
```

##### bun

```bash
bun add @langchain/google deepagents
```

##### default parameters

```typescript
import { createDeepAgent } from "deepagents";

process.env.GOOGLE_API_KEY = "your-api-key";

const agent = createDeepAgent({ model: "google:gemini-3.1-pro-preview" });
// this calls initChatModel for the specified model with default parameters
// to use specific model parameters, use initChatModel directly
```

##### initChatModel

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

process.env.GOOGLE_API_KEY = "your-api-key";

const model = await initChatModel("google:gemini-3.1-pro-preview");
const agent = createDeepAgent({
  model,
  temperature: 0,
});
```

##### Model Class

```typescript
import { ChatGoogle } from "@langchain/google/node";
import { createDeepAgent } from "deepagents";

const agent = createDeepAgent({
  model: new ChatGoogle({
    model: "gemini-3.1-pro-preview",
    apiKey: "your-api-key",
    temperature: 0,
  }),
});
```

#### Bedrock Converse

👉 阅读 [AWS Bedrock chat model integration docs](/oss/javascript/integrations/chat/bedrock_converse/)

##### npm

```bash
npm install @langchain/aws deepagents
```

##### pnpm

```bash
pnpm install @langchain/aws deepagents
```

##### yarn

```bash
yarn add @langchain/aws deepagents
```

##### bun

```bash
bun add @langchain/aws deepagents
```

##### default parameters

```typescript
import { createDeepAgent } from "deepagents";

// Follow the steps here to configure your credentials:
// https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started.html

const agent = createDeepAgent({ model: "bedrock:anthropic.claude-sonnet-4-6" });
// this calls initChatModel for the specified model with default parameters
// to use specific model parameters, use initChatModel directly
```

##### initChatModel

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

// Follow the steps here to configure your credentials:
// https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started.html

const model = await initChatModel("bedrock:anthropic.claude-sonnet-4-6");
const agent = createDeepAgent({
  model,
  temperature: 0,
});
```

##### Model Class

```typescript
import { ChatBedrockConverse } from "@langchain/aws";
import { createDeepAgent } from "deepagents";

// Follow the steps here to configure your credentials:
// https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started.html

const agent = createDeepAgent({
  model: new ChatBedrockConverse({
    model: "anthropic.claude-sonnet-4-6",
    region: "us-east-2",
    temperature: 0,
  }),
});
```

#### Other

传入任意 [supported model string](/oss/javascript/deepagents/models#supported-models)，或一个已初始化的 model 实例：

```typescript
import { initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";

const model = await initChatModel("provider:model-name");
const agent = createDeepAgent({ model });
```

> **提示**：Chat models 会自动重试瞬时的 API 失败（采用指数退避）。关于 `max_retries` / `timeout` 调优的默认值、限制和代码示例，请见 LangChain 的 [Models](/oss/javascript/langchain/models#connection-resilience) 页面。

## Tools

除了用于文件管理和 subagent 生成（spawning）的 [built-in tools](/oss/javascript/deepagents/overview#execution-environment) 之外，你还可以提供自定义工具：

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

### MCP tools

> **提示**：Deep Agents 完全支持 [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp) 工具。你可以从任何 MCP server 加载工具——数据库、APIs、文件系统等等——并把它们直接传给 `create_deep_agent`。

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

关于包括 stdio servers、OAuth 认证、tool filtering 和有状态会话在内的详细配置选项，请参见完整的 [MCP guide](/oss/javascript/langchain/mcp)。

## System prompt

传入 `system_prompt=` 来给 agent 你自己的指令：

#### Google

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt: researchInstructions,
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt: researchInstructions,
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt: researchInstructions,
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt: researchInstructions,
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt: researchInstructions,
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt: researchInstructions,
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const researchInstructions =
  `You are an expert researcher. ` +
  `Your job is to conduct thorough research, and then ` +
  `write a polished report.`;

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt: researchInstructions,
});
```

> **注意**：除了字符串之外，主 agent 也接受带有结构化 [content blocks](/oss/javascript/langchain/messages#standard-content-blocks) 的 [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage)；Deep Agents 会保留这些 blocks（[subagent](/oss/javascript/deepagents/subagents) 的字典规格仍然保持为字符串）。

#### Subagent prompts

声明式 [subagents](/oss/javascript/deepagents/subagents) 会针对它们各自的 model 解析 profile overlays，然后把解析出的 profile 的 `base_system_prompt` / `system_prompt_suffix` 应用到 subagent 编写的 `system_prompt` 上。只带有 `system_prompt_suffix` 的 profile（内置 Anthropic / OpenAI profiles 的常见情况）会追加到编写的 prompt 之后。设置了 `base_system_prompt` 的 profile 则直接替换它。

#### General-purpose subagent prompt

自动添加的 [general-purpose subagent](/oss/javascript/deepagents/subagents#the-general-purpose-subagent) 按 **`general_purpose_subagent.system_prompt`（如果设置）-> `HarnessProfile.base_system_prompt`（如果设置）-> SDK general-purpose 默认值** 的顺序解析其基础 prompt，并在其上叠加 profile suffix。当两个 override 字段都设置时，general-purpose 专属的那个优先生效，这样同时调这两个字段的调用者永远不会看到自己的 GP override 被静默丢弃：

```python
from deepagents import (
    GeneralPurposeSubagentProfile,
    HarnessProfile,
    register_harness_profile,
)

register_harness_profile(
    "anthropic",
    HarnessProfile(
        base_system_prompt="You are ACME's support orchestrator.",  # main agent
        general_purpose_subagent=GeneralPurposeSubagentProfile(
            system_prompt="You are a research subagent. Cite sources.",  # GP subagent
        ),
        system_prompt_suffix="Always think step by step.",
    ),
)
```

| 层          | 最终 system prompt                                     |
| ----------- | ------------------------------------------------------- |
| 主 agent    | `"You are ACME's support orchestrator." + SUFFIX`       |
| GP 子代理   | `"You are a research subagent. Cite sources." + SUFFIX` |

## Middleware

Deep Agents 支持任何 [middleware](/oss/javascript/langchain/middleware/overview)，包括下面列出的内置 middleware、来自 LangChain 的 prebuilt middleware、provider 专属的 middleware，以及你自己编写的自定义 middleware。

把 middleware 传给 `createDeepAgent` 的 `middleware` 参数。自定义 middleware 会被追加到 [Deep Agents stack](#deep-agents-stack) 中 [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware) 之后。

### Deep Agents stack

`createDeepAgent` 按固定顺序构建 middleware。[bare stack](#bare-stack) 是仅传入一个 model 时得到的内容。[full stack](#full-stack) 是完整的组装顺序，包含只有在传入可选参数、或解析出的 [harness profile](/oss/javascript/deepagents/profiles) 贡献它们时才会出现的槽位。

#### Bare stack

仅有一个 `model`（没有其他可选参数）时，主 agent 通常包含：

1. [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware)
2. [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware)（因为除非 harness profile 禁用它，否则会自动添加 [general-purpose subagent](/oss/javascript/deepagents/subagents#default-subagent)）
3. [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware)
4. [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware)
5. **Prompt caching** middleware（为受支持的 providers 添加；在其他地方为 no-op）
6. **Harness profile extras** 和 **excluded-tool filtering**（如果解析出的 model profile 定义了它们）

#### Full stack

从第一个到最后一个：

1. [`SkillsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSkillsMiddleware)：仅当你传入 `skills` 时。它会**在** filesystem middleware **之前**注入，这样在文件工具运行之前就能获得 skill 元数据。

2. [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware)：处理文件系统操作，例如读取、写入和浏览目录。当你传入 `permissions` 时，文件系统权限执行（enforcement）会包含在这里，以便它能评估 agent 可能调用的每一个工具。

3. [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware)：仅当至少有一个同步 subagent 可用时。它生成并协调 subagents 以委派任务。由于默认会自动添加 general-purpose subagent，因此它被包含在 [bare stack](#bare-stack) 中；通过禁用该 subagent 并且不传入任何同步 `subagents` 即可省略它。参见 [Running without subagents](/oss/javascript/deepagents/subagents#running-without-subagents)。

4. [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware)：当对话变长时压缩消息历史以保持在上下文限制之内（通过 [createSummarizationMiddleware](https://reference.langchain.com/javascript/deepagents/middleware/createSummarizationMiddleware)）。

5. [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware)：当一次运行在中断后恢复、或收到格式错误的 tool-call 参数时，修复消息历史中悬空的工具调用。它在 Anthropic prompt caching 以及下面的 tail stack **之前**运行。

6. [`AsyncSubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/agent/createDeepAgent)：仅当你配置了 async subagents 时。

7. **你的 middleware 参数**：你作为 `middleware` 参数传入的可选 middleware 会追加到这里（在 Patch 之后、tail stack 之前）。

8. **Harness profile extras**：来自解析出的 model profile 的 provider 专属 middleware（如果有）。

9. **Excluded-tool filtering**：当 harness profile 列出了被排除的工具时，middleware 会从 agent 中移除这些工具。

10. **Prompt caching**（[`AnthropicPromptCachingMiddleware`](https://reference.langchain.com/javascript/langchain/index/anthropicPromptCachingMiddleware) 和 [`BedrockPromptCachingMiddleware`](https://reference.langchain.com/javascript/langchain/index/bedrockPromptCachingMiddleware)）：分别为 Anthropic models 和 Amazon Bedrock Converse models 自动添加。两者都在 Patch 之后、你的 middleware 之后运行，这样被缓存的 prefix 就会与实际发送给模型的内容相匹配。

11. [`MemoryMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createMemoryMiddleware)：仅当你传入 `memory` 时。

    > **注意**：`MemoryMiddleware` 被放在 profile extras 和 prompt caching middleware **之后**，这样对注入的 memory 的更新就不太可能使 cache prefix 失效。`createDeepAgent` 实现代码的注释中同样指出了这一顺序上的考量。

12. `HumanInTheLoopMiddleware`：仅当你传入 `interruptOn` 时。在配置好的工具调用处暂停，以等待人工批准或输入。

### Synchronous subagent stack

内置的 **general-purpose** subagent 以及每个声明式同步 `SubAgent` graph 都使用一套由 `createDeepAgent` 在代码中构建的 stack。它在总体形态上与主 agent 一致（文件系统、摘要、Patch、profile extras、Anthropic 和 Bedrock caching、可选 permissions），但在两个方面有所不同：

* **Skills 在这些内部 agents 上于** [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware) **之后运行**（在主 agent 上，当设置了 `skills` 时，skills 在 filesystem middleware **之前**运行）。
* subagent graph 内部**没有** [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware)（只有父 agent 暴露 `task` 工具）。

当声明式 subagent 设置了 `interruptOn` 时，该值会被转发给该 subagent 的 `createAgent`，从而为配置好的工具调用接好 human-in-the-loop 处理。

### Prebuilt middleware

LangChain 还暴露了额外的 prebuilt middleware，让你可以附加各种功能，例如重试、fallback 或 PII 检测。更多内容请参见 [Prebuilt middleware](/oss/javascript/langchain/middleware/built-in)。

`deepagents` 包也暴露了用于同一工作流的 [`createSummarizationMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSummarizationMiddleware)。更多细节请参见 [Summarization](/oss/javascript/deepagents/context-engineering#summarization)。

### Provider-specific middleware

关于针对特定 LLM providers 优化过的 provider 专属 middleware，请参见 [Middleware integrations](/oss/javascript/integrations/middleware)。

### Custom middleware

你可以提供额外的 middleware 来扩展功能、添加工具或实现自定义 hooks：

#### Google

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### OpenAI

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### Anthropic

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### OpenRouter

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### Fireworks

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### Baseten

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

#### Ollama

```ts
import { tool, createMiddleware } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const getWeather = tool(
  ({ city }: { city: string }) => {
    return `The weather in ${city} is sunny.`;
  },
  {
    name: "get_weather",
    description: "Get the weather in a city.",
    schema: z.object({
      city: z.string(),
    }),
  },
);

let callCount = 0;

const logToolCallsMiddleware = createMiddleware({
  name: "LogToolCallsMiddleware",
  wrapToolCall: async (request, handler) => {
    // Intercept and log every tool call - demonstrates cross-cutting concern
    callCount += 1;
    const toolName = request.toolCall.name;

    console.log(`[Middleware] Tool call #${callCount}: ${toolName}`);
    console.log(
      `[Middleware] Arguments: ${JSON.stringify(request.toolCall.args)}`,
    );

    // Execute the tool call
    const result = await handler(request);

    // Log the result
    console.log(`[Middleware] Tool call #${callCount} completed`);

    return result;
  },
});

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [getWeather] as any,
  middleware: [logToolCallsMiddleware] as any,
});
```

> **警告**：**不要在初始化之后修改属性（Do not mutate attributes after initialization）**
>
> 如果你需要在多次 hook 调用之间跟踪值（例如计数器或累计数据），请使用 graph state。graph state 在设计上以 thread 为作用域，因此在并发下更新是安全的。

**这样做：**

```ts
const customMiddleware = createMiddleware({
  name: "CustomMiddleware",
  beforeAgent: async (state) => {
    return { x: (state.x ?? 0) + 1 }; // Update graph state instead
  },
});
```

**不要**这样做：

```ts
let x = 1;

const customMiddlewareBad = createMiddleware({
  name: "CustomMiddleware",
  beforeAgent: async () => {
    x += 1; // Mutation causes race conditions
  },
});
```

原地修改（in-place mutation），例如在 `beforeAgent` 中修改 `state.x`、在 `beforeAgent` 中修改一个共享变量、或在 hooks 中更改其他共享值，都可能导致难以察觉的 bug 和竞态条件，因为许多操作会并发运行（subagents、并行工具，以及在不同 threads 上的并行调用）。

如果你必须在自定义 middleware 中使用 mutation，请考虑当 subagents、并行工具或并发的 agent 调用同时运行时会发生什么。

### Override a default middleware instance

### Interpreters

使用 [interpreters](/oss/javascript/deepagents/interpreters) 添加一个 `eval` 工具，它在作用域受限的 QuickJS runtime 中运行 JavaScript。当 agent 需要以编程方式组合工具、批量处理工作、在代码中处理错误，或在不使用完整 shell 环境的情况下转换结构化数据时，interpreters 很有用。

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  middleware: [createCodeInterpreterMiddleware()],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  middleware: [createCodeInterpreterMiddleware()],
});
```

关于设置、编程式工具调用、subagent 编排和限制，请参见 [Interpreters](/oss/javascript/deepagents/interpreters)。

## Subagents

为了隔离细节工作并避免上下文膨胀，请使用 subagents：

#### Google

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "google:gemini-3.6-flash", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### OpenAI

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "openai:gpt-5.5", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### Anthropic

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "anthropic:claude-sonnet-5", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### OpenRouter

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "openrouter:z-ai/glm-5.2", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### Fireworks

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "fireworks:accounts/fireworks/models/glm-5p2", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### Baseten

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "baseten:zai-org/GLM-5.2", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

#### Ollama

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
import { createDeepAgent, type SubAgent } from "deepagents";
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

const researchSubagent: SubAgent = {
  name: "research-agent",
  description: "Used to research more in depth questions",
  systemPrompt: "You are a great researcher",
  tools: [internetSearch],
  model: "ollama:north-mini-code-1.0", // Optional override, defaults to main agent model
};
const subagents = [researchSubagent];

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  subagents,
});
```

更多信息请参见 [Subagents](/oss/javascript/deepagents/subagents)。

## Backends

deep agent 的工具可以利用虚拟文件系统来存储、访问和编辑文件。默认情况下，deep agents 使用 [`StateBackend`](https://reference.langchain.com/javascript/deepagents/backends/StateBackend)。

如果你使用了 [skills](#skills) 或 [memory](#memory)，则必须在创建 agent 之前把预期的 skill 或 memory 文件添加到 backend 中。

#### StateBackend

一个存储在 `langgraph` state 中、以 thread 为作用域的文件系统 backend。

文件在一个 thread 内跨多轮持久存在（通过你的 checkpointer），并且不会跨 threads 共享。

```ts
import { createDeepAgent, StateBackend } from "deepagents";

// By default we provide a StateBackend
const agent = createDeepAgent();

// Under the hood, it looks like
const agent2 = createDeepAgent({
  backend: new StateBackend(),
});
```

#### FilesystemBackend

本地机器的文件系统。

> **警告**：该 backend 授予 agent 直接的文件系统读写访问权限。请谨慎使用，并且只在合适的环境中使用。更多信息请参见 [`FilesystemBackend`](/oss/javascript/deepagents/backends#filesystembackend-local-disk)。

##### Google

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### OpenAI

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### Anthropic

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### OpenRouter

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### Fireworks

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### Baseten

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

##### Ollama

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
});
```

> **提示**：把 `FilesystemBackend` 包裹在 `CompositeBackend` 中，以防止 agent 内部数据（卸载的 tool results、对话历史）与你的项目文件一起被写入磁盘。请参见[推荐模式](/oss/javascript/deepagents/backends#filesystembackend-local-disk)。

#### LocalShellBackend

一个可直接在宿主机上执行 shell 的文件系统。除了文件系统工具外，还提供用于运行命令的 `execute` 工具。

> **警告**：该 backend 授予 agent 直接的文件系统读写访问权限，**以及**在你的宿主机上不受限制的 shell 执行权限。请极其谨慎地使用，并且只在合适的环境中使用。更多信息请参见 [`LocalShellBackend`](/oss/javascript/deepagents/backends#localshellbackend-local-shell)。

##### Google

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend,
});
```

##### OpenAI

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  backend,
});
```

##### Anthropic

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend,
});
```

##### OpenRouter

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend,
});
```

##### Fireworks

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend,
});
```

##### Baseten

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend,
});
```

##### Ollama

```ts
import { createDeepAgent, LocalShellBackend } from "deepagents";

const backend = new LocalShellBackend({ workingDirectory: "." });

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend,
});
```

#### StoreBackend

一种提供长期存储的文件系统，其数据*跨 threads 持久化*。

##### Google

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### OpenAI

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### Anthropic

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### OpenRouter

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### Fireworks

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### Baseten

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

##### Ollama

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new StoreBackend({
    namespace: (rt) => [rt.serverInfo.user.identity],
  }),
  store,
});
```

> **注意**：部署到 [LangSmith Deployment](/langsmith/deployment) 时，请省略 `store` 参数。平台会为你的 agent 自动配置一个 store。

> **提示**：`namespace` 参数控制数据隔离。对于多用户部署，务必设置 [namespace factory](/oss/javascript/deepagents/backends#namespace-factories) 以按用户或租户隔离数据。

#### ContextHubBackend

位于 LangSmith Hub repo 中的持久文件系统存储。

更多细节请参见 [`ContextHubBackend`](/oss/javascript/deepagents/backends#contexthubbackend)。

#### CompositeBackend

一种灵活的 backend，你可以在文件系统中指定不同的路由，把它们指向不同的 backends。

##### Google

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### OpenAI

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### Anthropic

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### OpenRouter

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### Fireworks

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### Baseten

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

##### Ollama

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  store,
});
```

更多信息请参见 [Backends](/oss/javascript/deepagents/backends)。

### Sandboxes

Sandboxes 是一类特殊的 [backends](/oss/javascript/deepagents/backends)，它们在隔离环境中运行 agent 代码，拥有自己的文件系统以及用于 shell 命令的 `execute` 工具。
当你希望 deep agent 写入文件、安装依赖并运行命令，而又不想改变本地机器上的任何东西时，请使用 sandbox backend。

创建 deep agent 时，把 sandbox backend 传给 `backend` 即可配置 sandboxes：

```typescript
import { createDeepAgent, LangSmithSandbox } from "deepagents";
import { ChatAnthropic } from "@langchain/anthropic";
import { SandboxClient } from "langsmith/sandbox";

const client = new SandboxClient();
const lsSandbox = await client.createSandbox();

try {
  const agent = createDeepAgent({
    model: new ChatAnthropic({ model: "claude-opus-4-8" }),
    systemPrompt: "You are a coding assistant with sandbox access.",
    backend: new LangSmithSandbox({ sandbox: lsSandbox }),
  });

  const result = await agent.invoke({
    messages: [
      {
        role: "user",
        content: "Create a hello world Python script and run it",
      },
    ],
  });
} finally {
  await client.deleteSandbox(lsSandbox.name);
}
```

更多信息请参见 [Sandboxes](/oss/javascript/deepagents/sandboxes)。

## Human-in-the-loop

某些工具操作可能比较敏感，需要在执行前由人工批准。
你可以为每个工具配置批准方式：

```ts
import { tool } from "langchain";
import { createDeepAgent } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";
import { z } from "zod";

const removeFile = tool(
  async ({ path }: { path: string }) => {
    return `Deleted ${path}`;
  },
  {
    name: "remove_file",
    description: "Delete a file from the filesystem.",
    schema: z.object({
      path: z.string(),
    }),
  },
);

const fetchFile = tool(
  async ({ path }: { path: string }) => {
    return `Contents of ${path}`;
  },
  {
    name: "fetch_file",
    description: "Read a file from the filesystem.",
    schema: z.object({
      path: z.string(),
    }),
  },
);

const notifyEmail = tool(
  async ({
    to,
    subject,
    body,
  }: {
    to: string;
    subject: string;
    body: string;
  }) => {
    return `Sent email to ${to}`;
  },
  {
    name: "notify_email",
    description: "Send an email.",
    schema: z.object({
      to: z.string(),
      subject: z.string(),
      body: z.string(),
    }),
  },
);

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [removeFile, fetchFile, notifyEmail],
  interruptOn: {
    remove_file: true, // Default: approve, edit, reject, respond
    fetch_file: false, // No interrupts needed
    notify_email: { allowedDecisions: ["approve", "reject"] }, // No editing
  },
  checkpointer, // Required!
});
```

你可以为 agents 和 subagents 配置在工具调用时以及在工具调用内部的中断。
更多信息请参见 [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)。

## Skills

你可以使用 [skills](/oss/javascript/deepagents/overview) 为你的 deep agent 提供新的能力和专业知识。
虽然 [tools](/oss/javascript/deepagents/customization#tools) 往往涵盖较低层的功能（例如原生文件系统操作），但 skills 可以包含关于如何完成任务的详细指令、参考信息以及其他资产（例如模板）。
这些文件只有在 agent 判定该 skill 对当前 prompt 有用时才会被 agent 加载。
这种渐进式披露（progressive disclosure）减少了 agent 在启动时必须考虑的 tokens 和上下文数量。

示例 skills 请参见 [Deep Agents example skills](https://github.com/langchain-ai/deepagentsjs/tree/main/examples/skills)。

要为你的 deep agent 添加 skills，请把它们作为参数传给 `create_deep_agent`：

#### StateBackend

```ts
import { createDeepAgent, StateBackend, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();
const backend = new StateBackend();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

const skillsFiles: Record<string, FileData> = {};
const skillUrl =
  "https://raw.githubusercontent.com/langchain-ai/deepagentsjs/refs/heads/main/examples/skills/langgraph-docs/SKILL.md";
const response = await fetch(skillUrl);
const skillContent = await response.text();

skillsFiles["/skills/langgraph-docs/SKILL.md"] = createFileData(skillContent);

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  backend,
  checkpointer, // Required !
  // IMPORTANT: deepagents skill source paths are virtual (POSIX) paths relative to the backend root.
  skills: ["/skills/"],
});

const config = { configurable: { thread_id: `thread-${Date.now()}` } };
const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "what is langraph?" }],
    files: skillsFiles,
  },
  config,
);
```

#### StoreBackend

```ts
import { createDeepAgent, StoreBackend } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();
const store = new InMemoryStore();
const backend = new StoreBackend({
  namespace: () => ["filesystem"],
  store,
});

const skillUrl =
  "https://raw.githubusercontent.com/langchain-ai/deepagentsjs/refs/heads/main/examples/skills/langgraph-docs/SKILL.md";

const response = await fetch(skillUrl);
const skillContent = await response.text();

await backend.uploadFiles([
  ["/skills/langgraph-docs/SKILL.md", new TextEncoder().encode(skillContent)],
]);

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  backend,
  store,
  checkpointer,
  // IMPORTANT: deepagents skill source paths are virtual (POSIX) paths relative to the backend root.
  skills: ["/skills/"],
});

const config = {
  recursionLimit: 50,
  configurable: { thread_id: `thread-${Date.now()}` },
};
const result = await agent.invoke(
  { messages: [{ role: "user", content: "what is langraph?" }] },
  config,
);
```

#### FilesystemBackend

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();

const skillUrl =
  "https://raw.githubusercontent.com/langchain-ai/deepagentsjs/refs/heads/main/examples/skills/langgraph-docs/SKILL.md";
const response = await fetch(skillUrl);
const skillContent = await response.text();

let backend = new FilesystemBackend({
  rootDir: process.cwd(),
  virtualMode: true,
});
await backend.uploadFiles([
  ["/skills/langgraph-docs/SKILL.md", new TextEncoder().encode(skillContent)],
]);

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  backend,
  // IMPORTANT: deepagents skill source paths are virtual (POSIX) paths relative to the backend root.
  skills: ["/skills/"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required for filesystem operations!
});

const config = { configurable: { thread_id: `thread-${Date.now()}` } };
const result = await agent.invoke(
  { messages: [{ role: "user", content: "what is langraph?" }] },
  config,
);
```

## Memory

使用 [`AGENTS.md` 文件](https://agents.md/) 为你的 deep agent 提供额外上下文。

> **提示**：要生成一个能被 coding agents 通过 `AGENTS.md` 发现的仓库 wiki，请参见 [OpenWiki](/oss/openwiki/overview)。

创建 deep agent 时，你可以向 `memory` 参数传入一个或多个文件路径：

#### StateBackend

##### Google

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### OpenAI

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Anthropic

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### OpenRouter

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Fireworks

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Baseten

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Ollama

```ts
import { createDeepAgent, type FileData } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);
const checkpointer = new MemorySaver();

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  memory: ["/AGENTS.md"],
  checkpointer: checkpointer,
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
    // Seed the default StateBackend's in-state filesystem (virtual paths must start with "/").
    files: { "/AGENTS.md": createFileData(agentsMd) },
  },
  { configurable: { thread_id: "12345" } },
);
```

> **View example trace**（[链接](https://smith.langchain.com/public/5c07e925-9d8f-4792-b73c-9dbee94890cd/r)）：Open a public LangSmith run for this example.

#### StoreBackend

##### Google

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### OpenAI

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Anthropic

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### OpenRouter

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Fireworks

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Baseten

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

##### Ollama

```ts
import { createDeepAgent, StoreBackend, type FileData } from "deepagents";
import { InMemoryStore, MemorySaver } from "@langchain/langgraph";

const AGENTS_MD_URL =
  "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md";

async function fetchText(url: string): Promise<string> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`Failed to fetch ${url}: ${res.status} ${res.statusText}`);
  }
  return await res.text();
}

const agentsMd = await fetchText(AGENTS_MD_URL);

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content,
    mimeType: "text/plain",
    created_at: now,
    modified_at: now,
  };
}

const store = new InMemoryStore();
const fileData = createFileData(agentsMd);
await store.put(["filesystem"], "/AGENTS.md", fileData);

const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new StoreBackend({
    namespace: () => ["filesystem"],
  }),
  store: store,
  checkpointer: checkpointer,
  memory: ["/AGENTS.md"],
});

const result = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "Please tell me what's in your memory files.",
      },
    ],
  },
  { configurable: { thread_id: "12345" } },
);
```

> **View example trace**（[链接](https://smith.langchain.com/public/ff0f387f-c0c3-4456-8081-d6fc23504942/r)）：Open a public LangSmith run for this example.

#### Filesystem

##### Google

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### OpenAI

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### Anthropic

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### OpenRouter

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### Fireworks

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### Baseten

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

##### Ollama

```ts
import { createDeepAgent, FilesystemBackend } from "deepagents";
import { MemorySaver } from "@langchain/langgraph";

// Checkpointer is REQUIRED for human-in-the-loop
const checkpointer = new MemorySaver();

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new FilesystemBackend({ rootDir: "/Users/user/{project}" }),
  memory: ["./AGENTS.md", "./.deepagents/AGENTS.md"],
  interruptOn: {
    read_file: true,
    write_file: true,
    delete_file: true,
  },
  checkpointer, // Required!
});
```

## Structured output

Deep Agents 支持 [structured output](/oss/javascript/langchain/structured-output)。

你可以把所需的结构化输出 schema 作为 `responseFormat` 参数传给 `createDeepAgent()` 调用。
当模型生成结构化数据时，它会被捕获、校验，并返回到 agent state 的 `structuredResponse` 键中。

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

const weatherReportSchema = z.object({
  location: z.string().describe("The location for this weather report"),
  temperature: z.number().describe("Current temperature in Celsius"),
  condition: z
    .string()
    .describe("Current weather condition (e.g., sunny, cloudy, rainy)"),
  humidity: z.number().describe("Humidity percentage"),
  windSpeed: z.number().describe("Wind speed in km/h"),
  forecast: z.string().describe("Brief forecast for the next 24 hours"),
});

const agent = await createDeepAgent({
  responseFormat: weatherReportSchema,
  tools: [internetSearch],
});

const result = await agent.invoke({
  messages: [
    {
      role: "user",
      content: "What's the weather like in San Francisco?",
    },
  ],
});

console.log(result.structuredResponse);
// {
//   location: 'San Francisco, California',
//   temperature: 18.3,
//   condition: 'Sunny',
//   humidity: 48,
//   windSpeed: 7.6,
//   forecast: 'Clear skies with temperatures remaining mild. High of 18°C (64°F) during the day, dropping to around 11°C (52°F) at night.'
// }
```

> **View example trace**（[链接](https://smith.langchain.com/public/e4587886-ebb6-4fbc-8b57-d41b27180770/r)）：Open a public LangSmith run for this example.

更多信息和示例请参见 [response format](/oss/javascript/langchain/structured-output#response-format)。

## Advanced

`createDeepAgent` 在 `createAgent` 之上预先组装了一套 middleware stack。若要构建一个完全自定义的 agent——精确选择要包含哪些能力——请参见 [Configure the harness](/oss/javascript/langchain/agents#configure-the-harness)。

***
