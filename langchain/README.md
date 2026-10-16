# LangChain (TypeScript) 官方文档中文整理

本目录是 [LangChain TypeScript 官方文档](https://docs.langchain.com/oss/javascript/langchain/overview) 的中文整理版。

- 正文、标题、列表、提示框均翻译为简体中文。
- **所有代码块原样保留**（代码、变量名、命令、包名、模型字符串均不翻译），可直接对照编写。
- 原文的提示框转换为引用块，并标注【提示】【注意】【警告】【信息】；多提供商/多标签代码拆分为带小标题的独立代码块。

> 建议阅读顺序：先读 `01`～`04` 建立整体认识，再按 `05`～`12`（核心组件）逐个学习，之后是 `13`～`16`（中间件/前端）、`17`～`23`（进阶用法）、`24`～`27`（开发与生产）。

## 目录

### 一、Get started（快速开始）

| 文件 | 内容 |
| --- | --- |
| [01-overview.md](./01-overview.md) | 概览：LangChain 是什么、核心概念与整体架构 |
| [02-install.md](./02-install.md) | 安装与依赖 |
| [03-quickstart.md](./03-quickstart.md) | 快速开始：从零构建第一个智能体（含真实研究型 agent） |
| [04-philosophy.md](./04-philosophy.md) | 设计理念 |

### 二、Core components（核心组件）

| 文件 | 内容 |
| --- | --- |
| [05-agents.md](./05-agents.md) | 智能体（Agents）：`createAgent` 与智能体循环 |
| [06-models.md](./06-models.md) | 模型（Models）：初始化、参数、多提供商 |
| [07-messages.md](./07-messages.md) | 消息（Messages）：消息类型与内容块 |
| [08-tools.md](./08-tools.md) | 工具（Tools）：定义、schema、运行时上下文 |
| [09-short-term-memory.md](./09-short-term-memory.md) | 短期记忆：会话状态与 checkpointer |
| [10-event-streaming.md](./10-event-streaming.md) | 事件流（Event streaming） |
| [11-streaming.md](./11-streaming.md) | 流式输出（Streaming） |
| [12-structured-output.md](./12-structured-output.md) | 结构化输出（Structured output） |

### 三、Middleware（中间件）

| 文件 | 内容 |
| --- | --- |
| [13-middleware-overview.md](./13-middleware-overview.md) | 中间件概览与钩子机制 |
| [14-middleware-built-in.md](./14-middleware-built-in.md) | 内置中间件（摘要、重试、回退、PII、限流等） |
| [15-middleware-custom.md](./15-middleware-custom.md) | 自定义中间件 |

### 四、Frontend（前端）

| 文件 | 内容 |
| --- | --- |
| [16-frontend-overview.md](./16-frontend-overview.md) | 前端集成概览 |

### 五、Advanced usage（进阶用法）

| 文件 | 内容 |
| --- | --- |
| [17-guardrails.md](./17-guardrails.md) | 护栏（Guardrails） |
| [18-runtime.md](./18-runtime.md) | 运行时上下文（Runtime） |
| [19-context-engineering.md](./19-context-engineering.md) | 上下文工程（Context engineering） |
| [20-mcp.md](./20-mcp.md) | Model Context Protocol（MCP） |
| [21-human-in-the-loop.md](./21-human-in-the-loop.md) | 人在环路（Human-in-the-loop） |
| [22-long-term-memory.md](./22-long-term-memory.md) | 长期记忆（Long-term memory） |
| [23-retrieval.md](./23-retrieval.md) | 检索（Retrieval / RAG） |

### 六、Agent development（智能体开发）

| 文件 | 内容 |
| --- | --- |
| [24-studio.md](./24-studio.md) | LangSmith Studio |
| [25-agent-chat-ui.md](./25-agent-chat-ui.md) | Agent Chat UI |

### 七、Production（生产环境）

| 文件 | 内容 |
| --- | --- |
| [26-deployment.md](./26-deployment.md) | 部署（Deployment） |
| [27-observability.md](./27-observability.md) | 可观测性（Observability） |

### 八、API 参考（`api/`）

> 与上面 `01`～`27` 的**概念教程**不同，`api/` 目录是对
> [LangChain JS 官方 API 参考](https://reference.langchain.com/javascript/langchain)
> 的**逐条中文整理**，覆盖 `langchain` 包导出的**全部类、函数、接口、类型**及其**属性和方法**。

| 文件 | 内容 |
| --- | --- |
| [api/00-overview.md](./api/00-overview.md) | API 参考总览：模块结构、文档索引、全部导出清单 |
| [api/01-messages.md](./api/01-messages.md) | 消息（Messages）：消息类、内容块、`filterMessages`/`trimMessages` |
| [api/02-tools.md](./api/02-tools.md) | 工具（Tools）：`Tool`/`StructuredTool`/`DynamicTool`/`HeadlessTool`、结构化输出策略 |
| [api/03-agents.md](./api/03-agents.md) | 智能体（Agents）：`createAgent`、`ReactAgent`、`createMiddleware`、生命周期钩子 |
| [api/04-middleware.md](./api/04-middleware.md) | 中间件（Middleware）：全部内置中间件工厂、PII 检测、配置类型 |
| [api/05-human-in-the-loop.md](./api/05-human-in-the-loop.md) | 人在环路（HITL）：`HITLRequest`/`HITLResponse`、`*Decision`、`Interrupt` |
| [api/06-models.md](./api/06-models.md) | 模型（Models）：`initChatModel`、模型工具函数、测试模型 |
| [api/07-storage-and-serialization.md](./api/07-storage-and-serialization.md) | 存储与序列化：`*Store`、`Serializable` 系列、`load` |
| [api/08-hub.md](./api/08-hub.md) | Hub：`pull`/`push`、`inferModelProviderFromNamespace` |
| [api/09-errors.md](./api/09-errors.md) | 错误（Errors）：全部导出的错误类 |
| [api/10-types-and-utilities.md](./api/10-types-and-utilities.md) | 类型与工具：`Infer*`/`Combine*`/`Resolve*` 等类型辅助、`ModelRequest` |

## 说明

- 原始文档会持续更新，本目录内容为整理时的快照，以官方文档为准。
- 每篇文件顶部都标注了对应的原文链接，便于对照查证。
