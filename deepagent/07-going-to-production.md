# 走向生产环境

> 原文：Going to production
> 来源：https://docs.langchain.com/oss/javascript/deepagents/going-to-production

---

> 借助持久化 memory、sandboxes、韧性 middleware 和多种部署选项，将你的 deep agent 推向生产环境。

本指南涵盖将 deep agent 从本地原型推进到生产部署时需要考虑的事项。它逐一介绍如何划定 memory 的作用范围、配置执行环境、添加 guardrails 以及连接前端。

## 概述

Agent 使用来自 memory 及其执行环境的信息来完成任务。
在生产环境中，有几个基础原语（primitives）决定了信息如何被共享和访问：

* **Thread**：单次对话。消息历史和临时文件默认限定在该 thread 内，不会延续到其他 thread。
* **User**：与你的 agent 交互的人。Memory 和文件可以是用户私有的，也可以在用户之间共享。身份与授权来自你的 [auth layer](/langsmith/auth)。
* **Assistant**：一个已配置的 agent 实例。Memory 和文件可以绑定到单个 assistant，也可以在所有 assistant 之间共享。

本页涵盖：

* **[LangSmith Deployments](#langsmith-deployments)**：托管基础设施，内置 auth、webhooks 和 cron
* **[生产环境考量](#production-considerations)**：调用、多租户、认证、凭据、异步与持久性
* **[Memory](#memory)**：跨对话持久化信息
* **[执行环境](#execution-environment)**：文件存储与代码执行
* **[Guardrails](#guardrails)**：权限与数据隐私
* **[前端](#frontend)**：将你的 UI 连接到已部署的 agent

## LangSmith Deployments

![Managed Deep Agents packages your agent configuration, tools, and runtime settings for LangSmith](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/deepagents-deploy-config.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=5c3f2961994afe5fe67a2f5c9e9ba7ac)

将 Deep Agent 投入生产环境的推荐路径是 [Managed Deep Agents](/langsmith/javascript/managed-deep-agents-overview)，这是一个 CLI 优先的托管运行时，用于在 LangSmith 中创建、运行和运维 deep agents。Managed Deep Agents 目前处于私有预览阶段（[加入等待列表](https://www.langchain.com/langsmith-managed-deep-agents-waitlist)）。对于需要自定义应用代码、自定义路由、高级认证的团队，你可以直接配置一个 [LangSmith Deployment](/langsmith/deployment)。无论哪条路径，都会为你的 agent 预置所需的基础设施：[threads](/langsmith/use-threads)、[runs](/langsmith/runs)、一个 store 以及一个 checkpointer，因此你无需自行搭建这些。传统的 LangSmith Deployment 还会开箱即用地为你提供 [authentication](/langsmith/auth)、[webhooks](/langsmith/use-webhooks)、[cron jobs](/langsmith/cron-jobs) 和 [observability](/langsmith/observability)，并可以通过 [MCP](/langsmith/server-mcp) 或 [A2A](/langsmith/server-a2a) 暴露你的 agent。

你也可以在不使用 LangSmith Cloud 的情况下，部署到 JavaScript 框架和托管平台上。

**框架与平台**

[查看所有指南](/langsmith/deploy-frameworks-and-platforms)

* [LangSmith](/langsmith/deploy-vite-langsmith)
* [Next.js](/langsmith/deploy-nextjs)
* [SvelteKit](/langsmith/deploy-sveltekit)
* [Nuxt](/langsmith/deploy-nuxt)
* [Cloudflare](/langsmith/deploy-cloudflare-workers)
* [Deno](/langsmith/deploy-deno)

> **提示**：LangSmith Cloud 部署会自动将 trace 发送到以你的部署命名的项目中。打开 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-deepagents-going-to-production) 即可调试 run 并监控用量。对于混合或自托管部署，请参阅 [LangSmith tracing](/langsmith/data-plane#langsmith-tracing)。我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的 trace、检测问题并提出修复方案。

除非另有说明，本页所有代码片段都使用以下 `langgraph.json`：

```json
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent.ts:agent"
  },
  "env": ".env"
}
```

`langgraph.json` 是配置文件，用于告诉 LangGraph 平台如何构建和运行你的应用。它位于项目根目录，是本地开发（使用 `langgraph dev`）和生产部署都必需的。关键字段如下：

| Field          | Description                                                                                                                                                                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dependencies` | 要安装的包。`["."]` 会将当前目录作为包安装（读取自 `requirements.txt`、`pyproject.toml` 或 `package.json`）。                                                                                                  |
| `graphs`       | 将 graph ID 映射到其代码位置。每一项格式为 `"<id>": "./<file>:<variable>"`，其中 `<id>` 是你通过 API 调用 graph 时使用的名称，`<variable>` 是从 `<file>` 导出的已编译 graph 或构造函数。 |
| `env`          | 指向含有环境变量（API keys、secrets）的 `.env` 文件的路径。这些变量在构建时设置，并在运行时可用。                                                                                                                     |

关于完整的配置选项集合（自定义 Docker 步骤、store 索引、auth handlers 等），请参阅 [application structure](/oss/javascript/langgraph/application-structure)。

## 生产环境考量

### 调用 agent

在生产环境中，每次调用都应携带两个 run 级别的参数：

* **`thread_id`**（通过 `config={"configurable": {"thread_id": ...}}` 传入）：对话的稳定标识符。[checkpointer](#durability) 用它来持久化并恢复消息历史，因此后续轮次会延续同一个对话。生成新的 `thread_id` 即可开启全新对话。
* **`context`**：每次 run 的数据，由你的工具和 middleware 在调用时读取，例如 `user_id`、API keys、feature flags 或会话元数据。使用 `context_schema` 定义其结构，并通过 `runtime.context` 访问它。参阅 [Runtime context](/oss/javascript/deepagents/context-engineering#runtime-context)。

两者相互独立，且几乎总是同时传入：

#### Google

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "google:gemini-3.6-flash",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### OpenAI

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "openai:gpt-5.5",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### Anthropic

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "anthropic:claude-sonnet-5",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### OpenRouter

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "openrouter:z-ai/glm-5.2",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### Fireworks

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "fireworks:accounts/fireworks/models/glm-5p2",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### Baseten

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "baseten:zai-org/GLM-5.2",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

#### Ollama

```ts
import { createDeepAgent } from "deepagents";
import { z } from "zod";

const contextSchema = z.object({ userId: z.string() });

const agent = createDeepAgent({
  model: "ollama:north-mini-code-1.0",
  contextSchema,
});

// Start a conversation
const config = { configurable: { thread_id: crypto.randomUUID() } };
await agent.invoke(
  { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
  { ...config, context: { userId: "user-123" } },
);

// Follow-up on the same conversation: reuse the same thread_id
await agent.invoke(
  { messages: [{ role: "user", content: "Make it 5 days instead" }] },
  { ...config, context: { userId: "user-123" } },
);
```

> **查看示例 trace**（[链接](https://smith.langchain.com/public/92d99101-7d19-46ac-8050-24d1cad6039b/r)）：打开此示例的公开 LangSmith run。

使用 LangGraph SDK 部署时，SDK 会为你管理 threads，你只需将返回的 `thread_id` 传给每次 run：

```typescript
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>", apiKey: "<LANGSMITH_API_KEY>" });

const thread = await client.threads.create();
for await (const chunk of client.runs.stream(
  thread.thread_id,  // [!code highlight]
  "agent",
  {
    input: { messages: [{ role: "user", content: "Plan a 3-day trip to Tokyo" }] },
    context: { userId: "user-123" },  // [!code highlight]
    streamMode: "updates",
  },
)) {
  console.log(chunk.data);
}
```

> **提示**：`thread_id` 划定的是*对话*（消息历史、checkpoints）的范围。`context` 承载你的工具和 middleware 读取的*每次 run* 的数据。二者相互独立：改变其中一个不会影响另一个，你可以只传其中一个，也可以都传。

### 多租户

当你的 agent 服务于多个用户时，你需要处理三件事：验证每个用户是谁、控制他们能访问什么，以及管理 agent 代表他们行事所使用的凭据。

![Three authentication layers compose: end-user auth, agent-acting-as-user auth, and team RBAC](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/auth-layers.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=11137d8bae74093c459416fd3e541d68)

#### 用户身份与访问控制

[LangSmith Deployments](/langsmith/deployment) 支持通过 [custom authentication](/langsmith/custom-auth) 建立用户身份，并通过 [authorization handlers](/langsmith/auth) 控制对 threads、assistants 和 store namespaces 等资源的访问。Authorization handlers 在认证成功后运行，可以：

* 用所有权元数据为资源打标签（例如 `owner: user_id`）
* 返回过滤器，使用户只能看到自己的资源
* 对未授权的操作以 HTTP 403 拒绝访问

关于分步教程，请参阅 [Make conversations private](/langsmith/resource-auth)。关于演练，请观看 [custom auth video](https://www.youtube.com/watch?v=DkNqgCz8cjE)。

你如何[划定 memory 范围](#scoping)以及[执行环境](#execution-environment)，决定了用户之间共享哪些数据。详情见下文各节。

#### 团队访问控制（RBAC）

LangSmith 的 [role-based access control](/langsmith/rbac) 管辖团队中谁可以部署、配置和监控 agents。这与上面的终端用户授权是分开的。

| Role             | Access                                                                |
| ---------------- | --------------------------------------------------------------------- |
| Workspace Admin  | 拥有全部权限，包括设置和成员管理             |
| Workspace Editor | 可以创建和修改资源，但不能删除 runs 或管理成员 |
| Workspace Viewer | 只读访问                                                      |

更细粒度权限的自定义角色在 Enterprise 计划中可用。关于完整的权限模型，请参阅 [RBAC reference](/langsmith/rbac)。

#### 终端用户凭据

当你的 agent 需要代表用户调用外部 API 时（例如读取其 GitHub repos、发送 Slack 消息、查询其数据仓库），你需要一种方式将用户的凭据传递给 agent，而不必将它们硬编码。

**通过 Agent Auth 使用 OAuth。** [Agent Auth](/langsmith/agent-auth) 提供托管的 OAuth 2.0 流程。配置 OAuth provider 后，agent 可以请求限定到每个用户的 tokens。首次使用时，agent 会[中断](/oss/javascript/langgraph/interrupts)执行并给出一个 OAuth 同意 URL。用户完成认证后，agent 携带有效 token 恢复执行。Tokens 会被自动存储和刷新。

```typescript
import { Client } from "@langchain/auth";

const authClient = new Client();

// Inside your agent's tool:
// Access the authenticated user via runtime.serverInfo
const authResult = await authClient.authenticate({
  provider: "github",
  scopes: ["repo", "read:org"],
  userId: runtime.serverInfo.user.identity,  // [!code highlight]
});
// Use authResult.token for GitHub API calls on the user's behalf
```

**为 sandboxes 注入凭据。** 如果你的 agent 在 [sandbox](#sandboxes) 中运行调用外部 API 的代码，[sandbox auth proxy](/langsmith/sandbox-auth-proxy) 可以自动将凭据注入到出站请求中，因此 sandbox 代码永远不会接触原始 API keys。设置细节请参阅 [Managing secrets](#managing-secrets)。

**Workspace secrets。** 对于在所有用户之间共享的 API keys（例如你所在组织的 LLM provider keys、search API keys），请将它们作为 [workspace secrets](/langsmith/set-up-hierarchy#configure-workspace-settings) 存储在 LangSmith 中。详情请参阅 [Managing secrets](#managing-secrets)。

### 异步

基于 LLM 的应用高度受 I/O 约束：调用 language models、数据库和外部服务。异步编程让这些操作并发执行而非阻塞，从而提升吞吐量和响应性。

> **注意**：LangChain 遵循在异步方法名前加 `a` 的约定（例如 `ainvoke`、`abefore_agent`、`astream`）。同步和异步变体位于同一个类或命名空间中。

构建生产环境时：

* **创建异步 tools。** LangChain 会在单独的线程中运行同步 tools 以避免阻塞，但原生异步完全避免了线程开销。
* **使用异步 middleware 方法。** 自定义 [middleware](/oss/javascript/langchain/middleware/custom) 应实现异步钩子（例如 `abefore_agent` 而非 `before_agent`）。
* **在外部资源生命周期中使用异步。** 创建 [sandboxes](#sandboxes) 或连接 [MCP servers](/oss/javascript/langchain/mcp) 涉及网络调用，应当 await。这正是那些预置这些资源的 [graph factories](/langsmith/graph-rebuild) 是异步的原因。

### 持久性

Deep Agents 运行在 LangGraph 上，后者开箱即用地提供持久化执行。[persistence](/oss/javascript/langgraph/persistence) 层在每一步都对状态做 checkpoint，因此被失败、超时或 [human-in-the-loop](/oss/javascript/langgraph/interrupts) 暂停中断的 run 会从最后一次记录的状态恢复，无需重新处理之前的步骤。对于会派生许多 subagents 的长时间运行的 deep agents 来说，这意味着 run 中途的失败不会丢失已完成的工作。

![Durable execution: when a worker crashes mid-run, another worker picks the run up from the latest checkpoint](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/durable-execution.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=d3bfd69460769dba142c68c7a20ae43b)

Checkpointing 还能实现：

* **无限期的 [interrupts](/oss/javascript/langgraph/interrupts)。** Human-in-the-loop 工作流可以暂停数分钟甚至数天，并精确地从上次中断处恢复。
* **[Time travel](/oss/javascript/langgraph/use-time-travel)。** 每个已 checkpoint 的步骤都是一个可回退到的快照，让你在出错时可以从更早的状态重放。
* **安全处理敏感操作。** 对于涉及支付或其它不可逆操作的工作流，checkpoints 提供了审计轨迹和恢复点，以便检查导致某操作的确切状态。

> **提示**：[LangSmith Deployments](/langsmith/deployment) 会自动配置持久化 checkpointer。如果你自托管，请参阅 [persistence](/oss/javascript/langgraph/persistence) 了解设置说明。

## Memory

没有 memory，每次对话都从零开始。Memory 让你的 agent 跨对话保留信息（用户偏好、已学到的指令、过往经验），从而能够随时间推移个性化其行为。关于 memory 类型的概述，请参阅 [memory concepts guide](/oss/javascript/concepts/memory)。

![Short-term memory is scoped to a single thread via checkpoints; long-term memory persists across threads via the store](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/memory.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=3ec585271dcd8d62e0207d79d68c296b)

### 划定范围

Memory 始终跨对话持久化。主要问题在于它如何在用户和 assistant 边界之间划定范围。正确的范围取决于谁应当查看和修改这些数据：

| Scope                          | Namespace        | Use case                                        | Example                           |
| ------------------------------ | ---------------- | ----------------------------------------------- | --------------------------------- |
| **User**（推荐的默认值） | `(user_id)`      | 每个用户的偏好和上下文                | "I prefer concise responses"      |
| **Assistant**                  | `(assistant_id)` | 单个 assistant 的共享指令           | "Cap posts at 280 characters"     |
| **Global**                     | `(org_id)`       | 适用于所有用户和 assistant 的只读策略 | "Never disclose internal pricing" |

> **警告**：共享 memory（assistant、user 或 organization 范围）是 prompt injection 的载体。如果一个用户可以写入另一个用户对话所读取的 memory，恶意用户就可能向该共享状态注入指令。请在适当的地方强制只读访问。例如，让组织级策略只能通过应用代码写入，而不能由 agent 自身写入。使用 [permissions](/oss/javascript/deepagents/permissions) 以声明方式拒绝写入共享路径，或使用 [backend policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks) 实现自定义校验逻辑。

### 配置

在 Deep Agents 中，memory 以文件形式存储在虚拟文件系统中。默认情况下，文件限定在单个 thread（对话）内，不跨 threads 共享。
否则，要跨 threads 共享 memory，请将诸如 `/memories/` 这样的路径路由到一个写入 LangGraph [Store](/langsmith/custom-store) 的 [StoreBackend](https://reference.langchain.com/javascript/deepagents/backends/StoreBackend)。使用 [CompositeBackend](https://reference.langchain.com/javascript/deepagents/backends/CompositeBackend) 让 agent 同时拥有 thread 作用域的临时空间和跨 thread 的[长期 memory](/oss/javascript/deepagents/memory)。

> **注意**：下面展示的 `rt.serverInfo` 和 `rt.executionInfo` 命名空间模式需要 `deepagents>=1.9.0`。

#### User（推荐）

按 `user_id` 划定命名空间。每个用户获得自己私有的 memory。这是推荐的默认值，因为大多数应用只部署单个 assistant。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

export const agent = createDeepAgent({
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [
          rt.serverInfo.assistantId,  // [!code highlight]
          rt.serverInfo.user.identity,  // [!code highlight]
        ],
      }),
    },
  ),
  systemPrompt: `You have persistent memory at /memories/.

  Read /memories/instructions.txt at the start of each conversation for
  accumulated knowledge and preferences. When you learn something that
  should persist, update that file.`,
});
```

#### Assistant

按 `assistant_id` 划定命名空间。Memory 在同一个 assistant 的所有用户之间共享，因此任何用户都可以读取或更新它。用于适用于使用某个 assistant 的所有人的共享指令或知识（例如 "always reply in formal tone"）。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

export const agent = createDeepAgent({
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.assistantId],  // [!code highlight]
      }),
    },
  ),
});
```

#### User

仅按 `user_id` 划定命名空间。Memory 跟随用户跨所有 assistants。用于应适用于用户无论与哪个 assistant 交谈的全局用户档案（姓名、时区、沟通偏好）。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

export const agent = createDeepAgent({
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.serverInfo.user.identity],  // [!code highlight]
      }),
    },
  ),
});
```

#### Organization

按 `org_id` 划定命名空间。Memory 在所有用户和所有 assistants 之间共享。通常用于应对 agent 保持只读的组织级策略（合规规则、品牌指南）。写入访问应限制在应用代码中，以防止 prompt injection。

```typescript
import { createDeepAgent, CompositeBackend, StateBackend, StoreBackend } from "deepagents";

export const agent = createDeepAgent({
  backend: new CompositeBackend(
    new StateBackend(),
    {
      "/memories/": new StoreBackend({
        namespace: (rt) => [rt.context.orgId],
      }),
    },
  ),
});
```

你还可以使用 [Store API](/langsmith/custom-store) 从应用代码中读写 store。示例请参阅 [Advanced usage](/oss/javascript/deepagents/memory#advanced-usage)。

关于完整的 namespace factory API，请参阅 [namespace factories](/oss/javascript/deepagents/backends#namespace-factories)。关于自我改进指令和知识库等 memory 模式，请参阅[长期 memory](/oss/javascript/deepagents/memory)。

## 执行环境

在本地，agent 可以读写磁盘上的文件并直接运行 shell 命令。在生产环境中，你需要考虑隔离和持久性。正确的设置取决于你的 agent 是否需要执行代码：

* 如果你的 agent 只读写文件，**Filesystem backends** 就足够了。选择一个符合你持久性需求的 backend：thread 作用域的临时空间、跨 thread 存储，或两者兼有。
* **Sandboxes** 增加了一个带有 `execute` tool 的隔离容器，用于运行 shell 命令。如果你的 agent 需要运行代码、安装包或做任何超出文件 I/O 的事情，请使用 sandbox。

### 文件系统

根据需要持久化的内容来选择 backend：

* [StateBackend](https://reference.langchain.com/javascript/deepagents/backends/StateBackend)（默认）：thread 作用域的临时空间。文件在一个 thread 内通过 checkpointer 跨轮次持久化，但不跨 threads 共享。每一步都会 checkpoint，因此避免写入大文件。
* [StoreBackend](https://reference.langchain.com/javascript/deepagents/backends/StoreBackend)：跨 thread 存储，可跨对话存活。使用 [namespace factory](/oss/javascript/deepagents/backends#namespace-factories) 划定范围。
* [CompositeBackend](https://reference.langchain.com/javascript/deepagents/backends/CompositeBackend)：混合两者。默认是 thread 作用域的临时空间，并为诸如 `/memories/` 之类的特定路径提供跨 thread 路由。

关于完整的 backends 列表以及如何构建自定义 backend，请参阅 [backends](/oss/javascript/deepagents/backends)。

> **警告**：`FilesystemBackend` 和 `LocalShellBackend` 直接访问宿主机。不要在已部署的 agents 中使用它们。

### Sandboxes

如果你的 agent 需要运行代码（而不仅仅是读写文件），请使用 [sandbox](/oss/javascript/deepagents/sandboxes)。Sandboxes 同时提供文件系统和一个用于运行 shell 命令的 `execute` tool，全部位于一个隔离容器内。这种隔离也保护你的宿主机：如果 agent 的代码耗尽内存或崩溃，只会影响 sandbox。你的服务器会继续运行。

#### 生命周期

关键决策是 sandbox 存活多久。每次对话都获得一个全新的，还是多个对话共享一个持久环境？

| 作用域               | Sandbox ID 存储位置                      | 生命周期                                 | 示例用例                                                             |
| -------------------- | ----------------------------------------- | ----------------------------------------- | -------------------------------------------------------------------- |
| **Thread 作用域**    | [Thread](/langsmith/use-threads) 元数据  | 每次对话都是全新的，在 TTL 时清理 | 一个数据分析 bot，每次对话都从干净状态开始             |
| **Assistant 作用域** | [Assistant](/langsmith/assistants) 配置  | 在所有对话之间共享           | 一个编码 assistant，跨对话维持一个已克隆的 repo |

> **注意**：下面的示例使用异步 [graph factory](/langsmith/graph-rebuild) 而非静态 graph，因为 sandbox 需要 `thread_id` 或 `assistant_id` 来查找或创建正确的 sandbox。Graph factories 不会收到完整的 `Runtime`（没有 `server_info` 或 `execution_info`）；相反，它接收一个 `RunnableConfig`，并从 `config["configurable"]` 读取 `thread_id` 和 `assistant_id`。该 factory 是异步的，因为 sandbox 创建是一种受 I/O 约束的操作，需要仅在调用时才能获得的每次 run 信息。

#### Thread-scoped（最常见）

每次对话获得自己的 sandbox。[graph factory](/langsmith/graph-rebuild) 从 run config 读取 `thread_id`，因此每个 [thread](/langsmith/use-threads) 自动获得自己隔离的环境。具名 sandbox 查找会处理跨 run 的去重。当 sandbox 的 [TTL](/langsmith/configure-ttl) 过期时会被清理。

```typescript
import { createDeepAgent, LangSmithSandbox } from "deepagents";
import { SandboxClient } from "langsmith/sandbox";
import type { LangGraphRunnableConfig } from "@langchain/langgraph";

const client = new SandboxClient();

export async function agent(config: LangGraphRunnableConfig) {
  const threadId = config.configurable?.thread_id as string;  // [!code highlight]
  const sandboxName = `thread-${threadId}`;
  const existing = (await client.listSandboxes()).filter(
    (sb) => sb.name === sandboxName,
  );
  const lsSandbox =
    existing[0] ??
    (await client.createSandbox({
      name: sandboxName,
      idleTtlSeconds: 3600, // TTL: clean up when idle
    }));
  return createDeepAgent({
    model: "google:gemini-3.6-flash",
    backend: new LangSmithSandbox({ sandbox: lsSandbox }),
  });
}
```

#### Assistant-scoped

所有对话共享一个 sandbox。[graph factory](/langsmith/graph-rebuild) 从 `config["configurable"]` 读取 [assistant](/langsmith/assistants) ID，因此同一 assistant 上的每个 thread 都会回到同一个环境。文件、已安装的包以及克隆的仓库跨对话持久存在。

```typescript
import { createDeepAgent, LangSmithSandbox } from "deepagents";
import { SandboxClient } from "langsmith/sandbox";
import type { LangGraphRunnableConfig } from "@langchain/langgraph";

const client = new SandboxClient();

export async function agent(config: LangGraphRunnableConfig) {
  const assistantId = config.configurable?.assistant_id as string;  // [!code highlight]
  const sandboxName = `assistant-${assistantId}`;
  const existing = (await client.listSandboxes()).filter(
    (sb) => sb.name === sandboxName,
  );
  const lsSandbox =
    existing[0] ??
    (await client.createSandbox({
      name: sandboxName,
    }));
  return createDeepAgent({
    model: "google:gemini-3.6-flash",
    backend: new LangSmithSandbox({ sandbox: lsSandbox }),
  });
}
```

> **警告**：Assistant-scoped sandboxes 会随时间累积文件、已安装的包以及其它 sandbox 内状态。请在你的 sandbox provider 上配置 TTL，使用快照定期重置，或实现清理逻辑，以防止 sandbox 的磁盘和内存无界增长。

因为 `agent` 变量是一个异步函数（而非已编译的 graph），服务器将其视为 [graph factory](/langsmith/graph-rebuild)，并在每次 run 时调用它，注入 config。该 factory 按名称查找或创建 sandbox，并返回一个连接到该 sandbox 的全新 agent graph。

一旦使用 `langgraph deploy` 部署，即可从你的应用代码使用 SDK 调用该 agent。无论作用域如何，客户端代码都相同。作用域完全在上面的 agent factory 中处理，但行为有所不同：

#### Thread-scoped

每个 thread 获得自己的 sandbox。同一 thread 内的后续消息复用同一个 sandbox，但新 thread 总是从干净状态开始，没有之前对话遗留的文件或已安装的包。

```typescript
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>", apiKey: "<LANGSMITH_API_KEY>" });

// Conversation 1: install pandas and analyze data
const thread1 = await client.threads.create();
for await (const chunk of client.runs.stream(
  thread1.thread_id,
  "agent",
  { input: { messages: [{ role: "human", content: "Install pandas and analyze sales_data.csv" }] } },
)) {
  console.log(chunk.data);
}

// Follow-up in the same conversation — pandas is still installed
for await (const chunk of client.runs.stream(
  thread1.thread_id,
  "agent",
  { input: { messages: [{ role: "human", content: "Now plot the results" }] } },
)) {
  console.log(chunk.data);
}

// Conversation 2: fresh sandbox — pandas is NOT installed, no files from conversation 1
const thread2 = await client.threads.create();
for await (const chunk of client.runs.stream(
  thread2.thread_id,
  "agent",
  { input: { messages: [{ role: "human", content: "What packages are installed?" }] } },
)) {
  console.log(chunk.data);
}
```

#### Assistant-scoped

所有 threads 共享一个 sandbox。当 sandbox 含有重建成本高昂的状态时（例如已克隆的 repo、已安装的依赖或构建产物），这很有用。同一 assistant 上的任何对话都会从上一次离开的地方继续，无需重复设置。

```typescript
import { Client } from "@langchain/langgraph-sdk";

const client = new Client({ apiUrl: "<DEPLOYMENT_URL>", apiKey: "<LANGSMITH_API_KEY>" });

// Conversation 1: clone and set up the project
const thread1 = await client.threads.create();
for await (const chunk of client.runs.stream(
  thread1.thread_id,
  "agent",
  { input: { messages: [{ role: "human", content: "Clone https://github.com/org/repo and install dependencies" }] } },
)) {
  console.log(chunk.data);
}

// Conversation 2: repo and dependencies are still there
const thread2 = await client.threads.create();
for await (const chunk of client.runs.stream(
  thread2.thread_id,
  "agent",
  { input: { messages: [{ role: "human", content: "Run the test suite and fix any failures" }] } },
)) {
  console.log(chunk.data);
}
```

#### 文件传输

Sandboxes 是隔离的容器，因此你的应用代码无法直接访问其中的文件。使用 `upload_files()` 和 `download_files()` 跨 sandbox 边界移动数据：

* **在 agent 运行前为 sandbox 播种**：上传用户文件、[skill](/oss/javascript/deepagents/skills) 脚本、配置或[持久化 memories](/oss/javascript/deepagents/memory)，使 agent 从一开始就拥有所需内容
* **在 agent 完成后取回结果**：下载生成的产物（报告、图表、导出文件），并将更新后的 memories 同步回去以供未来对话使用

关于特定 provider 的文件传输示例，请参阅 [working with files](/oss/javascript/deepagents/sandboxes#working-with-files)。关于 provider 设置、安全性和生命周期模式，请参阅完整的 [sandboxes guide](/oss/javascript/deepagents/sandboxes)。

#### 示例：使用自定义 middleware 同步 skills 与 memories

Agent 需要执行的 [Skill](/oss/javascript/deepagents/skills) 脚本必须在 agent 运行前上传到 sandbox 中。你可能还希望同步 [memories](/oss/javascript/deepagents/memory)，以便 agent 能在容器内读取和更新它们。使用带有 `before_agent` 和 `after_agent` 钩子的[自定义 middleware](/oss/javascript/langchain/middleware/custom)跨 sandbox 边界移动文件：

```typescript
import { createMiddleware } from "langchain";
import {
  createDeepAgent,
  CompositeBackend,
  LangSmithSandbox,
  StoreBackend,
} from "deepagents";
import { SandboxClient } from "langsmith/sandbox";

function safeFilename(key: string): string {
  const name = key.split("/").pop()!;
  if (name.includes("..") || /[*?]/.test(name)) {
    throw new Error(`Invalid key: ${key}`);
  }
  return name;
}

const createSandboxSyncMiddleware = (backend: CompositeBackend) => {
  return createMiddleware({
    name: "SandboxSyncMiddleware",
    beforeAgent: async (state, runtime) => {
      // Upload skill scripts and memories into the sandbox
      const userId = runtime.serverInfo.user.identity;  // [!code highlight]
      const store = runtime.store;
      const encoder = new TextEncoder();
      const files: [string, Uint8Array][] = [];
      for (const item of await store.search(["skills", userId])) {
        const name = safeFilename(item.key);
        files.push([`/skills/${name}`, encoder.encode(item.value.content)]);
      }
      for (const item of await store.search(["memories", userId])) {
        const name = safeFilename(item.key);
        files.push([`/memories/${name}`, encoder.encode(item.value.content)]);
      }
      if (files.length > 0) {
        await backend.uploadFiles(files);
      }
    },
    afterAgent: async (state, runtime) => {
      // Sync updated memories back to the store
      const userId = runtime.serverInfo.user.identity;  // [!code highlight]
      const store = runtime.store;
      const items = await store.search(["memories", userId]);
      const results = await backend.downloadFiles(
        items.map((item) => `/memories/${item.key}`),
      );
      const decoder = new TextDecoder();
      for (const result of results) {
        if (result.content) {
          await store.put(
            ["memories", userId],
            result.path.split("/").pop()!,
            { content: decoder.decode(result.content) },
          );
        }
      }
    },
  });
};

const client = new SandboxClient();
const lsSandbox = await client.createSandbox();

const backend = new CompositeBackend(
  new LangSmithSandbox({ sandbox: lsSandbox }),
  {
    "/skills/": new StoreBackend({
      namespace: (rt) => ["skills", rt.serverInfo.user.identity],  // [!code highlight]
    }),
    "/memories/": new StoreBackend({
      namespace: (rt) => ["memories", rt.serverInfo.user.identity],  // [!code highlight]
    }),
  },
);

export const agent = createDeepAgent({
  backend,
  middleware: [createSandboxSyncMiddleware(backend)],
});
```

#### 管理 secrets

Sandboxes 是隔离的容器，因此宿主机上的环境变量在其中不可用。有两种方式向 sandbox 代码提供 API keys 和其它 secrets：

**Auth proxy（推荐）。** [sandbox auth proxy](/langsmith/sandbox-auth-proxy) 拦截来自 sandbox 的出站请求并自动注入认证头。Sandbox 代码正常调用外部 API，proxy 根据目标 host 添加正确的凭据。这意味着 API keys 永远不会出现在 sandbox 代码、环境变量或日志中。

![The sandbox auth proxy injects credentials into outbound requests so secrets never enter the sandbox](https://mintcdn.com/langchain-5e9cc07a/9oyV6nbtSbBRfaE1/oss/images/deepagents/production/sandbox-auth-proxy.png?fit=max&auto=format&n=9oyV6nbtSbBRfaE1&q=85&s=632c4a493f1d5928e41c6865ab86d1da)

```json
{
  "proxy_config": {
    "rules": [
      {
        "name": "openai-api",
        "match_hosts": ["api.openai.com"],
        "inject_headers": {
          "Authorization": "Bearer ${OPENAI_API_KEY}"
        }
      },
      {
        "name": "anthropic-api",
        "match_hosts": ["api.anthropic.com"],
        "inject_headers": {
          "x-api-key": "${ANTHROPIC_API_KEY}"
        }
      }
    ]
  }
}
```

`${SECRET_KEY}` 引用会对照存储在你 LangSmith [workspace settings](/langsmith/set-up-hierarchy#configure-workspace-settings) 中的 secrets 进行解析。在创建引用它们的 template 之前，请先在那里配置 secrets。

**Workspace secrets。** 对于不需要基于 proxy 注入的 API keys（例如由 agent server 自身而非 sandbox 代码使用的 keys），请将它们作为 [workspace secrets](/langsmith/set-up-hierarchy#configure-workspace-settings) 存储在 LangSmith 中。这些会在运行时作为环境变量供 workspace 中的所有 agents 使用。

> **警告**：避免通过环境变量或文件上传将 secrets 传入 sandboxes。Agents 可以读取 sandbox 内任何可访问的文件或环境变量，包括凭据。Auth proxy 让 secrets 完全不进入 sandbox。

## Guardrails

生产环境中的 agents 自主运行，这意味着它们可能无限循环、触及速率限制，或处理含有敏感信息的用户数据。Deep Agents 提供两层保护：

* **[Permissions](#permissions)**：声明式的允许/拒绝规则，控制 agent 可以读取或写入哪些文件和目录。
* **[Fault tolerance](#fault-tolerance)**：速率限制、重试、回退和错误处理。
* **[Data privacy](#data-privacy)**：在 PII 到达模型或存入日志之前检测并处理它的 middleware。

### Permissions

[Permissions](/oss/javascript/deepagents/permissions) 是声明式的允许/拒绝规则，控制 agent 可以读取或写入哪些文件和目录。使用 permissions 将 agent 限制在某个工作目录内、保护敏感文件，或强制只读 memory。规则按声明顺序求值，第一个匹配的规则生效。

### Fault tolerance

关于速率限制、重试、回退和错误处理，请参阅 [Fault tolerance](/oss/javascript/deepagents/fault-tolerance)。

### 数据隐私

如果你的 agent 处理可能包含电子邮件、信用卡号或其它 PII 的用户输入，你可以在它到达模型或存入日志之前检测并处理它：

```typescript
import { createAgent, piiMiddleware } from "langchain";

const agent = createAgent({
  model: "google:gemini-3.6-flash",
  middleware: [
    piiMiddleware("email", { strategy: "redact", applyToInput: true }),
    piiMiddleware("credit_card", { strategy: "mask", applyToInput: true }),
  ],
});
```

策略包括 `redact`（替换为 `[REDACTED_EMAIL]`）、`mask`（部分掩码，如 `****-****-****-1234`）、`hash`（确定性哈希）和 `block`（抛出错误）。你还可以为特定领域的模式编写自定义检测器。

关于完整配置，请参阅 [piiMiddleware](https://reference.langchain.com/javascript/langchain/index/piiMiddleware)。

关于默认的 Deep Agents middleware 栈，请参阅 [Customization](/oss/javascript/deepagents/customization#middleware)。关于额外的 LangChain 预构建 middleware（重试、回退、PII 检测等），请参阅 [Prebuilt middleware](/oss/javascript/langchain/middleware/built-in)。

## 前端

Deep Agents 使用 [`useStream`](/oss/javascript/langchain/frontend/overview) 将你的 UI 连接到 agent backend。[`useStream`](https://reference.langchain.com/javascript/langchain-react/index/useStream) 是一个前端 hook（可用于 React、Vue、Svelte 和 Angular），能实时流式传输来自你 agent 的消息、subagent 进度和自定义状态。

在本地，`useStream` 指向 `http://localhost:2024`。在生产环境中，将它指向你的 [LangSmith Deployment](/langsmith/deployment) 并配置重连，这样即使用户连接中断也不会丢失进度。

```tsx
import { useStream } from "@langchain/react";

function App() {
  const stream = useStream<typeof agent>({
    apiUrl: "https://your-deployment.langsmith.dev",
    assistantId: "agent",
  });
}
```

对于会派生许多 subagents 的 deep agent 工作流，提交时设置较高的 `recursionLimit`，以避免截断长时间运行的执行：

```tsx
stream.submit(
  { messages: [{ type: "human", content: text }] },
  {
    streamSubgraphs: true,
    config: { recursionLimit: 10000 },
  },
);
```

关于 deep agents 特有的 UI 模式，例如 subagent 卡片、todo 列表和自定义状态渲染，请参阅 [frontend guide](/oss/javascript/deepagents/frontend/overview)。

***
