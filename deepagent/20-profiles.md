# 配置档案（Profiles）

> 原文：Profiles
> 来源：https://docs.langchain.com/oss/javascript/deepagents/profiles

---

**Harness 配置档案（Harness profiles）**让你可以为特定模型或 provider 定制 Deep Agents 的 harness。你可以调整 system prompt 和 tool 描述、排除 tools 或 middleware、添加 middleware，以及配置通用子代理（general-purpose subagent）。只要选择了匹配的模型，Deep Agents 就会应用这些设置，无需修改你创建 agent 的代码。

> **注意**：Provider 配置档案（用于控制创建模型时使用的设置）以及插件注册系统是 Python 专属特性。TypeScript SDK 仅支持 harness 配置档案。

## Harness 配置档案

Deep Agents 内置**harness 配置档案（built-in harness profiles）**，为特定 provider 和模型提供默认设置。

使用 `HarnessProfileOptions` 定义 `createDeepAgent` 在构建 chat model 之后应用的设置：

```ts
import { registerHarnessProfile } from "deepagents";

registerHarnessProfile("openai:gpt-6-astra", {
  systemPromptSuffix: "Respond in under 100 words.",
  excludedTools: ["execute"],
  excludedMiddleware: ["SummarizationMiddleware"],
  generalPurposeSubagent: { enabled: false },
});
```

- **`baseSystemPrompt`**（`string`）：设置该配置档案的基础指令。对于主 agent，这些指令跟随调用者的 system 指令；默认不添加基础指令。对于声明式（declarative）subagents，这会替换其作者编写的 system prompt。

- **`systemPromptSuffix`**（`string`）：在调用者的指令和该配置档案的基础指令之后追加文本。应用于主 agent、声明式 subagents 以及自动添加的通用子代理。

- **`toolDescriptionOverrides`**（`Record<string, string>`）：以工具名作为键，覆盖单个工具的 description。

- **`excludedTools`**（`string[]`）：从工具集中移除特定的 harness 级工具。按工具名匹配，作为注入后的过滤器应用，因此能捕获用户提供的工具和 middleware 提供的工具。

- **`excludedMiddleware`**（`string[]`）：从组装好的 stack 中剥离特定的 middleware。与每个 middleware 的 `.name` 属性匹配。不能包含必需的脚手架名称（`FilesystemMiddleware`、`SubAgentMiddleware`）。

- **`extraMiddleware`**（`AgentMiddleware[] | (() => AgentMiddleware[])`）：在用户 middleware 之后追加到 stack 的额外 middleware。可以是静态数组，或返回每次 agent 构造时全新实例的零参工厂函数。

- **`generalPurposeSubagent`**（`GeneralPurposeSubagentConfig`）：禁用、重命名或重新提示通用子代理（`enabled`、`description`、`systemPrompt`）。

> **注意**：调用者提供的 `systemPrompt` 始终位于组装 prompt 的开头，而 `systemPromptSuffix` 始终位于末尾——无论选择了哪个模型都是如此。同样的叠加规则也适用于 subagents：每个 subagent 都会针对自己的模型重新运行配置档案解析。关于自定义指令和 subagent prompt 行为，请参阅[系统提示词](/oss/javascript/deepagents/customization#system-prompt)。

> **警告**：在 `excludedMiddleware` 中列出 `FilesystemMiddleware` 或 `SubAgentMiddleware` 会在构造时抛错——它们是必需的脚手架。若要在不移除 middleware 的情况下向模型隐藏其工具，请改用 `excludedTools`。

在创建 agent 之前注册配置档案。传入模型字符串或你自己构造的模型对象；示例请参阅[配置模型参数](/oss/javascript/deepagents/models#configure-model-parameters)。harness 配置档案在两种情况下都会应用。

#### 预配置模型实例的查找顺序

当你传入模型对象时，harness 会使用该对象报告的 provider 和标识符来查找其配置档案。

1. 如果标识符不含冒号，则查找 `provider:identifier`，回退到该 provider 的默认值。
2. 如果标识符含冒号，则直接查找它，回退到其前缀的默认值。
3. 如果两种查找都不匹配，则使用所报告 provider 的默认值。

## 注册键

配置档案注册使用以下键：

* **Provider 级**——像 `"openai"` 这样的裸 provider 名称，适用于该 provider 的所有模型。
* **模型级**——像 `"openai:gpt-6-astra"` 这样的完整 `provider:model` 键，只适用于该特定模型。

当 provider 级和模型级配置档案同时存在时，它们会在解析时合并。未设置的模型级字段继承自 provider 级配置档案；显式的模型级值会覆盖它们。

TypeScript 中的配置档案注册不支持含冒号的模型标识符。请使用诸如 `my_provider:my-model` 这样的键，用单个冒号分隔 provider 和模型标识符。

例如，为一个假设的 provider 的模型排除某个工具，然后为某一个模型定制 prompt 后缀：

```ts
import { registerHarnessProfile } from "deepagents";

// Set defaults for a hypothetical provider.
registerHarnessProfile("my_provider", {
  excludedTools: ["execute"],
  systemPromptSuffix: "Respond in under 500 words.",
});

// Override the prompt suffix for one model; inherit the excluded tool.
registerHarnessProfile("my_provider:my-model", {
  systemPromptSuffix: "Respond in under 100 words.",
});
```

使用模型特定注册的 agent 会排除 `execute`，并获得 100 词的后缀。来自 `my_provider` 的其他模型会排除 `execute`，并获得 500 词的后缀。

在已有键下重新注册会将新配置档案合并到前一个之上；它不会替换前一个。这也让你可以通过在其键下注册来定制内置配置档案。逐字段规则请参阅[合并语义](#merge-semantics)。

延续该示例，为同一个模型再排除一个工具：

```ts
registerHarnessProfile("my_provider:my-model", {
  excludedTools: ["grep"],
});
```

此后使用该模型创建的 agents 会同时排除 `execute` 和 `grep`，并保留 100 词的后缀。其他模型保持 provider 默认值。

> **注意**：不存在能匹配每个 provider 的通配符键。若要在所有地方应用相同的覆盖——比如无论选择哪个模型都去掉 `SummarizationMiddleware`——请在你使用的每个 provider 键下注册该配置档案。配置档案用于依赖所选模型的调整。无论模型如何都应应用的全局调整，应在 `createDeepAgent` 调用处进行。

## 合并语义

| 字段 | 合并行为 |
| ---------------------------------------- | ------------------------------------------------------------------------------------ |
| `baseSystemPrompt`、`systemPromptSuffix` | 设置时新值优先；否则继承 |
| `toolDescriptionOverrides` | 映射按 key 合并；在共享 key 上新值优先 |
| `excludedTools`、`excludedMiddleware` | 取集合并集 |
| `extraMiddleware` | 按名称合并：新实例替换其位置上的已有项，新条目追加 |
| `generalPurposeSubagent` | 逐字段合并（未设置的字段继承） |

## Provider 配置档案

Provider 配置档案（用于控制创建模型时使用的设置，例如 `temperature`）是 Python 专属特性，在 TypeScript SDK 中不可用。

## 从配置文件加载配置档案

对于由 YAML/JSON 支持的工作流，请使用 `parseHarnessProfileConfig`。它会从使用 camelCase 键的普通对象验证并构建 `HarnessProfile`。诸如 `extraMiddleware` 实例之类的仅运行时状态无法在 JSON/YAML 中表示，必须以编程方式设置。

```yaml
# profile.yaml
baseSystemPrompt: You are helpful.
systemPromptSuffix: Respond briefly.
excludedTools:
  - execute
  - grep
excludedMiddleware:
  - SummarizationMiddleware
generalPurposeSubagent:
  enabled: false
```

```ts
import { readFileSync } from "fs";
import YAML from "yaml";
import { parseHarnessProfileConfig, registerHarnessProfile } from "deepagents";

const raw = YAML.parse(readFileSync("profile.yaml", "utf-8"));
registerHarnessProfile("openai", parseHarnessProfileConfig(raw));
```

要将配置档案序列化回 JSON/YAML，请使用 `serializeProfile`：

```ts
import { serializeProfile } from "deepagents";

const data = serializeProfile(profile); // JSON-compatible object
```

`extraMiddleware` 非空的配置档案无法被序列化；如果存在 middleware 实例，`serializeProfile` 会抛错。

## 将配置档案作为插件发布

插件注册系统（通过 package entry points）是 Python 专属特性。在 TypeScript 中，请在应用程序启动时或包的初始化代码中直接调用 `registerHarnessProfile`。

## 相关

* [Harness 概览](/oss/javascript/deepagents/overview)——harness 能力概述

* [Models](/oss/javascript/deepagents/models)——配置模型 provider 和参数

* [自定义（Customization）](/oss/javascript/deepagents/customization)——完整的 `createDeepAgent` 配置面
