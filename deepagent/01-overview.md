# Deep Agents 概览

> 原文：Deep Agents overview
> 来源：https://docs.langchain.com/oss/javascript/deepagents/overview

---

Deep Agents 是开始构建由 LLM 驱动的 agents 和应用最简单的方式——内置了用于上下文管理的文件系统、子代理生成以及长期记忆等能力。
诸如[任务规划](#task-planning)和[技能](#skills)之类的可选能力，会在你的用例需要时扩展 harness。
你可以将 deep agents 用于任何任务，包括复杂的、多步骤的任务。

Deep Agents 附带以下能力：

* **在环境中采取行动**：通过工具采取行动、读写文件、执行代码
* **连接你的数据**：在恰当的时刻加载记忆、技能和领域知识
* **管理不断增长的上下文**：在长时间运行中总结历史并卸载大型结果
* **并行化任务**：委派给在隔离上下文窗口中运行的通用或专用子代理
* **保持在环路中**：在关键决策点暂停以等待人工批准
* **随时间改进**：根据真实使用情况更新记忆、技能和提示词

关于每个组成部分的完整拆解，请参阅[核心能力](#core-capabilities)。

## 试用

> 官方原页面此处为交互式 Playground 演示，请在官网查看。

## 快速开始

```ts
import * as z from "zod";
// npm install deepagents langchain @langchain/core
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";

const getWeather = tool(({ city }) => `It's always sunny in ${city}!`, {
  name: "get_weather",
  description: "Get the weather for a given city",
  schema: z.object({
    city: z.string(),
  }),
});

const agent = await createDeepAgent({
  tools: [getWeather],
  systemPrompt: "You are a helpful assistant",
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in Tokyo?" }],
  }),
);
```

> **View example trace**（[链接](https://smith.langchain.com/public/7837efa6-ba5d-44f4-a88c-7cabaa4d18cb/r)）：打开该示例的公开 LangSmith 运行记录。

请参阅[快速开始](/oss/javascript/deepagents/quickstart/)和[定制指南](/oss/javascript/deepagents/customization/)，开始用 Deep Agents 构建你自己的 agents 和应用。

> **提示**：使用 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-deepagents-overview) 追踪请求、调试 agent 行为并评估输出。按照[可观测性快速开始](/langsmith/observability-quickstart)进行设置。当准备投入生产时，请参阅[投入生产](/oss/javascript/deepagents/going-to-production)了解 LangSmith 部署选项。

## 核心能力

![Agent harness capabilities by category](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/agent_harness_capabilities.svg?fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=0ff671d72badd0844826660dfcb04391)

Deep Agents 是一个["agent harness"](/oss/javascript/concepts/products#agent-harnesses-like-the-deep-agents-sdk)。它与其他 agent 框架共享相同的核心工具调用循环，但内置了一些能力，使 agents 在实际任务中保持可靠：

> **执行环境**（[链接](#execution-environment)）：工具、虚拟文件系统、可选 sandbox，以及 REPL（解释器）

> **上下文管理**（[链接](#context-management)）：技能、记忆、摘要、上下文卸载和提示词缓存

> **任务委派**（[链接](#delegation)）：子代理生成和可选的任务规划

> **引导**（[链接](#steering)）：Human-in-the-loop 批准和中断

[`deepagents`](https://www.npmjs.com/package/deepagents) 是一个独立库，构建在 [LangChain](/oss/javascript/langchain/) 的 agents 核心构建块之上，并使用 [LangGraph](/oss/javascript/langgraph/) 的工具在生产环境中运行 agents。

[LangChain](/oss/javascript/langchain/) 是提供 agents 核心构建块的框架。
要了解 LangChain、LangGraph 和 Deep Agents 之间的区别，请参阅[框架、运行时和 harness](/oss/javascript/concepts/products)。关于与 Anthropic harness 的并排对比，请参阅 [Deep Agents vs. Claude Agent SDK](/oss/javascript/deepagents/comparison)。

如果你想构建不具备这些内置能力的自定义 agents，可以考虑使用 LangChain 的 [`createAgent`](/oss/javascript/langchain/agents)，或构建自定义的 [LangGraph](/oss/javascript/langgraph/overview) 工作流。

## 执行环境

执行环境是 agent 采取行动的地方。它包含四层：

* **[工具](#tools-and-mcp)**：agent 可以调用的自定义函数、API 和数据库
* **[虚拟文件系统](#virtual-filesystem-access)**：由可插拔 backends 支持的文件工具
* **[文件系统权限](#filesystem-permissions)**：对 agent 可读或可写哪些路径的声明式访问控制
* **[代码执行](#code-execution)**：沙箱化的 shell 执行和进程内 JavaScript 解释器

**[流式传输](#streaming)** 允许你通过面向消息、工具、值和委派任务的类型化事件流，跟上发生的一切。

### 工具与 MCP

通过 `tools=` 参数传入自定义函数、LangChain 工具，或来自任意 [MCP server](/oss/javascript/deepagents/tools#mcp-tools) 的工具。Deep Agents 完全支持 [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp)，让你可以通过标准接口连接数据库、API、文件系统等。

关于定义自定义工具、使用 MCP servers，以及内置 harness 工具的完整列表，请参阅[工具](/oss/javascript/deepagents/tools)。

### 虚拟文件系统访问

harness 提供了一个可配置的虚拟文件系统，它可以由不同的[可插拔 backends](/oss/javascript/deepagents/backends)支持：内存状态、本地磁盘、LangGraph store、组合路由，或带有用于读写访问的[权限规则](/oss/javascript/deepagents/permissions)的自定义 backend。

这些 backends 支持以下文件系统操作：

| 工具         | 描述                                                                                                                                                                                                              |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ls`         | 列出目录中的文件及其元数据（大小、修改时间）                                                                                                                                                            |
| `read_file`  | 读取文件内容并带行号，对大型文件支持 offset/limit。还支持对非文本文件（图片、视频、音频和文档）返回多模态内容块。参见下方支持的扩展名。 |
| `write_file` | 创建新文件                                                                                                                                                                                                         |
| `edit_file`  | 在文件中执行精确字符串替换（带全局替换模式）                                                                                                                                                    |
| `glob`       | 查找匹配模式的文件（例如 `**/*.py`）                                                                                                                                                                           |
| `grep`       | 搜索文件内容，支持多种输出模式（仅文件名、带上下文的匹配内容，或计数）                                                                                                                            |
| `execute`    | 在环境中运行 shell 命令（仅在 [sandbox backends](/oss/javascript/deepagents/sandboxes) 时可用）                                                                                                     |

#### 支持的多模态文件扩展名

| 类型                                                   | 扩展名                                                                |
| ------------------------------------------------------ | ------------------------------------------------------------------------- |
| [图片](/oss/javascript/langchain/messages#multimodal) | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.heic`, `.heif`                |
| [视频](/oss/javascript/langchain/messages#multimodal) | `.mp4`, `.mpeg`, `.mov`, `.avi`, `.flv`, `.mpg`, `.webm`, `.wmv`, `.3gpp` |
| [音频](/oss/javascript/langchain/messages#multimodal) | `.wav`, `.mp3`, `.aiff`, `.aac`, `.ogg`, `.flac`                          |
| [文件](/oss/javascript/langchain/messages#multimodal)  | `.pdf`, `.ppt`, `.pptx`                                                   |

#### 在不使用默认文件系统工具的情况下运行

要从模型中隐藏上面列出的文件系统工具，请通过 `excluded_tools` 注册一个 [harness profile](/oss/javascript/deepagents/profiles#harness-profiles)：

```python
from deepagents import HarnessProfile, register_harness_profile

register_harness_profile(
    "anthropic:claude-sonnet-4-6",
    HarnessProfile(
        excluded_tools=frozenset(
            {"ls", "read_file", "write_file", "edit_file", "delete", "glob", "grep"}
        ),
    ),
)
```

通过 `excluded_middleware` 移除 [`FilesystemMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createFilesystemMiddleware) 本身会被有意拒绝——它是 [Deep Agents stack](/oss/javascript/deepagents/customization#deep-agents-stack) 中必需的脚手架。请使用 `excluded_tools` 仅隐藏模型可见的工具面，并保留 middleware 原样不动。要移除 `task` 工具，请参阅[在不使用子代理的情况下运行](/oss/javascript/deepagents/subagents#running-without-subagents)。

虚拟文件系统被若干其他 harness 能力使用，例如技能、记忆、代码执行和上下文管理。
你也可以在为 Deep Agents 构建自定义工具和 middleware 时使用该文件系统。

关于更多信息，请参阅 [backends](/oss/javascript/deepagents/backends)。要生成一个 agents 可从文件系统读取的持久化仓库 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

### 文件系统权限

harness 支持声明式权限规则，用于控制 agent 可以读取或写入哪些文件和目录。权限适用于上面列出的内置文件系统工具，并按照声明顺序以“先匹配者胜出”的语义进行评估。

在创建 agent 时通过向 `permissions=` 传入一组规则来定义权限。每条规则包括：

* `operations`：`"read"` 和/或 `"write"`
* `paths`：文件或目录的 Glob 模式
* `mode`：`"allow"` 或 `"deny"`

规则自上而下评估，第一条匹配的规则胜出。如果没有规则匹配，则允许该操作。

这种模型让你可以把 agents 限制在特定目录（例如 `/workspace/`），保护 `.env` 或凭据等敏感文件，并给子代理比父 agent 更窄的访问权限。

权限不适用于 [sandbox backends](/oss/javascript/deepagents/sandboxes)，后者通过 `execute` 工具支持任意命令执行。对于自定义校验逻辑，请使用 [backend policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks)。

关于完整的规则结构、示例和子代理继承，请参阅[权限](/oss/javascript/deepagents/permissions)。

### 代码执行

Deep Agents 以两种方式支持代码执行：

* [Sandbox backends](/oss/javascript/deepagents/sandboxes) 暴露一个 `execute` 工具，用于在隔离环境中执行 shell 命令。
* [Interpreters](/oss/javascript/deepagents/interpreters) 添加一个 `eval` 工具，用于在作用域化的 QuickJS 运行时中运行 JavaScript。

当 agent 需要安装依赖、运行测试、调用 CLI，或使用操作系统文件系统时，请使用 sandbox backends。Sandbox backends 实现了 `SandboxBackendProtocolV2`；当检测到时，harness 会把 `execute` 工具添加到 agent 的可用工具中。

当 agent 需要一个轻量级的可编程层来处理循环、批处理、确定性数据转换或程序化工具调用时，请使用 interpreters。Interpreters 不提供 shell 访问、包安装，也不提供文件系统和网络访问。

关于 sandbox 的设置、提供商和文件传输 API，请参阅 [Sandboxes](/oss/javascript/deepagents/sandboxes)。关于 QuickJS 运行时和程序化工具调用，请参阅 [Interpreters](/oss/javascript/deepagents/interpreters)。

### 流式传输

[事件流式传输](/oss/javascript/deepagents/event-streaming) 将 agent 运行暴露为面向消息、工具调用、值和输出的类型化投影。Deep Agents 增加了 `stream.subagents`，因此每个委派的任务都会获得自己的句柄，具有独立的消息、工具调用和嵌套子代理流。

## 上下文管理

上下文管理组件控制 agent 知道什么、它能在 token 限制内运行多久，以及它跨会话保留什么。它包含四层：

* **[技能](#skills)**：从 skill 文件中逐步按需加载的领域知识
* **[记忆](#memory)**：启动时从 `AGENTS.md` 文件加载的持久化指令和偏好
* **[摘要与上下文卸载](#summarization-and-context-offloading)**：对对话历史和大型工具结果的自动压缩
* **[提示词缓存](#prompt-caching)**：静态提示词片段可被缓存，以加快推理并在受支持的模型上降低成本

### 技能

技能为你的 deep agent 打包专门的工作流、领域知识和自定义指令。

每个技能都遵循 [Agent Skills 标准](https://agentskills.io/)，并位于一个包含 `SKILL.md` 文件的目录中。技能还可以包含脚本、模板、参考文档和其他支持性资源。

Deep Agents 以渐进式披露（progressive disclosure）的方式加载技能：agent 在启动时读取 `SKILL.md` 的 frontmatter，然后仅在任务需要时才读取完整的技能内容。这使启动上下文保持紧凑，同时仍按需提供丰富的能力。

关于更多信息，请参阅[技能](/oss/javascript/deepagents/skills)。

### 记忆

记忆为你的 deep agent 提供跨对话的持久化上下文，例如编码风格、偏好、约定和项目指南。

记忆使用你在创建 agent 时通过 `memory` 参数传入的 [`AGENTS.md` 文件](https://agents.md/)。与技能不同，记忆文件总是被加载，其内容存储在配置的 backend（`StateBackend`、`StoreBackend` 或 `FilesystemBackend`）中。

agent 还可以根据交互和反馈更新记忆，因此偏好和模式可以延续下去，而无需在每个话题中重新陈述。

关于配置细节和示例，请参阅[记忆](/oss/javascript/deepagents/customization#memory)。要生成一个编码 agents 通过 `AGENTS.md` 发现的仓库 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

### 摘要与上下文卸载

harness 管理上下文，使 deep agents 能够在 token 限制内处理长时间运行的工作，同时将最相关的信息保持在范围内。

这个上下文流程包含四个部分：

* **输入上下文**：系统提示词、记忆、技能和工具提示词定义了 agent 的起始内容。
* **压缩**：内置的卸载和摘要会压缩对话历史和大型中间结果。
* **隔离**：子代理隔离繁重的子任务，只返回最终结果（请参阅[任务委派](#delegation)）。
* **长期记忆**：虚拟文件系统中的持久化存储跨话题携带信息。

这些机制共同支持超出单一上下文窗口的多步骤任务，同时减少手动裁剪上下文和 token 使用量。

关于配置细节，请参阅[上下文工程](/oss/javascript/deepagents/context-engineering)。关于多模态输入和工具输出，请参阅[多模态](/oss/javascript/deepagents/multimodal)。

### 提示词缓存

对于 Anthropic 和 Amazon Bedrock 模型，`create_deep_agent` 会自动对系统提示词中的静态部分应用提示词缓存——即在每一轮都会重复的基础 agent 指令、记忆和技能内容。这避免在多次调用之间重复处理相同的 token，从而降低长时间运行 agents 的延迟和成本。

在使用 Anthropic 模型或 Bedrock 模型（Claude 或 Nova）时，提示词缓存默认启用。无需任何配置。

对于其他提供商，请参阅[中间件集成](/oss/javascript/integrations/middleware)以了解可用的提供商特定缓存 middleware。

## 任务委派

任务委派组件使 agents 能够将大问题拆分为更小、可并行的工作单元。它包含两层：

* **[任务规划](#task-planning)**：一个可选的 `write_todos` 工具，用于结构化任务跟踪
* **[子代理](#subagents)**：处理隔离子任务的临时子 agents

### 任务规划

任务规划是一个可选的 harness 能力，让 agents 在执行期间维护一个结构化的任务列表。

从 v0.7 开始，任务规划仅是可选的（opt-in）。在更早的版本中，任务规划 middleware 默认包含在内。

规划通常对以下情况有用：

* 较长或复杂的多步骤任务
* 能力较弱的模型，它们能从一个明确的问责工具中受益
* 从 agent 状态流式传输进度的 UI（请参阅[待办列表](/oss/javascript/deepagents/frontend/todo-list)）

将 [`TodoListMiddleware`](https://reference.langchain.com/javascript/langchain/index/todoListMiddleware) 传给 middleware 参数，为 agent 提供一个 `write_todos` 工具，用于在执行期间维护结构化任务列表。

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  middleware: [todoListMiddleware()],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "openai:gpt-5.5",
  middleware: [todoListMiddleware()],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  middleware: [todoListMiddleware()],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  middleware: [todoListMiddleware()],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  middleware: [todoListMiddleware()],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  middleware: [todoListMiddleware()],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { todoListMiddleware } from "langchain";

const agent = await createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  middleware: [todoListMiddleware()],
});
```

任务支持状态跟踪（`'pending'`、`'in_progress'`、`'completed'`），并持久化在 agent 状态中。这为 agents 提供了一个轻量级的规划层，用于组织长时间运行和多步骤的工作。

关于配置选项和行为细节，请参阅[待办列表](/oss/javascript/langchain/middleware/built-in#to-do-list)。

### 子代理

harness 包含一个内置的 `task` 工具，让主 agent 能够为隔离的、长时间运行的、多步骤的或并行的任务创建临时子代理。

子代理执行提供：

* **全新上下文**：每次调用都会创建一个带有自己上下文的新 agent 实例。
* **自主执行**：子代理独立运行直至完成。
* **单次交接**：它向主 agent 返回一份最终报告。
* **可配置策略**：使用[默认的 `general-purpose` 子代理](/oss/javascript/deepagents/subagents#default-subagent)（默认启用）或定义[自定义子代理](/oss/javascript/deepagents/subagents#custom-subagents)。
* **无状态消息传递**：子代理是无状态的，无法回传多条消息。
* **上下文与 token 效率**：繁重的子任务工作保持隔离，并被压缩为一份紧凑的结果。

#### 在不使用子代理的情况下运行（无 `task` 工具）

要在没有 `task` 工具的情况下运行 agent，请参阅[在不使用子代理的情况下运行](/oss/javascript/deepagents/subagents#running-without-subagents)。不要尝试通过 `excluded_middleware` 移除 [`SubAgentMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSubAgentMiddleware)——那会被有意拒绝。相反，请通过 [harness profile](/oss/javascript/deepagents/profiles#harness-profiles) 禁用自动添加的子代理，并且不要通过 `subagents=` 传入任何同步子代理。异步子代理不受影响。完整的顺序请参阅[完整 stack](/oss/javascript/deepagents/customization#full-stack)。

关于更多信息，请参阅[子代理](/oss/javascript/deepagents/subagents)。

## 引导

引导组件让人类在运行时控制 agent 行为，并为 agent 的工作设置文件系统权限。

### Human-in-the-loop

Deep Agents 与 LangGraph 中断集成，因此你可以在敏感工具调用上暂停以请求批准。通过 `create_deep_agent` 中的 `interrupt_on` 参数启用此行为。

`interrupt_on` 接受一个从工具名到中断配置的映射。例如，`interrupt_on={"edit_file": True}` 会在每次编辑前暂停，让你批准该调用、补充指导，或在执行前修改工具输入。

这为你提供了一个运行时安全与控制层，用于破坏性操作、昂贵的 API 调用和交互式调试。

关于更多信息，请参阅 [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)。

## 开始使用

> **Quickstart**（[链接](/oss/javascript/deepagents/quickstart)）：构建你的第一个 deep agent

> **Customization**（[链接](/oss/javascript/deepagents/customization)）：了解定制选项

> **Code**（[链接](/oss/deepagents/code/overview)）：使用 Deep Agents Code

> **ACP**（[链接](/oss/javascript/deepagents/acp)）：在代码编辑器中使用 ACP 运行 deep agents

> **Reference**（[链接](https://reference.langchain.com/javascript/modules/deepagents.html)）：查看 `deepagents` API 参考
