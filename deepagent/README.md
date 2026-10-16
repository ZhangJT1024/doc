# Deep Agents（TypeScript）中文文档

本目录是对 [Deep Agents 官方文档（TypeScript 版）](https://docs.langchain.com/oss/javascript/deepagents/overview) 的**全章节中文整理**。

- 面向人群：使用 **TypeScript / JavaScript** 的开发者。
- 翻译原则：**正文用中文**；**代码、专有名词、API/类/函数/参数/字段名、命令、路径、URL 一律保留英文**。
- 章节结构与官网左侧导航**保持一致**（Get started / Deployment / Execution environment / Context management / Delegation / Steering / Frontend / Protocols）。
- 每个文件都带有原文标题与官方来源链接，方便与官网对照。
- `_source/` 目录保存了官方英文原文（`.md`），仅作校对参考，可自行删除。

---

## 目录（按官网章节组织）

### 一、Get started（入门）

| 文件 | 标题 |
| --- | --- |
| [01-overview.md](01-overview.md) | Deep Agents 概览 |
| [02-quickstart.md](02-quickstart.md) | 快速开始 |
| [03-customization.md](03-customization.md) | 自定义（Customization） |
| [04-models.md](04-models.md) | 模型（Models） |
| [05-comparison.md](05-comparison.md) | 与 Claude Agent SDK 的对比 |

### 二、Deployment（部署）

| 文件 | 标题 |
| --- | --- |
| [06-fault-tolerance.md](06-fault-tolerance.md) | 容错（Fault tolerance） |
| [07-going-to-production.md](07-going-to-production.md) | 投入生产（Going to production） |

### 三、Execution environment（执行环境）

| 文件 | 标题 |
| --- | --- |
| [08-tools.md](08-tools.md) | 工具（Tools）与 MCP |
| [09-backends.md](09-backends.md) | 后端（Backends）与虚拟文件系统 |
| [10-permissions.md](10-permissions.md) | 权限（Permissions） |
| [11-multimodal.md](11-multimodal.md) | 多模态输入输出（Multimodal） |
| [12-sandboxes.md](12-sandboxes.md) | 沙箱（Sandboxes） |
| [13-interpreters.md](13-interpreters.md) | 解释器（Interpreters，Beta） |
| [14-event-streaming.md](14-event-streaming.md) | 事件流（Event streaming，Beta） |
| [15-streaming.md](15-streaming.md) | 流式传输（Streaming） |

### 四、Context management（上下文管理）

| 文件 | 标题 |
| --- | --- |
| [16-skills.md](16-skills.md) | 技能（Skills） |
| [17-memory.md](17-memory.md) | 记忆（Memory） |
| [18-retrieval.md](18-retrieval.md) | 检索（Retrieval / RAG） |
| [19-context-engineering.md](19-context-engineering.md) | 上下文工程（Context engineering） |
| [20-profiles.md](20-profiles.md) | 配置档案（Profiles，Beta） |

### 五、Delegation（任务委派）

| 文件 | 标题 |
| --- | --- |
| [21-subagents.md](21-subagents.md) | 子代理（Subagents） |
| [22-dynamic-subagents.md](22-dynamic-subagents.md) | 动态子代理（Dynamic subagents，Beta） |
| [23-async-subagents.md](23-async-subagents.md) | 异步子代理（Async subagents） |

### 六、Steering（人工引导）

| 文件 | 标题 |
| --- | --- |
| [24-human-in-the-loop.md](24-human-in-the-loop.md) | Human-in-the-loop（人工介入） |

### 七、Frontend（前端）

| 文件 | 标题 |
| --- | --- |
| [25-frontend-overview.md](25-frontend-overview.md) | 前端总览（Frontend overview） |

### 八、Protocols（协议）

| 文件 | 标题 |
| --- | --- |
| [26-acp.md](26-acp.md) | Agent Client Protocol (ACP) |
| [27-ag-ui.md](27-ag-ui.md) | Agent User Interaction Protocol (AG-UI) |

> 说明：官网上的 **Middleware（中间件）** 章节属于 LangChain 文档的一部分，不在本套 Deep Agents 文档范围内。如需，可参阅 `D:\code\langchain` 中的 `13-middleware-overview.md`、`14-middleware-built-in.md`、`15-middleware-custom.md`。

---

## 建议学习路径（TypeScript 使用者）

1. **先跑通最小示例**
   阅读 `01-overview.md` 与 `02-quickstart.md`，理解 Deep Agents 是什么、`createDeepAgent` 的最小用法。
2. **掌握核心配置**
   阅读 `03-customization.md` 与 `04-models.md`，搞清 `model`、`tools`、`systemPrompt`、`middleware`、`subagents`、`backend` 等参数的写法。
3. **执行环境（agent 能做什么）**
   依次阅读 `08-tools.md` → `09-backends.md` → `10-permissions.md` → `12-sandboxes.md` / `13-interpreters.md` → `15-streaming.md`，理解工具、虚拟文件系统、权限、代码执行与流式输出。
4. **上下文管理（agent 记得什么）**
   依次阅读 `19-context-engineering.md` → `16-skills.md` → `17-memory.md` → `18-retrieval.md` → `20-profiles.md`，理解技能、记忆、摘要/上下文卸载、检索。
5. **任务委派（并行与拆分）**
   阅读 `21-subagents.md` → `22-dynamic-subagents.md` → `23-async-subagents.md`，用子代理隔离上下文并并行化任务。
6. **人工引导与生产化**
   阅读 `24-human-in-the-loop.md` → `06-fault-tolerance.md` → `07-going-to-production.md`，为上线做好准备。
7. **前端与协议（可选）**
   阅读 `25-frontend-overview.md`、`26-acp.md`、`27-ag-ui.md`，把 agent 接入你的 UI / 编辑器。

---

## 最小可用示例

```ts
import * as z from "zod";
// npm install deepagents langchain @langchain/core
import { createDeepAgent } from "deepagents";
import { tool } from "langchain";

const getWeather = tool(({ city }) => `It's always sunny in ${city}!`, {
  name: "get_weather",
  description: "Get the weather for a given city",
  schema: z.object({
    city: z.string(),
  }),
});

const agent = await createDeepAgent({
  tools: [getWeather],
  systemPrompt: "You are a helpful assistant",
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in Tokyo?" }],
  }),
);
```

---

## 术语约定

- **harness**：agent 运行框架 / 载体（保留 harness）。
- **subagent**：子代理。
- **backend**：后端（保留 backend）。
- **skill / memory**：技能 / 记忆（保留 skill、memory）。
- **middleware**：中间件。
- **human-in-the-loop**：人工介入（保留英文）。
- **context offloading**：上下文卸载。
- **prompt caching**：提示词缓存。

> 所有 API、字段、枚举值、命令与路径均保留英文原文，请以官网 TypeScript 文档为准。
