# Agent Client Protocol (ACP)

> 原文：Agent Client Protocol (ACP)
> 来源：https://docs.langchain.com/oss/javascript/deepagents/acp

---

> 通过 Agent Client Protocol (ACP) 暴露 Deep Agents，以集成到代码编辑器和 IDE 中。

[Agent Client Protocol (ACP)](https://agentclientprotocol.com/get-started/introduction) 标准化了编码 agent 与代码编辑器或 IDE 之间的通信。借助 ACP 协议，你可以将自定义 deep agents 用于任何兼容 ACP 的客户端，让代码编辑器提供项目上下文并接收丰富的更新。

> **注意**：ACP 是为 agent-editor 集成而设计的。如果你想让 agent 调用由外部服务器托管的工具，请参阅 [Model Context Protocol (MCP)](/oss/javascript/langchain/mcp/)。

## 快速开始

安装 ACP 集成包：

#### npm

```bash
npm install deepagents-acp
```

#### yarn

```bash
yarn add deepagents-acp
```

#### pnpm

```bash
pnpm add deepagents-acp
```

然后通过 ACP 暴露一个 deep agent。

这会在 stdio 模式（从 stdin 读取请求并将响应写入 stdout）下启动一个 ACP server。在实践中，你通常将其作为由 ACP 客户端（例如你的编辑器）启动的命令来运行，编辑器随后通过 stdio 与服务器通信。

```ts
import { startServer } from "deepagents-acp";

await startServer({
  agents: {
    name: "coding-assistant",
    description: "AI coding assistant with filesystem access",
  },
  workspaceRoot: process.cwd(),
});
```

你也可以在不编写任何代码的情况下使用 CLI：

```bash
npx deepagents-acp
```

> **Deep Agents ACP on npm**（[链接](https://www.npmjs.com/package/deepagents-acp)）：`deepagents-acp` 包同时提供 CLI 和编程式 API，用于通过 ACP 暴露 deep agents。

## 客户端

Deep agents 可以在任何你能运行 ACP agent server 的地方工作。一些值得注意的 ACP 客户端包括：

* [Zed](https://zed.dev/docs/ai/external-agents)
* [JetBrains IDEs](https://www.jetbrains.com/help/ai-assistant/acp.html)
* Visual Studio Code（通过 [vscode-acp](https://github.com/formulahendry/vscode-acp)）
* Neovim（通过兼容 ACP 的插件）

### Zed

通过将你的 deep agent 添加到 Zed 设置（Linux 上为 `~/.config/zed/settings.json`，macOS 上为 `~/Library/Application Support/Zed/settings.json`）来向 [Zed](https://zed.dev/docs/ai/external-agents) 注册它：

**简单设置（无需代码）：**

```json
{
  "agent": {
    "profiles": {
      "deepagents": {
        "name": "DeepAgents",
        "command": "npx",
        "args": ["deepagents-acp"],
        "env": {
          "ANTHROPIC_API_KEY": "sk-ant-..."
        }
      }
    }
  }
}
```

**使用 CLI 选项：**

```json
{
  "agent": {
    "profiles": {
      "deepagents": {
        "name": "DeepAgents",
        "command": "npx",
        "args": [
          "deepagents-acp",
          "--name", "my-assistant",
          "--skills", "./skills",
          "--debug"
        ],
        "env": {
          "ANTHROPIC_API_KEY": "sk-ant-..."
        }
      }
    }
  }
}
```

**自定义 server 脚本：**

要获得更多控制，创建一个 TypeScript server 脚本：

```ts
// server.ts
import { startServer } from "deepagents-acp";

await startServer({
  agents: {
    name: "my-agent",
    description: "My custom coding agent",
    skills: ["./skills/"],
  },
});
```

然后将 Zed 指向它：

```json
{
  "agent": {
    "profiles": {
      "my-agent": {
        "name": "My Agent",
        "command": "npx",
        "args": ["tsx", "./server.ts"]
      }
    }
  }
}
```

打开 Zed 的 Agents 面板并启动一个 Deep Agents thread。

### ACP Registry

Deep Agents 已在 [ACP Agent Registry](https://agentclientprotocol.com/registry/index) 中提供，可在 Zed 和 JetBrains IDEs 中一键安装。当 ACP 客户端支持该 registry 时，用户无需任何手动配置即可发现并安装 Deep Agents。

## CLI 参考

CLI 是启动 ACP server 最快的方式。它不需要代码——只需运行 `npx deepagents-acp` 并连接你的编辑器。

```bash
npx deepagents-acp [options]
```

| 选项                   | 简写  | 描述                                                |
| ---------------------- | ----- | --------------------------------------------------- |
| `--name <name>`        | `-n`  | Agent 名称（默认：`"deepagents"`）                   |
| `--description <desc>` | `-d`  | Agent 描述                                          |
| `--model <model>`      | `-m`  | LLM 模型（默认：`"claude-sonnet-4-5-20250929"`）     |
| `--workspace <path>`   | `-w`  | 工作区根目录（默认：cwd）                            |
| `--skills <paths>`     | `-s`  | 逗号分隔的 skill 路径                                |
| `--memory <paths>`     |       | 逗号分隔的 AGENTS.md 路径                            |
| `--debug`              |       | 将 debug 日志输出到 stderr                           |
| `--help`               | `-h`  | 显示帮助信息                                        |
| `--version`            | `-v`  | 显示版本                                            |

### 环境变量

| 变量                | 描述                                           |
| ------------------- | ---------------------------------------------- |
| `ANTHROPIC_API_KEY` | Anthropic/Claude 模型的 API key（必需）          |
| `OPENAI_API_KEY`    | OpenAI 模型的 API key                          |
| `DEBUG`             | 设为 `"true"` 以启用 debug 日志                 |
| `WORKSPACE_ROOT`    | `--workspace` flag 的替代方式                   |

## 编程式 API

### `startServer`

用于在一次调用中创建并启动服务器的便捷函数：

```ts
import { startServer } from "deepagents-acp";

await startServer({
  agents: {
    name: "coding-assistant",
    description: "AI coding assistant with filesystem access",
  },
  workspaceRoot: process.cwd(),
});
```

### `DeepAgentsServer`

要获得完全控制，可直接使用 `DeepAgentsServer` 类：

#### Google

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "google:gemini-3.6-flash",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### OpenAI

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "openai:gpt-5.5",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### Anthropic

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "anthropic:claude-sonnet-5",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### OpenRouter

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "openrouter:z-ai/glm-5.2",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### Fireworks

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "fireworks:accounts/fireworks/models/glm-5p2",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### Baseten

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "baseten:zai-org/GLM-5.2",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### Ollama

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    {
      name: "code-agent",
      description: "Full-featured coding assistant",
      model: "ollama:north-mini-code-1.0",
      skills: ["./skills/"],
      memory: ["./.deepagents/AGENTS.md"],
    },
    {
      name: "reviewer",
      description: "Code review specialist",
      systemPrompt: "You are a code review expert...",
    },
  ],
  serverName: "my-deepagents-acp",
  serverVersion: "1.0.0",
  workspaceRoot: process.cwd(),
  debug: true,
});

await server.start();
```

#### Server 选项

| 选项            | 类型                                   | 默认值             | 描述                     |
| --------------- | -------------------------------------- | ------------------ | ------------------------ |
| `agents`        | `DeepAgentConfig \| DeepAgentConfig[]` | required           | Agent 配置（单个或多个）  |
| `serverName`    | `string`                               | `"deepagents-acp"` | 用于 ACP 的 server 名称   |
| `serverVersion` | `string`                               | `"0.0.1"`          | Server 版本              |
| `workspaceRoot` | `string`                               | `process.cwd()`    | 工作区根目录             |
| `debug`         | `boolean`                              | `false`            | 启用 debug 日志          |

#### Agent 配置

| 选项           | 类型                                           | 描述                                                                                                              |
| -------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `name`         | `string`                                       | 唯一的 agent 名称（必需）                                                                                          |
| `description`  | `string`                                       | Agent 描述                                                                                                        |
| `model`        | `string`                                       | LLM 模型（默认：`"claude-sonnet-4-5-20250929"`）                                                                   |
| `tools`        | `StructuredTool[]`                             | 自定义 LangChain tools                                                                                            |
| `systemPrompt` | `string`                                       | 自定义 system prompt                                                                                              |
| `middleware`   | `AgentMiddleware[]`                            | 追加到 [Deep Agents stack](/oss/javascript/deepagents/customization#deep-agents-stack) 的自定义 middleware         |
| `backend`      | `AnyBackendProtocol`                           | 文件系统 backend                                                                                                   |
| `skills`       | `string[]`                                     | Skill 源路径                                                                                                       |
| `memory`       | `string[]`                                     | Memory 源路径（AGENTS.md）                                                                                         |
| `interruptOn`  | `Record<string, boolean \| InterruptOnConfig>` | 需要用户审批的工具（HITL）                                                                                         |
| `commands`     | `Array<{ name, description, input? }>`         | 自定义 slash commands                                                                                             |

## 自定义

### 多个 agent

你可以从单个服务器暴露多个 agent。ACP 客户端在创建会话时选择使用哪个 agent：

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: [
    { name: "code-agent", description: "General coding" },
    { name: "reviewer", description: "Code reviews" },
  ],
});
```

> **注意**：某些 ACP 客户端（如 Zed）目前没有提供在 agent 之间进行选择的 UI。在这种情况下，可考虑运行各自只带一个 agent 的独立服务器实例。

### Slash commands

服务器会向 IDE 注册内置的 slash commands：`/plan`、`/agent`、`/ask`、`/clear` 和 `/status`。你也可以为每个 agent 定义自定义命令：

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: {
    name: "my-agent",
    commands: [
      { name: "test", description: "Run the project's test suite" },
      { name: "lint", description: "Run linter and fix issues" },
      {
        name: "deploy",
        description: "Deploy to staging",
        input: { hint: "environment (staging or production)" },
      },
    ],
  },
});
```

### Human-in-the-loop

使用 `interruptOn` 要求在 agent 运行敏感工具之前，在 IDE 中获得用户审批：

```ts
import { DeepAgentsServer } from "deepagents-acp";

const server = new DeepAgentsServer({
  agents: {
    name: "careful-agent",
    interruptOn: {
      execute: { allowedDecisions: ["approve", "edit", "reject"] },
      write_file: true,
    },
  },
});
```

当 agent 调用受保护的工具时，IDE 会提示用户允许或拒绝该操作，并提供为该会话记住该决策的选项。

### 自定义 tools

```ts
import { DeepAgentsServer } from "deepagents-acp";
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const searchTool = tool(
  async ({ query }) => {
    return `Results for: ${query}`;
  },
  {
    name: "search",
    description: "Search the codebase",
    schema: z.object({ query: z.string() }),
  },
);

const server = new DeepAgentsServer({
  agents: {
    name: "search-agent",
    tools: [searchTool],
  },
});


await server.start();
```

### 自定义 backend

```ts
import { DeepAgentsServer } from "deepagents-acp";
import { CompositeBackend, FilesystemBackend, StateBackend } from "deepagents";

const server = new DeepAgentsServer({
  agents: {
    name: "custom-agent",
    backend: new CompositeBackend(new StateBackend(), {
      "/workspace/": new FilesystemBackend({ rootDir: "./workspace" }),
    }),
  },
});
```

### Skills 和 memory

```ts
import { startServer } from "deepagents-acp";

await startServer({
  agents: {
    name: "project-agent",
    description: "Agent with project-specific knowledge",
    skills: ["./skills/", "~/.deepagents/skills/"],
    memory: ["./.deepagents/AGENTS.md"],
  },
  workspaceRoot: process.cwd(),
});
```

> **说明**：协议细节和编辑器支持请参阅上游的 ACP 文档：
>
> * 简介：[https://agentclientprotocol.com/get-started/introduction](https://agentclientprotocol.com/get-started/introduction)
> * 客户端/编辑器：[https://agentclientprotocol.com/get-started/clients](https://agentclientprotocol.com/get-started/clients)
