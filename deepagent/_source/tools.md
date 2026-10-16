> ## Documentation Index
> Fetch the complete documentation index at: https://docs.langchain.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Tools

> Connect Deep Agents to custom functions, APIs, databases, and any MCP server

Deep Agents can call any tool you define, any [LangChain tool](https://python.langchain.com/docs/concepts/tools/), and tools from any [MCP server](#mcp-tools).
Pass them to `create_deep_agent` via the `tools=` parameter alongside the [built-in harness tools](/oss/javascript/deepagents/overview#execution-environment) for file management and subagent spawning.

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "google:gemini-3.6-flash",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "openai:gpt-5.5",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "anthropic:claude-sonnet-5",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "openrouter:z-ai/glm-5.2",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "fireworks:accounts/fireworks/models/glm-5p2",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "baseten:zai-org/GLM-5.2",
    tools: [search, fetchUrl, runQuery],
  });
  ```

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";


  const agent = await createDeepAgent({
    model: "ollama:north-mini-code-1.0",
    tools: [search, fetchUrl, runQuery],
  });
  ```
</CodeGroup>

## Custom tools

Pass any callable, such as plain functions, LangChain `@tool`-decorated functions, or tool dicts—directly to `tools=`.
Deep Agents infers the tool schema from the function signature and docstring, so you don't need to define a separate schema in most cases.

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
</CodeGroup>

For full details on defining and using LangChain tools (tool dicts, `StructuredTool`, return types, error handling, and more), see [Tools](/oss/javascript/langchain/tools).

## MCP tools

<Note>
  Deep Agents fully support [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp), the open standard for connecting agents to external services. Load tools from any MCP server and pass them directly to `create_deep_agent`.
</Note>

MCP is an open protocol that lets agents connect to a growing ecosystem of servers—databases, APIs, file systems, browsers, and more—through a standard interface. Instead of writing custom integration code for each service, you point Deep Agents at an MCP server and it gets all the tools that server exposes.

Install `@langchain/mcp-adapters` to connect to MCP servers:

```bash theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
npm install @langchain/mcp-adapters
```

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
</CodeGroup>

For detailed configuration options—including stdio servers, OAuth authentication, tool filtering, and stateful sessions—see the full [MCP guide](/oss/javascript/langchain/mcp).

## Built-in harness tools

In addition to the tools you provide, every Deep Agent comes with a built-in set of tools from the harness:

| Tool         | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `ls`         | List files in a directory.                                   |
| `read_file`  | Read file contents (with pagination and multimodal support). |
| `write_file` | Create new files.                                            |
| `edit_file`  | Perform exact string replacements in files.                  |
| `glob`       | Find files matching a glob pattern.                          |
| `grep`       | Search file contents.                                        |
| `execute`    | Run shell commands (sandbox backends only).                  |
| `task`       | Spawn a subagent to handle a delegated task.                 |

To add structured task planning with `write_todos`, opt in with [`TodoListMiddleware`](https://reference.langchain.com/javascript/langchain/index/todoListMiddleware). See [Task planning](/oss/javascript/deepagents/overview#task-planning).

For a full breakdown of what each built-in tool does, see [Harness overview](/oss/javascript/deepagents/overview#execution-environment).

## Multimodal tool outputs

Custom tools can return plain text or [standard content blocks](/oss/javascript/langchain/messages#standard-content-blocks) (text, images, audio, video, and files) when the selected model supports multimodal tool results. The built-in `read_file` tool also returns multimodal blocks for supported non-text file types.

Return a string for text-only results, or an ordered list of content blocks for text plus media or interleaved multimodal output. See [Multimodal](/oss/javascript/deepagents/multimodal) and [Tool return values](/oss/javascript/langchain/tools#return-multimodal-content) for examples and context-compression considerations.

***

<div className="source-links">
  <Callout icon="terminal-2">
    [Connect these docs](/use-these-docs) to your agent of choice via MCP for real-time answers.
  </Callout>

  <Callout icon="edit">
    [Edit this page on GitHub](https://github.com/langchain-ai/docs/edit/main/src/oss/deepagents/tools.mdx) or [file an issue](https://github.com/langchain-ai/docs/issues/new/choose).
  </Callout>
</div>
