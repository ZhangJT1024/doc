# Deep Agents 中的上下文工程

> 原文：Context engineering in Deep Agents
> 来源：https://docs.langchain.com/oss/javascript/deepagents/context-engineering

---

上下文工程（context engineering）是指以正确的格式提供正确的信息和工具，让 deep agent 能够可靠地完成任务。

Deep agents 可以访问多种上下文。有些来源在启动时就提供给 agent；其他来源在运行期间才可用，例如用户输入。Deep agents 内置了在长时间运行的会话中管理上下文的机制。

本页概述了你的 deep agent 可以访问和管理的各种上下文。

> **提示**：刚接触上下文工程？请参阅[概念性概述](/oss/javascript/concepts/context)，了解不同类型的上下文以及何时使用它们。

## 上下文的类型

| 上下文类型 | 你控制的内容 | 作用范围 |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------- | --------------------------------- |
| **[输入上下文（Input context）](#input-context)** | 启动时进入 agent prompt 的内容（system prompt、memory、skills） | 静态，每次运行都会应用 |
| **[运行时上下文（Runtime context）](#runtime-context)** | 调用时传入的静态配置（用户元数据、API key、连接） | 每次运行，传播到 subagents |
| **[上下文压缩（Context compression）](#context-compression)** | 内置的卸载与摘要机制，使上下文保持在窗口限制内 | 自动，在接近限制时触发 |
| **[上下文隔离（Context isolation）](#context-isolation-with-subagents)** | 使用 subagents 隔离繁重工作，只向主 agent 返回结果 | 每个 subagent，在委派时 |
| **[长期记忆（Long-term memory）](#long-term-memory)** | 使用虚拟文件系统跨 thread 持久化存储 | 跨对话持久 |

## 输入上下文

输入上下文是在启动时提供给 deep agent 的信息，它会成为其 system prompt 的一部分。最终的 prompt 由多个来源组成：

> **系统提示词（System prompt）**（[链接](#system-prompt)）：你提供的自定义指令加上内置的 agent 引导。

> **记忆（Memory）**（[链接](#memory)）：配置后会始终加载的持久化 `AGENTS.md` 文件。

> **技能（Skills）**（[链接](#skills)）：在相关时按需加载的能力（渐进式披露）。

> **工具提示词（Tool prompts）**（[链接](#tool-prompts)）：使用内置工具或自定义工具的说明。

### 系统提示词

你的自定义 system prompt 会被前置到内置 system prompt（其中包含针对文件系统工具和 subagents 的引导）之前。用它来定义 agent 的角色、行为和知识：

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  systemPrompt: `You are a research assistant specializing in scientific literature.
  Always cite sources. Use subagents for parallel research on different topics.`,
});
```

`systemPrompt` 参数是静态的，这意味着它不会随每次调用而变化。对于某些用例，你可能需要动态 prompt：例如，告诉模型「You have admin access」还是「You have read-only access」，或者从[长期记忆](#long-term-memory)注入用户偏好，例如「User prefers concise responses」。如果你的 prompt 依赖上下文或 `runtime.store`，请使用 `dynamicSystemPromptMiddleware` 来构建上下文感知的指令。你的 middleware 可以读取 `request.runtime.context` 和 `request.runtime.store`。关于 [Deep Agents stack](/oss/javascript/deepagents/customization#deep-agents-stack) 以及如何添加[自定义 middleware](/oss/javascript/langchain/middleware)，请参阅[自定义（Customization）](/oss/javascript/deepagents/customization#middleware)。示例请参阅 [LangChain 上下文工程](/oss/javascript/langchain/context-engineering#system-prompt)指南。

当只有工具使用上下文或 `runtime.store` 时，你**不**需要 middleware；工具会直接接收到 `runtime` 对象（包括 `runtime.context` 和 `runtime.store`）。只有当 system prompt 本身必须随请求而变化时，才添加 middleware。

> **提示**：要为特定 provider 或模型调整组装后的 system prompt，请使用 [harness 配置档案](/oss/javascript/deepagents/profiles#harness-profiles)：`base_system_prompt` 会彻底替换基础 prompt，而 `system_prompt_suffix` 会在其后追加。

### 记忆

记忆文件（[`AGENTS.md`](https://agents.md/)）提供**始终加载**到 system prompt 中的持久化上下文。将记忆用于应适用于每次对话的项目约定、用户偏好和关键准则：

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  memory: ["/project/AGENTS.md", "~/.deepagents/preferences.md"],
});
```

与 skills 不同，memory 总是被注入——没有渐进式披露。保持 memory 精简以避免上下文过载；将详细工作流和领域特定内容交给 [skills](/oss/javascript/deepagents/skills)。配置细节请参阅[记忆（Memory）](/oss/javascript/deepagents/customization#memory)。

要生成一个供编码 agent 通过 `AGENTS.md` 发现的仓库 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

### 技能

Skills 提供**按需**能力。agent 在启动时读取每个 `SKILL.md` 的 frontmatter，然后仅在判断技能相关时才加载完整的技能内容。这减少了 token 消耗，同时仍能提供专门化的工作流：

#### Google

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  skills: ["/skills/research/", "/skills/web-search/"],
});
```

让每个 skill 专注于单一工作流或领域；宽泛或重叠的 skills 会稀释相关性，并在加载时使上下文膨胀。在一个 skill 内，保持主内容精简，并将详细参考资料移到被 skill 文件引用的单独文件中。将始终相关的约定放入 [memory](#memory)。编写和配置请参阅 [Skills](/oss/javascript/deepagents/skills)。

### 工具提示词

[工具（Tool）](/oss/javascript/langchain/tools) prompt 是塑造模型如何使用工具的指令。所有工具都会暴露模型在其 prompt 中看到的元数据——通常是一个 schema 和一个 description。你通过 `tools` 参数传入的工具会将这些工具元数据（schema 和 description）暴露给模型。deep agent 的内置工具打包在 [Deep Agents stack](/oss/javascript/deepagents/customization#deep-agents-stack) 中，通常还会更新 system prompt，为这些工具提供更多引导。

**内置工具**：添加 harness 能力（filesystem、subagents 以及可选的 planning）的 middleware 会自动向 system prompt 追加工具特定的指令，从而创建解释如何有效使用这些工具的工具 prompt。完整列表请参阅[自定义（Customization）](/oss/javascript/deepagents/customization#middleware)：

* 文件系统 prompt：`ls`、`read_file`、`write_file`、`edit_file`、`glob`、`grep` 的文档（以及使用 sandbox backend 时的 `execute`）

* Subagent prompt：使用 `task` 工具进行任务委派的指南

* Human-in-the-loop prompt：在指定工具调用处暂停的用法（当设置了 `interrupt_on` 时）

* 本地上下文 prompt：当前目录和项目信息（仅 CLI）

**你提供的工具**：通过 `tools` 参数传入的工具，其 description（来自工具 schema）会被发送给模型。你还可以添加[自定义 middleware](/oss/javascript/langchain/middleware)，用于添加工具并追加其自身的 system prompt 指令。

对于你提供的工具，请确保提供清晰的名称、description 和参数描述。这些会引导模型推理何时以及如何使用该工具。在 description 中说明*何时*使用该工具，并描述每个参数的作用。

```ts
import { tool } from "langchain";
import * as z from "zod";

const searchOrders = tool(
  async ({ userId, status, limit }) =>
    `orders for ${userId} with status ${status} (limit ${limit})`,
  {
    name: "search_orders",
    description: `Search for user orders by status.

Use this when the user asks about order history or wants to check
order status. Always filter by the provided status.`,
    schema: z.object({
      userId: z.string().describe("Unique identifier for the user"),
      status: z
        .enum(["pending", "shipped", "delivered"])
        .describe("Order status to filter by"),
      limit: z
        .number()
        .default(10)
        .describe("Maximum number of results to return"),
    }),
  },
);
```

> **提示**：要为特定 provider 或模型覆盖内置工具或用户提供的工具的 description，请使用 [harness 配置档案](/oss/javascript/deepagents/profiles#harness-profiles)的 `tool_description_overrides`，以工具名作为键。
>
> 未使用的内置工具仍会在每一轮发送其完整 schema。使用 `excluded_tools` 移除 agent 绝不应调用的工具（例如只读 agent 上的 `write_file` 或 `execute`）。这会缩减整个运行期间的基线 prompt 大小。它属于配置，而非[上下文压缩](#context-compression)中的自动卸载或摘要。
>
> 请参阅 [Harness 配置档案](/oss/javascript/deepagents/profiles#harness-profiles)和[不使用默认文件系统工具运行](/oss/javascript/deepagents/overview#virtual-filesystem-access)。

关于内置能力请参阅[概览（Overview）](/oss/javascript/deepagents/overview#execution-environment)，关于直接传入工具请参阅[自定义（Customization）](/oss/javascript/deepagents/customization#tools)。

### 完整的 system prompt

deep agent 的 system message——即在一次运行开始时模型接收到的组装后的 system prompt——由以下部分组成：

1. 自定义 `system_prompt`（如果提供）
2. [基础 agent prompt](https://github.com/langchain-ai/deepagents/blob/e18e9dcd0e6edc72c0a4a5b76ae752c4bc539752/libs/deepagents/deepagents/graph.py#L37)
3. Memory prompt：`AGENTS.md` + memory 使用指南（仅当提供 `memory` 时）
4. Skills prompt：Skills 位置 + 带 frontmatter 信息的 skills 列表 + 用法（仅当提供 skills 时）
5. 虚拟文件系统 prompt（filesystem + 适用时的 execute 工具文档）
6. Subagent prompt：Task 工具用法
7. 用户提供的 middleware prompt（如果提供了自定义 middleware）
8. Human-in-the-loop prompt（当设置了 `interrupt_on` 时）

## 运行时上下文

运行时上下文是你在调用 agent 时传入的按次运行配置。它不会自动包含在模型 prompt 中；只有当某个工具、middleware 或其他逻辑读取它并将其添加到 messages 或 system prompt 中时，模型才会看到它。将运行时上下文用于用户元数据（ID、偏好、角色）、API key、数据库连接、feature flag，或你的工具和 harness 所需的其他值。

使用 `contextSchema`（通常是一个 Zod 对象 schema，例如 `z.object({ ... })`）定义该数据的形状。在传给 `invoke` / `ainvoke` 的 options 对象的 **`context`** 字段中传入运行时值。完整细节请参阅[运行时（Runtime）](/oss/javascript/langchain/runtime)和 [LangGraph 运行时上下文](/oss/javascript/langgraph/graph-api#runtime-context)。

在工具内部，从作为工具 handler 的 `runtime` 参数提供的 `ToolRuntime` 实例中读取 `runtime.context`：

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";
import type { ToolRuntime } from "@langchain/core/tools";
import * as z from "zod";

const contextSchema = z.object({
  userId: z.string(),
  apiKey: z.string(),
});

const fetchUserData = tool(
  async (input, runtime: ToolRuntime<unknown, typeof contextSchema>) => {
    const userId = runtime.context?.userId;
    return `Data for user ${userId}: ${input.query}`;
  },
  {
    name: "fetch_user_data",
    description: "Fetch data for the current user",
    schema: z.object({ query: z.string() }),
  },
);

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  tools: [fetchUserData],
  contextSchema,
});

const result = await agent.invoke(
  { messages: [{ role: "user", content: "Get my recent activity" }] },
  { context: { userId: "user-123", apiKey: "sk-..." } },
);
```

运行时上下文**会传播到所有 subagents**。当 subagent 运行时，它会收到与父级相同的运行时上下文。关于每个 subagent 的上下文（带命名空间的键），请参阅 [Subagents](/oss/javascript/deepagents/subagents#context-management)。

## 上下文压缩

每次 `create_deep_agent` 调用都包含内置的上下文压缩。你无需添加 middleware 即可让卸载或摘要生效。

长时间运行的任务会产生大量工具输出和很长的对话历史。上下文压缩在保留与任务相关细节的同时，减小 agent 工作记忆中信息的大小。以下技术是确保传给 LLM 的上下文保持在其上下文窗口限制内的内置机制：

> **卸载（Offloading）**（[链接](#offloading)）：大量的工具输入和结果被存储到文件系统中，并替换为引用。

> **摘要（Summarization）**（[链接](#summarization)）：当接近限制时，旧消息会被压缩成由 LLM 生成的摘要。

若要在压缩运行之前就缩减每一轮发送的工具 schema，请通过 [harness 配置档案](/oss/javascript/deepagents/profiles#harness-profiles)（`excluded_tools`）排除未使用的内置工具。请参阅[工具提示词](#tool-prompts)。

### 卸载

Deep Agents 使用[内置文件系统工具](/oss/javascript/deepagents/overview#virtual-filesystem-access)自动卸载内容，并按需搜索和检索被卸载的内容。当工具调用输入或结果超过某个 token 阈值（默认 20,000）时，就会发生内容卸载：

1. **工具调用输入超过 20,000 tokens**：文件写入和编辑操作会在 agent 的对话历史中留下包含完整文件内容的工具调用。由于这些内容已经持久化到文件系统中，它们往往是冗余的。当会话上下文超过模型可用窗口的 85% 时，deep agents 会截断较旧的工具调用，将其替换为指向磁盘上文件的指针，从而减小活动上下文的大小。

   ![卸载示例：一个较大的输入被保存到磁盘，工具调用使用被截断的版本](https://mintcdn.com/langchain-5e9cc07a/0G7fpRWZQ2tFN1wL/oss/images/deepagents/offloading-inputs.png?fit=max&auto=format&n=0G7fpRWZQ2tFN1wL&q=85&s=fa18372080684d661965ea6f5ed1edd0)

2. **工具调用结果超过 20,000 tokens**：发生这种情况时，deep agent 会将响应卸载到配置的 backend，并用一个文件路径引用加上前 10 行的预览来替代它。agent 随后可以按需重新读取或搜索该内容。

   ![卸载示例：一个较大的工具响应被替换为关于卸载结果位置的说明以及结果的前 10 行](https://mintcdn.com/langchain-5e9cc07a/0G7fpRWZQ2tFN1wL/oss/images/deepagents/offloading-results.png?fit=max&auto=format&n=0G7fpRWZQ2tFN1wL&q=85&s=11f3da2f37cae63b8aa4c440549f1a67)

> **注意**：内置上下文压缩不会调整图像大小、降低图像分辨率或生成视觉嵌入。关于多模态输入、工具输出以及压缩如何与媒体交互，请参阅[多模态（Multimodal）](/oss/javascript/deepagents/multimodal)。

### 摘要

> **注意**：当前的摘要行为（通过 `wrapModelCall` 的模型内摘要、精确的 token 计数，以及自动的 `ContextOverflowError` 回退）需要 `deepagents>=1.6.0`。

每次 `create_deep_agent` 调用都会在 [bare stack](/oss/javascript/deepagents/customization#bare-stack) 中包含 [`SummarizationMiddleware`](https://reference.langchain.com/javascript/langchain/index/summarizationMiddleware)。当上下文大小超过模型的上下文窗口限制（例如 `max_input_tokens` 的 85%），且没有更多可供卸载的上下文时，deep agent 会自动对消息历史进行摘要。

此过程有两个组成部分：

* **上下文内摘要（In-context summary）**：由 LLM 生成对话的结构化摘要，包括会话意图、创建的人工产物以及后续步骤——它会在 agent 的工作记忆中替换完整的对话历史。
* **文件系统保留（Filesystem preservation）**：原始对话消息的文本渲染会作为权威记录写入文件系统。

这种双重方法确保 agent 保持对其目标和进展的感知（通过摘要），同时保留在需要时恢复文本细节的能力（通过文件系统搜索）。

![摘要示例：展示了一个 agent 的对话历史，其中若干步骤被压缩](https://mintcdn.com/langchain-5e9cc07a/0G7fpRWZQ2tFN1wL/oss/images/deepagents/summarization.png?fit=max&auto=format&n=0G7fpRWZQ2tFN1wL&q=85&s=a8fea59d4365dd688e49ce118e706e76)

**配置：**

* 在模型的 `max_input_tokens`（来自其[模型配置档案](/oss/javascript/langchain/models#model-profiles)）的 85% 处触发
* 保留 10% 的 token 作为近期上下文
* 如果模型配置档案不可用，则回退到 170,000-token 触发 / 保留 6 条消息
* 如果任何模型调用抛出标准的 [ContextOverflowError](https://reference.langchain.com/javascript/langchain-core/errors/ContextOverflowError)，deep agent 会立即回退到摘要，并用摘要 + 保留的近期消息进行重试
* 较旧的消息由模型摘要

> **提示**：来自 agent 的[流式 token](/oss/javascript/deepagents/streaming#llm-tokens)通常会包含摘要步骤生成的 token。你可以使用其关联的元数据过滤掉这些 token：
>
> ```typescript
> for await (const [namespace, chunk] of await agent.stream(
>   { messages: [...] },
>   { streamMode: "messages" },
> )) {
>   const [message, metadata] = chunk;
>   if (metadata?.lcSource === "summarization") {  // [!code highlight]
>     continue;
>   } else {
>     ...
>   }
> }
> ```

## 使用 subagents 进行上下文隔离

Subagents 解决了**上下文膨胀问题**。当主 agent 使用具有大量输出的工具（web 搜索、文件读取、数据库查询）时，上下文窗口会迅速填满。Subagents 将这项工作隔离——主 agent 只接收最终结果，而不是产生结果的数十次工具调用。你还可以将每个 subagent 与主 agent 分开配置（例如 model、tools、system prompt 和 skills）。

**工作方式：**

* 主 agent 有一个用于委派工作的 `task` 工具
* Subagent 以其自己全新的上下文运行
* Subagent 自主执行直至完成
* Subagent 向主 agent 返回单个最终报告
* 主 agent 的上下文保持干净

**最佳实践：**

1. **委派复杂任务**：对于会弄乱主 agent 上下文的多步骤工作，使用 subagents。

2. **保持 subagent 响应简洁**：指示 subagents 返回摘要而非原始数据：

   ```ts
   const researchSubagent = {
     name: "researcher",
     description: "Conducts research on a topic",
     systemPrompt: `You are a research assistant.
       IMPORTANT: Return only the essential summary (under 500 words).
       Do NOT include raw search results or detailed tool outputs.`,
     tools: [webSearch],
   };
   ```

   > **查看示例 trace**（[链接](https://smith.langchain.com/public/3f16f08a-46ef-4b67-9dab-5d2478341843/r)）：打开该示例的公开 LangSmith 运行记录。

3. **对大数据使用文件系统**：Subagents 可以将结果写入文件；主 agent 读取它所需的内容。

关于配置请参阅 [Subagents](/oss/javascript/deepagents/subagents)，关于运行时上下文传播和每个 subagent 的命名空间，请参阅[上下文管理](/oss/javascript/deepagents/subagents#context-management)。

## 长期记忆

使用默认文件系统时，你的 deep agent 会将其工作记忆文件存储在 agent state 中，而它只在单个 thread 内持久。长期记忆使你的 deep agent 能够跨不同 thread 和对话持久化信息。Deep agents 可以使用长期记忆来存储用户偏好、累积的知识、研究进展，或任何应在单个会话之外持久化的信息。

要使用长期记忆，你必须使用一个 `CompositeBackend`，将特定路径（通常是 `/memories/`）路由到 LangGraph Store，后者提供持久的跨 thread 持久化。`CompositeBackend` 是一种混合存储系统，其中一些文件无限期持久，而另一些则保持在单个 thread 范围内。

#### Google

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### OpenAI

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### Anthropic

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### OpenRouter

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### Fireworks

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### Baseten

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

#### Ollama

```ts
import {
  CompositeBackend,
  createDeepAgent,
  StateBackend,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  store: new InMemoryStore(),
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: () => ["memories"],
    }),
  }),
  systemPrompt: `When users tell you their preferences, save them to /memories/user_preferences.txt so you remember them in future conversations.`,
});
```

> **查看示例 trace**（[链接](https://smith.langchain.com/public/5b8bfe15-a23f-4b74-b406-3e4e6f0ab709/r)）：打开该示例的公开 LangSmith 运行记录。

你无需预先用文件填充 `/memories/`。你提供 backend 配置、store 以及 system prompt 指令，告诉 agent *要保存什么*以及*保存在哪里*。例如，你可以提示 agent 将偏好存储在 `/memories/preferences.txt`。该路径初始为空，当用户分享值得记住的信息时，agent 会使用其文件系统工具（`write_file`、`edit_file`）按需创建文件。

要预置记忆，请在 LangSmith 上部署时使用 [Store API](/langsmith/custom-store)。关于设置和用例，请参阅[长期记忆（Long-term memory）](/oss/javascript/deepagents/memory)。

## 最佳实践

1. **从正确的输入上下文开始**：让 memory 保持精简，只放始终相关的约定；对于任务特定的能力，使用专注的 skills。
2. **对繁重工作利用 subagents**：将多步骤、输出繁重的任务委派出去，以保持主 agent 的上下文干净。
3. **在配置中调整 subagent 输出**：如果你在调试时注意到 subagents 生成了很长的输出，可以往 subagent 的 `system_prompt` 中添加引导，要求其生成摘要和综合后的发现。
4. **使用文件系统**：将大量输出持久化到文件（例如 subagent 的写入或[自动卸载](#offloading)），使活动上下文保持较小；当需要细节时，模型可以用 `read_file` 和 `grep` 拉取片段。
5. **记录长期记忆结构**：告诉 agent `/memories/` 中有什么以及如何使用它。
6. **为工具传入运行时上下文**：将 `context` 用于工具所需的用户元数据、API key 和其他静态配置。

## 相关资源

* [Harness](/oss/javascript/deepagents/overview)：上下文管理概述、卸载、摘要
* [多模态（Multimodal）](/oss/javascript/deepagents/multimodal)：图像、音频、视频以及多模态工具输出
* [Subagents](/oss/javascript/deepagents/subagents)：上下文隔离、运行时上下文传播
* [长期记忆（Long-term memory）](/oss/javascript/deepagents/memory)：跨 thread 持久化
* * [OpenWiki](/oss/openwiki/overview)：编码 agent 通过 `AGENTS.md` 发现的仓库 wiki
* [Skills](/oss/javascript/deepagents/skills)：渐进式披露与 skill 编写
* [Backends](/oss/javascript/deepagents/backends)：文件系统 backends 与 CompositeBackend
* [上下文概念性概述](/oss/javascript/concepts/context)：上下文类型与生命周期
