# LangSmith Studio

> 原文：https://docs.langchain.com/oss/javascript/langchain/studio
> LangChain (TypeScript) 官方文档中文整理

在本地使用 LangChain 构建智能体时，可视化智能体内部发生的情况、与它实时交互、并在问题出现时进行调试会很有帮助。**LangSmith Studio** 是一个免费的视觉界面，用于在你的本地机器上开发和测试你的 LangChain 智能体。

Studio 连接到你在本地运行的智能体，向你展示智能体所采取的每一步：发送给模型的提示（prompts）、工具调用及其结果，以及最终输出。你可以测试不同的输入、检查中间状态，并在无需额外代码或部署的情况下迭代智能体的行为。

本页介绍如何为你的本地 LangChain 智能体设置 Studio。

## 前置条件

在开始之前，请确保你具备以下条件：

* **一个 LangSmith 账户**：在 [smith.langchain.com](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=snippets-oss-studio-js) 免费注册或登录。
* **一个 LangSmith API key**：遵循 [创建 API key](https://docs.langchain.com/langsmith/create-account-api-key) 指南。
* 如果你不希望数据被[追踪（traced）](https://docs.langchain.com/langsmith/observability-concepts#traces)到 LangSmith，请在应用的 `.env` 文件中设置 `LANGSMITH_TRACING=false`。禁用追踪后，不会有任何数据离开你的本地服务器。

## 设置本地 Agent 服务器

### 1. 安装 LangGraph CLI

[LangGraph CLI](https://docs.langchain.com/langsmith/cli) 提供一个本地开发服务器（也称为 [Agent Server](https://docs.langchain.com/langsmith/agent-server)），它将你的智能体连接到 Studio。

```shell
npx @langchain/langgraph-cli
```

### 2. 准备你的智能体

如果你已经有一个 LangChain 智能体，可以直接使用它。本示例使用一个简单的邮件智能体：

**文件：agent.ts**
```typescript
import { createAgent } from "@langchain/agents";

function sendEmail(to: string, subject: string, body: string): string {
  const email = {
    to,
    subject,
    body,
  };
  // ... email sending logic

  return `Email sent to ${to}`;
}

const agent = createAgent({
  model: "gpt-5.5",
  tools: [sendEmail],
  systemPrompt: "You are an email assistant. Always use the send_email tool.",
});
```

### 3. 环境变量

Studio 需要一个 LangSmith API key 来连接你的本地智能体。在你的项目根目录创建一个 `.env` 文件，并添加来自 [LangSmith](https://smith.langchain.com/settings) 的 API key。

> 【警告】确保你的 `.env` 文件没有被提交到版本控制系统（例如 Git）中。

#### .env
```bash
LANGSMITH_API_KEY=lsv2...
```

### 4. 创建 LangGraph 配置文件

LangGraph CLI 使用一个配置文件来定位你的智能体并管理依赖。在你的应用目录中创建一个 `langgraph.json` 文件：

**文件：langgraph.json**
```json
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent.ts:agent"
  },
  "env": ".env"
}
```

[`createAgent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 函数会自动返回一个已编译的 LangGraph 图（graph），这正是配置文件中 `graphs` 键所期望的内容。

> 【信息】关于配置文件的 JSON 对象中每个键的详细说明，请参阅 [LangGraph 配置文件参考](https://docs.langchain.com/langsmith/cli#configuration-file)。

此时，项目结构看起来会是这样：

```bash
my-app/
├── src
│   └── agent.ts
├── .env
└── langgraph.json
```

### 5. 安装依赖

```shell
yarn install
```

### 6. 在 Studio 中查看你的智能体

启动开发服务器，将你的智能体连接到 Studio：

```shell
npx @langchain/langgraph-cli dev
```

> 【警告】Safari 会阻止到 Studio 的 `localhost` 连接。要绕过此问题，请使用 `--tunnel` 运行上述命令，通过安全隧道访问 Studio。

服务器运行后，你的智能体既可以通过 `http://127.0.0.1:2024` 的 API 访问，也可以通过 `https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024` 的 Studio UI 访问：

![Agent view in the Studio UI](https://mintcdn.com/langchain-5e9cc07a/TCDks4pdsHdxWmuJ/oss/images/studio_create-agent.png?fit=max&auto=format&n=TCDks4pdsHdxWmuJ&q=85&s=ebd259e9fa24af7d011dfcc568f74be2)

Studio 连接到你的本地智能体后，你可以快速迭代智能体的行为。运行一个测试输入，检查完整的执行追踪，包括提示、工具参数、返回值以及 token/延迟指标。当出现问题时，Studio 会连同周围的状态一起捕获异常，帮助你理解发生了什么。

开发服务器支持热重载——在代码中修改提示或工具签名，Studio 会立即反映这些更改。你可以从任意步骤重新运行对话线程（threads），无需从头开始即可测试你的更改。这种工作流可从小型的单工具智能体扩展到复杂的多节点图。

有关如何运行 Studio 的更多信息，请参阅 [LangSmith 文档](https://docs.langchain.com/langsmith/observability)中的以下指南：

* [运行应用](https://docs.langchain.com/langsmith/use-studio#run-application)
* [管理助手（assistants）](https://docs.langchain.com/langsmith/use-studio#manage-assistants)
* [管理线程（threads）](https://docs.langchain.com/langsmith/use-studio#manage-threads)
* [迭代提示](https://docs.langchain.com/langsmith/observability-studio)
* [调试 LangSmith 追踪](https://docs.langchain.com/langsmith/observability-studio#debug-langsmith-traces)
* [将节点添加到数据集](https://docs.langchain.com/langsmith/observability-studio#add-node-to-dataset)

## 视频指南

[Studio 视频](https://www.youtube.com/embed/Mi1gSlHwZLM?si=zA47TNuTC5aH0ahd)

> 【提示】关于已部署智能体的更多信息，请参阅[部署](https://docs.langchain.com/oss/javascript/langchain/deploy)。

***

> [通过 MCP 将这些文档连接到](https://docs.langchain.com/use-these-docs) Claude、VSCode 等，以获得实时答案。
>
> [在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/studio.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
