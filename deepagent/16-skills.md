# 技能（Skills）

> 原文：Skills
> 来源：https://docs.langchain.com/oss/javascript/deepagents/skills

---

> 了解如何通过 skills 扩展 deep agent 的能力

Skills 将领域专业知识（例如工作流、最佳实践、脚本、参考文档和模板）打包为可复用的目录。agent 在启动时会获得其内容的摘要，并且只在相关时才发现和读取其中包含的文件。

Skills 通过启动时只加载摘要、在任务需要时才读取完整指令，帮助你避免上下文膨胀。你可以在多个 agent 和项目之间共享 skills，并在单个 agent 中组合多个 skills，使每个 skill 覆盖一项不同的能力。

> **注意**：Skills 需要 `deepagents>=1.7.0`。

> **提示**：关于可提升 agent 在 LangChain 生态任务上表现的开箱即用 skills，请参阅 [LangChain Skills](https://github.com/langchain-ai/langchain-skills) 仓库。

## 用法

1. **创建一个顶层 skills 目录**
   创建一个目录来存放你项目的所有 skills，例如 backend 根目录下的 `skills/`。

2. **在你的 skills 目录中为你的 skill 创建一个子目录**
   每个 skill 是一个包含 `SKILL.md` 文件的目录：一个带有 YAML [frontmatter](#frontmatter-fields)（`name` 和 `description`）的 markdown 文件，后面跟着 skill 被激活时 agent 遵循的指令。skill 目录还可以可选地包含支持文件，例如脚本、参考文档和模板。

   ```
   skills/
   └── langgraph-docs/
       ├── SKILL.md
       ├── scripts/
       │   └── fetch_docs.py
       ├── references/
       │   ├── api-patterns.md
       │   └── style-guide.md
       └── assets/
           ├── report-template.md
           └── schema.json
   ```

   Deep agent skills 遵循 [Agent Skills specification](https://agentskills.io/specification)。

3. **添加一个包含 YAML frontmatter 和指令的 `SKILL.md` 文件。**
   `SKILL.md` 以 YAML [frontmatter](#frontmatter-fields) 开头，后面跟着 markdown 指令：

   ```md
   ---
   name: langgraph-docs
   description: Use this skill for requests related to LangGraph in order to fetch relevant documentation to provide accurate, up-to-date guidance.
   ---

   # langgraph-docs

   ## Overview

   This skill explains how to access LangGraph documentation to help answer questions and guide implementation.

   ## Instructions

   ### 1. Fetch the documentation index

   Use the fetch_url tool to read the following URL:
   https://docs.langchain.com/llms.txt

   This provides a structured list of all available documentation with descriptions.

   ### 2. Select relevant documentation

   Based on the question, identify 2-4 most relevant documentation URLs from the index. Prioritize:

   - Specific how-to guides for implementation questions
   - Core concept pages for understanding questions
   - Tutorials for end-to-end examples
   - Reference docs for API details

   ### 3. Fetch and synthesize

   Use the fetch_url tool to read the selected documentation URLs, then answer the user's question. Give a direct answer first, include the minimum necessary context, and link to the source pages rather than quoting long passages.
   ```

   > **注意**：在你的 `SKILL.md` 中引用任何[支持资源](#add-supporting-resources)，并说明每个文件包含什么内容以及何时使用它。agent 通过 skill 指令中的引用发现这些文件。

4. **创建 agent 时传入 skills 路径**
   在创建 agent 时，将顶层 skills 目录的路径传给 `skills` 参数：

   ```ts
   import { createDeepAgent, FilesystemBackend } from "deepagents";

   const backend = new FilesystemBackend({ rootDir: process.cwd() });

   const agent = await createDeepAgent({
     model: "anthropic:claude-sonnet-4-6",
     backend,
     skills: ["/skills/"],
   });
   ```

   此示例使用 `FilesystemBackend` 从磁盘加载 skills。关于其他存储选项，包括从远程来源加载 skills，请参阅 [Backends 与远程 skill 加载](#backends-and-remote-skill-loading)。

   每个源路径可以指向一个包含 skill 目录的目录，也可以直接指向一个包含 `SKILL.md` 的 skill 目录。

   **`skills`**（`list[str]`，可选）

   skill 源路径的列表。

   路径必须使用正斜杠指定，并且相对于 backend 的根目录。

   * 如果省略，则不会加载任何 skills。

   * 使用 `StateBackend`（默认）时，通过 `invoke(files={...})` 提供 skill 文件。使用 `deepagents.backends.utils` 中的 `create_file_data()` 来格式化文件内容；不支持原始字符串。

   * 使用 `FilesystemBackend` 和 `StoreBackend` 时，先创建 backend，调用 `backend.uploadFiles()` 添加 skill 文件，然后将该 backend 传给 `createDeepAgent`。对于 `FilesystemBackend`，使用 `virtualMode: true` 将路径沙箱化到 `rootDir` 下。已存在于 `rootDir` 下磁盘上的 skills 无需上传即可加载。

   对于同名的 skills，后面的源会覆盖前面的源（last one wins）。

   > **注意**：当多个 skill 源包含同名 skill 时，`skills` 数组中靠后列出的源中的 skill 优先（last one wins）。这让你可以分层组合来自不同来源的 skills，例如被项目特定版本覆盖的基础 skills。

5. **调用 agent**
   使用 `invoke()` 向 agent 发送任务。在启动时，agent 会从 [frontmatter](#frontmatter-fields) 中加载每个 skill 的 [`name`](#frontmatter-fields) 和 [`description`](#frontmatter-fields) 到 system prompt。当你的任务匹配某个 skill 的描述时，agent 会读取该 skill 的 `SKILL.md` 并遵循其指令。

   ```ts
   const result = await agent.invoke(
     { messages: [{ role: "user", content: "What is LangGraph?" }] },
     { configurable: { thread_id: "1" } },
   );
   ```

   > **View example trace**（[链接](https://smith.langchain.com/public/637067d5-ee65-481b-a13e-3b3944551e9a/r)）：打开此示例的公开 LangSmith run。

## Skills 的工作原理

随着 agent 承担越来越复杂的任务，它们所需的上下文也随之增长。将所有指令加载到 system prompt 会在与当前任务无关的信息上浪费 tokens，而在不同会话中手动提供相同的指导也无法扩展。

> **说明**：Skills 使用**渐进式披露（progressive disclosure）**：agent 分层加载 skill 信息，而不是一次性全部加载。在启动时，它只看到每个 skill 的 name 和 description。当某个 skill 被调用时，它读取完整的 `SKILL.md` 指令。支持文件之后才加载，并且只在指令需要它们时才加载。

Skills 分三个层级加载。每个层级仅在任务需要时才添加更多细节：

| 层级              | 加载的内容                                                                                                                | 何时                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **1. 元数据**     | 来自 `SKILL.md` [frontmatter](#frontmatter-fields) 的 [`name`](#frontmatter-fields) 和 [`description`](#frontmatter-fields) | agent 启动时，对每个已配置的 skill                               |
| **2. 指令**       | 完整的 `SKILL.md` 正文                                                                                                    | 当该 skill 被调用时                                              |
| **3. 资源**       | `scripts/`、`references/` 和 `assets/` 下的[支持文件](#add-supporting-resources)                                          | 调用后按需加载，当指令引用它们时                                 |

下图展示了在某一时刻 agent 上下文中出现的内容。在启动时，每个 skill 的层级 1 元数据都在 system prompt 中。当某个 skill 被调用时，层级 2 指令加入上下文。层级 3 文件停留在 backend 上，直到 agent 在调用后读取它们。

![skill 组件在启动和激活时如何映射到 agent 上下文](https://mintcdn.com/langchain-5e9cc07a/-Q4wgirblfw7Ioet/oss/images/deepagents/skills-composition.svg?fit=max&auto=format&n=-Q4wgirblfw7Ioet&q=85&s=9450c441ece57465053644ede8991271)

当 agent 处理任务时，它分层加载 skill 信息：

![skills 如何从元数据到指令再到资源分层加载](https://mintcdn.com/langchain-5e9cc07a/-Q4wgirblfw7Ioet/oss/images/deepagents/skills-progressive-disclosure.svg?fit=max&auto=format&n=-Q4wgirblfw7Ioet&q=85&s=ba55587d858f5588bea425dc503e3246)

在 Deep Agents 中，[`SkillsMiddleware`](https://reference.langchain.com/javascript/deepagents/middleware/createSkillsMiddleware)（当你传入 `skills` 时属于 [Deep Agents stack](/oss/javascript/deepagents/customization#deep-agents-stack) 的一部分）处理前两个层级，第三个层级由 LLM 处理：

1. **发现（Discovery）**（层级 1）：在 agent 启动时，middleware 扫描已配置的 skill 路径，解析每个 `SKILL.md` [frontmatter](#frontmatter-fields)，并将 [`name`](#frontmatter-fields) 和 [`description`](#frontmatter-fields) 字段注入 system prompt。
2. **读取（Read）**（层级 2）：当 agent 调用某个 skill 时，它通过 `read_file` 读取完整的 `SKILL.md` 内容。
3. **执行（Execute）**（层级 3）：调用后，agent 遵循 skill 的指令，并且只在指令要求时读取支持文件（scripts、references、assets）。

## 何时使用 skills

如果你发现自己反复向 agent 给出类似的指令，尤其是当它们很详细且包含多个步骤时，请考虑将这些指令编写成 skill 交给 agent。这样，将来当你想完成类似任务时，agent 就已经知道该怎么做。

> **提示**：你也可以让 agent 为你曾与它一起完成的任务编写一个 skill。

Skills 对编写以下内容特别有帮助：

* **分步骤的工作流**：跨越多个步骤的工作流，类似于食谱。
* **领域特定知识**：指导 agent 如何为工作流使用工具。例如，包含从哪里获取信息的信息，包括该 skill 可能有权访问的其他参考信息或脚本。
* **带可执行代码的指令**：将流程与 agent 可以运行的脚本或模块打包在一起，这样它遵循经过测试的逻辑，而不是每次都从指令重新生成。请参阅[使用 skills 执行代码](#execute-code-with-skills)。
* **指导准则**：为 agent 提供有关需要遵守的护栏的支持性指令。例如，遵循特定格式或风格指南，或指定始终将运行测试作为工作流的一部分。

## 编写有效的 skills

[Agent Skills specification](https://agentskills.io/specification) 包含了关于如何组织 skills 以实现可靠发现和激活的指导。以下建议在此基础之上，为 Deep Agents 提供了实用的模式。

**保持 [frontmatter](#frontmatter-fields) 简洁**，并使 `SKILL.md` 正文保持在 5,000 tokens 以内。每个 skill 的 frontmatter 都会在[发现](#how-skills-work)阶段被加入 system prompt，而完整正文只在激活时才被读取。保持两个层级都很小，意味着你可以加载许多 skills 而不会挤占上下文窗口。

**编写具体的 description。** 在[发现](#how-skills-work)期间，[`description`](#frontmatter-fields) 字段是 agent 对每个 skill 唯一能看到的信息。一个好的 description 会告诉 agent 这个 skill 做什么以及何时激活它，并带有 agent 可以匹配的具体关键词：

```yaml
# Good: specific about what and when
description: >-
  Extract text and tables from PDF files, fill PDF forms, and merge
  multiple PDFs. Use when working with PDF documents or when the user
  mentions PDFs, forms, or document extraction.

# Poor: too vague for reliable matching
description: Helps with PDFs.
```

当你在相关领域有多个 skills 时，请清晰地差异化它们的 description。重叠的 description 会导致 agent 激活错误的 skill，或在多个选项之间犹豫。如果两个 skills 服务于类似的目的，请将它们合并为一个。

**让指令保持聚焦。** Agent Skills specification 建议将你的 `SKILL.md` 保持在 500 行以内。当指令变长时，将详细的参考资料移入[支持资源文件](#add-supporting-resources)，并在主 `SKILL.md` 中引用它们：

```
skills/
└── data-pipeline/
    ├── SKILL.md
    └── references/
        ├── schema-reference.md
        └── error-codes.md
```

agent 只在指令要求时才加载参考文件，从而使渐进式披露的每个层级都保持适当的大小。保持文件引用距离 `SKILL.md` 只有一层，并避免深度嵌套的引用链——这会迫使 agent 通过多次读取才能到达所需信息。

**为 agent 组织指令结构。** 将你的 `SKILL.md` 正文写成 agent 可以遵循的清晰指令：

* 面向多步骤工作流的**分步骤流程**
* 用于在多种方法之间做选择的**决策标准**
* **预期输入和输出的示例**，以便 agent 知道成功是什么样子
* agent 应处理或向用户标记的**边界情况**

**管理 skill 数量。** 更少且范围明确的 skills 胜过许多相互重叠的 skills。随着具有相似 description 的 skill 数量增加，agent 选择正确 skill 的能力会下降。如果你发现自己有许多相关的 skills，请考虑：

* 将相关能力合并为单个 skill，并为每个子任务设置分节
* 使用参考文件在覆盖多个子任务的同时保持主 `SKILL.md` 简洁

> **提示**：使用 [`skills-ref` 验证工具](https://github.com/agentskills/agentskills/tree/main/skills-ref)检查你的 `SKILL.md` [frontmatter](#frontmatter-fields) 是否遵循 Agent Skills specification 的命名和格式约定。

## 添加支持资源

除了 `SKILL.md` 之外，skill 目录还可以包含任何额外的文件或目录。[Agent Skills specification](https://agentskills.io/specification) 为常见资源类型定义了三个可选目录。Deep Agents 不会在发现或激活时加载这些文件。agent 只在你的 `SKILL.md` 指令指示时才读取或执行它们。

### `scripts/`

`scripts/` 目录存放 agent 可以运行的可执行代码，例如 API 客户端、数据转换或验证检查。脚本应当：

* 自包含，或清晰地记录依赖项
* 包含有用的错误消息
* 优雅地处理边界情况

支持的语言取决于你的 agent 设置。常见选项包括 Python、Bash 以及 JavaScript 或 TypeScript。要执行脚本而不只是读取它们，请参阅[使用 skills 执行代码](#execute-code-with-skills)。当 agent 需要 shell 时，请使用[沙箱脚本](#sandbox-scripts)。

### `references/`

`references/` 目录存放 agent 按需读取的补充文档。将其用于对 `SKILL.md` 来说过于详细但仍与任务相关的材料，例如：

* `REFERENCE.md` 用于详细的技术参考
* `FORMS.md` 用于表单模板或结构化数据格式
* 领域特定指南（`finance.md`、`legal.md` 及类似文件）

保持各个参考文件聚焦。agent 只在需要时加载它们，因此更小的文件占用更少的上下文。

### `assets/`

`assets/` 目录存放 agent 使用但不需要作为指令读取的静态资源，例如：

* 文档或配置模板
* 图像（图表、示例）
* 数据文件（查找表、schemas）

在 `SKILL.md` 中描述 agent 何时应打开或复制每个 asset。

### 从 `SKILL.md` 引用文件

当你引用支持文件时，请使用相对于 skill 根目录的路径：

```md
For API details, see the [reference guide](references/api-patterns.md).

To extract tables from a PDF, run:
scripts/extract.py
```

对于你引用的每个文件，说明它包含什么内容以及 agent 何时应使用它。保持引用距离 `SKILL.md` 只有一层。避免深度嵌套的引用链——这会迫使 agent 通过多次读取才能到达所需信息。

## Backends 与远程 skill 加载

Deep Agents 支持不同的 backends，具体取决于你想如何存储和管理 skill 文件：

* `StateBackend`：将文件存储在 LangGraph agent state 中，供当前 thread 使用。
* `StoreBackend`：将文件存储在 LangGraph store 中，实现持久的、跨 thread 的存储。
* `FilesystemBackend`：从可配置的 `root_dir` 下的磁盘读取和写入 skill 文件。

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

## Skill 可用性

并非每次运行都需要每个 skill。通过选择传给 `skills` 的路径、将这些路径解析为特定于上下文的存储，或组合多个 skill 源，来控制 agent 可以看到哪些 skills。上下文可以来自请求处理器、auth claims、graph factory、部署配置或任何其他应用逻辑。

### 按上下文选择 skills

在创建 agent 之前选择 `skills` 数组。当 skills 位于一个公共位置，而你的应用决定哪些路径适用于当前角色、租户、用户或请求类型时，请使用这种方式：

```ts
import { createDeepAgent } from "deepagents";

const SKILLS_BY_ROLE: Record<string, string[]> = {
  engineering: [
    "/skills/code-review/",
    "/skills/testing/",
    "/skills/deployment/",
  ],
  data: [
    "/skills/sql-analysis/",
    "/skills/visualization/",
    "/skills/data-pipeline/",
  ],
  support: ["/skills/ticket-triage/", "/skills/runbook/"],
};

function createAgentForUser(userRole: string) {
  return createDeepAgent({
    model: "anthropic:claude-sonnet-4-6",
    skills: SKILLS_BY_ROLE[userRole] ?? [],
  });
}
```

这种模式为每个 skill 保留一份维护副本，只改变传给每个 agent 的路径。在部署中，graph factory 是解析用户信息并使用正确的 skill 路径构造 agent 的自然位置。

> **注意**：SDK 只加载你在 `skills` 中传入的源。它不会自动扫描 CLI 目录，例如 `~/.deepagents/...` 或 `~/.agents/...`。
>
> 关于 CLI 存储约定，请参阅 [App data](/oss/deepagents/code/configuration#data-locations)。
>
> #### 在 SDK 中模拟 CLI 的源顺序
>
> 如果你希望在 SDK 代码中实现 CLI 风格的分层，请按从最低到最高优先级显式传入所有期望的源：
>
> ```text
> [
> "<user-home>/.deepagents/{agent}/skills/",
> "<user-home>/.agents/skills/",
> "<project-root>/.deepagents/skills/",
> "<project-root>/.agents/skills/",
> ]
> ```
>
> 然后在创建 agent 时将该有序列表作为 `skills` 传入。

### 隔离 skill 库

当同一个 skill 路径应对不同用户、角色、租户、工作区或环境解析为不同文件时，请使用隔离的库。例如，每个 agent 都可以接收 `skills=["/skills/"]`，而 backend 将 `/skills/` 解析为限定于当前身份或部署上下文的存储。

以下示例使用 [StoreBackend](https://reference.langchain.com/javascript/deepagents/backends/StoreBackend)，但同样的模式适用于任何能够根据运行时上下文解析文件的 backend：

```ts
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  skills: ["/skills/"],
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (ctx) => [
        ctx.assistantId ?? "default",
        ctx.config?.configurable?.user_id ?? "anonymous",
      ],
    }),
  }),
});
```

这在 backend 强制 agent 可以访问哪个库的同时，保持 agent 配置稳定。作用域无需特定于用户。它可以是你应用使用的任何边界，包括角色、租户、工作区、组织、环境或请求类型。

要填充一个共享库，请从你的应用代码或管理流程中预置 store：

* 按 org ID 设置 namespace，用于工作区范围的 skills。当 agent 应使用该库但不应修改它时，将其与[只读 skills](#read-only-skills)搭配使用。
* 当每个用户都需要一个独立的库时（如上面的示例），按 user ID 设置 namespace。

使用诸如 `/company-policies/SKILL.md` 的键以及包含 `content` 和 `encoding` 字段的值来预置 store。在从 store 读取记录之前，`/skills/` 路由前缀会被剥离。

你还可以组合共享库和个人库：将 `/skills/approved/` 路由到组织作用域的 `StoreBackend`，将 `/skills/editable/` 路由到用户作用域的 backend，并在 `skills` 中传入这两个路径。请参阅[可写 skills](#writable-skills)。

### 组合多个源

当 agent 应看到不止一个库时，传入多个 skill 路径。例如，你可以组合已批准的组织 skills、团队特定的工作流和请求特定的指令：

```ts
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  skills: ["/skills/org/", "/skills/team/", "/skills/request/"],
});
```

每个路径可以指向同一个 backend、不同的 backend 路由，或由运行时上下文限定作用域的 backend。关于处理 skill 访问、共享和工作区级可见性的托管解决方案，请参阅 [Fleet skills](/langsmith/fleet/skills)。

## 重新加载 skills

Skills 每个 thread 只加载一次，并保留在 agent state 中，因此该 thread 上之后每一次 model call 都复用同一份列表。此后添加、编辑或删除的 skills 永远不会到达模型。这适用于配置了 checkpointer 的情况。没有 checkpointer 时，state 不会在多次运行之间保留，因此每次运行都会重新加载 skills。

重置已存储的元数据，以便在下一次 model call 时重新加载每个源。

> **注意**：本地运行通常没有 checkpointer，因此 skill 编辑会立即生效，skills 看起来会自行重新加载。而 [LangSmith Deployments](/langsmith/deployment) 会自动配置持久 checkpointer，因此同一个 agent 一旦部署，就会在其 thread 的整个生命周期内持有它的 skills。请参阅[投入生产](/oss/javascript/deepagents/going-to-production)。

> **注意**：重新加载 skills 需要 `deepagents>=1.14.0`。

将重置作为运行输入传入，以便在单次调用中重新加载并运行：

```ts
const result = await agent.invoke(
  {
    messages: [{ role: "user", content: "What is LangGraph?" }],
    skillsMetadata: null,
  },
  config,
);
```

在多次运行之间更新 state 以重置某个 thread：

```ts
await agent.updateState(config, { skillsMetadata: null });
```

使用 `null`，而不是空列表。空列表意味着源已加载但不含任何 skills，因此 agent 在该 thread 的剩余时间内将看不到任何 skills。

> **注意**：一次发现不同 skills 集合的重新加载会改变 system prompt，并使该 thread 的 prompt cache 失效。一次发现相同 skills 的重新加载会产生相同的 prompt，没有任何成本。

Middleware 也可以重置该值，这让你的应用决定何时某个 thread 已过期。

加载发生在每次 model call 之前，因此来自 hook 的重置会在下一次 model call 时生效，包括在一次运行的中途。

```ts
import { createMiddleware } from "langchain";
import { StateSchema } from "@langchain/langgraph";
import { skillsMetadataValue } from "deepagents";

const reloadEditedSkills = createMiddleware({
  name: "ReloadEditedSkills",
  stateSchema: new StateSchema({ skillsMetadata: skillsMetadataValue }),
  afterModel: (state) =>
    agentEditedSkills(state) ? { skillsMetadata: null } : undefined,
});
```

## 面向 subagents 的 skills

当你使用 [subagents](/oss/javascript/deepagents/subagents) 时，你可以配置每种类型可以访问哪些 skills：

* **通用 subagent**：当你将 `skills` 传给 `create_deep_agent` 时，它自动从主 agent 继承 skills。无需额外配置。
* **自定义 subagents**：不继承主 agent 的 skills。为每个 subagent 定义添加一个 `skills` 参数，并填入该 subagent 的 skill 源路径。

Skill 状态是完全隔离的：主 agent 的 skills 对 subagents 不可见，而 subagent 的 skills 对主 agent 也不可见。

```ts
import { createDeepAgent } from "deepagents";

const researchSubagent = {
  name: "researcher",
  description: "Research assistant with specialized skills",
  systemPrompt: "You are a researcher.",
  tools: [webSearch],
  skills: ["/skills/research/", "/skills/web-search/"], // Subagent-specific skills
};

const agent = await createDeepAgent({
  model: "google:gemini-3.6-flash",
  skills: ["/skills/main/"], // Main agent and GP subagent get these
  subagents: [researchSubagent], // Researcher gets only its own skills
});
```

关于 subagent 配置和 skills 继承的更多信息，请参阅 [Subagents](/oss/javascript/deepagents/subagents)。

## Skill 权限

Skill 可用性控制 agent 可以看到哪些 skill 文件。Skill 权限控制 agent 是否可以读取这些文件、写入它们，或在写入前暂停以等待人工批准。使用[文件系统权限](/oss/javascript/deepagents/permissions)实现基于路径的读/写规则，并使用 [`interrupt_on`](/oss/javascript/deepagents/human-in-the-loop) 或带 `mode="interrupt"` 的权限规则来处理写入需要批准的情况。

### 只读 skills

要让 agent 使用精心策划的 skill 库而不修改它，请将该 skill 路径路由到包含已批准库的 backend，并拒绝该路径下的写操作。agent 可以发现和读取 skills；只有你的应用代码或管理流程才会更新 backend。

#### Google

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### OpenAI

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### Anthropic

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### OpenRouter

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### Fireworks

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### Baseten

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

#### Ollama

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["curated-skills", rt.context.orgId],
    }),
  }),
  skills: ["/skills/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "deny",
    },
  ],
  store,
});
```

将此用于企业知识库、已批准的工具指令或集中管理的 skill 包，在这些场景中 agent 应使用这些内容，但不应重写其真实来源（source of truth）。

### 可写 skills

默认情况下，如果 backend 允许且没有权限规则阻止该路径，agent 可以写入 skill 文件。要让 agent 创建或改进某些 skills，同时保护其他 skills：

1. 将可写路径路由到能够持久化 agent 编辑的 backend。
2. 在 `skills` 中传入这些路径。
3. 为任何应保持只读的路径添加 `deny` 规则。如果路径重叠，请将更具体的规则放在更宽泛的 deny 规则之前（[规则顺序](/oss/javascript/deepagents/permissions#rule-ordering)）。

```ts
import { InMemoryStore } from "@langchain/langgraph";
import {
  createDeepAgent,
  CompositeBackend,
  StateBackend,
  StoreBackend,
} from "deepagents";

const store = new InMemoryStore(); // Good for local dev; omit for LangSmith Deployment

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  backend: new CompositeBackend(new StateBackend(), {
    "/skills/approved/": new StoreBackend({
      namespace: (rt) => ["approved-skills", rt.context.orgId],
    }),
    "/skills/editable/": new StoreBackend({
      namespace: (ctx) => [
        "editable-skills",
        ctx.config?.configurable?.user_id ?? "anonymous",
      ],
    }),
  }),
  skills: ["/skills/approved/", "/skills/editable/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/approved/**"],
      mode: "deny",
    },
  ],
  store,
});
```

agent 使用 `write_file` 和 `edit_file` 在可写路径下创建或更新 `SKILL.md` 及支持文件。要在 skills 格式之外捕获一般性的学习成果，请将单独的路径（例如 `/memories/`）路由到另一个可写 backend。关于路由和 store 设置，请参阅 [Backends](/oss/javascript/deepagents/backends)。

### 写入批准

如果 agent 可以写入 skill 文件但你希望先经过人工介入，请使用 [`interrupt_on`](/oss/javascript/deepagents/human-in-the-loop) 或带 `mode="interrupt"` 的权限规则。两者都会在 `write_file` 或 `edit_file` 运行前暂停，并使用相同的恢复流程。

```ts
import { MemorySaver } from "@langchain/langgraph";
import { createDeepAgent } from "deepagents";

const agent = await createDeepAgent({
  model: "anthropic:claude-sonnet-4-6",
  skills: ["/skills/editable/"],
  permissions: [
    {
      operations: ["write"],
      paths: ["/skills/**"],
      mode: "interrupt",
    },
  ],
  checkpointer: new MemorySaver(), // Required to pause and resume
});
```

另外，也可以配置 `interrupt_on={"write_file": True, "edit_file": True}`，以要求对所有文件系统写入（而不仅是 skills 路径）进行批准。关于处理和恢复 interrupts，请参阅 [Human-in-the-loop](/oss/javascript/deepagents/human-in-the-loop)。

## 使用 skills 执行代码

没有代码执行时，skills 是被动的：agent 阅读指令并使用其可用的 tools 遵循它们。代码执行将 skills 转变为主动的能力。一个 skill 可以附带一个经过测试的脚本，用于调用 API、转换数据、验证输出或运行 pipeline —— 而 agent 会确定性地执行它，而不是每次从指令重新生成逻辑。这对于需要精确行为的工作流（数据转换、API 集成、合规检查），或依赖于 agent 仅通过 tool calls 无法使用的库的工作流，尤其有价值。

Skills 通过[沙箱脚本](#sandbox-scripts)执行代码：当 agent 需要安装依赖、运行测试、调用 CLIs 或使用操作系统文件系统时，它会运行打包的脚本。

### 沙箱脚本

Skills 可以在 `SKILL.md` 文件旁包含脚本。在你的 `SKILL.md` 中引用脚本，以便 agent 知道它们存在以及何时运行它们：

```
skills/
└── arxiv-search/
    ├── SKILL.md
    └── scripts/
        └── search.ts
```

```md
---
name: arxiv-search
description: Search the arXiv preprint repository for research papers. Use when the user asks about academic papers, recent research, or scientific literature.
---

# arxiv-search

Search arXiv for papers matching the user's query.

## Instructions

1. Run `scripts/search.ts` with the user's query as an argument.
2. Parse the results and present them with title, authors, abstract summary, and link.
3. If the user asks for more detail on a specific paper, fetch the full abstract.
```

agent 可以从任何 backend *读取*脚本，但要*执行*它们，agent 需要访问 shell，而只有[沙箱 backends](/oss/javascript/deepagents/sandboxes) 提供 shell。

[沙箱 backends](/oss/javascript/deepagents/sandboxes) 在隔离的容器中运行。存储在沙箱之外的 skill 文件在沙箱内不可用，这意味着除非先将它们传输进去，否则 agent 无法执行 skill 脚本或访问 skill 资源。使用[自定义 middleware](/oss/javascript/langchain/middleware/custom)来处理这种传输：

* **`before_agent`**：从 backend 读取 skill 文件并将它们上传到沙箱，以便 agent 从一开始就能执行脚本。
* **`after_agent`**：从沙箱下载任何更新或新建的 skill 文件，并将它们写回 backend，以便更改在多次运行之间持久化。

#### Google

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "google:gemini-3.6-flash",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### OpenAI

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "openai:gpt-5.5",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### Anthropic

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "anthropic:claude-sonnet-5",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### OpenRouter

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "openrouter:z-ai/glm-5.2",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### Fireworks

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "fireworks:accounts/fireworks/models/glm-5p2",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### Baseten

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "baseten:zai-org/GLM-5.2",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

#### Ollama

```ts
import { readFile, readdir } from "node:fs/promises";
import { join, posix, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";

import { createMiddleware } from "langchain";
import {
  CompositeBackend,
  createDeepAgent,
  type FileData,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";
import { SandboxClient } from "langsmith/sandbox";

/** Identical skill bundles for every user: one shared store namespace. */
const SKILLS_SHARED_NAMESPACE = ["skills", "builtin"] as const;

function createFileData(content: string): FileData {
  const now = new Date().toISOString();
  return {
    content: content.split("\n"),
    created_at: now,
    modified_at: now,
  };
}

function normalizeSkillsStoreKey(key: string): string {
  const k = String(key);
  if (k.includes("..") || /[*?]/.test(k)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return k.startsWith("/") ? k : `/${k}`;
}

async function walkFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const fullPath = join(dir, entry.name);
    if (entry.isDirectory()) {
      files.push(...(await walkFiles(fullPath)));
    } else if (entry.isFile()) {
      files.push(fullPath);
    }
  }
  return files.sort((a, b) => a.localeCompare(b));
}

/** Load canonical skill files from disk into the shared store namespace (run once at deploy).
 *  You can retrieve skills from any source (local filesystem, remote URL, etc.).
 */
async function seedSkillStore(store: InMemoryStore) {
  const moduleDir = resolve(fileURLToPath(new URL(".", import.meta.url)));
  const skillsDir = resolve(moduleDir, "skills");
  const filePaths = await walkFiles(skillsDir);
  for (const filePath of filePaths) {
    const rel = relative(skillsDir, filePath);
    // StoreBackend keys are paths *relative to the routed backend root*.
    // CompositeBackend strips the route prefix (`/skills/`) before delegating,
    // so store keys should look like "/<skillname>/SKILL.md".
    const key = `/${posix.normalize(rel.split("\\").join("/"))}`;
    const content = await readFile(filePath, "utf8");
    await store.put([...SKILLS_SHARED_NAMESPACE], key, createFileData(content));
  }
}

/** Copy shared skill files from the store into the sandbox before each agent run. */
function createSkillSandboxSyncMiddleware(backend: CompositeBackend) {
  return createMiddleware({
    name: "SkillSandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      const store = (runtime as any).store;
      if (!store) {
        throw new Error(
          "Store is required for syncing skills into the sandbox. " +
            "Pass `store` to createDeepAgent and ensure your runtime provides it.",
        );
      }

      const encoder = new TextEncoder();
      const files: Array<[string, Uint8Array]> = [];

      for (const item of await store.search([...SKILLS_SHARED_NAMESPACE])) {
        const normalized = normalizeSkillsStoreKey(String(item.key));
        const data = item.value as FileData;
        // CompositeBackend routes paths and batches uploads to the right backend.
        files.push([
          `/skills${normalized}`,
          encoder.encode(data.content.join("\n")),
        ]);
      }

      if (files.length > 0) await backend.uploadFiles(files);

      return state;
    },
  });
}

async function main() {
  const store = new InMemoryStore();
  await seedSkillStore(store);

  const client = new SandboxClient();
  const lsSandbox = await client.createSandbox();

  const backend = new CompositeBackend(
    new LangSmithSandbox({ sandbox: lsSandbox }),
    {
      "/skills/": new StoreBackend({
        store,
        namespace: () => [...SKILLS_SHARED_NAMESPACE],
      } as any),
    },
  );

  try {
    const agent = await createDeepAgent({
      model: "ollama:north-mini-code-1.0",
      backend,
      skills: ["/skills/"],
      store,
      middleware: [createSkillSandboxSyncMiddleware(backend)],
    });

  } finally {
    await client.deleteSandbox(lsSandbox.name);
  }
}

main().catch((err) => {
  console.error(err);
  process.exitCode = 1;
});
```

> **View example trace**（[链接](https://smith.langchain.com/public/a81cf81d-7f30-4bb8-99a0-1e90554fc8c4/r)）：打开此示例的公开 LangSmith run。

关于在执行前预置 skills 和 memories，并在之后将两者都同步回去的完整示例，请参阅[使用自定义 middleware 同步 skills 和 memories](/oss/javascript/deepagents/going-to-production#example-syncing-skills-and-memories-with-custom-middleware)。

## 故障排查

使用 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-deepagents-skills) traces 来调试 skill 发现、对 `SKILL.md` 的 `read_file` 调用以及支持资源的访问。按照 [tracing quickstart](/langsmith/observability-quickstart) 进行设置。我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的 traces、检测问题并提出修复建议。

### Skill 未被激活

**问题**：agent 在没有读取该 skill 的 `SKILL.md` 的情况下处理了任务。

**解决方案**：

1. **让 description 更具体。** agent 在[发现](#how-skills-work)阶段仅根据 [`description`](#frontmatter-fields) 字段选择 skills。请包含这个 skill 做什么、何时使用它，以及 agent 可以匹配的关键词：

   ```yaml
   # Good
   description: >-
     Search the arXiv preprint repository for research papers. Use when the
     user asks about academic papers, recent research, or scientific literature.

   # Poor
   description: Helps with research.
   ```

2. **减少 skills 之间的重叠。** 如果多个 skills 有相似的 description，agent 可能会跳过正确的那个或选错。请差异化 description 或[合并相关的 skills](#write-effective-skills)。

3. **确认该 skill 在 `skills` 数组中。** Skills 只会从你在创建 agent 时传入的路径，或从 subagent 特定的 `skills` 参数加载。

### 启动时缺少 skills

**问题**：agent 没有在其 system prompt 中列出某个 skill，或者对 `SKILL.md` 的 `read_file` 失败。

**解决方案**：

1. **检查 skill 路径。** 路径必须使用正斜杠，并且相对于 backend 根目录。使用 `FilesystemBackend` 时，路径相对于 `root_dir`。使用 `StateBackend` 时，在 `invoke(files={...})` 中使用 `create_file_data()` 传入 skill 文件。

2. **验证 `SKILL.md` [frontmatter](#frontmatter-fields)。** [`name`](#frontmatter-fields) 必须与父目录名称匹配，并遵循 [Agent Skills specification](https://agentskills.io/specification)。使用 [`skills-ref` 验证工具](https://github.com/agentskills/agentskills/tree/main/skills-ref)检查格式。

3. **检查文件大小。** Deep Agents 会在发现阶段跳过超过 10 MB 的 `SKILL.md` 文件。

4. **检查分层源。** 当同一个 skill 名称出现在多个源中时，[最后一个源生效](#usage)。来自靠后路径的较旧或空 skill 可能会覆盖你期望的那个。

### Skill 更改未被采纳

**问题**：在你添加、编辑或删除某个 skill 后，agent 仍在使用其较早版本。

**解决方案**：Skills 每个 thread 只加载一次，因此已经运行过的 thread 看不到该更改。请重置已存储的元数据。请参阅[重新加载 skills](#reload-skills)。

### 找不到支持文件

**问题**：agent 读取了 `SKILL.md`，但无法访问 scripts、references 或 assets。

**解决方案**：

1. **从 `SKILL.md` 引用文件。** agent 不会自动发现支持文件。请说明每个文件包含什么内容以及何时使用它。使用相对于 skill 根目录的[相对路径](#reference-files-from-skill-md)。

2. **将路径保持在 skill 目录内。** 文件路径会相对于 backend 解析。请确认支持文件存在于你的指令所引用的路径上。

3. **将 skills 同步到沙箱中。** 如果你使用[沙箱 backends](/oss/javascript/deepagents/sandboxes)，容器之外的 skill 文件在你将它们复制进去之前不可用。请参阅[沙箱脚本](#sandbox-scripts)以及[使用自定义 middleware 同步 skills 和 memories](/oss/javascript/deepagents/going-to-production#example-syncing-skills-and-memories-with-custom-middleware)。

### 脚本无法运行

**问题**：agent 读取了脚本，但无法运行它。

**解决方案**：agent 可以从任何 backend 读取脚本，但运行它们需要一个[沙箱 backend](/oss/javascript/deepagents/sandboxes)。请参阅[使用 skills 执行代码](#execute-code-with-skills)。

### Subagent 无法访问某个 skill

**问题**：自定义 subagent 看不到主 agent 使用的 skills。

**解决方案**：自定义 subagents 不继承主 agent 的 skills。为每个 [subagent 定义](#skills-for-subagents)添加一个 `skills` 参数，并填入该 subagent 的 skill 源路径。通用 subagent 会自动从 `create_deep_agent` 继承 skills。

## 参考

### Skills、memory 与 tools

Skills、[memory](/oss/javascript/deepagents/memory)（`AGENTS.md` 文件）和 tools 都为 agent 提供上下文或能力。下表总结了何时使用每一种：

|              | Skills（技能）                                                    | Memory（记忆）                                                  | Tools（工具）                                                                     |
| ------------ | ----------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **用途**     | 通过渐进式披露发现、按需加载的能力                                | 启动时加载的持久上下文                                          | agent 可以调用的程序化操作                                                        |
| **加载**     | 仅在 agent 判断相关时读取                                         | 在 agent 启动时加载                                             | 每一轮都可用                                                                      |
| **格式**     | 位于命名目录中的 `SKILL.md`                                       | `AGENTS.md` 文件                                                | 绑定到 agent 的函数                                                               |
| **分层**     | 用户，然后项目（last wins）                                       | 用户，然后项目（组合）                                          | 在创建 agent 时定义                                                               |
| **何时使用** | 指令是任务特定的且可能很大                                        | 上下文始终相关（项目约定、偏好）                                | agent 需要程序化操作，或无法访问文件系统                                          |

这些是指南，而非硬性边界。在实践中，skills 和 memory 处于一个连续谱上。agent 可以在工作时更新它自己的 skills，捕获新流程并随时间改进指令。这样，skills 可以作为渐进式披露 memory 的一种形式：agent 逐步积累并按需检索的上下文，而不是每次 prompt 都加载。

### Frontmatter 字段

[Agent Skills specification](https://agentskills.io/specification) 定义了以下 frontmatter 字段：

| 字段            | 必需   | 描述                                                                                        |
| --------------- | ------ | ------------------------------------------------------------------------------------------- |
| `name`          | 是     | 小写字母数字加连字符，1-64 个字符。必须与父目录名称匹配。                                    |
| `description`   | 是     | 该 skill 做什么以及何时使用它。最多 1,024 个字符。                                          |
| `license`       | 否     | 许可证名称或对随附许可证文件的引用。                                                        |
| `compatibility` | 否     | 环境要求（系统包、网络访问）。最多 500 个字符。                                             |
| `metadata`      | 否     | 用于附加属性的任意键值对。                                                                  |
| `allowed-tools` | 否     | 该 skill 可以使用的预先批准工具的以空格分隔的列表。实验性。                                 |

```md
---
name: langgraph-docs
description: Use this skill for requests related to LangGraph in order to fetch relevant documentation to provide accurate, up-to-date guidance.
license: MIT
compatibility: Requires internet access for fetching documentation URLs
metadata:
  author: langchain
  version: "1.0"
allowed-tools: fetch_url
---

# langgraph-docs

Instructions for the agent go here. See [Usage](#usage) for a complete example of skill instructions.
```

> **警告**：有关详细的约束和验证规则，请参阅完整的 [Agent Skills specification](https://agentskills.io/specification)。在 Deep Agents 中，`SKILL.md` 文件必须小于 10 MB。超过此限制的文件会在 skill 加载期间被跳过。

关于更多示例 skills，请参阅 [Deep Agents example skills](https://github.com/langchain-ai/deepagentsjs/tree/main/examples/skills)。
