# 与 Claude Agent SDK 的对比

> 原文：Comparison with Claude Agent SDK
> 来源：https://docs.langchain.com/oss/javascript/deepagents/comparison

---

本页说明 [LangChain Deep Agents](/oss/javascript/deepagents/overview) 与 [Claude Agent SDK](https://platform.anthropic.com/docs/en/agent-sdk/overview) 的对比。二者都是用于构建自定义 agents 的 harness，但它们在执行环境、部署和供应商耦合方面做出了不同的取舍。

> **说明**：Deep Agents 已在生产环境中被 [OpenSWE](https://github.com/langchain-ai/open-swe) 和 [LangSmith Fleet](/langsmith/fleet/index) 使用。

## 一览

|                               | **Deep Agents**                                                                                                                                                                                              | **Claude Agent SDK**                                                                                                                                                                                                        |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **agent 在哪里运行**      | 在 sandbox 内部运行，或让 sandbox 在外部远程执行命令                                                                                                                                           | 在 sandbox 内部                                                                                                                                                                                                            |
| **执行 backend**         | 可插拔：[本地、虚拟文件系统、远程 sandbox 或自定义](/oss/javascript/deepagents/backends)                                                                                                       | 它所运行 sandbox 的本地文件系统                                                                                                                                                                                  |
| **模型提供商**            | 任意（Anthropic、OpenAI、Google 及 100+ 其他）                                                                                                                                                                 | Claude（Anthropic、Bedrock、Vertex、Azure）                                                                                                                                                                                  |
| **按提供商/模型调优** | [Harness profiles](/oss/javascript/deepagents/profiles)（beta）：以声明式打包 system prompt、工具、middleware 和 subagent 调整，按提供商或特定模型注册                       | 在每个模型调用点用代码配置                                                                                                                                                                                   |
| **部署**                | 在 LangSmith 中使用 [Managed Deep Agents](/langsmith/javascript/managed-deep-agents-overview)，或通过 [`langgraph build`](/langsmith/cli#build) 自托管一个[独立镜像](/langsmith/deploy-standalone-server) | [自托管](https://code.claude.com/docs/en/agent-sdk/hosting)。你需要自行构建 server、认证和流式传输层。[Claude managed agents](https://platform.claude.com/docs/en/managed-agents/overview) 是独立产品 |
| **多租户**             | [内置](/oss/javascript/deepagents/going-to-production#multi-tenancy)：作用域化 threads、每用户 sandboxes、RBAC                                                                                           | 自行构建                                                                                                                                                                                                           |
| **许可证**                   | MIT                                                                                                                                                                                                          | MIT（Claude Code 本身为专有）                                                                                                                                                                                     |

## 主要区别

### Agent 与执行环境

关于[将 agents 连接到 sandboxes 的两种模式](https://www.langchain.com/blog/the-two-patterns-by-which-agents-connect-sandboxes)：在 sandbox *内部*运行 agent，还是在外部运行 agent 并**把 sandbox 当作工具使用**。

Claude Agent SDK 只支持第一种。你的 agent 在 sandbox 内运行，并针对 sandbox 的本地文件系统执行工具。Anthropic 的托管模型 [Claude managed agents](https://platform.claude.com/docs/en/managed-agents/overview) 使用解耦模型，这反映了生产 agent 架构的发展方向。

Deep Agents 两者都支持，并让你选择一个 [backend](/oss/javascript/deepagents/backends#quickstart) 来将它们连接起来。在实践中，这意味着你可以：

* 在 sandbox 内运行 agent（与 Claude Agent SDK 相同的模型）。
* 在长生命周期的容器中运行 agent，并[把远程 sandbox 当作工具使用](https://www.langchain.com/blog/the-two-patterns-by-which-agents-connect-sandboxes)，通过网络执行命令。
* 为测试替换为虚拟文件系统，或为你自己的基础设施替换为自定义 backend。

### 多租户

当你把应用程序投入生产时，通常需要将其暴露给许多最终用户，并且必须为每个用户隔离环境。

在 Claude Agent SDK 中，SDK 将 agent 与其 sandbox 绑定。要为每个用户提供隔离的执行环境，你必须构建一个 API 封装层，为每个用户启动一个 sandbox，跟踪哪个 sandbox 属于谁，并在之后将其拆除。

Deep Agents 直接处理这一点：在 harness 中为[每个用户或每个 assistant](/oss/javascript/deepagents/going-to-production#lifecycle)配置一个 sandbox，并内置作用域化的 threads、运行历史和 [RBAC](/oss/javascript/deepagents/going-to-production#team-access-control-rbac)。如果你使用 [LangSmith Sandbox](/langsmith/sandbox-auth-proxy)，你还开箱即得一个 auth proxy，使最终用户可以从 sandbox 调用第三方 API，而无需你为每个用户配置凭据。

### 生产级 agent server

要将[自托管的 Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/hosting) 应用暴露给最终用户，你需要编写自己的 HTTP/WebSocket 或 SSE server，用于调用 agent、将 token 流式返回，并管理对话 threads。那个 server 需要你自己构建、运维和保护。

Deep Agents 部署开箱即包含一个 [agent server](/langsmith/agent-server)：流式端点、thread 管理、运行历史、webhooks 和[认证](/langsmith/auth)。

### 托管云或自托管

Claude Agent SDK 部署是[自托管](https://code.claude.com/docs/en/agent-sdk/hosting)的。SDK 和 [Claude managed agents](https://platform.claude.com/docs/en/managed-agents/overview) 是分开的产品。针对 SDK 编写的代码不能直接部署到托管方案。

Deep Agents 无需更改代码即可在两种模式下运行：

* **托管**：使用 LangSmith 中的 [Managed Deep Agents](/langsmith/javascript/managed-deep-agents-overview) 创建、运行和运维 deep agents。
* **自托管**：运行 [`langgraph build`](/langsmith/cli#build) 生成一个可部署到任何地方的[独立 Docker image](/langsmith/deploy-standalone-server)。

> **提示**：若要一个可跨任意模型提供商工作的托管 agent 平台，请使用 [LangSmith Fleet](/langsmith/fleet/index)。[Claude managed agents](https://platform.claude.com/docs/en/managed-agents/overview) 仅限于 Anthropic 生态。

### LLM

Claude Agent SDK 的执行将模型、backend 和部署捆绑在一起，并在三者之间优化支持。

使用 Deep Agents，你可以独立选择模型提供商、执行 backend 和部署目标。选择这个 harness，你就能在模型和基础设施的选择上保留最大的灵活性。

### 生态

Claude Agent SDK 是为 Claude 和 Anthropic 的产品面专门构建的。Deep Agents 与更广泛的 LangChain 生态集成，包括用于可观测性、评估和部署的 LangSmith，并可跨任意模型提供商工作。

## 总结

* **选择 Deep Agents**，如果你想要模型和基础设施的灵活性、内置的多租户部署，以及无需更改代码即可托管或自托管的选项。
* **选择 Claude Agent SDK**，如果你已经投入 Anthropic 生态，并希望自托管并自行构建 API、认证和多租户层。

> **注意**：**发现错误？**
>
> 我们在 2026 年 4 月 16 日起草了这份对比。如果产品发生了变化，请[提交 issue](https://github.com/langchain-ai/docs/issues)。
