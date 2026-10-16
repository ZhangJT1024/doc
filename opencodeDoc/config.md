# 配置
 
在你的项目中创建 `opencode.jsonc` 来配置 OpenCode。添加用于编辑器验证的 schema，然后仅设置你需要的选项。
 
```jsonc title="opencode.jsonc"
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
}
```
 
你也可以要求 OpenCode 为你更新此文件。
 
## 格式
 
OpenCode 支持 JSON 和 JSONC。当你需要注释或尾随逗号时，请使用 JSONC。
 
```jsonc title="opencode.jsonc"
{
  "$schema": "https://opencode.ai/config.json",
  // 默认使用此模型。
  "model": "anthropic/claude-sonnet-4-5",
}
```
 
## 位置
 
将每个项目的设置放在全局配置中：
 
```text
~/.config/opencode/opencode.json(c)
```
 
将项目设置放在以下两个文件之一：
 
```text
/home/user/projects/my-app/opencode.json(c)
/home/user/projects/my-app/.opencode/opencode.json(c)
```
 
OpenCode 从当前目录搜索到文件系统根目录。它首先
合并从最远目录到最近的直接 `opencode.json(c)` 文件，然后按相同顺序合并 `.opencode` 目录中的文件。
这意味着每个发现的 `.opencode` 配置都会覆盖每个直接的配置。
除非你需要这种行为，否则在一个目录树中请使用一种形式。
 
例如，从 `/home/user/projects/acme/packages/web` 启动 OpenCode：
 
```text
~/.config/opencode/opencode.json
 
/home/user/projects/acme/
├── opencode.json
└── packages/
    └── web/
        ├── opencode.json
        └── src/
```
 
OpenCode 按从低到高的优先级应用这些文件：
 
1. `~/.config/opencode/opencode.json`
2. `/home/user/projects/acme/opencode.json`
3. `/home/user/projects/acme/packages/web/opencode.json`
 
软件包配置覆盖来自仓库配置的匹配设置，
而仓库配置覆盖全局配置。不冲突的设置将被保留。
 
## Schema
 
完整的 OpenCode 配置 schema 可在
[opencode.ai/config.json](https://opencode.ai/config.json) 获取。
 
在你的配置文件中添加 `$schema` 字段，以在支持 JSON Schema 的编辑器中启用验证和
自动补全：
 
```json title="opencode.json"
{
  "$schema": "https://opencode.ai/config.json"
}
```
 
将 schema 作为可用字段、接受值
和嵌套配置形状的真相来源。
 
### Shell
 
设置终端和 shell 工具使用的 shell。
 
```jsonc
{
  "shell": "/bin/zsh",
}
```
 
### Model
 
设置 `provider/model` 格式的默认模型。当前的根默认值
不保留 `#variant`；智能体和命令模型引用可以选择其中一个。
 
```jsonc
{
  "model": "anthropic/claude-sonnet-4-5",
}
```
 
查看 [模型指南](/models) 以获取模型选择和本地模型信息。
 
### Agent
 
选择当会话未显式选择时使用的主要智能体。
 
```jsonc
{
  "default_agent": "build",
}
```
 
查看 [智能体指南](/agents) 以获取内置和自定义
智能体信息。
 
### Updates
 
从全局配置控制更新检查。将 `update` 设置为 `"disable"` 以
跳过更新，设置为 `"notify"` 在安装前显示可用更新，或
设置为 `"auto"` 自动安装更新。如果省略，`update` 默认为 `"notify"`。
 
自动安装不会重新启动正在运行的服务器。手动重启它以激活已安装的更新。
项目级的数值将被忽略。
 
```jsonc
{
  "update": "notify",
}
```
 
### Sharing
 
设置会话共享策略。OpenCode 接受此字段，但目前尚不支持会话共享。
 
```jsonc
{
  "share": "manual",
}
```
 
查看 [共享指南](/sharing) 获取更多详情。
 
### Username
 
设置用户名。OpenCode 接受此字段但不会在
对话中显示它。
 
```jsonc
{
  "username": "alice",
}
```
 
### Permissions
 
定义有序的规则，在智能体在匹配资源上使用工具之前允许、拒绝或询问。
 
```jsonc
{
  "permissions": [
    {
      "action": "shell",
      "resource": "git push *",
      "effect": "ask",
    },
  ],
}
```
 
查看 [权限指南](/permissions) 获取规则匹配和可用操作。
 
### Policies
 
通过有序语句允许或拒绝提供商的使用，或硬性拒绝权限检查，更宽泛的配置可以覆盖这些语句。
 
```jsonc
{
  "experimental": {
    "policies": [
      { "action": "provider.use", "resource": "*", "effect": "deny" },
      { "action": "provider.use", "resource": "anthropic", "effect": "allow" },
      { "action": "permission", "resource": "shell:git push *", "effect": "deny" },
    ],
  },
}
```
 
查看 [策略指南](/policies) 获取匹配、跨配置
文件的优先级以及 Console 管理的策略。
 
### Agents
 
覆盖内置智能体或定义具有自己模型、
指令、模式和权限的专门智能体。
 
```jsonc
{
  "agents": {
    "reviewer": {
      "description": "Review changes without editing files",
      "mode": "subagent",
      "system": "Focus on correctness, security, and missing tests.",
      "permissions": [{ "action": "edit", "resource": "*", "effect": "deny" }],
    },
  },
}
```
 
查看 [智能体指南](/agents) 获取所有智能体选项和基于文件的智能体。
 
### Snapshots
 
启用或禁用用于撤销（undo）和还原（revert）行为的文件系统快照。
 
```jsonc
{
  "snapshots": false,
}
```
 
查看 [快照指南](/snapshots) 获取撤销和重做行为。
 
### Watcher
 
忽略不应触发文件系统更新的文件和目录。
 
```jsonc
{
  "watcher": {
    "ignore": ["dist/**", "coverage/**"],
  },
}
```
 
### Formatter
 
在 `write`、`edit` 或 `patch` 工具更改文件后格式化文件。设置
`formatter` 为 `true` 以启用可用的内置格式化程序。
 
```jsonc
{
  "formatter": true,
}
```
 
查看 [格式化程序指南](/formatters) 获取内置和自定义格式化程序。
 
### Media
 
控制由 `read` 工具加载的超大图片在发送到模型之前如何调整大小或被拒绝。
 
```jsonc
{
  "media": {
    "image": {
      "auto_resize": true,
      "max_width": 2000,
      "max_height": 2000,
      "max_base64_bytes": 5242880,
    },
  },
}
```
 
查看 [附件指南](/attachments) 获取图像处理和限制。
 
### Output
 
设置从工具结果中保留的最大行数和字节数。
 
```jsonc
{
  "tool_output": {
    "max_lines": 2000,
    "max_bytes": 51200,
  },
}
```
 
### Search
 
选择 OpenCode 如何搜索 Web。使用 `"random"` 自动选择一个可用的
提供商。
 
```jsonc
{
  "websearch": {
    "provider": "random",
  },
}
```
 
查看 [Web 搜索指南](/websearch) 获取提供商、凭据、选择、
速率限制以及禁用搜索。
 
### MCP
 
配置本地和远程 Model Context Protocol 服务器。全局超时可以
由单个服务器覆盖。
 
```jsonc
{
  "mcp": {
    "servers": {
      "playwright": {
        "type": "local",
        "command": ["bunx", "@playwright/mcp"],
      },
    },
  },
}
```
 
查看 [MCP 指南](/mcp-servers) 获取远程服务器、OAuth、环境变量和超时。
 
### Compaction
 
控制自动上下文压缩以及它保留多少最近的上下文。
 
```jsonc
{
  "compaction": {
    "auto": true,
    "keep": {
      "tokens": 15000,
    },
    "buffer": 20000,
  },
}
```
 
本地摘要仍为默认值。对于自动和手动请求，使用提供商或模型策略加入原生提供商压缩：
 
```jsonc
{
  "providers": {
    "openai": {
      "settings": { "compaction": { "type": "native" } },
      "models": {
        "gpt-4.1": { "settings": { "compaction": { "type": "summary" } } },
      },
    },
  },
}
```
 
模型设置覆盖提供商设置。自动压缩使用所选模型的可用输入预算。提供商检查点保留在相同
`compaction.tokens` 预算内的最近用户消息，这是本地摘要用于保留尾部时使用的预算。
顶级 `compaction.auto: false` 在不丢弃已安装检查点的情况下禁用新的自动压缩。查看 [压缩指南](/compaction) 以获取
预算分配和溢出恢复。
 
### Warming
 
通过周期性的瞬态请求保持最近活跃的模型会话处于预热状态。
默认禁用预热；将其设置为 `true` 以使用四分钟的空闲
间隔和 30 分钟的活跃窗口。
 
```jsonc
{
  "warming": {
    "prompt": "Do not perform any work. Reply with exactly: OK",
    "interval": "4 minutes",
    "duration": "30 minutes",
  },
}
```
 
查看 [预热指南](/warming) 获取请求行为、自定义
以及成本考虑。
 
### Skills
 
添加 OpenCode 应搜索智能体技能的目录或 URL。
 
```jsonc
{
  "skills": ["./team-skills", "https://example.com/.well-known/skills/"],
}
```
 
查看 [技能指南](/skills) 获取技能结构和 `.opencode/skills/` 下的自动发现。
 
### Commands
 
将可重用的斜杠命令定义为命名的提示模板。
 
```jsonc
{
  "commands": {
    "review": {
      "description": "Review the current changes",
      "template": "Review the current diff for correctness and missing tests.",
    },
  },
}
```
 
查看 [命令指南](/commands) 获取参数、模型、智能体和基于文件的命令。
 
### Instructions
 
声明额外的指令文件、通配符或 URL。OpenCode 接受此
字段但不会加载其条目；请使用 `AGENTS.md` 获取指令。
 
```jsonc
{
  "instructions": ["CONTRIBUTING.md", "docs/guidelines/*.md"],
}
```
 
查看 [指令指南](/instructions) 获取项目指令和 `AGENTS.md`。
 
### References
 
将本地目录或 Git 仓库作为命名的支持
上下文提供。
 
```jsonc
{
  "references": {
    "docs": {
      "path": "../product-docs",
      "description": "Product behavior and terminology",
    },
    "effect": {
      "repository": "Effect-TS/effect",
      "branch": "main",
    },
  },
}
```
 
查看 [引用指南](/references) 获取简写、可见性和路径解析。
 
### Worktrees
 
设置新本地工作树的父目录。OpenCode 会附加请求或生成的工作树名称。
 
```jsonc
{
  "worktree": {
    "directory": "../worktrees",
  },
}
```
 
相对路径相对于项目保存的规范检出路径进行解析。这同样适用于全局和项目
配置；绝对路径按原样使用，且 `~/` 解析
为用户的主目录。
 
例如，此全局配置将新的工作树放在每个项目自己的 `.lane/trees/` 目录下：
 
```jsonc
{
  "worktree": {
    "directory": ".lane/trees",
  },
}
```
 
若无此设置，创建将使用服务器的数据目录 `worktree/<first-six-project-ID-characters>`。
工作树的创建和刷新会从项目保存的规范检出中加载配置，包括当其他
克隆共享其项目 ID 时的情况。更改它不会移动现有的工作树。
 
Git 是内置默认值。注册了策略的 [插件](/build/plugins#worktrees) 会自动成为
该插件运行时的默认值。项目操作使用规范检出的运行时。策略特定选项
属于该插件，而非 `worktree` 配置对象。
 
### Plugins
 
从软件包或本地插件目录加载插件。当插件
接受选项时，使用对象形式。
 
```jsonc
{
  "plugins": [
    "opencode-example-plugin",
    {
      "package": "./plugins/local",
      "options": {
        "enabled": true,
      },
    },
  ],
}
```
 
查看 [插件指南](/plugins) 获取插件加载和配置。
 
### Providers
 
配置提供商并添加或覆盖它们的模型、请求设置、
标头和模型变体。
 
```jsonc
{
  "providers": {
    "openai": {
      "models": {
        "gpt-5.2-custom": {
          "modelID": "gpt-5.2",
          "name": "GPT-5.2 Custom",
          "limit": {
            "context": 200000,
            "output": 32000,
          },
        },
      },
    },
  },
}
```
 
提供商 `settings.transport: "websocket"` 选择其会话 WebSocket。
 
查看 [提供商指南](/providers) 获取凭据、自定义端点、提供商软件包、WebSocket 传输和模型配置。
