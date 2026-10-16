# 智能体聊天 UI（Agent Chat UI）

> 原文：https://docs.langchain.com/oss/javascript/langchain/ui
> LangChain (TypeScript) 官方文档中文整理

[Agent Chat UI](https://github.com/langchain-ai/agent-chat-ui) 是一个 Next.js 应用，为与任何 LangChain 智能体交互提供对话式界面。它支持实时聊天、工具可视化，以及诸如时间旅行调试（time-travel debugging）和状态分叉（state forking）之类的高级功能。Agent Chat UI 能够与使用 [`create_agent`](https://reference.langchain.com/javascript/langchain/index/createAgent) 创建的智能体无缝协作，并且只需极少的设置，就能为你的智能体提供交互式体验——无论你是在本地运行，还是在已部署的环境（例如 [LangSmith](https://docs.langchain.com/langsmith/observability)）中运行。

Agent Chat UI 是开源的，可以按你的应用需求进行调整。

[Agent Chat UI 视频](https://www.youtube.com/embed/lInrwVnZ83o?si=Uw66mPtCERJm0EjU)

> 【提示】你可以在 Agent Chat UI 中使用生成式 UI（generative UI）。更多信息，请参阅[使用 LangGraph 实现生成式用户界面](https://docs.langchain.com/langsmith/generative-ui-react)。

### 快速开始

最快的入门方式是使用托管版本：

1. **访问 [Agent Chat UI](https://agentchat.vercel.app)**
2. **连接你的智能体**：输入你的部署 URL 或本地服务器地址
3. **开始聊天** —— UI 会自动检测并渲染工具调用和中断（interrupts）

### 本地开发

如果需要进行自定义或本地开发，你可以在本地运行 Agent Chat UI：

#### 使用 npx

```bash
# Create a new Agent Chat UI project
npx create-agent-chat-app --project-name my-chat-ui
cd my-chat-ui

# Install dependencies and start
pnpm install
pnpm dev
```

#### 克隆仓库

```bash
# Clone the repository
git clone https://github.com/langchain-ai/agent-chat-ui.git
cd agent-chat-ui

# Install dependencies and start
pnpm install
pnpm dev
```

### 连接到你的智能体

Agent Chat UI 既可以连接到[本地智能体](https://docs.langchain.com/oss/javascript/langchain/studio)，也可以连接到[已部署的智能体](https://docs.langchain.com/oss/javascript/langchain/deploy)。

启动 Agent Chat UI 后，你需要配置它以连接到你的智能体：

1. **Graph ID**：输入你的图名称（可在 `langgraph.json` 文件的 `graphs` 下找到）
2. **Deployment URL**：你的 Agent 服务器的端点（例如本地开发使用 `http://localhost:2024`，或者你已部署智能体的 URL）
3. **LangSmith API key（可选）**：添加你的 LangSmith API key（如果使用的是本地 Agent 服务器，则不需要）

配置完成后，Agent Chat UI 会自动获取并显示来自你智能体的所有已中断线程（interrupted threads）。

> 【提示】Agent Chat UI 开箱即支持渲染工具调用和工具结果消息。要自定义显示哪些消息，请参阅[在聊天中隐藏消息（Hiding Messages in the Chat）](https://github.com/langchain-ai/agent-chat-ui?tab=readme-ov-file#hiding-messages-in-the-chat)。

***

> [通过 MCP 将这些文档连接到](https://docs.langchain.com/use-these-docs) Claude、VSCode 等，以获得实时答案。
>
> [在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/ui.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
