# 部署

> 原文：https://docs.langchain.com/oss/javascript/langchain/deploy
> LangChain (TypeScript) 官方文档中文整理

当你准备好将 LangChain agent 部署到生产环境时，请选择适合你技术栈的托管模型。**[LangSmith Cloud](/langsmith/deploy-to-cloud)** 为有状态、长时间运行的 agent 提供全托管基础设施，支持持久化状态和后台执行。

你也可以在 **JavaScript 框架与平台** 上部署，例如 Next.js、SvelteKit、Nuxt、Cloudflare Workers 和 Deno Deploy，使用相同的 [Agent Streaming Protocol](https://github.com/langchain-ai/agent-protocol/tree/main/streaming)。

**框架与平台**

- **LangSmith** — [查看指南](/langsmith/deploy-vite-langsmith)
- **Next.js** — [查看指南](/langsmith/deploy-nextjs)
- **SvelteKit** — [查看指南](/langsmith/deploy-sveltekit)
- **Nuxt** — [查看指南](/langsmith/deploy-nuxt)
- **Cloudflare** — [查看指南](/langsmith/deploy-cloudflare-workers)
- **Deno** — [查看指南](/langsmith/deploy-deno)

> 【提示】LangSmith 除了 Cloud 之外还提供多种部署选项，包括 [hybrid](/langsmith/hybrid)、[standalone servers](/langsmith/deploy-standalone-server) 以及 [self-hosted with control plane](/langsmith/deploy-with-control-plane)。更多信息请参阅 [LangSmith 部署概览](/langsmith/deployment)。

## LangSmith Cloud

本节将带你从 GitHub 仓库把 agent 部署到 LangSmith Cloud。LangSmith 负责基础设施、扩缩容以及运维相关事项。

### 前置条件

开始之前，请确保你拥有以下内容：

* 一个 [GitHub 账号](https://github.com/)
* 一个 [LangSmith 账号](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=oss-langchain-deploy)（可免费注册）

### 部署你的 agent

#### 1. 在 GitHub 上创建仓库

要在 LangSmith 上部署，你的应用代码必须位于一个 GitHub 仓库中。公共仓库和私有仓库均受支持。对于本快速上手，请先按照 [本地服务器设置指南](/oss/javascript/langchain/studio) 确保你的应用与 LangGraph 兼容。然后，将代码推送到该仓库。

#### 2. 部署到 LangSmith

**步骤 1：前往 LangSmith Deployment**

登录 [LangSmith](https://smith.langchain.com?utm_source=docs&utm_medium=cta&utm_campaign=langsmith-signup&utm_content=snippets-oss-deploy-js)。在左侧边栏中，选择 **Deployments**。

**步骤 2：创建新部署**

点击 **+ New Deployment** 按钮。会打开一个面板，你可以在其中填写必填字段。

**步骤 3：关联仓库**

如果你是首次使用，或要添加一个之前未连接过的私有仓库，请点击 **Add new account** 按钮，并按照说明连接你的 GitHub 账号。

**步骤 4：部署仓库**

选择你的应用仓库。点击 **Submit** 进行部署。这大约需要 15 分钟才能完成。你可以在 **Deployment details** 视图中查看状态。

#### 3. 在 Studio 中测试你的应用

应用部署完成后：

1. 选择你刚刚创建的部署，以查看更多详情。
2. 点击右上角的 **Studio** 按钮。Studio 将打开并显示你的 graph。

#### 4. 获取部署的 API URL

1. 在 LangGraph 的 **Deployment details** 视图中，点击 **API URL** 将其复制到剪贴板。
2. 点击 `URL` 将其复制到剪贴板。

#### 5. 测试 API

现在你可以测试该 API：

**JavaScript**

1. 安装 LangGraph JS：

```shell
npm install @langchain/langgraph-sdk
```

2. 向 agent 发送一条消息：

```ts
const { Client } = await import("@langchain/langgraph-sdk");

const client = new Client({ apiUrl: "your-deployment-url", apiKey: "your-langsmith-api-key" });

const streamResponse = client.runs.stream(
    null,    // Threadless run
    "agent", // Name of agent. Defined in langgraph.json.
    {
        input: {
            "messages": [
                { "role": "user", "content": "What is LangGraph?"}
            ]
        },
        streamMode: "messages",
    }
);

for await (const chunk of streamResponse) {
    console.log(`Receiving new event of type: ${chunk.event}...`);
    console.log(JSON.stringify(chunk.data));
    console.log("\n\n");
}
```

**Rest API**

```bash
curl -s --request POST \
    --url <DEPLOYMENT_URL>/runs/stream \
    --header 'Content-Type: application/json' \
    --header "X-Api-Key: <LANGSMITH API KEY> \
    --data "{
        \"assistant_id\": \"agent\", `# Name of agent. Defined in langgraph.json.`
        \"input\": {
            \"messages\": [
                {
                    \"role\": \"human\",
                    \"content\": \"What is LangGraph?\"
                }
            ]
        },
        \"stream_mode\": \"updates\"
    }"
```

> 【提示】LangSmith 还提供其他托管选项，包括自托管和混合部署。更多信息请参阅 [平台设置概览](/langsmith/platform-setup)。

***

[通过 MCP 将这些文档连接](/use-these-docs) 到 Claude、VSCode 等，以获得实时答案。

[在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/deploy.mdx) 或 [提交 issue](https://github.com/langchain-ai/docs/issues/new/choose)。
