> ## Documentation Index
> Fetch the complete documentation index at: https://docs.langchain.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Customize Deep Agents

> Learn how to customize Deep Agents with system prompts, tools, subagents, and more

Build the harness around your goal. `create_deep_agent` gives you a production-ready foundation: connect it to your data, shape its behavior, and add the capabilities your use case needs.

`createDeepAgent` ships with a pre-assembled harness: filesystem, summarization, subagents, and prompt caching by default. The parameters below let you define the agent's persona, connect it to your data and tools, and extend the [Deep Agents stack](#deep-agents-stack) with additional middleware.

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "google:gemini-3.6-flash",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "openai:gpt-5.5",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "anthropic:claude-sonnet-5",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "openrouter:z-ai/glm-5.2",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "fireworks:accounts/fireworks/models/glm-5p2",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "baseten:zai-org/GLM-5.2",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";

  const agent = await createDeepAgent({
    model: "ollama:north-mini-code-1.0",
    systemPrompt: "You are a helpful assistant.",
    tools: [search, fetchUrl],
    memory: ["./AGENTS.md"],
    skills: ["./skills/"],
  });
  ```
</CodeGroup>

| Parameter                                                                         | What it does                                                             |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `model`                                                                           | Which model to use                                                       |
| `systemPrompt`                                                                    | Custom instructions for the agent                                        |
| `tools`                                                                           | Domain tools the agent can call                                          |
| `memory`                                                                          | AGENTS.md files loaded at startup                                        |
| `skills`                                                                          | Skills directory for on-demand knowledge                                 |
| `backend`                                                                         | Filesystem backend (StateBackend by default)                             |
| `permissions`                                                                     | Path-level access control for the filesystem                             |
| `subagents`                                                                       | Custom subagents for delegated tasks                                     |
| `middleware`                                                                      | Extra middleware appended to the [Deep Agents stack](#deep-agents-stack) |
| `interruptOn`                                                                     | Pause before tool calls for human approval                               |
| `responseFormat`                                                                  | Structured output schema                                                 |
| [`contextSchema`](/oss/javascript/deepagents/context-engineering#runtime-context) | Per-run runtime context schema (user IDs, API keys, feature flags)       |

For the full parameter list, see the [`createDeepAgent`](https://reference.langchain.com/javascript/deepagents/types/CreateDeepAgentParams) API reference. To compose a fully custom harness from scratch, see [Configure the harness](/oss/javascript/langchain/agents#configure-the-harness).

<Tip>
  As you add tools, subagents, and backends, use [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-deepagents-customization) to trace how each piece behaves together. Follow the [observability quickstart](/langsmith/observability-quickstart) to get set up, and see [Going to production](/oss/javascript/deepagents/going-to-production) for deployment on LangSmith.

  We recommend you also set up [LangSmith Engine](/langsmith/engine), which monitors your traces, detects issues, and proposes fixes.
</Tip>

## Model

Pass a `model` string in `provider:model` format, or an initialized model instance. See [supported models](/oss/javascript/deepagents/models#supported-models) for all providers and [suggested models](/oss/javascript/deepagents/models#suggested-models) for tested recommendations.

<Tip>
  Use the `provider:model` format (for example `openai:gpt-5.5`) to quickly switch between models.
</Tip>

<Tabs>
  <Tab title="OpenAI">
    👉 Read the [OpenAI chat model integration docs](/oss/javascript/integrations/chat/openai/)

    <CodeGroup>
      ```bash npm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      npm install @langchain/openai deepagents
      ```

      ```bash pnpm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      pnpm install @langchain/openai deepagents
      ```

      ```bash yarn theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      yarn add @langchain/openai deepagents
      ```

      ```bash bun theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      bun add @langchain/openai deepagents
      ```
    </CodeGroup>

    <CodeGroup>
      ```typescript default parameters theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent } from "deepagents";

      process.env.OPENAI_API_KEY = "your-api-key";

      const agent = createDeepAgent({ model: "gpt-5.5" });
      // this calls initChatModel for the specified model with default parameters
      // to use specific model parameters, use initChatModel directly
      ```

      ```typescript initChatModel theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { initChatModel } from "langchain";
      import { createDeepAgent } from "deepagents";

      process.env.OPENAI_API_KEY = "your-api-key";

      const model = await initChatModel("gpt-5.5");
      const agent = createDeepAgent({
        model,
        temperature: 0,
      });
      ```

      ```typescript Model Class theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>

  <Tab title="Anthropic">
    👉 Read the [Anthropic chat model integration docs](/oss/javascript/integrations/chat/anthropic/)

    <CodeGroup>
      ```bash npm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      npm install @langchain/anthropic deepagents
      ```

      ```bash pnpm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      pnpm install @langchain/anthropic deepagents
      ```

      ```bash yarn theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      yarn add @langchain/anthropic deepagents
      ```

      ```bash bun theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      bun add @langchain/anthropic deepagents
      ```
    </CodeGroup>

    <CodeGroup>
      ```typescript default parameters theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent } from "deepagents";

      process.env.ANTHROPIC_API_KEY = "your-api-key";

      const agent = createDeepAgent({ model: "anthropic:claude-sonnet-4-6" });
      // this calls initChatModel for the specified model with default parameters
      // to use specific model parameters, use initChatModel directly
      ```

      ```typescript initChatModel theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { initChatModel } from "langchain";
      import { createDeepAgent } from "deepagents";

      process.env.ANTHROPIC_API_KEY = "your-api-key";

      const model = await initChatModel("claude-sonnet-4-6");
      const agent = createDeepAgent({
        model,
        temperature: 0,
      });
      ```

      ```typescript Model Class theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>

  <Tab title="Azure">
    👉 Read the [Azure chat model integration docs](/oss/javascript/integrations/chat/azure/)

    <CodeGroup>
      ```bash npm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      npm install @langchain/azure deepagents
      ```

      ```bash pnpm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      pnpm install @langchain/azure deepagents
      ```

      ```bash yarn theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      yarn add @langchain/azure deepagents
      ```

      ```bash bun theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      bun add @langchain/azure deepagents
      ```
    </CodeGroup>

    <CodeGroup>
      ```typescript default parameters theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent } from "deepagents";

      process.env.AZURE_OPENAI_API_KEY = "your-api-key";
      process.env.AZURE_OPENAI_ENDPOINT = "your-endpoint";
      process.env.OPENAI_API_VERSION = "your-api-version";

      const agent = createDeepAgent({ model: "azure_openai:gpt-5.5" });
      // this calls initChatModel for the specified model with default parameters
      // to use specific model parameters, use initChatModel directly
      ```

      ```typescript initChatModel theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```typescript Model Class theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>

  <Tab title="Google Gemini">
    👉 Read the [ChatGoogle chat model integration docs](/oss/javascript/integrations/chat/google)

    <CodeGroup>
      ```bash npm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      npm install @langchain/google deepagents
      ```

      ```bash pnpm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      pnpm install @langchain/google deepagents
      ```

      ```bash yarn theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      yarn add @langchain/google deepagents
      ```

      ```bash bun theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      bun add @langchain/google deepagents
      ```
    </CodeGroup>

    <CodeGroup>
      ```typescript default parameters theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent } from "deepagents";

      process.env.GOOGLE_API_KEY = "your-api-key";

      const agent = createDeepAgent({ model: "google:gemini-3.1-pro-preview" });
      // this calls initChatModel for the specified model with default parameters
      // to use specific model parameters, use initChatModel directly
      ```

      ```typescript initChatModel theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { initChatModel } from "langchain";
      import { createDeepAgent } from "deepagents";

      process.env.GOOGLE_API_KEY = "your-api-key";

      const model = await initChatModel("google:gemini-3.1-pro-preview");
      const agent = createDeepAgent({
        model,
        temperature: 0,
      });
      ```

      ```typescript Model Class theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>

  <Tab title="Bedrock Converse">
    👉 Read the [AWS Bedrock chat model integration docs](/oss/javascript/integrations/chat/bedrock_converse/)

    <CodeGroup>
      ```bash npm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      npm install @langchain/aws deepagents
      ```

      ```bash pnpm theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      pnpm install @langchain/aws deepagents
      ```

      ```bash yarn theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      yarn add @langchain/aws deepagents
      ```

      ```bash bun theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      bun add @langchain/aws deepagents
      ```
    </CodeGroup>

    <CodeGroup>
      ```typescript default parameters theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent } from "deepagents";

      // Follow the steps here to configure your credentials:
      // https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started.html

      const agent = createDeepAgent({ model: "bedrock:anthropic.claude-sonnet-4-6" });
      // this calls initChatModel for the specified model with default parameters
      // to use specific model parameters, use initChatModel directly
      ```

      ```typescript initChatModel theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```typescript Model Class theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>

  <Tab title="Other">
    Pass any [supported model string](/oss/javascript/deepagents/models#supported-models), or an initialized model instance:

    ```typescript theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
    import { initChatModel } from "langchain";
    import { createDeepAgent } from "deepagents";

    const model = await initChatModel("provider:model-name");
    const agent = createDeepAgent({ model });
    ```
  </Tab>
</Tabs>

<Tip>
  Chat models automatically retry transient API failures (with exponential backoff). For defaults, limits, and code samples for tuning `max_retries` / `timeout` live on the LangChain [Models](/oss/javascript/langchain/models#connection-resilience) page.
</Tip>

## Tools

In addition to [built-in tools](/oss/javascript/deepagents/overview#execution-environment) for file management and subagent spawning, you can provide custom tools:

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

### MCP tools

<Tip>
  Deep Agents fully support [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp) tools. You can load tools from any MCP server—databases, APIs, file systems, and more—and pass them directly to `create_deep_agent`.
</Tip>

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

For detailed configuration options including stdio servers, OAuth authentication, tool filtering, and stateful sessions, see the full [MCP guide](/oss/javascript/langchain/mcp).

## System prompt

Pass `system_prompt=` to give the agent your own instructions:

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
</CodeGroup>

<Note>
  Besides a string, the main agent also accepts a [`SystemMessage`](https://reference.langchain.com/javascript/langchain-core/messages/SystemMessage) with structured [content blocks](/oss/javascript/langchain/messages#standard-content-blocks); Deep Agents preserve those blocks ([subagent](/oss/javascript/deepagents/subagents) dictionary specs remain strings).
</Note>

<AccordionGroup>
  <Accordion title="Subagent prompts">
    Declarative [subagents](/oss/javascript/deepagents/subagents) resolve profile overlays against their own model, then apply the resolved profile's `base_system_prompt` / `system_prompt_suffix` to the subagent's authored `system_prompt`. A profile that ships only a `system_prompt_suffix` (the common case for built-in Anthropic / OpenAI profiles) appends to the authored prompt. A profile that sets `base_system_prompt` replaces it outright.
  </Accordion>

  <Accordion title="General-purpose subagent prompt">
    The auto-added [general-purpose subagent](/oss/javascript/deepagents/subagents#the-general-purpose-subagent) resolves its base prompt as **`general_purpose_subagent.system_prompt` (if set) -> `HarnessProfile.base_system_prompt` (if set) -> SDK general-purpose default**, with the profile suffix layered on top. When both override fields are set, the general-purpose-specific one wins so a caller tuning both fields never sees their GP override silently dropped:

    ```python theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

    | Stack       | Final system prompt                                     |
    | ----------- | ------------------------------------------------------- |
    | Main agent  | `"You are ACME's support orchestrator." + SUFFIX`       |
    | GP subagent | `"You are a research subagent. Cite sources." + SUFFIX` |
  </Accordion>
</AccordionGroup>

## Middleware

Deep Agents support any [middleware](/oss/javascript/langchain/middleware/overview), including the built-in middleware listed below, prebuilt middleware from LangChain, provider-specific middleware, and custom middleware you write yourself.

Pass middleware to the `middleware` argument of `createDeepAgent`. Custom middleware is appended after [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware) in the [Deep Agents stack](#deep-agents-stack).

### Deep Agents stack

`createDeepAgent` builds middleware in a fixed order. The [bare stack](#bare-stack) is what you get with only a model. The [full stack](#full-stack) is the complete assembly order, including slots that appear only when you pass optional arguments or when the resolved [harness profile](/oss/javascript/deepagents/profiles) contributes them.

#### Bare stack

With only a `model` (no other optional arguments), the main agent typically includes:

1. [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware)
2. [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware) (because the [general-purpose subagent](/oss/javascript/deepagents/subagents#default-subagent) is auto-added unless a harness profile disables it)
3. [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware)
4. [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware)
5. **Prompt caching** middleware (added for supported providers; no-ops elsewhere)
6. **Harness profile extras** and **excluded-tool filtering**, if the resolved model profile defines them

#### Full stack

From first to last:

1. [`SkillsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSkillsMiddleware): Only when you pass `skills`. Injected **before** filesystem middleware so skill metadata is available before file tools run.

2. [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware): Handles file system operations such as reading, writing, and navigating directories. When you pass `permissions`, filesystem permissions enforcement is included here so it can evaluate every tool the agent might call.

3. [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware): Only when at least one synchronous subagent is available. Spawns and coordinates subagents for delegating tasks. Included in the [bare stack](#bare-stack) because the general-purpose subagent is auto-added by default; omit it by disabling that subagent and passing no synchronous `subagents`. See [Running without subagents](/oss/javascript/deepagents/subagents#running-without-subagents).

4. [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware): Condenses message history to stay within context limits when conversations grow long (via [createSummarizationMiddleware](https://reference.langchain.com/javascript/deepagents/middleware/createSummarizationMiddleware)).

5. [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware): Repairs dangling tool calls in message history when a run resumes after an interruption or receives malformed tool-call arguments. Runs **before** Anthropic prompt caching and the tail stack below.

6. [`AsyncSubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/agent/createDeepAgent): Only when you configure async subagents.

7. **Your middleware argument**: Optional middleware you pass as the `middleware` argument is appended here (after Patch, before the tail stack).

8. **Harness profile extras**: Provider-specific middleware from the resolved model profile, if any.

9. **Excluded-tool filtering**: When the harness profile lists excluded tools, middleware removes those tools from the agent.

10. **Prompt caching** ([`AnthropicPromptCachingMiddleware`](https://reference.langchain.com/javascript/langchain/index/anthropicPromptCachingMiddleware) and [`BedrockPromptCachingMiddleware`](https://reference.langchain.com/javascript/langchain/index/bedrockPromptCachingMiddleware)): Added automatically for Anthropic models and Amazon Bedrock Converse models, respectively. Both run **after** Patch and after your middleware so the cached prefix matches what is actually sent to the model.

11. [`MemoryMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createMemoryMiddleware): Only when you pass `memory`.

    <Note>
      `MemoryMiddleware` is placed **after** profile extras and the prompt caching middleware so updates to injected memory are less likely to invalidate the cache prefix. The same ordering concern is called out in the `createDeepAgent` implementation comments.
    </Note>

12. `HumanInTheLoopMiddleware`: Only when you pass `interruptOn`. Pauses for human approval or input at configured tool calls.

### Synchronous subagent stack

The built-in **general-purpose** subagent and each declarative synchronous `SubAgent` graph use a stack that `createDeepAgent` builds in code. It matches the main agent in broad shape (filesystem, summarization, Patch, profile extras, Anthropic and Bedrock caching, optional permissions) but differs in two ways:

* **Skills run after** [`PatchToolCallsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createPatchToolCallsMiddleware) on these inner agents (on the main agent, skills run **before** filesystem middleware when `skills` is set).
* There is **no** [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware) inside a subagent graph (only the parent agent exposes the `task` tool).

When a declarative subagent sets `interruptOn`, that value is forwarded to `createAgent` for the subagent, which wires up human-in-the-loop handling for the configured tool calls.

### Prebuilt middleware

LangChain exposes additional prebuilt middleware that let you add-on various features, such as retries, fallbacks, or PII detection. See [Prebuilt middleware](/oss/javascript/langchain/middleware/built-in) for more.

The `deepagents` package also exposes [`createSummarizationMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSummarizationMiddleware) for the same workflow. For more detail, see [Summarization](/oss/javascript/deepagents/context-engineering#summarization).

### Provider-specific middleware

For provider-specific middleware that is optimized for specific LLM providers, see [Middleware integrations](/oss/javascript/integrations/middleware).

### Custom middleware

You can provide additional middleware to extend functionality, add tools, or implement custom hooks:

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
</CodeGroup>

<Warning>
  **Do not mutate attributes after initialization**

  If you need to track values across hook invocations (for example, counters or accumulated data), use graph state.
  Graph state is scoped to a thread by design, so updates are safe under concurrency.

  **Do this:**

  ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  const customMiddleware = createMiddleware({
    name: "CustomMiddleware",
    beforeAgent: async (state) => {
      return { x: (state.x ?? 0) + 1 }; // Update graph state instead
    },
  });
  ```

  Do **not** do this:

  ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  let x = 1;

  const customMiddlewareBad = createMiddleware({
    name: "CustomMiddleware",
    beforeAgent: async () => {
      x += 1; // Mutation causes race conditions
    },
  });
  ```

  Mutation in place, such as modifying `state.x` in `beforeAgent`, mutating a shared variable in `beforeAgent`, or changing other shared values in hooks, can lead to subtle bugs and race conditions because many operations run concurrently (subagents, parallel tools, and parallel invocations on different threads).

  If you must use mutation in custom middleware, consider what happens when subagents, parallel tools, or concurrent agent invocations run at the same time.
</Warning>

### Override a default middleware instance

### Interpreters

Use [interpreters](/oss/javascript/deepagents/interpreters) to add an `eval` tool that runs JavaScript in a scoped QuickJS runtime. Interpreters are useful when the agent needs to compose tools programmatically, batch work, handle errors in code, or transform structured data without a full shell environment.

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "google:gemini-3.6-flash",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "openai:gpt-5.5",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "anthropic:claude-sonnet-5",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "openrouter:z-ai/glm-5.2",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "fireworks:accounts/fireworks/models/glm-5p2",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "baseten:zai-org/GLM-5.2",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
  import { createDeepAgent } from "deepagents";
  import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

  const agent = createDeepAgent({
    model: "ollama:north-mini-code-1.0",
    middleware: [createCodeInterpreterMiddleware()],
  });
  ```
</CodeGroup>

For setup, programmatic tool calling, subagent orchestration, and limits, see [Interpreters](/oss/javascript/deepagents/interpreters).

## Subagents

To isolate detailed work and avoid context bloat, use subagents:

<CodeGroup>
  ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

  ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
</CodeGroup>

For more information, see [Subagents](/oss/javascript/deepagents/subagents).

## Backends

Tools for a deep agent can make use of virtual file systems to store, access, and edit files. By default, deep agents use a [`StateBackend`](https://reference.langchain.com/javascript/deepagents/backends/StateBackend).

If you are using [skills](#skills) or [memory](#memory), you must add the expected skill or memory files to the backend before creating the agent.

<Tabs>
  <Tab title="StateBackend">
    A thread-scoped filesystem backend stored in `langgraph` state.

    Files persist across turns within a thread (via your checkpointer) and are not shared across threads.

    ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
    import { createDeepAgent, StateBackend } from "deepagents";

    // By default we provide a StateBackend
    const agent = createDeepAgent();

    // Under the hood, it looks like
    const agent2 = createDeepAgent({
      backend: new StateBackend(),
    });
    ```
  </Tab>

  <Tab title="FilesystemBackend">
    The local machine's filesystem.

    <Warning>
      This backend grants agents direct filesystem read/write access.
      Use with caution and only in appropriate environments.
      For more information, see [`FilesystemBackend`](/oss/javascript/deepagents/backends#filesystembackend-local-disk).
    </Warning>

    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "google:gemini-3.6-flash",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "openai:gpt-5.5",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "anthropic:claude-sonnet-5",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "openrouter:z-ai/glm-5.2",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "fireworks:accounts/fireworks/models/glm-5p2",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "baseten:zai-org/GLM-5.2",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, FilesystemBackend } from "deepagents";

      const agent = createDeepAgent({
        model: "ollama:north-mini-code-1.0",
        backend: new FilesystemBackend({ rootDir: ".", virtualMode: true }),
      });
      ```
    </CodeGroup>

    <Tip>
      Wrap `FilesystemBackend` in a `CompositeBackend` to prevent internal agent data (offloaded tool results, conversation history) from being written to disk alongside your project files. See the [recommended pattern](/oss/javascript/deepagents/backends#filesystembackend-local-disk).
    </Tip>
  </Tab>

  <Tab title="LocalShellBackend">
    A filesystem with shell execution directly on the host. Provides filesystem tools plus the `execute` tool for running commands.

    <Warning>
      This backend grants agents direct filesystem read/write access **and** unrestricted shell execution on your host.
      Use with extreme caution and only in appropriate environments.
      For more information, see [`LocalShellBackend`](/oss/javascript/deepagents/backends#localshellbackend-local-shell).
    </Warning>

    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "google:gemini-3.6-flash",
        backend,
      });
      ```

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "openai:gpt-5.5",
        backend,
      });
      ```

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "anthropic:claude-sonnet-5",
        backend,
      });
      ```

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "openrouter:z-ai/glm-5.2",
        backend,
      });
      ```

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "fireworks:accounts/fireworks/models/glm-5p2",
        backend,
      });
      ```

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "baseten:zai-org/GLM-5.2",
        backend,
      });
      ```

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
      import { createDeepAgent, LocalShellBackend } from "deepagents";

      const backend = new LocalShellBackend({ workingDirectory: "." });

      const agent = createDeepAgent({
        model: "ollama:north-mini-code-1.0",
        backend,
      });
      ```
    </CodeGroup>
  </Tab>

  <Tab title="StoreBackend">
    A filesystem that provides long-term storage that is *persisted across threads*.

    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>

    <Note>
      When deploying to [LangSmith Deployment](/langsmith/deployment), omit the `store` parameter. The platform automatically provisions a store for your agent.
    </Note>

    <Tip>
      The `namespace` parameter controls data isolation. For multi-user deployments, always set a [namespace factory](/oss/javascript/deepagents/backends#namespace-factories) to isolate data per user or tenant.
    </Tip>
  </Tab>

  <Tab title="ContextHubBackend">
    Durable filesystem storage in a LangSmith Hub repo.

    For more details, see [`ContextHubBackend`](/oss/javascript/deepagents/backends#contexthubbackend).
  </Tab>

  <Tab title="CompositeBackend">
    A flexible backend where you can specify different routes in the filesystem to point towards different backends.

    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>
</Tabs>

For more information, see [Backends](/oss/javascript/deepagents/backends).

### Sandboxes

Sandboxes are specialized [backends](/oss/javascript/deepagents/backends) that run agent code in an isolated environment with their own filesystem and an `execute` tool for shell commands.
Use a sandbox backend when you want your deep agent to write files, install dependencies, and run commands without changing anything on your local machine.

You configure sandboxes by passing a sandbox backend to `backend` when creating your deep agent:

```typescript theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

For more information, see [Sandboxes](/oss/javascript/deepagents/sandboxes).

## Human-in-the-loop

Some tool operations may be sensitive and require human approval before execution.
You can configure the approval for each tool:

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

You can configure interrupt for agents and subagents on tool call as well as from within tool calls.
For more information, see [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop).

## Skills

You can use [skills](/oss/javascript/deepagents/overview) to provide your deep agent with new capabilities and expertise.
While [tools](/oss/javascript/deepagents/customization#tools) tend to cover lower level functionality like native file system actions, skills can contain detailed instructions on how to complete tasks, reference info, and other assets, such as templates.
These files are only loaded by the agent when the agent has determined that the skill is useful for the current prompt.
This progressive disclosure reduces the amount of tokens and context the agent has to consider upon startup.

For example skills, see [Deep Agents example skills](https://github.com/langchain-ai/deepagentsjs/tree/main/examples/skills).

To add skills to your deep agent, pass them as an argument to `create_deep_agent`:

<Tabs>
  <Tab title="StateBackend">
    ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
  </Tab>

  <Tab title="StoreBackend">
    ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
  </Tab>

  <Tab title="FilesystemBackend">
    ```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
  </Tab>
</Tabs>

## Memory

Use [`AGENTS.md` files](https://agents.md/) to provide extra context to your deep agent.

<Tip>
  To generate a repository wiki that coding agents discover through `AGENTS.md`, see [OpenWiki](/oss/openwiki/overview).
</Tip>

You can pass one or more file paths to the `memory` parameter when creating your deep agent:

<Tabs>
  <Tab title="StateBackend">
    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>

    <Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/5c07e925-9d8f-4792-b73c-9dbee94890cd/r" arrow horizontal>
      Open a public LangSmith run for this example.
    </Card>
  </Tab>

  <Tab title="StoreBackend">
    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>

    <Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/ff0f387f-c0c3-4456-8081-d6fc23504942/r" arrow horizontal>
      Open a public LangSmith run for this example.
    </Card>
  </Tab>

  <Tab title="Filesystem">
    <CodeGroup>
      ```ts Google theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenAI theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Anthropic theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts OpenRouter theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Fireworks theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Baseten theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

      ```ts Ollama theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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
    </CodeGroup>
  </Tab>
</Tabs>

## Structured output

Deep Agents support [structured output](/oss/javascript/langchain/structured-output).

You can set a desired structured output schema by passing it as the `responseFormat` argument to the call to `createDeepAgent()`.
When the model generates the structured data, it's captured, validated, and returned in the 'structuredResponse' key of the agent's state.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
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

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/e4587886-ebb6-4fbc-8b57-d41b27180770/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

For more information and examples, see [response format](/oss/javascript/langchain/structured-output#response-format).

## Advanced

`createDeepAgent` pre-assembles a middleware stack on top of `createAgent`. To build a fully custom agent—choosing exactly which capabilities to include—see [Configure the harness](/oss/javascript/langchain/agents#configure-the-harness).

***

<div className="source-links">
  <Callout icon="terminal-2">
    [Connect these docs](/use-these-docs) to your agent of choice via MCP for real-time answers.
  </Callout>

  <Callout icon="edit">
    [Edit this page on GitHub](https://github.com/langchain-ai/docs/edit/main/src/oss/deepagents/customization.mdx) or [file an issue](https://github.com/langchain-ai/docs/issues/new/choose).
  </Callout>
</div>
