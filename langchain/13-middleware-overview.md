# 概览

> 原文：https://docs.langchain.com/oss/javascript/langchain/middleware/overview
> LangChain (TypeScript) 官方文档中文整理

> 控制并自定义智能体（agent）执行的每一步

中间件（Middleware）提供了一种更精细地控制智能体内部行为的方式。中间件适用于以下场景：

* 通过日志、分析和调试跟踪智能体行为。
* 转换提示词、[工具选择](/oss/javascript/langchain/middleware/built-in#llm-tool-selector)以及输出格式。
* 添加[重试](/oss/javascript/langchain/middleware/built-in#tool-retry)、[回退](/oss/javascript/langchain/middleware/built-in#model-fallback)和提前终止逻辑。
* 应用[速率限制](/oss/javascript/langchain/middleware/built-in#model-call-limit)、护栏（guardrails）以及[PII 检测](/oss/javascript/langchain/middleware/built-in#pii-detection)。

将中间件传给 `createAgent` 即可添加：

```typescript
import {
  createAgent,
  summarizationMiddleware,
  humanInTheLoopMiddleware,
} from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [...],
  middleware: [summarizationMiddleware, humanInTheLoopMiddleware],
});
```

## 智能体循环

核心的智能体循环包括：调用模型、让模型选择要执行的工具，然后在模型不再调用任何工具时结束：

![核心智能体循环示意图](https://mintcdn.com/langchain-5e9cc07a/Tazq8zGc0yYUYrDl/oss/images/core_agent_loop.png?fit=max&auto=format&n=Tazq8zGc0yYUYrDl&q=85&s=ac72e48317a9ced68fd1be64e89ec063)

中间件在上述每个步骤的前后都暴露了钩子（hook）：

![中间件流程图](https://mintcdn.com/langchain-5e9cc07a/RAP6mjwE5G00xYsA/oss/images/middleware_final.png?fit=max&auto=format&n=RAP6mjwE5G00xYsA&q=85&s=eb4404b137edec6f6f0c8ccb8323eaf1)

## 在 LangGraph 工作流中使用中间件

中间件并不是一个独立的运行时：钩子运行在 [`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 返回的、已编译的 [LangGraph](/oss/javascript/langgraph/overview) 内部。你可以把整个智能体（包括中间件在内的全部内容）作为节点或子图放入一个更大的 [StateGraph](https://reference.langchain.com/javascript/langchain-langgraph/index/StateGraph) 中，所有中间件钩子仍会继续运行。

当外围拓扑不只是标准的“循环直到完成”时，就可以采用这种模式：在路由到多个智能体之一之前先对输入进行分类、并行分发工作，或将智能体调用与确定性步骤拼接在一起。

`HumanInTheLoopMiddleware` 会根据每个工具的 `.name` 进行匹配。

匹配的键就是你传给 `tool({...}, { name })` 的 `name`。

```typescript
import { AgentState, createAgent, humanInTheLoopMiddleware } from "langchain";
import { StateGraph, START } from "@langchain/langgraph";

// 假设 readEmail、sendEmail、classifyNode 和 route 已在别处定义。
// readEmail / sendEmail 注册时使用的 name 分别为 "read_email" / "send_email"。
const emailAgent = createAgent({
  model: "claude-sonnet-4-6",
  tools: [readEmail, sendEmail],
  middleware: [humanInTheLoopMiddleware({ interruptOn: { send_email: true } })],
});

const graph = new StateGraph(AgentState)
  .addNode("classify", classifyNode)
  .addNode("emailAgent", emailAgent)
  .addEdge(START, "classify")
  .addConditionalEdges("classify", route)
  .compile();
```

HITL 中断、摘要、PII 脱敏、重试以及任何自定义钩子都会随智能体节点一起迁移。完整的组合模式（包括子图检查点作用域——按调用还是按线程）请参见[使用子图](/oss/javascript/langgraph/use-subgraphs)。

## 其他资源

* **内置中间件** —— 探索用于常见用例的内置中间件：`/oss/javascript/langchain/middleware/built-in`
* **自定义中间件** —— 使用钩子和装饰器构建你自己的中间件：`/oss/javascript/langchain/middleware/custom`
* **中间件 API 参考** —— 中间件的完整 API 参考：https://reference.langchain.com/python/langchain/middleware/
* **中间件集成** —— 面向 Anthropic、AWS、OpenAI 等提供方的专用中间件：`/oss/javascript/integrations/middleware/`
* **测试智能体** —— 使用 LangSmith 测试你的智能体：`/oss/javascript/langchain/test/`

---

> 【信息】[通过 MCP 将这些文档连接到](/use-these-docs) Claude、VSCode 等，以获得实时答案。

> 【信息】[在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/middleware/overview.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
