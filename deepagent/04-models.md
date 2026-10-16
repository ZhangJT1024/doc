# 模型

> 原文：Models
> 来源：https://docs.langchain.com/oss/javascript/deepagents/models

---

Deep Agents 可与任何支持[工具调用](/oss/javascript/langchain/models#tool-calling)的 [LangChain chat model](/oss/javascript/langchain/models) 配合使用。

## 支持的模型

以 `provider:model` 格式指定模型（例如 `google:gemini-3.6-flash`、`openai:gpt-5.4` 或 `anthropic:claude-sonnet-4-6`）。提供商前缀用于选择 LangChain 集成，冒号之后的所有内容作为模型标识符传递给该提供商。关于有效的提供商字符串，请参阅 [`initChatModel`](https://reference.langchain.com/javascript/langchain/chat_models/universal/initChatModel)。关于提供商特定的配置，请参阅 [chat model 集成](/oss/javascript/integrations/chat)。

模型标识符必须与提供商期望的格式一致。有些提供商使用像 `gpt-5.5` 这样的简单名称；其他提供商使用命名空间 ID 或部署路径，如 `zai-org/GLM-5.2`，因此完整的 Deep Agents 字符串会是 `baseten:zai-org/GLM-5.2`。请查看提供商的模型目录或集成文档以获取当前标识符。

### 建议的模型

这些模型在 [Deep Agents eval 套件](https://github.com/langchain-ai/deepagents/tree/main/libs/evals#readme)上表现出色，该套件测试基本的 agent 操作。通过这些 evals 是在更长、更复杂的任务上取得良好性能的必要条件，但并非充分条件。

| 提供商                                                        | 模型                                                  |
| ------------------------------------------------------------- | ------------------------------------------------------- |
| [Google](/oss/javascript/integrations/providers/google)       | `gemini-3.1-pro-preview`, `gemini-3.6-flash`            |
| [OpenAI](/oss/javascript/integrations/providers/openai)       | `gpt-5.5`, `gpt-5.4`                                    |
| [Anthropic](/oss/javascript/integrations/providers/anthropic) | `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6` |
| 开放权重                                                      | `GLM-5.2`, `Kimi-K2.7 Code`, `MiniMax-M3`               |

开放权重模型可通过 [OpenRouter](/oss/javascript/integrations/chat/openrouter) 和 [Ollama](/oss/javascript/integrations/chat/ollama) 等提供商获取。

### 模型评估

[Deep Agents eval 套件](https://github.com/langchain-ai/deepagents/tree/main/libs/evals#readme)测试了流行的模型：

| 模型                                             |                                                                        总分 |                                                                        文件操作 |                                                                       检索 |                                                                       工具使用 |                                                                         记忆 |                                                                   对话 |                                                                   摘要 |
| :----------------------------------------------- | -----------------------------------------------------------------------------: | ------------------------------------------------------------------------------: | ------------------------------------------------------------------------------: | -----------------------------------------------------------------------------: | -----------------------------------------------------------------------------: | -----------------------------------------------------------------------------: | ------------------------------------------------------------------------------: |
| google\_genai:gemini-3.6-flash                   |     [82%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** | **[90%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |     [54%](https://github.com/langchain-ai/deepagents/actions/runs/25290479270) |     [38%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |
| openai:gpt-5.4                                   |     [18%](https://github.com/langchain-ai/deepagents/actions/runs/24906955930) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** |     [18%](https://github.com/langchain-ai/deepagents/actions/runs/24906955930) |     [51%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583) |     [38%](https://github.com/langchain-ai/deepagents/actions/runs/24425363630) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** |
| openai:gpt-5.5                                   |     [80%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |     [84%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |     [64%](https://github.com/langchain-ai/deepagents/actions/runs/25345307822) | **[52%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |
| anthropic:claude-opus-4-6                        |     [26%](https://github.com/langchain-ai/deepagents/actions/runs/24906955930) |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** |     [26%](https://github.com/langchain-ai/deepagents/actions/runs/24906955930) | **[69%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** |     [22%](https://github.com/langchain-ai/deepagents/actions/runs/24363491527) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/24172638583)** |
| anthropic:claude-opus-4-7                        |     [80%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |     [82%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |                                                                              — |     [48%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |
| baseten:moonshotai/Kimi-K2.6                     |     [79%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906) |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906)** |     [84%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906) |                                                                              — |     [43%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906) |      [60%](https://github.com/langchain-ai/deepagents/actions/runs/25475600906) |
| baseten:zai-org/GLM-5                            |     [77%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424)** |     [89%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424) |     [44%](https://github.com/langchain-ai/deepagents/actions/runs/23872647281) |     [24%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424) |      [60%](https://github.com/langchain-ai/deepagents/actions/runs/25403850424) |
| fireworks:accounts/fireworks/models/glm-5p1      |     [81%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650)** |     [87%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650) |                                                                              — |     [33%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650) |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25461031650) |
| fireworks:accounts/fireworks/models/minimax-m2p7 |     [79%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412)** | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412)** |     [85%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412) |                                                                              — |     [43%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412) |      [60%](https://github.com/langchain-ai/deepagents/actions/runs/25403894412) |
| ollama:minimax-m2.7:cloud                        |     [73%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |      [90%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |     [82%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |     [38%](https://github.com/langchain-ai/deepagents/actions/runs/23872647281) |     [29%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |      [60%](https://github.com/langchain-ai/deepagents/actions/runs/24106499785) |
| openrouter:deepseek/deepseek-v4-flash            |     [81%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395)** |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395) | **[90%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395)** |                                                                              — |     [33%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395) |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25677815395) |
| openrouter:minimax/minimax-m2.7                  |     [80%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535)** |     [89%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |                                                                              — |     [43%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |      [60%](https://github.com/langchain-ai/deepagents/actions/runs/25455998535) |
| openrouter:z-ai/glm-5.1                          | **[89%](https://github.com/langchain-ai/deepagents/actions/runs/25387853856)** |      [92%](https://github.com/langchain-ai/deepagents/actions/runs/25234719085) | **[100%](https://github.com/langchain-ai/deepagents/actions/runs/25234686782)** |     [89%](https://github.com/langchain-ai/deepagents/actions/runs/25387853856) |                                                                              — |     [33%](https://github.com/langchain-ai/deepagents/actions/runs/25225620506) |      [80%](https://github.com/langchain-ai/deepagents/actions/runs/25235579950) |

关于更多信息，请参阅 [Eval 运行记录](https://github.com/langchain-ai/deepagents/actions/workflows/evals.yml)。

## 配置模型参数

向 [`createDeepAgent`](https://reference.langchain.com/javascript/deepagents/agent/createDeepAgent) 传入 `provider:model` 格式的模型字符串，或传入一个已配置的模型实例以获得完全控制。在底层，模型字符串通过 [`init_chat_model`](https://reference.langchain.com/javascript/langchain/chat_models/universal/initChatModel) 解析。

要配置模型特定的参数，请使用 [`init_chat_model`](https://reference.langchain.com/javascript/langchain/chat_models/universal/initChatModel)，或直接实例化一个提供商模型类：

#### initChatModel

```ts
import { initChatModel } from "langchain/chat_models/universal";
import { createDeepAgent } from "deepagents";

const model = await initChatModel("google:gemini-3.6-flash", {
  reasoningEffort: "medium", // [!code highlight]
});
const agent = createDeepAgent({ model });
```

#### Provider package

```ts
import { ChatGoogle } from "@langchain/google";
import { createDeepAgent } from "deepagents";

const model = new ChatGoogle({
  model: "gemini-3.1-pro-preview",
  reasoningEffort: "medium", // [!code highlight]
});
const agent = createDeepAgent({ model });
```

> **注意**：可用参数因提供商而异。关于提供商特定的配置选项，请参阅 [chat model 集成](/oss/javascript/integrations/chat)页面。

### 提供商 profiles

一个 [`ProviderProfile`](/oss/javascript/deepagents/profiles#provider-profiles) 打包了初始化参数，这些参数在你于创建 deep agent 时提供 `provider:model` 字符串时生效。当你通过 [`init_chat_model`](https://reference.langchain.com/javascript/langchain/chat_models/universal/initChatModel) 传入预配置的模型时，它不生效。

你可以在两个层级上注册，且二者可以共存：

* **提供商层级**：像 `"openai"` 这样的裸提供商键适用于来自 `openai` 提供商的每个模型。
* **模型层级**：像 `"openai:gpt-5.4"` 这样的 `provider:model` 键仅适用于该特定模型，并会合并到任何匹配的提供商层级 profile 之上。

关于完整字段列表、合并语义和插件打包，请参阅 [Profiles](/oss/javascript/deepagents/profiles)。

> **提示**：要塑造模型构建后 *agent* 的行为，请使用 [harness profile](/oss/javascript/deepagents/profiles#harness-profiles)。

### OpenAI 与 Responses API

当你传入 `openai:...` 字符串作为模型时，Deep Agents 使用内置的 OpenAI [provider profile](/oss/javascript/deepagents/profiles#provider-profiles) 默认启用 [Responses API](/oss/javascript/integrations/chat/openai#responses-api)。该路径支持 OpenAI 对话状态及相关功能；请参阅 [OpenAI 提供商](/oss/javascript/integrations/providers/openai)和 [chat model](/oss/javascript/integrations/chat/openai) 文档以了解当前 API 选项。

> **注意**：内置的 OpenAI provider profile 仅适用于模型字符串。要设置 `use_responses_api`、`store` 或 `include` 等 kwargs，请传入一个已配置的模型实例，或者在 Python 中通过 [provider profile](/oss/javascript/deepagents/profiles#provider-profiles) 的 `init_kwargs` 分层设置。使用 [`init_chat_model`](https://reference.langchain.com/javascript/langchain/chat_models/universal/initChatModel)（Python）或 `initChatModel` / `ChatOpenAI`（TypeScript）构建实例。

使用 [OpenAI chat 集成](/oss/javascript/integrations/chat/openai#responses-api)可选择 Responses API 还是 Chat Completions（`useResponsesApi`、模型 kwargs，以及你的版本暴露的任何存储或输出选项）。将生成的模型实例传给 `createDeepAgent({ model })`，以便保留这些设置。

## 在运行时选择模型

如果你的应用程序允许用户选择模型（例如使用 UI 中的下拉菜单），请使用 [middleware](/oss/javascript/langchain/middleware) 在运行时切换模型，而无需重建 agent。

```ts
import { createMiddleware, initChatModel } from "langchain";
import { createDeepAgent } from "deepagents";
import * as z from "zod";

const contextSchema = z.object({
  model: z.string(),
});

const configurableModel = createMiddleware({
  name: "ConfigurableModel",
  wrapModelCall: async (request, handler) => {
    const modelName = request.runtime.context.model;
    const model = await initChatModel(modelName);
    return handler({ ...request, model });
  },
});

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  middleware: [configurableModel],
  contextSchema,
});

// Invoke with the user's model selection
const result = await agent.invoke(
  { messages: [{ role: "user", content: "Hello!" }] },
  { context: { model: "openai:gpt-5.5" } },
);
```

> **View example trace**（[链接](https://smith.langchain.com/public/89e1089d-632c-4338-b4bb-c019f4c18e14/r)）：打开该示例的公开 LangSmith 运行记录。

> **提示**：关于更动态的模型模式（例如基于对话复杂度或成本优化的路由），请参阅 LangChain agents 指南中的[动态模型](/oss/javascript/langchain/models#dynamic-model-selection)。

## 了解更多

* [LangChain 中的模型](/oss/javascript/langchain/models)：chat model 功能，包括工具调用、结构化输出和多模态
