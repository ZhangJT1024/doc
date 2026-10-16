# 记忆（Memory）

> 原文：Memory
> 来源：https://docs.langchain.com/oss/javascript/deepagents/memory

---

> 为使用 Deep Agents 构建的 agent 添加持久记忆，使其能够跨对话学习和改进

Memory 让 agent 能够跨对话学习和改进。Deep Agents 通过基于文件系统的 memory 将 memory 提升为一等公民：agent 以文件形式读写 memory，而你可以使用 [backends](/oss/javascript/deepagents/backends) 控制这些文件的存储位置。

> **提示**：要生成可供 coding agents 通过 [`AGENTS.md`](https://agents.md/) 发现的仓库 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

> **注意**：本页介绍的是**长期记忆（long-term memory）**：跨对话持久化的 memory。关于短期记忆（单次会话内的对话历史和临时文件），请参阅[上下文工程](/oss/javascript/deepagents/context-engineering)指南。短期记忆作为 agent [state](/oss/javascript/langgraph/graph-api#state) 的一部分被自动管理。
>
> ![短期记忆通过 checkpoints 限定于单个 thread；长期记忆通过 store 跨 thread 持久化](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/memory.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=3ec585271dcd8d62e0207d79d68c296b)

## Memory 的工作原理

1. **将 agent 指向 memory 文件。** 在创建 agent 时将文件路径传给 `memory=`。你还可以通过 `skills=` 传入 [skills](/oss/javascript/deepagents/skills) 作为程序性记忆（procedural memory，即告诉 agent *如何*执行任务的可复用指令）。[backend](/oss/javascript/deepagents/backends) 控制文件存储在哪里以及谁可以访问它们。
2. **Agent 读取 memory。** agent 可以在启动时将 memory 文件加载到 system prompt，或在对话过程中按需读取它们。例如，[skills](/oss/javascript/deepagents/skills) 使用按需加载：agent 在启动时只读取 skill 描述，只有当某个任务匹配时才读取完整的 skill 文件。这样在需要某项能力之前，上下文始终保持精简。
3. **Agent 更新 memory（可选）。** 当 agent 学到新信息时，它可以使用内置的 `edit_file` tool 更新 memory 文件。更新可以发生在对话期间（默认），也可以在对话之间通过[后台整合](#background-consolidation)在后台进行。更改会被持久化，并可在下一次对话中使用。并非所有 memory 都是可写的：开发者定义的 [skills](/oss/javascript/deepagents/skills) 和[组织策略](#organization-level-memory)通常是只读的。详情请参阅[只读 vs 可写 memory](#read-only-vs-writable-memory)。

两种最常见的模式是 [agent 作用域 memory](#agent-scoped-memory)（所有用户共享）和[用户作用域 memory](#user-scoped-memory)（每个用户隔离）。

关于可供 coding agents 通过 [`AGENTS.md`](https://agents.md/) 发现的生成的仓库 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

## 作用域 memory

Agent memory 可以设置作用域，使得同一个 memory 文件对所有使用该 agent 的人都可访问，或者使 memory 文件对每个用户都是独立的。

### Agent 作用域 memory

为 agent 赋予它自己的、随时间演进的持久身份。Agent 作用域 memory 在所有用户之间共享，因此 agent 在每次对话中都会积累自己的角色形象、累积的知识和习得的偏好。在与用户交互时，它会发展专长、改进方法并记住哪些做法有效。当它具有写访问权限时，它还可以学习并更新 [skills](/oss/javascript/deepagents/skills)。

关键在于 backend namespace：将其设置为 `(assistant_id,)` 意味着该 agent 的每次对话都读写同一个 memory 文件。

> **注意**：访问 `rt.serverInfo` 需要 `deepagents>=1.9.0`。在更早的版本中，请改为从 `getConfig().metadata.assistantId` 读取 assistant ID。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

const agent = createDeepAgent({
  memory: ["/memories/AGENTS.md"],
  skills: ["/skills/"],
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.assistantId],  // [!code highlight]
      }),
      "/skills/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.assistantId],  // [!code highlight]
      }),
    },
  ),
});
```

#### 完整示例：填充 memory 并调用

向 store 中填充初始 memories，然后跨两个 thread 调用 agent，观察它记住并更新所学内容。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend, createFileData } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();  // Use platform store when deploying to LangSmith

// Seed the memory file
await store.put(
  ["my-agent"],
  "/memories/AGENTS.md",
  createFileData(`## Response style
- Keep responses concise
- Use code examples where possible
`),
);

// Seed a skill
await store.put(
  ["my-agent"],
  "/skills/langgraph-docs/SKILL.md",
  createFileData(`---
name: langgraph-docs
description: Fetch relevant LangGraph documentation to provide accurate guidance.
---

# langgraph-docs

Use the fetch_url tool to read https://docs.langchain.com/llms.txt, then fetch relevant pages.
`),
);

const agent = createDeepAgent({
  memory: ["/memories/AGENTS.md"],
  skills: ["/skills/"],
  backend: (rt) => new CompositeBackend(
    new StateBackend(rt),
    {
      "/memories/": new StoreBackend(rt, {
        namespace: (rt) => ["my-agent"],
      }),
      "/skills/": new StoreBackend(rt, {
        namespace: (rt) => ["my-agent"],
      }),
    },
  ),
  store,
});

// Thread 1: the agent learns a new preference and saves it to memory
const config1 = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke({
  messages: [{ role: "user", content: "I prefer detailed explanations. Remember that." }],
}, config1);

// Thread 2: the agent reads memory and applies the preference
const config2 = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke({
  messages: [{ role: "user", content: "Explain how transformers work." }],
}, config2);
```

### 用户作用域 memory

为每个用户赋予他们自己的 memory 文件。agent 按用户记住偏好、上下文和历史，而核心 agent 指令保持固定。如果存储在用户作用域的 backend 中，用户也可以拥有各自的 [skills](/oss/javascript/deepagents/skills)。

namespace 使用 `(user_id,)`，因此每个用户都会得到一份隔离的 memory 文件副本。用户 A 的偏好绝不会泄漏到用户 B 的对话中。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

const agent = createDeepAgent({
  memory: ["/memories/preferences.md"],
  skills: ["/skills/"],
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.user.identity],
      }),
      "/skills/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.user.identity],
      }),
    },
  ),
});
```

#### 完整示例：跨用户的隔离 memory

为每个用户填充各自的 memories，并以两个不同用户的身份调用 agent。每个用户只能看到自己的偏好。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend, createFileData } from "deepagents";
import { InMemoryStore } from "@langchain/langgraph";

const store = new InMemoryStore();  // Use platform store when deploying to LangSmith

// Seed preferences for two users
await store.put(
  ["user-alice"],
  "/memories/preferences.md",
  createFileData(`## Preferences
- Likes concise bullet points
- Prefers Python examples
`),
);
await store.put(
  ["user-bob"],
  "/memories/preferences.md",
  createFileData(`## Preferences
- Likes detailed explanations
- Prefers TypeScript examples
`),
);

// Seed a skill for Alice
await store.put(
  ["user-alice"],
  "/skills/langgraph-docs/SKILL.md",
  createFileData(`---
name: langgraph-docs
description: Fetch relevant LangGraph documentation to provide accurate guidance.
---

# langgraph-docs

Use the fetch_url tool to read https://docs.langchain.com/llms.txt, then fetch relevant pages.
`),
);

const agent = createDeepAgent({
  memory: ["/memories/preferences.md"],
  skills: ["/skills/"],
  backend: (rt) => new CompositeBackend(
    new StateBackend(rt),
    {
      "/memories/": new StoreBackend(rt, {
        namespace: (rt) => [rt.serverInfo.user.identity],
      }),
      "/skills/": new StoreBackend(rt, {
        namespace: (rt) => [rt.serverInfo.user.identity],
      }),
    },
  ),
  store,
});

// When deployed, each authenticated request resolves
// `rt.serverInfo.user.identity` to the calling user, so Alice and Bob
// automatically see only their own preferences.
await agent.invoke(
  { messages: [{ role: "user", content: "How do I read a CSV file?" }] },
  { configurable: { thread_id: crypto.randomUUID() } },
);
```

## 高级用法

除了 memory 路径和作用域的基本配置选项之外，你还可以为 memory 配置更高级的参数：

| 维度                    | 它回答的问题                    | 选项                                                                                                                                                                                              |
| ----------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **持续时间**            | 它能持续多久？                  | [短期](/oss/javascript/deepagents/context-engineering)（单次对话）或[长期](#scoped-memory)（跨对话）                                                                                             |
| **信息类型**            | 它是什么类型的信息？            | [情景式](#episodic-memory)（过往经历）、[程序性](/oss/javascript/deepagents/skills)（指令和 skills）或[语义式](/oss/javascript/concepts/memory#semantic-memory)（事实）                          |
| **作用域**              | 谁可以查看和修改它？            | [用户](#user-scoped-memory)、[agent](#agent-scoped-memory) 或[组织](#organization-level-memory)                                                                                                   |
| **更新策略**            | memory 何时被写入？             | 对话期间（默认）或[对话之间](#background-consolidation)                                                                                                                                           |
| **检索（Retrieval）**   | memory 如何被读取？             | 加载到 prompt（默认）或按需加载（例如 [skills](/oss/javascript/deepagents/skills)）                                                                                                              |
| **Agent 权限**          | agent 可以写入 memory 吗？      | [读写](#read-only-vs-writable-memory)（默认）或[只读](#read-only-vs-writable-memory)（用于共享策略）                                                                                              |

### 情景式 memory

情景式记忆（episodic memory）存储过往经历的记录：发生了什么、以什么顺序发生，以及结果如何。与语义式记忆（semantic memory，存储在诸如 `AGENTS.md` 文件中的事实和偏好）不同，情景式记忆保留完整的对话上下文，因此 agent 能够回忆问题是*如何*解决的，而不仅仅是*学到了什么*。关于为 coding agents 生成并维护仓库级 wiki，请参阅 [OpenWiki](/oss/openwiki/overview)。

Deep Agents 已经使用 [checkpointers](/oss/javascript/langgraph/checkpointers#checkpoints)，这正是支持情景式记忆的机制：每次对话都会作为 checkpointed thread 持久化。

要使过往对话可被搜索，请将 thread 搜索封装到一个 tool 中。`user_id` 从 runtime context 中提取，而不是作为参数传入：

```typescript
import { Client } from "@langchain/langgraph-sdk";
import { tool } from "@langchain/core/tools";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>" });

const searchPastConversations = tool(
  async ({ query }, runtime) => {
    const userId = runtime.serverInfo.user.identity;  // [!code highlight]
    const threads = await client.threads.search({
      metadata: { userId },
      limit: 5,
    });
    const results = [];
    for (const thread of threads) {
      const history = await client.threads.getHistory(thread.threadId);
      results.push(history);
    }
    return JSON.stringify(results);
  },
  {
    name: "search_past_conversations",
    description: "Search past conversations for relevant context.",
  }
);
```

你可以通过调整 metadata 过滤器，按用户或组织限定 thread 搜索的作用域：

```typescript
// Search conversations for a specific user
const userThreads = await client.threads.search({
  metadata: { userId },
  limit: 5,
});

// Search conversations across an organization
const orgThreads = await client.threads.search({
  metadata: { orgId },
  limit: 5,
});
```

这对于执行复杂、多步骤任务的 agent 很有用。例如，一个 coding agent 可以回顾过去的调试会话，并直接跳到可能的根本原因。

### 组织级 memory

组织级 memory 遵循与用户作用域 memory 相同的模式，只是使用组织范围的 namespace 而非每个用户的 namespace。将其用于应在组织中所有用户和 agent 之间生效的策略或知识。

组织 memory 通常是**只读的**，以防止通过共享状态进行提示注入（prompt injection）。详情请参阅[只读 vs 可写 memory](#read-only-vs-writable-memory)。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

const agent = createDeepAgent({
  memory: [
    "/memories/preferences.md",
    "/policies/compliance.md",
  ],
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.user.identity],
      }),
      "/policies/": new StoreBackend({
        namespace: (rt) => [rt.context.orgId],
      }),
    },
  ),
});
```

从你的应用代码中填充组织 memory：

```typescript
import { Client } from "@langchain/langgraph-sdk";
import { createFileData } from "deepagents";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>" });

await client.store.putItem(
  [orgId],
  "/compliance.md",
  createFileData(`## Compliance policies
- Never disclose internal pricing
- Always include disclaimers on financial advice
`),
);
```

使用 [permissions](/oss/javascript/deepagents/permissions) 强制组织级 memory 为只读，或使用 [policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks) 实现自定义验证逻辑。

### 后台整合

默认情况下，agent 在对话期间（热路径）写入 memories。另一种方法是在**对话之间**作为后台任务处理 memories，有时称为 **sleep time compute**。一个单独的 deep agent 会审查最近的对话，提取关键事实，并将它们与现有 memories 合并。

| 方法                              | 优点                                                     | 缺点                                                                    |
| --------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------- |
| **热路径**（对话期间）            | memories 立即可用，对用户透明                            | 增加延迟，agent 必须同时处理多项任务                                    |
| **后台**（对话之间）              | 无面向用户的延迟，可跨多次对话进行综合                   | memories 要到下一次对话才可用，需要第二个 agent                         |

对于大多数应用，热路径就足够了。当你需要降低延迟或改善多次对话之间的 memory 质量时，再添加后台整合。

推荐的模式是：在你的主 agent 之外部署一个**整合 agent（consolidation agent）**——它读取最近对话历史、提取关键事实并合并到 memory store 中——并通过 [cron 计划](#cron)触发它。选择能反映用户实际与 agent 交互频率的节奏：一个每天有稳定流量的聊天产品可能每隔几小时整合一次，而一个每周只使用几次的工具只需要每晚或每周运行一次。整合频率远高于用户对话频率，只会在无操作的空跑中浪费 tokens。

#### 整合 agent

整合 agent 读取最近的对话历史，并将关键事实合并到 memory store 中。在 `langgraph.json` 中将其与你的主 agent 一起注册：

```typescript src/consolidation-agent.ts
import { createDeepAgent } from "deepagents";
import { Client } from "@langchain/langgraph-sdk";
import { tool } from "@langchain/core/tools";

const sdkClient = new Client({ apiUrl: "<DEPLOYMENT_URL>" });

const searchRecentConversations = tool(
  async ({ query }, runtime) => {
    const userId = runtime.serverInfo.user.identity;  // [!code highlight]

    const since = new Date(Date.now() - 6 * 60 * 60 * 1000).toISOString();
    const threads = await sdkClient.threads.search({
      metadata: { userId },
      updatedAfter: since,
      limit: 20,
    });
    const conversations = [];
    for (const thread of threads) {
      const history = await sdkClient.threads.getHistory(thread.threadId);
      conversations.push(history.values.messages);
    }
    return JSON.stringify(conversations);
  },
  {
    name: "search_recent_conversations",
    description: "Search this user's conversations updated in the last 6 hours.",
  }
);

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  systemPrompt: `Review recent conversations and update the user's memory file.
Merge new facts, remove outdated information, and keep it concise.`,
  tools: [searchRecentConversations],
});

export { agent };
```

```json langgraph.json
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent.ts:agent",
    "consolidation_agent": "./src/consolidation-agent.ts:agent"
  },
  "env": ".env"
}
```

#### Cron

[cron job](/langsmith/cron-jobs) 按固定计划运行整合 agent。agent 搜索最近的对话并将它们综合到 memory 中。将计划与你的使用模式相匹配，使整合运行的频率大致跟踪真实活动。

```mermaid
graph LR
    Store[(Memory store)] -.->|reads| Conv1[Conversation 1]
    Store -.->|reads| Conv2[Conversation 2]
    Cron[Cron schedule] -->|periodic| Agent[Consolidation agent]
    Agent -->|writes| Store

    classDef trigger fill:#F6FFDB,stroke:#6E8900,stroke-width:2px,color:#2E3900
    classDef process fill:#E5F4FF,stroke:#006DDD,stroke-width:2px,color:#030710
    classDef output fill:#EBD0F0,stroke:#885270,stroke-width:2px,color:#441E33
    classDef schedule fill:#FDF3FF,stroke:#7E65AE,stroke-width:2px,color:#504B5F

    class Conv1,Conv2 trigger
    class Agent process
    class Store output
    class Cron schedule
```

使用 cron job 调度整合 agent：

```typescript
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>" });

const cronJob = await client.crons.create(
  "consolidation_agent",
  {
    schedule: "0 */6 * * *",
    input: { messages: [{ role: "user", content: "Consolidate recent memories." }] },
  },
);
```

> **注意**：所有 cron 计划都以 **UTC** 解释。关于管理和删除 cron jobs 的详细信息，请参阅 [cron jobs](/langsmith/cron-jobs)。

> **警告**：cron 间隔必须与整合 agent 内部的回看窗口匹配。上面的示例每 6 小时运行一次（`0 */6 * * *`），而 agent 的 `search_recent_conversations` tool 回看 `timedelta(hours=6)` —— 保持两者同步。如果 cron 运行频率高于回看窗口，你会重新处理相同的对话；如果运行频率低于回看窗口，你就会丢弃落在窗口之外的 memories。

关于部署带有后台进程的 agent 的更多内容，请参阅[投入生产](/oss/javascript/deepagents/going-to-production)。

### 只读 vs 可写 memory

默认情况下，agent 既可以读取也可以写入 memory 文件。对于像组织策略或合规规则这样的共享状态，你可能希望将 memory 设为**只读**，使 agent 可以引用它但不能修改它。这可以防止通过共享 memory 进行提示注入，并确保只有你的应用代码控制文件中的内容。

| 权限                        | 用例                                                                                                                          | 工作原理                                                                                                                                                                                                                                                                                         |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **读写**（默认）            | 用户偏好、agent 自我改进、习得的 [skills](/oss/javascript/deepagents/skills)                                                  | agent 通过 `edit_file` tool 更新文件                                                                                                                                                                                                                                                            |
| **只读**                    | 组织策略、合规规则、共享知识库、开发者定义的 [skills](/oss/javascript/deepagents/skills)                                       | 通过应用代码或 [Store API](/langsmith/custom-store) 填充。使用 [permissions](/oss/javascript/deepagents/permissions) 拒绝对特定路径的写入，或使用 [policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks) 实现自定义验证逻辑。                                                    |

**安全考量：** 如果一个用户可以写入另一个用户读取的 memory，恶意用户就可能向共享状态注入指令。为缓解此问题：

* **默认采用用户作用域** `(user_id)`，除非你有特定理由要共享
* 对共享策略使用**只读 memory**（通过应用代码填充，而不是通过 agent）
* 在 agent 写入共享 memory 之前添加 **Human-in-the-loop** 验证。使用 [interrupt](/oss/javascript/langgraph/interrupts) 要求对写入敏感路径的操作进行人工批准。

要强制 memory 只读，请使用 [permissions](/oss/javascript/deepagents/permissions) 以声明方式拒绝对特定路径的写入。对于自定义验证逻辑（速率限制、审计日志、内容检查），请使用 [backend policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks)。

### 并发写入

多个 thread 可以并行写入 memory，但对**同一文件**的并发写入可能导致后写覆盖先写（last-write-wins）冲突。对于用户作用域 memory，这种情况很少见，因为用户通常一次只有一个活跃对话。对于 agent 作用域或组织作用域 memory，请考虑使用[后台整合](#background-consolidation)来串行化写入，或按主题将 memory 组织为多个独立文件以减少争用。

在实践中，如果一次写入因冲突而失败，LLM 通常足够聪明，能够重试或优雅地恢复，因此单次写入丢失并不致命。

### 同一部署中的多个 agent

要使每个 agent 在共享部署中拥有自己的 memory，请将 `assistant_id` 添加到 namespace：

```typescript
new StoreBackend({
  namespace: (rt) => [
    rt.serverInfo.assistantId,  // [!code highlight]
    rt.serverInfo.user.identity,
  ],
})
```

如果你只需要按 agent 隔离而无需按用户设置作用域，可以单独使用 `assistant_id`。

> **提示**：使用 [LangSmith tracing](/langsmith/trace-with-langgraph) 审计 agent 写入 memory 的内容。每次文件写入都会在 trace 中显示为一次 tool call。

## 另请参阅

* [OpenWiki](/oss/openwiki/overview)：生成并维护 coding agents 通过 `AGENTS.md` 找到的仓库 wiki
* [Backends](/oss/javascript/deepagents/backends)：选择 memory 文件的存储位置
* [Context engineering](/oss/javascript/deepagents/context-engineering)：短期记忆、卸载和摘要
* [Skills](/oss/javascript/deepagents/skills)：按需的程序性记忆
