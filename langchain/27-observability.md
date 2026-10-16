# LangSmith 可观测性

> 原文：https://docs.langchain.com/oss/javascript/langchain/observability
> LangChain (TypeScript) 官方文档中文整理

在使用 LangChain 构建和运行 agent 时，你需要了解它们的行为方式：它们调用了哪些 [工具](/oss/javascript/langchain/tools)、生成了什么提示词，以及如何做出决策。使用 [`createAgent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 构建的 LangChain agent 自动支持通过 [LangSmith](/langsmith/observability) 进行追踪（tracing），LangSmith 是一个用于捕获、调试、评估和监控 LLM 应用行为的平台。

[*Trace（追踪）*](/langsmith/observability-concepts#traces) 会记录 agent 执行的每一步，从最初的用户输入到最终响应，包括所有工具调用、模型交互和决策点。这些执行数据可帮助你调试问题、评估不同输入下的性能，并监控生产环境中的使用模式。

本指南将向你展示如何为 LangChain agent 启用追踪，并使用 LangSmith 分析它们的执行过程。

## 前置条件

开始之前，请确保你拥有以下内容：

* **一个 LangSmith 账号**：在 [smith.langchain.com](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-langchain-observability) 免费注册或登录。
* **一个 LangSmith API key**：按照 [创建 API key](/langsmith/create-account-api-key) 指南操作。

## 启用追踪

所有 LangChain agent 都自动支持 LangSmith 追踪。要启用它，请设置以下环境变量：

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=<your-api-key>
```

## 快速上手

将 trace 记录到 LangSmith 无需额外代码。只需像平常一样运行你的 agent 代码：

```ts
import { createAgent } from "@langchain/agents";

function sendEmail(to: string, subject: string, body: string): string {
    // ... email sending logic
    return `Email sent to ${to}`;
}

function searchWeb(query: string): string {
    // ... web search logic
    return `Search results for: ${query}`;
}

const agent = createAgent({
    model: "gpt-5.5",
    tools: [sendEmail, searchWeb],
    systemPrompt: "You are a helpful assistant that can send emails and search the web."
});

// Run the agent - all steps will be traced automatically
const response = await agent.invoke({
    messages: [{ role: "user", content: "Search for the latest AI news and email a summary to john@example.com" }]
});
```

默认情况下，trace 会记录到名为 `default` 的项目中。要配置自定义项目名称，请参阅 [记录到某个项目](/langsmith/log-traces-to-project)。

## 选择性追踪

你可以使用 LangSmith 的 `tracing_context` 上下文管理器，选择性地追踪特定的调用或应用中的某些部分：

```ts
import { LangChainTracer } from "@langchain/core/tracers/tracer_langchain";

// This WILL be traced
const tracer = new LangChainTracer();
await agent.invoke(
  {
    messages: [{role: "user", content: "Send a test email to alice@example.com"}]
  },
  { callbacks: [tracer] }
);

// This will NOT be traced (if LANGSMITH_TRACING is not set)
await agent.invoke(
  {
    messages: [{role: "user", content: "Send another email"}]
  }
);
```

## 记录到某个项目

### 静态设置

你可以通过设置 `LANGSMITH_PROJECT` 环境变量，为整个应用设置自定义项目名称：

```bash
export LANGSMITH_PROJECT=my-agent-project
```

### 动态设置

你可以以编程方式为特定操作设置项目名称：

```ts
import { LangChainTracer } from "@langchain/core/tracers/tracer_langchain";

const tracer = new LangChainTracer({ projectName: "email-agent-test" });
await agent.invoke(
  {
    messages: [{role: "user", content: "Send a test email to alice@example.com"}]
  },
  { callbacks: [tracer] }
);
```

## 为 trace 添加元数据

你可以使用自定义元数据和标签来标注你的 trace：

```ts
import { LangChainTracer } from "@langchain/core/tracers/tracer_langchain";

const tracer = new LangChainTracer({ projectName: "email-agent-test" });
await agent.invoke(
  {
    messages: [{role: "user", content: "Send a test email to alice@example.com"}]
  },
  {
    tags: ["production", "email-assistant", "v1.0"],
    metadata: {
      userId: "user123",
      sessionId: "session456",
      environment: "production"
    }
  },
);

```

这些自定义元数据和标签会被附加到 LangSmith 中的该 trace 上。

> 【提示】要了解更多关于如何使用 trace 来调试、评估和监控 agent 的内容，请参阅 [LangSmith 文档](/langsmith/observability)。

***

[通过 MCP 将这些文档连接](/use-these-docs) 到 Claude、VSCode 等，以获得实时答案。

[在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/observability.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
