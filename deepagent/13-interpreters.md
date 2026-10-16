# 解释器（Interpreters）

> 原文：Interpreters
> 来源：https://docs.langchain.com/oss/javascript/deepagents/interpreters

---

> 在 Deep Agents 中运行轻量级代码，以组合工具、编排 subagents 并转换结构化数据。

解释器在 agent 循环内为 agents 提供一个可编程的、**内存中（in-memory）**的工作区。agent 编写代码来完成任务，运行时执行代码并只返回相关结果。中间结果不会成为模型上下文的一部分。

如果说 [sandboxes](/oss/javascript/deepagents/sandboxes) 是以代码为先、作用于环境的方式（例如运行命令、安装依赖和编辑文件），那么解释器则是以代码为先、用于组合工具、保留状态并决定哪些信息应返回给模型的方式。

> **警告**：解释器处于 [**beta**](/oss/javascript/versioning) 阶段。API 和生命周期行为可能在不同版本之间发生变化。

> **说明**：解释器需要 `@langchain/quickjs`。

## 为什么要使用解释器？

大多数 agent 工作都在模型推理和工具调用之间交替进行。模型可以在一个回合中触发多个工具调用，但这一批次在发出的那一刻就固定了。没有另一个模型回合，就无法循环、根据结果分支、重试失败，或将一个调用的输出馈入下一个调用，而且每个结果都会返回模型上下文。模型还决定要发出多少调用，因此让它把工作分派到数百个项目上是不可靠的，而且它往往只覆盖样本而非每一项。

解释器将这种编排移入代码中，让模型只推理*要做什么*，而不是每一个中间步骤。

> **Programmatic tool calling (PTC)**（[链接](#programmatic-tool-calling-ptc)）：从解释器代码中调用选定的工具，包括循环、重试、分支和并行批次。

> **Dynamic subagents**（[链接](#dynamic-subagents)）：从代码中分派 subagents，用于扇出、验证以及针对大型输入的递归工作流。

> **Stateful work**（[链接](#how-interpreters-work)）：将中间值保留在运行时状态中，而不使模型上下文过载。

> **Deterministic transforms**（[链接](#how-interpreters-work)）：无需另一个模型回合即可对结构化数据进行排序、分组、解析、验证、评分和聚合。

## 选择一种模式

在 agent 循环内运行代码时使用解释器：组合工具、保留状态，并控制哪些内容返回给模型。

针对环境运行代码时使用 [sandboxes](/oss/javascript/deepagents/sandboxes)：shell 命令、安装软件包、测试、文件系统编辑以及操作系统级执行。

| 需求                                                                                             | 使用方式                                                                            |
| ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| 一到两个简单的外部调用                                                                           | 普通工具调用                                                                        |
| 纯内存中的 JavaScript：循环、分支、重试或数据转换（无外部工具）                                  | Interpreter                                                                        |
| 从代码编排的许多外部工具调用（需要 [PTC](#programmatic-tool-calling-ptc)）                       | 带 [programmatic tool calling (PTC)](#programmatic-tool-calling-ptc) 的 Interpreter |
| 许多独立的工作单元、多个视角，或针对大型输入的递归分析                                           | 带 [dynamic subagents](/oss/javascript/deepagents/dynamic-subagents) 的 Interpreter |
| Shell 命令、安装软件包、测试或完整的操作系统文件系统访问                                         | [Sandboxes](/oss/javascript/deepagents/sandboxes)                                  |

## 快速开始

安装 QuickJS middleware 包，然后通过 `createDeepAgent` 上的 `middleware` 参数传入 interpreter middleware。

#### npm

```bash
npm install deepagents @langchain/quickjs
```

#### pnpm

```bash
pnpm add deepagents @langchain/quickjs
```

#### yarn

```bash
yarn add deepagents @langchain/quickjs
```

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

## 解释器如何工作

middleware 会向 agent 添加一个 `eval` 工具。在有用时，agent 编写 JavaScript 并调用 `eval`；你不会直接调用解释器。该工具在 QuickJS 上下文中运行代码，其变量可以在 `eval` 调用之间持久保留，具体取决于持久化 `mode`。它会捕获 `console.log`、`console.warn` 和 `console.error`，并返回最后一个表达式的结果。

agent 可以编写如下代码：

```ts
const rows = [
  { team: "alpha", score: 8 },
  { team: "beta", score: 13 },
  { team: "alpha", score: 21 },
];

const totals = rows.reduce((acc, row) => {
  acc[row.team] = (acc[row.team] ?? 0) + row.score;
  console.log(`${row.team} score: ${acc[row.team]}`);
  return acc;
}, {});

totals;
```

代码运行在 [**QuickJS**](https://github.com/quickjs-ng/quickjs) 之上，这是一个轻量级 JavaScript 运行时。默认情况下，解释器代码无法访问宿主文件系统、网络、shell、包管理器或时钟。它可以进行计算、保持状态，并写入 `console.log`、`console.warn` 或 `console.error`，仅此而已。

有两个显式的桥接扩展了这种能力范围：

* **Tools**，通过 [programmatic tool calling (PTC)](#programmatic-tool-calling-ptc)。将工具允许列表作为异步函数提供在 `tools` 命名空间下。这些可以是 agent 自身的工具，也可以是你定义并传入的独立工具。
* **Subagents**，通过 [dynamic subagents](/oss/javascript/deepagents/dynamic-subagents)。当 agent 配置了 subagents 时，解释器会暴露一个 `task()` 全局函数，用于从代码中分派它们。

程序化工具调用在你[启用它](#enable-ptc)之前处于关闭状态。只要 agent 配置了 subagents，通过 `task()` 进行 subagent 分派就默认开启，你也可以将其关闭。除此之外没有其他内容跨越 QuickJS 边界。

## Programmatic tool calling (PTC)

Programmatic tool calling (PTC) 在解释器内部通过全局 `tools` 命名空间暴露选定的 agent 工具。与其让模型发出一次工具调用、等待结果、再决定下一次调用，agent 可以编写代码，在循环、分支、重试或并行批次中调用工具。

当中间结果只是下一步的输入时，这会很有帮助：解释器在有任何内容返回模型之前先对它们进行过滤或聚合，从而让多步骤工作流保持 token 高效。它与模型无关，由 middleware 实现，而不是依赖某个提供方特定的工具调用 API。

middleware 将每个列入允许列表的工具作为异步函数暴露在 `tools` 下。agent 使用 `await` 调用它，在代码中处理结果，模型只会看到最终的解释器输出，而不是每一个中间值。工具名称会转换为 camelCase，而输入对象仍遵循该工具的 schema，因此名为 `web_search` 的工具会变成 `tools.webSearch(...)`：

```ts
const result: string = await tools.webSearch({
  query: "deepagents interpreters",
});
```

### 启用 PTC

使用显式的允许列表启用 PTC：

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { createCodeInterpreterMiddleware } from "@langchain/quickjs";

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  middleware: [createCodeInterpreterMiddleware({ ptc: ["web_search"] })],
});
```

启用 PTC 后，agent 可以从解释器代码中调用列入允许列表的工具。此示例并行搜索多个主题，并在返回给模型之前合并结果：

```ts
const topics = ["retrieval", "memory", "evaluation"];

const results = await Promise.all(
  topics.map((topic) =>
    tools.webSearch({ query: `${topic} best practices 2025` }),
  ),
);

results.join("\n\n");
```

> **警告**：PTC 调用目前通过解释器桥接执行，不会走常规的工具调用路径。因此，针对每个由 PTC 调用的工具调用，`interruptOn` 审批工作流不会被强制执行。

## Dynamic subagents

下面的概述涵盖了何时使用 dynamic subagents 以及一个最小的 `task()` 模式。有关配置、编排示例、工作流触发条件和安全说明，请参阅 [Dynamic subagents](/oss/javascript/deepagents/dynamic-subagents)。

Dynamic subagents 让解释器可以使用内置的 `task()` 全局函数，从代码中分派已配置的 [subagents](/oss/javascript/deepagents/subagents)。一项跨越许多独立单元的任务，例如审查目录中的每个文件或对一批工单进行分类，会变成一个循环，扇出工作并综合结果。

在以下情况使用 dynamic subagents：

* **扇出并综合（Fan-out and synthesize）**：对许多项目并行运行同类工作，然后合并结果。
* **验证（Verification）**：将发现发送给独立的验证器 subagents，并只保留已确认的结果。
* **递归工作流（Recursive workflows）**：在解释器变量中保留工作集，选取切片，调用 subagents，并细化结果。

```ts
const paths = ["src/auth.ts", "src/routes/api.ts"];

const reviews = await Promise.all(
  paths.map((path) =>
    task({
      description: `Review ${path} for authentication issues`,
      subagentType: "reviewer",
    }),
  ),
);

reviews.join("\n\n");
```

## 安全

解释器使用 QuickJS 以严格的默认隔离运行不受信任的 JavaScript。请将其视为一个限定范围的解释器运行时，而不是完整的生产 sandbox backend。

你通过 PTC 暴露的每个工具都是解释器代码可以使用的外部能力。请将 PTC 允许列表视为权限边界：只暴露 agent 所需的工具，并避免桥接可能访问敏感系统、花钱、修改数据或调用不受限制网络的宽泛工具，除非这种行为是有意为之。

| 能力                                                       | 默认是否可用 | 如何暴露                                                                                                                 |
| ----------------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| JavaScript 执行                                             | 是                  | 添加 interpreter middleware                                                                                              |
| 顶层 `await`                                                | 是                  | 在解释器代码中使用 promises                                                                                              |
| 捕获 `console.log`、`warn`、`error`                         | 是                  | 使用 `captureConsole: false` 禁用                                                                                        |
| Agent 工具                                                  | 否                   | 添加 PTC 允许列表                                                                                                        |
| 文件系统访问                                                | 否                   | 通过 PTC 允许列表添加[内置文件系统工具](/oss/javascript/deepagents/overview#virtual-filesystem-access)                   |
| 网络访问                                                    | 否                   | 通过 PTC 暴露特定的网络工具                                                                                              |
| 墙钟或日期时间访问                                          | 否                   | 如有需要，暴露一个显式的时间工具                                                                                         |
| Shell 命令、安装软件包、测试、操作系统级执行                | 否                   | 使用 [sandbox backend](/oss/javascript/deepagents/sandboxes)                                                             |

> **说明**：**代码执行如何工作**
>
> 解释器代码通过 [QuickJS-Emscripten](https://github.com/justjake/quickjs-emscripten) 在 WASM 沙箱化的 QuickJS 运行时中运行，而不是在宿主 Node.js 进程中运行。请将解释器视为一个限定能力的执行层：只桥接 agent 所需的工具和 subagents，并保持 PTC 允许列表尽可能窄。

## 配置

`createCodeInterpreterMiddleware` 接受以下选项：

| 选项                 | 默认值                           | 用途                                                                                                                                                                                     |
| -------------------- | -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `memoryLimitBytes`   | `64 * 1024 * 1024` <br />(64 MB) | 限制每个会话的 QuickJS 堆内存。                                                                                                                                                          |
| `maxStackSizeBytes`  | `320 * 1024`                     | 限制每个会话的 QuickJS 栈大小。                                                                                                                                                          |
| `executionTimeoutMs` | `5000`                           | 每次 `eval` 调用的超时限制（毫秒）。负值会禁用超时。                                                                                                                                     |
| `toolName`           | `"eval"`                         | 暴露给模型的解释器工具名称。                                                                                                                                                             |
| `captureConsole`     | `true`                           | 在工具响应中捕获 `console.log`、`console.warn` 和 `console.error`。设置为 `false` 可丢弃控制台输出。                                                                                     |
| `maxResultChars`     | `4000`                           | 将返回给模型的结果、错误和控制台输出截断到最大字符数。                                                                                                                                   |
| `systemPrompt`       | `null`                           | 解释器工具的自定义 system prompt。为 `null` 时默认使用内置 prompt。                                                                                                                      |
| `ptc`                | 省略                             | 在解释器内部以 `tools.*` 暴露的工具名称或 `StructuredToolInterface` 实例的允许列表。省略则禁用。请参阅[启用 PTC](#enable-ptc)。                                                          |
| `maxPtcCalls`        | `256`                            | 每次 `eval` 允许的最大 `tools.*` 调用数。仅在受信任的环境中才设置为 `null`。请参阅 [Programmatic tool calling (PTC)](#programmatic-tool-calling-ptc) 和[安全](#security)。                |
| `subagents`          | `true`                           | 当 agent 配置了 subagents 时，暴露内置的 `task()` 全局函数。设置为 `false` 以要求通过常规 `task` 工具进行分派。请参阅 [Dynamic subagents](#dynamic-subagents)。                          |
