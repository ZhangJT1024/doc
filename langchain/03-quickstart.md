# 快速开始

> 原文：https://docs.langchain.com/oss/javascript/langchain/quickstart
> LangChain (TypeScript) 官方文档中文整理

> ## 文档索引
> 获取完整文档索引请访问：https://docs.langchain.com/llms.txt
> 使用该文件可以在进一步浏览之前发现所有可用的页面。

# 快速开始

> 几分钟内构建你的第一个智能体

本快速开始将向你展示如何在几分钟内创建一个功能完备的 AI 智能体。

> **【提示】构建 LangChain 快速开始智能体（Prompt 模板）**
>
> 按照 LangChain 快速开始，在当前工作目录中构建一个基础的 LangChain 智能体。
>
> ## 步骤 1：阅读指南
>
> 检测本项目使用的是 Python 还是 TypeScript/JavaScript。抓取并遵循匹配的页面；将其作为包名、模型字符串和代码的唯一可信来源：
>
> * Python: [https://docs.langchain.com/oss/python/langchain/quickstart.md](https://docs.langchain.com/oss/python/langchain/quickstart.md)
> * TypeScript: [https://docs.langchain.com/oss/javascript/langchain/quickstart.md](https://docs.langchain.com/oss/javascript/langchain/quickstart.md)
>
> ## 步骤 2：安装依赖
>
> 使用本项目已使用的包管理器（`uv`、`pip`、`npm`、`pnpm`、`yarn` 或 `bun`）安装指南中的包。当项目已经固定版本时，优先固定当前稳定版本，而不是浮动的 `latest` 标签。
>
> ## 步骤 3：配置模型凭证
>
> 检查是否已设置受支持的提供商 API 密钥（例如 `OPENAI_API_KEY`、`GOOGLE_API_KEY` 或 `ANTHROPIC_API_KEY`）。如果没有设置，询问用户使用哪个提供商，然后停止并等待他们创建密钥并在 shell 或 `.env` 文件中设置。不要编造、硬编码或提交 API 密钥。
>
> ## 步骤 4：实现基础智能体
>
> 创建指南中的“构建一个基础智能体”示例：`create_agent`、一个简单的工具（如 `get_weather`）、一段简短的系统提示词，以及一次 `invoke` 调用。使用用户所选提供商的模型字符串。打印最终消息内容（Python 为 `.content_blocks`），以便用户验证运行结果。
>
> ## 步骤 5：可选的 LangSmith 追踪
>
> 询问用户是否需要追踪。如果需要，请他们自行设置 `LANGSMITH_TRACING=true` 和 `LANGSMITH_API_KEY`，然后重新运行。不要编造 LangSmith 密钥。
>
> ## 规则
>
> * 保持在本快速开始范围内。不要添加无关的框架、评估或生产部署。
> * 如指南所示，优先使用来自 `langchain.agents` / `langchain` 的 `create_agent`。
> * 当密钥、提供商选择或项目约定不明确时，询问而非猜测。

> **【提示】正在使用 AI 编码助手？**
>
> * 安装 [LangChain 文档 MCP 服务器](/use-these-docs)，让你的智能体能够访问最新的 LangChain 文档和示例。
>
>      **【提示】连接 LangChain 文档 MCP 服务器（Prompt 模板）**
>
>      将两个 LangChain 文档 MCP 服务器连接到我的编码智能体，以便它可以查找当前的 LangChain、LangGraph 和 LangSmith 文档及 API 参考。
>
>      要添加的服务器：
>
>      * `docs-langchain`: [https://docs.langchain.com/mcp](https://docs.langchain.com/mcp)
>      * `reference-langchain`: [https://reference.langchain.com/mcp](https://reference.langchain.com/mcp)
>
>      检测我正在使用哪个智能体或编辑器（Claude Code、Cursor、Codex CLI、Claude Desktop、Deep Agents Code、VS Code、Antigravity 或其他兼容 MCP 的客户端）。使用 [https://docs.langchain.com/use-these-docs.md](https://docs.langchain.com/use-these-docs.md) 中匹配的设置：
>
>      * Claude Code：对每个服务器执行 `claude mcp add --transport http`（默认项目范围；只有在我要求全局访问时才使用 `--scope user`）。
>      * Codex CLI：对每个服务器 URL 执行 `codex mcp add`。
>      * Cursor、Deep Agents Code、VS Code 或 Antigravity：按照该页面上为我的客户端显示的字段名，将两个条目合并到 MCP 设置 JSON 中。
>      * Claude Desktop：在 设置 > 连接器 下添加两个 URL。
>
>      不要编造替代的 MCP URL。配置完成后，确认两个服务器都已列出且可访问。
>
> * 安装 [LangChain Skills](https://github.com/langchain-ai/langchain-skills)，以提升你的智能体在 LangChain 生态任务上的表现。
>
>      **【提示】安装 LangChain Skills（Prompt 模板）**
>
>      为我的编码智能体安装 LangChain Skills，使其在 LangChain、LangGraph 和 Deep Agents 任务上表现更好。
>
>      使用 [https://github.com/langchain-ai/langchain-skills](https://github.com/langchain-ai/langchain-skills) 中的 Agent Skills 安装器：
>
>      ```bash
>      npx skills add langchain-ai/langchain-skills --skill '*' --yes
>      ```
>
>      如果我要求全局安装，请使用：
>
>      ```bash
>      npx skills add langchain-ai/langchain-skills --skill '*' --yes --global
>      ```
>
>      检测我正在使用哪个智能体或编辑器。如果我使用 Claude Code 并偏好插件方式，请遵循该仓库 README 中的市场安装方式（`/plugin marketplace add` 然后 `/plugin install`）。不要编造替代的技能包名称或安装 URL。安装后，确认这些技能对智能体可用。

## 安装依赖

安装以下包以便跟随本教程：

#### npm
```bash
npm install langchain @langchain/core
# Requires Node.js 22+
```

#### pnpm
```bash
pnpm add langchain @langchain/core
# Requires Node.js 22+
```

#### yarn
```bash
yarn add langchain @langchain/core
# Requires Node.js 22+
```

#### bun
```bash
bun add langchain @langchain/core
# Requires Bun v1.0.0+
```

## 设置 API 密钥

从 [任意受支持的模型提供商](/oss/javascript/integrations/providers/overview)（例如 Google Gemini 或 OpenAI）获取 API 密钥。

在 shell 或 `.env` 文件中设置 API 密钥，例如：

### OpenAI

#### Shell
```bash
export OPENAI_API_KEY="your-api-key"
```

#### .env
```bash
OPENAI_API_KEY=your-api-key
```

### Google Gemini

#### Shell
```bash
export GOOGLE_API_KEY="your-api-key"
```

#### .env
```bash
GOOGLE_API_KEY=your-api-key
```

### Claude (Anthropic)

#### Shell
```bash
export ANTHROPIC_API_KEY="your-api-key"
```

#### .env
```bash
ANTHROPIC_API_KEY=your-api-key
```

### OpenRouter

#### Shell
```bash
export OPENROUTER_API_KEY="your-api-key"
```

#### .env
```bash
OPENROUTER_API_KEY=your-api-key
```

### Fireworks

#### Shell
```bash
export FIREWORKS_API_KEY="your-api-key"
```

#### .env
```bash
FIREWORKS_API_KEY=your-api-key
```

### Baseten

#### Shell
```bash
export BASETEN_API_KEY="your-api-key"
```

#### .env
```bash
BASETEN_API_KEY=your-api-key
```

### Ollama

#### Shell
```bash
# Local: Ollama must be running (https://ollama.com)
# Cloud: Set your Ollama API key for hosted inference
export OLLAMA_API_KEY="your-api-key"
```

#### .env
```bash
# Local: Ollama must be running (https://ollama.com)
# Cloud: Set your Ollama API key for hosted inference
OLLAMA_API_KEY=your-api-key
```

### Azure

#### Shell
```bash
export AZURE_OPENAI_API_KEY="your-api-key"
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com"
export AZURE_OPENAI_DEPLOYMENT_NAME="your-deployment"
```

#### .env
```bash
AZURE_OPENAI_API_KEY=your-api-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
AZURE_OPENAI_DEPLOYMENT_NAME=your-deployment
```

### AWS Bedrock

#### Shell
```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-1"
```

#### .env
```bash
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
```

### HuggingFace

#### Shell
```bash
export HUGGINGFACEHUB_API_TOKEN="hf_..."
```

#### .env
```bash
HUGGINGFACEHUB_API_TOKEN=hf_...
```

#### Other

### 其他

查看受支持的 [聊天模型集成](/oss/javascript/integrations/chat) 的完整列表。

要加载 `.env` 文件，请使用 [`dotenv`](https://www.npmjs.com/package/dotenv)（`import "dotenv/config"`）。

> **【提示】使用 LangSmith Gateway**
>
> [LangSmith Gateway](/langsmith/llm-gateway) 通过 LangSmith 路由大多数主流提供商。你可以 [自带提供商密钥](/langsmith/llm-gateway-quickstart#2-make-a-call)，或使用 [Gateway Credits](/langsmith/llm-gateway-credits) 在无需提供商密钥的情况下访问模型。

## 构建一个基础智能体

首先创建一个可以回答问题并调用工具的简单智能体。本示例中的智能体使用选定的语言模型、一个基础天气函数作为工具，以及一段简单的提示词来引导其行为：

#### OpenAI
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Google Gemini
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "google-genai:gemini-2.5-flash-lite",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Claude (Anthropic)
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "claude-sonnet-4-6",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### OpenRouter
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "openrouter:anthropic/claude-sonnet-4-6",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Fireworks
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/qwen3p5-397b-a17b",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Baseten
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Ollama
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "ollama:devstral-2",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Azure
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "azure_openai:gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### AWS Bedrock
```ts
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "bedrock:gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

当你运行代码并提示智能体告诉你旧金山的天气时，智能体会使用该输入及其可用上下文。
智能体理解你是在询问旧金山这座城市的天气，因此用提供的城市名调用天气工具。

> **【提示】** 你可以通过更改模型名称并设置相应的 API 密钥来使用 [任意受支持的模型](/oss/javascript/integrations/providers/overview)。使用 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-langchain-quickstart) 追踪智能体内部发生的一切。按照 [追踪快速开始](/langsmith/trace-with-langchain) 进行设置。
>
> 我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的追踪记录、检测问题并提出修复建议。

## 构建一个真实世界的智能体

在下面的示例中，你将构建一个可以回答关于文本文件问题的研究智能体。
在此过程中，你将探索以下概念：

1. **详细的系统提示词**，以获得更好的智能体行为
2. **创建工具**，与外部数据集成
3. **模型配置**，以获得一致的响应
4. **对话记忆**，用于类聊天交互
5. **Deep Agents**，提供内置功能
6. **测试**你的智能体

#### 定义系统提示词

### 第 1 步：定义系统提示词

系统提示词定义智能体的角色和行为。保持具体且可操作：

```ts
const SYSTEM_PROMPT = `You are a literary data assistant.

## Capabilities

- \`fetch_text_from_url\`: loads document text from a URL into the conversation.
Do not guess line counts or positions—ground them in tool results from the saved file.`;
```

#### 创建工具

### 第 2 步：创建工具

[工具（Tools）](/oss/javascript/langchain/tools) 让模型可以通过调用你定义的函数与外部系统交互。
工具可以依赖 [运行时上下文（runtime context）](/oss/javascript/langchain/runtime)，也可以与 [智能体记忆](/oss/javascript/langchain/short-term-memory) 交互。

本示例使用一个工具从给定 URL 加载文档：

```ts
import { tool } from "@langchain/core/tools";
import { createAgent, initChatModel } from "langchain";
import { z } from "zod";

const fetchTextFromUrl = tool(
    async ({ url }: { url: string }): Promise<string> => {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), 120_000);
        try {
            const resp = await fetch(url, {
                headers: {
                "User-Agent": "Mozilla/5.0 (compatible; quickstart-research/1.0)",
                },
                signal: controller.signal,
            });
            if (!resp.ok) {
                return `Fetch failed: HTTP ${resp.status} ${resp.statusText}`;
            }
            return await resp.text();
        } catch (e) {
            const msg = e instanceof Error ? e.message : String(e);
            return `Fetch failed: ${msg}`;
        } finally {
            clearTimeout(timeoutId);
        }
    },
    {
        name: "fetch_text_from_url",
        description: "Fetch the document from a URL.",
        schema: z.object({ url: z.string().url() }),
    },
);
```

> **【提示】** [Zod](https://zod.dev/) 是一个用于验证和解析预定义 schema 的库。你可以用它来定义工具输入 schema，确保智能体只以正确的参数调用工具。
>
> 或者，你可以将 `schema` 属性定义为 [JSON schema](https://json-schema.org/overview/what-is-jsonschema) 对象。请记住，JSON schema **不会** 在运行时被验证。
>
> **示例：为工具输入使用 JSON schema**
>
> ```ts
> import { tool } from "@langchain/core/tools";
>
> const fetchTextFromUrl = tool(
> async ({ url }: { url: string }): Promise<string> => {
>     const controller = new AbortController();
>     const timeoutId = setTimeout(() => controller.abort(), 120_000);
>     try {
>     const resp = await fetch(url, {
>         headers: {
>         "User-Agent": "Mozilla/5.0 (compatible; quickstart-research/1.0)",
>         },
>         signal: controller.signal,
>     });
>     if (!resp.ok) {
>         return `Fetch failed: HTTP ${resp.status} ${resp.statusText}`;
>     }
>     return await resp.text();
>     } catch (e) {
>     const msg = e instanceof Error ? e.message : String(e);
>     return `Fetch failed: ${msg}`;
>     } finally {
>     clearTimeout(timeoutId);
>     }
> },
> {
>     name: "fetch_text_from_url",
>     description: "Fetch the document from a URL.",
>     schema: {
>     type: "object",
>     properties: {
>         url: {
>         type: "string",
>         description: "The URL of the document to fetch.",
>         format: "uri",
>         },
>     },
>     required: ["url"],
>     },
> },
> );
> ```

#### 配置你的模型

### 第 3 步：配置你的模型

用适合你用例的正确参数设置你的 [语言模型](/oss/javascript/langchain/models)。例如：

#### OpenAI
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("gpt-5.5", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### Google Gemini
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("gemini-3.1-pro-preview", {
  modelProvider: "google-genai",
  temperature: 0.5,
  timeout: 600_000,
  maxTokens: 25000,
});
```

#### Claude (Anthropic)
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("claude-sonnet-4-6", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### OpenRouter
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("openrouter:anthropic/claude-sonnet-4-6", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### Fireworks
```ts
import { initChatModel } from "langchain";

const model = await initChatModel(
  "fireworks:accounts/fireworks/models/qwen3p5-397b-a17b",
  { temperature: 0.5, timeout: 300, maxTokens: 25000 }
);
```

#### Baseten
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("baseten:zai-org/GLM-5.2", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### Ollama
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("ollama:devstral-2", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### Azure
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("azure_openai:gpt-5.5", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

#### AWS Bedrock
```ts
import { initChatModel } from "langchain";

const model = await initChatModel("bedrock:gpt-5.5", {
  temperature: 0.5,
  timeout: 300,
  maxTokens: 25000,
});
```

根据所选模型和提供商的不同，初始化参数可能有所差异；详情请参阅它们的参考页面。

#### 添加记忆

### 第 4 步：添加记忆

为你的智能体添加 [记忆（memory）](/oss/javascript/langchain/short-term-memory)，以便在多次交互之间保持状态。这使智能体能够记住之前的对话和上下文。

```ts
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();
```

> **【信息】** 在生产环境中，请使用将消息历史保存到数据库的持久化 checkpointer。
> 更多细节请参阅 [添加和管理记忆](/oss/javascript/langgraph/add-memory#manage-short-term-memory)。

#### 创建并运行智能体

### 第 5 步：创建并运行智能体

现在用所有组件组装你的智能体并运行它。

创建智能体有两种不同的框架：LangChain 智能体和 deep agents。
LangChain 和 deep agents 都为你提供对工具、记忆等的细粒度控制。
两者的主要区别在于 deep agents 已经内置了一系列常用能力，例如规划、文件系统工具和子智能体。

当你希望以最小配置获得最大能力时，使用 deep agents；当你需要细粒度控制时，选择 LangChain 智能体。

为了在本步骤中对比两者，请安装 `deepagents` 包：

#### npm
```bash
npm install deepagents
```

#### pnpm
```bash
pnpm add deepagents
```

#### yarn
```bash
yarn add deepagents
```

#### bun
```bash
bun add deepagents
```

> **【警告】** 本示例会将《了不起的盖茨比》全文发送给模型，可能需要一到两分钟才能响应。你可以在下一步查看示例输出。

让我们两种都试试：

```ts
async function main() {
    const agent = createAgent({
        model,
        tools: [fetchTextFromUrl],
        systemPrompt: SYSTEM_PROMPT,
        checkpointer,
    });

    const deepAgent = createDeepAgent({
        model,
        tools: [fetchTextFromUrl],
        systemPrompt: SYSTEM_PROMPT,
        checkpointer,
    });

    const content = `Project Gutenberg hosts a full plain-text copy of F. Scott Fitzgerald's The Great Gatsby.
    URL: https://www.gutenberg.org/files/64317/64317-0.txt

    Answer as much as you can:

    1) How many lines in the complete Gutenberg file contain the substring \`Gatsby\` (count lines, not occurrences within a line, each line ends with a line break).
    2) The 1-based line number of the first line in the file that contains \`Daisy\`.
    3) A two-sentence neutral synopsis.

    Do your best on (1) and (2). If at any point you realize you cannot **verify** an exact answer with
    your available tools and reasoning, do not fabricate numbers: use \`null\` for that field and spell out
    the limitation in \`how_you_computed_counts\`. If you encounter any errors please report what the error was and what the error message was.`;

    console.log("Running createAgent...");
    const agentResult = await agent.invoke(
        { messages: [{ role: "user", content }] },
        { configurable: { thread_id: "great-gatsby-lc" } },
    );
    console.log("Running createDeepAgent...");
    const deepAgentResult = await deepAgent.invoke(
        { messages: [{ role: "user", content }] },
        { configurable: { thread_id: "great-gatsby-da" } },
    );

    const agentMessages = agentResult.messages;
    const deepMessages = deepAgentResult.messages;
    console.log("\ncreateAgent:");
    console.log(agentMessages[agentMessages.length - 1]!.contentBlocks);
    console.log("\ncreateDeepAgent:");
    console.log(deepMessages[deepMessages.length - 1]!.contentBlocks);
}

main().catch((err) => {
    console.error(err);
    process.exitCode = 1;
});
```

下面这个可展开区域包含了一个可运行的完整脚本：

**完整示例代码**

```ts
import { MemorySaver } from "@langchain/langgraph";
import { createDeepAgent } from "deepagents";
import { tool } from "@langchain/core/tools";
import { createAgent, initChatModel } from "langchain";
import { z } from "zod";
const SYSTEM_PROMPT = `You are a literary data assistant.

## Capabilities

- \`fetch_text_from_url\`: loads document text from a URL into the conversation.
Do not guess line counts or positions—ground them in tool results from the saved file.`;

const fetchTextFromUrl = tool(
    async ({ url }: { url: string }): Promise<string> => {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), 120_000);
        try {
            const resp = await fetch(url, {
                headers: {
                    "User-Agent": "Mozilla/5.0 (compatible; quickstart-research/1.0)",
                },
                signal: controller.signal,
            });
            if (!resp.ok) {
                return `Fetch failed: HTTP ${resp.status} ${resp.statusText}`;
            }
            return await resp.text();
        } catch (e) {
            const msg = e instanceof Error ? e.message : String(e);
            return `Fetch failed: ${msg}`;
        } finally {
           clearTimeout(timeoutId);
        }
    },
    {
        name: "fetch_text_from_url",
        description: "Fetch the document from a URL.",
        schema: z.object({ url: z.string().url() }),
    },
);

const model = await initChatModel("gemini-3.1-pro-preview", {
    modelProvider: "google-genai",
    temperature: 0.5,
    timeout: 600_000,
    maxTokens: 25000,
    streaming: true,
});

const checkpointer = new MemorySaver();

async function main() {
    const agent = createAgent({
        model,
        tools: [fetchTextFromUrl],
        systemPrompt: SYSTEM_PROMPT,
        checkpointer,
    });

    const deepAgent = createDeepAgent({
        model,
        tools: [fetchTextFromUrl],
        systemPrompt: SYSTEM_PROMPT,
        checkpointer,
    });

    const content = `Project Gutenberg hosts a full plain-text copy of F. Scott Fitzgerald's The Great Gatsby.
    URL: https://www.gutenberg.org/files/64317/64317-0.txt

    Answer as much as you can:

    1) How many lines in the complete Gutenberg file contain the substring \`Gatsby\` (count lines, not occurrences within a line, each line ends with a line break).
    2) The 1-based line number of the first line in the file that contains \`Daisy\`.
    3) A two-sentence neutral synopsis.

    Do your best on (1) and (2). If at any point you realize you cannot **verify** an exact answer with
    your available tools and reasoning, do not fabricate numbers: use \`null\` for that field and spell out
    the limitation in \`how_you_computed_counts\`. If you encounter any errors please report what the error was and what the error message was.`;

    console.log("Running createAgent...");
    const agentResult = await agent.invoke(
        { messages: [{ role: "user", content }] },
        { configurable: { thread_id: "great-gatsby-lc" } },
    );
    console.log("Running createDeepAgent...");
    const deepAgentResult = await deepAgent.invoke(
        { messages: [{ role: "user", content }] },
        { configurable: { thread_id: "great-gatsby-da" } },
    );

    const agentMessages = agentResult.messages;
    const deepMessages = deepAgentResult.messages;
    console.log("\ncreateAgent:");
    console.log(agentMessages[agentMessages.length - 1]!.contentBlocks);
    console.log("\ncreateDeepAgent:");
    console.log(deepMessages[deepMessages.length - 1]!.contentBlocks);
}

main().catch((err) => {
    console.error(err);
    process.exitCode = 1;
});
```

#### 查看结果

### 第 6 步：查看结果

结果会因模型和执行情况而异。

#### LangChain agents

**LangChain 智能体**

```txt
**1) Number of lines containing `Gatsby`:** `null`

**2) First line containing `Daisy`:** `null`

**3) Synopsis:**
The Great Gatsby follows the mysterious millionaire Jay Gatsby and his obsession with reuniting with his former lover, Daisy Buchanan, as narrated by his neighbor Nick Carraway. Set against the backdrop of the Roaring Twenties on Long Island, the novel explores themes of wealth, class, and the elusive nature of the American Dream.

**how_you_computed_counts:**
I successfully fetched the full text of the eBook using the `fetch_text_from_url` tool. However, because I do not have access to a code execution environment (like Python) or text-processing tools (like `grep`), I cannot deterministically split the text by line breaks, iterate through the thousands of lines, and verify the exact line numbers or match counts. LLMs cannot reliably perform exact line-counting or indexing over massive texts within their context window without external computational tools. As instructed, rather than fabricating or guessing a number, I have output `null` for the exact counts and positions.
```

#### Deep agents

**Deep agents**

```txt
Based on the text fetched directly from the Gutenberg URL and analyzed using filesystem search tools, here are the answers to your questions:

**1) Lines containing the substring `Gatsby`**
**258** lines contain the exact substring `Gatsby`.

**2) First line containing `Daisy`**
Line **181** is the first line in the file that contains the exact substring `Daisy`.
*(For context, the line reads: "Buchanans. Daisy was my second cousin once removed, and I’d known Tom")*

**3) Two-sentence neutral synopsis**
*The Great Gatsby* follows the mysterious millionaire Jay Gatsby and his obsessive pursuit to reunite with his former lover, Daisy Buchanan, in 1920s Long Island. The story is narrated by Nick Carraway, who observes the tragic consequences of Gatsby's relentless ambition and the shallow materialism of the era's wealthy elite.

***

**How counts were computed:**
When fetching the document from the URL, the file was too large for the standard output and was automatically saved to the local filesystem by the system (`/large_tool_results/x246ax2x`). I then used the `grep` tool to search the saved file for the exact literal substrings `Gatsby` and `Daisy`. The `grep` tool returned every matching line along with its 1-based line number. I manually counted the exact number of lines returned for `Gatsby` (which totaled 258) and identified the first line number returned for `Daisy` (which was 181). I also verified there were no uppercase variations (`GATSBY` or `DAISY`) that would have been missed. No errors were encountered during this process.
```

如果你查看两个标签页的输出，会注意到 LangChain 智能体提供了答案，但它们只是估计值。该智能体缺少回答此问题的工具。你也可能会遇到提示词过长的错误。

而 deep agent 则可以：

1. **规划其方法**，使用内置的 [`write_todos`](/oss/javascript/deepagents/harness#task-planning) 工具来拆解研究任务。
2. **加载文件**，通过调用 `fetch_text_from_url` 工具来收集信息。
3. **管理上下文**，使用文件系统工具（[`grep`](/oss/javascript/deepagents/harness#virtual-filesystem-access) 和 [`read_file`](/oss/javascript/deepagents/harness#virtual-filesystem-access)）。
4. **生成子智能体**，按需将复杂子任务委派给专门的子智能体。

对于 LangChain 智能体，你必须实现更多能力才能获得类似的服务水平，并且可以在此过程中按需对它们进行自定义。

## 追踪智能体调用

你用 LangChain 构建的大多数有趣应用都会对 LLM 进行多次调用。随着这些应用变得越来越复杂，能够检查智能体内部到底发生了什么就变得很重要。做到这一点最好的方式是使用 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-langchain-quickstart)。

注册一个 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-langchain-quickstart) 账号并设置以下内容以开始记录追踪：

```shell
export LANGSMITH_TRACING="true"
export LANGSMITH_API_KEY="..."
```

设置完成后，再次运行你的脚本，然后在 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-langchain-quickstart) 上检查智能体调用期间发生了什么。

> **【提示】** 要了解有关使用 LangSmith 追踪智能体的更多信息，请参阅 [LangSmith 文档](/langsmith/trace-with-langchain)。
>
> 我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的追踪记录、检测问题并提出修复建议。

## 后续步骤

现在你拥有了能够做到以下事情的智能体：

* **理解上下文** 并记住对话
* **智能地使用工具**
* **以一致的格式提供结构化响应**
* **通过上下文处理用户特定信息**
* **在多次交互之间保持对话状态**
* **规划、研究和综合**（仅 deep agents）

继续阅读：

* **LangChain 智能体**：[添加和管理记忆](/oss/javascript/langgraph/add-memory#manage-short-term-memory)、[部署到生产环境](/oss/javascript/langgraph/deploy)
* **Deep Agents**：[自定义选项](/oss/javascript/deepagents/customization)、[持久化记忆](/oss/javascript/deepagents/memory)、[部署到生产环境](/oss/javascript/langgraph/deploy)

***
