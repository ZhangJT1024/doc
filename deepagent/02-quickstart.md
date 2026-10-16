# 快速开始

> 原文：Quickstart
> 来源：https://docs.langchain.com/oss/javascript/deepagents/quickstart

---

本指南将引导你创建第一个具有文件系统工具和子代理能力的 deep agent。你将构建一个能够开展研究并撰写报告的研究型 agent。

> **Build the Deep Agents research quickstart**（可复制的提示词）：
>
> 通过遵循 Deep Agents 快速开始，在此工作目录中构建一个 Deep Agents 研究 agent。
>
> ## 步骤 1：阅读指南
>
> 检测此项目使用 Python 还是 TypeScript/JavaScript。获取并遵循对应的页面；将其视为包名、模型字符串、搜索工具设置和代码的唯一事实来源：
>
> * Python：[https://docs.langchain.com/oss/python/deepagents/quickstart.md](https://docs.langchain.com/oss/python/deepagents/quickstart.md)
> * TypeScript：[https://docs.langchain.com/oss/javascript/deepagents/quickstart.md](https://docs.langchain.com/oss/javascript/deepagents/quickstart.md)
>
> ## 步骤 2：安装依赖
>
> 使用此项目已经在用的包管理器安装 `deepagents`（在 TypeScript 上还有 `langchain` / `@langchain/core`）。只有当用户没有使用 Google、OpenAI 或 Anthropic 内置的提供商搜索工具时，才添加 Tavily。
>
> ## 步骤 3：配置模型凭据
>
> 检查是否存在受支持的提供商 API key（例如 `GOOGLE_API_KEY`、`OPENAI_API_KEY` 或 `ANTHROPIC_API_KEY`）。如果都没有设置，询问用户要使用哪个提供商，然后停止并等待他们创建 key 并在 shell 或 `.env` 文件中设置。不要杜撰、硬编码或提交 API key。如果他们需要 Tavily，请他们以同样方式设置 `TAVILY_API_KEY`。
>
> ## 步骤 4：实现研究 agent
>
> 按顺序遵循快速开始步骤：
>
> 1. 创建互联网搜索工具（当选定的模型支持时，优先使用提供商的内置搜索工具；否则使用 Tavily）。
> 2. 使用搜索工具、来自指南的 `provider:model` 字符串（或已初始化的模型）以及页面上展示的研究系统提示词调用 `create_deep_agent`。
> 3. 可选：通过让用户自行设置 `LANGSMITH_TRACING=true` 和 `LANGSMITH_API_KEY` 来启用 LangSmith 追踪。
> 4. 在来自指南的示例研究查询上运行 agent，并打印最终响应。
>
> ## 规则
>
> * 保持在本快速开始范围内。不要添加 Managed Deep Agents 部署、evals 或无关框架。
> * 在可用时优先使用提供商内置的 web search；仅在需要时使用 Tavily。
> * 当密钥、提供商选择或项目约定不明确时，询问而不是猜测。

> **提示**：**使用 AI 编码助手？**
>
> * 安装 [LangChain Docs MCP servers](/use-these-docs)，让你的 agent 能够访问最新的 LangChain 文档和示例。
>
> * 安装 [LangChain Skills](https://github.com/langchain-ai/langchain-skills)，以提升你的 agent 在 LangChain 生态任务上的表现。

> **Connect LangChain docs MCP servers**（可复制的提示词）：
>
> 将两个 LangChain 文档 MCP servers 连接到我的编码 agent，以便它能查询最新的 LangChain、LangGraph 和 LangSmith 文档及 API 参考。
>
> 要添加的 servers：
>
> * `docs-langchain`：[https://docs.langchain.com/mcp](https://docs.langchain.com/mcp)
> * `reference-langchain`：[https://reference.langchain.com/mcp](https://reference.langchain.com/mcp)
>
> 检测我正在使用哪个 agent 或编辑器（Claude Code、Cursor、Codex CLI、Claude Desktop、Deep Agents Code、VS Code、Antigravity 或其他 MCP 兼容客户端）。使用 [https://docs.langchain.com/use-these-docs.md](https://docs.langchain.com/use-these-docs.md) 中匹配的设置：
>
> * Claude Code：对每个 server 使用 `claude mcp add --transport http`（默认项目范围；只有当我要求全局访问时才使用 `--scope user`）。
> * Codex CLI：对每个 server URL 使用 `codex mcp add`。
> * Cursor、Deep Agents Code、VS Code 或 Antigravity：使用该页面为我的客户端展示的字段名，将两个条目合并到 MCP 设置 JSON 中。
> * Claude Desktop：在 Settings > Connectors 下添加两个 URL。
>
> 不要杜撰替代的 MCP URL。配置完成后，确认两个 servers 都已列出且可访问。

> **Install LangChain Skills**（可复制的提示词）：
>
> 为我的编码 agent 安装 LangChain Skills，使其在 LangChain、LangGraph 和 Deep Agents 任务上表现更好。
>
> 使用来自 [https://github.com/langchain-ai/langchain-skills](https://github.com/langchain-ai/langchain-skills) 的 Agent Skills 安装器：
>
> ```bash
> npx skills add langchain-ai/langchain-skills --skill '*' --yes
> ```
>
> 如果我改为要求全局安装，则使用：
>
> ```bash
> npx skills add langchain-ai/langchain-skills --skill '*' --yes --global
> ```
>
> 检测我正在使用哪个 agent 或编辑器。如果我使用 Claude Code 并且偏好插件路径，请遵循该仓库 README 中的 marketplace 安装方式（先 `/plugin marketplace add`，再 `/plugin install`）。不要杜撰替代的 skill 包名或安装 URL。安装后，确认这些 skills 对 agent 可用。

## 前置条件

在开始之前，请确保你拥有来自模型提供商（例如 Gemini、Anthropic、OpenAI）的 API key。

> **注意**：Deep Agents 需要一个支持[工具调用](/oss/javascript/langchain/models#tool-calling)的模型。关于如何配置你的模型，请参阅[定制](/oss/javascript/deepagents/customization#model)。

## 步骤 1：安装依赖

#### npm

```bash
npm install deepagents langchain @langchain/core
```

#### yarn

```bash
yarn add deepagents langchain @langchain/core
```

#### pnpm

```bash
pnpm add deepagents langchain @langchain/core
```

> **注意**：Google、OpenAI 和 Anthropic 都提供内置的 web search 工具：无需额外包或 API key。如果你使用其他提供商或偏好用 [Tavily](https://tavily.com/) 进行搜索，请一并安装 Tavily 包：
>
> ```bash
> npm install @langchain/tavily
> ```

## 步骤 2：设置你的 API keys

#### Google

```bash
export GOOGLE_API_KEY="your-api-key"
```

#### OpenAI

```bash
export OPENAI_API_KEY="your-api-key"
```

#### Anthropic

```bash
export ANTHROPIC_API_KEY="your-api-key"
```

#### OpenRouter

```bash
export OPENROUTER_API_KEY="your-api-key"
export TAVILY_API_KEY="your-tavily-api-key"
```

#### Fireworks

```bash
export FIREWORKS_API_KEY="your-api-key"
export TAVILY_API_KEY="your-tavily-api-key"
```

#### Baseten

```bash
export BASETEN_API_KEY="your-api-key"
export TAVILY_API_KEY="your-tavily-api-key"
```

#### Ollama

```bash
# Local: Ollama must be running on your machine
# Cloud: Set your Ollama API key for hosted inference
export OLLAMA_API_KEY="your-api-key"
export TAVILY_API_KEY="your-tavily-api-key"
```

#### Other

```bash
# Set the API key for your provider
export <PROVIDER>_API_KEY="your-api-key"
export TAVILY_API_KEY="your-tavily-api-key"
```

Deep Agents 可与任何 [LangChain chat model](/oss/javascript/deepagents/models#supported-models) 配合使用。为你的提供商设置 API key。

> **提示**：**使用 LangSmith Gateway**
>
> [LangSmith Gateway](/langsmith/llm-gateway) 通过 LangSmith 路由大多数主流提供商。你可以[带上自己的提供商密钥](/langsmith/llm-gateway-quickstart#send-a-request)，或使用 [Gateway Credits](/langsmith/llm-gateway-credits) 在没有提供商密钥的情况下访问模型。

## 步骤 3：创建搜索工具

Google、OpenAI 和 Anthropic 提供在服务端运行的内置 web search 工具：无需额外包或 API key。直接向 `create_deep_agent` 传入一个提供商工具 dict。

#### Provider search（推荐）

##### Google

```ts
import { createDeepAgent } from "deepagents";

// Google's built-in search — no extra install or API key needed
const internetSearch = { googleSearch: {} };
```

##### OpenAI

```ts
import { createDeepAgent } from "deepagents";

// OpenAI's built-in web search — no extra install or API key needed
const internetSearch = { type: "web_search_preview" };
```

##### Anthropic

```ts
import { createDeepAgent } from "deepagents";

// Anthropic's built-in web search — no extra install or API key needed
const internetSearch = { type: "web_search_20250305", name: "web_search" };
```

#### Tavily（任意提供商）

```ts
import { tool } from "langchain";
import { TavilySearch } from "@langchain/tavily";
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
      maxResults: z
        .number()
        .optional()
        .default(5)
        .describe("Maximum number of results to return"),
      topic: z
        .enum(["general", "news", "finance"])
        .optional()
        .default("general")
        .describe("Search topic category"),
      includeRawContent: z
        .boolean()
        .optional()
        .default(false)
        .describe("Whether to include raw content"),
    }),
  },
);
```

## 步骤 4：创建 deep agent

将你的搜索工具和模型传给 `create_deep_agent`。传入 `provider:model` 格式的 `model` 字符串，或一个[已初始化的模型实例](/oss/javascript/deepagents/models#configure-model-parameters)。有关所有提供商，请参阅[支持的模型](/oss/javascript/deepagents/models#supported-models)；有关经过测试的推荐，请参阅[建议的模型](/oss/javascript/deepagents/models#suggested-models)。

#### Google

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

// System prompt to steer the agent to be an expert researcher
const researchInstructions = `You are an expert researcher. Your job is to conduct thorough research and then write a polished report.

You have access to an internet search tool as your primary means of gathering information.

## \`internet_search\`

Use this to run an internet search for a given query. You can specify the max number of results to return, the topic, and whether raw content should be included.
`;

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [internetSearch],
  systemPrompt: researchInstructions,
});
```

## 步骤 5：设置 LangSmith 追踪

[LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-deepagents-quickstart) 让你能够洞察 agent 的执行，从而查看工具调用、子代理委派和 LLM 响应。

在 [smith.langchain.com](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-deepagents-quickstart) 注册，创建一个 API key，并设置以下环境变量：

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="your-langsmith-api-key"
```

## 步骤 6：运行 agent

```ts
const result = await agent.invoke({
  messages: [{ role: "user", content: "What is langgraph?" }],
});

// Print the agent's response
console.log(result.messages[result.messages.length - 1].content);
```

## 它是如何工作的？

你的 deep agent 会自动：

1. **开展研究**：调用 `internet_search` 工具来收集信息。
2. **管理上下文**：使用文件系统工具（[`write_file`](/oss/javascript/deepagents/overview#virtual-filesystem-access)、[`read_file`](/oss/javascript/deepagents/overview#virtual-filesystem-access)）卸载大型搜索结果。
3. **生成子代理**：按需将复杂的子任务委派给专门的子代理。
4. **综合成报告**：将发现汇编为一份连贯的响应。

要使用 `write_todos` 添加结构化任务规划，可通过 [`TodoListMiddleware`](https://reference.langchain.com/javascript/langchain/index/todoListMiddleware) 选择启用。请参阅[任务规划](/oss/javascript/deepagents/overview#task-planning)。

## 示例

关于你可以用 Deep Agents 构建的 agents、模式和应用程序，请参阅[示例](https://github.com/langchain-ai/deepagents/tree/main/examples)。

## 流式传输

Deep Agents 使用 LangGraph 内置了[流式传输](/oss/javascript/langchain/event-streaming)，可从 agent 执行中获得实时更新。
这让你能够逐步观察输出，并审查和调试 agent 及子代理的工作，例如工具调用、工具结果和 LLM 响应。

## 下一步

现在你已经构建了第一个 deep agent：

* **定制你的 agent**：了解[定制选项](/oss/javascript/deepagents/customization)，包括自定义系统提示词、工具和子代理。
* **添加长期记忆**：启用跨对话的[持久化记忆](/oss/javascript/deepagents/memory)。
* **部署到生产环境**：使用 [Managed Deep Agents](/langsmith/javascript/managed-deep-agents-overview) 在 LangSmith 中创建、运行和运维 deep agents。
* **测试与评估**：使用 [LangSmith 评估](/langsmith/evaluation-quickstart)运行自动化测试，并针对数据集衡量你的 agent 性能。
