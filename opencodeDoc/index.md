# 简介
 
OpenCode 是一个开源 AI 编程代理。它提供终端界面、桌面应用或 Web 应用。
 
 
## CLI
 
<CodeTabs
  id="install"
  label="安装方法"
  tabs={[
    { label: "curl", code: "curl -fsSL https://opencode.ai/v2/install | bash" },
    { label: "homebrew", code: "brew install anomalyco/tap/opencode-v2" },
    { label: "npm", code: "npm install -g @opencode/cli" },
    { label: "bun", code: "bun install -g --trust @opencode/cli" },
    { label: "pnpm", code: "pnpm add -g --allow-build=@opencode/cli @opencode/cli" },
    { label: "yarn", code: "yarn global add @opencode/cli" },
    { label: "vite+", code: "vp install -g @opencode/cli" },
    { label: "aur", code: "paru -S opencode-beta" },
  ]}
/>
 
npm 软件包使用 postinstall 脚本为你的平台选择原生 `opencode` 二进制文件。Bun 和 pnpm 命令
显式允许所需的脚本；Vite+ 不需要额外标志。
 
Windows 软件包管理器不被支持。
 
 
下载适用于你平台的独立 CLI 二进制文件。
 
- **macOS:** [Apple silicon](https://opencode.ai/files/bin/2.0.6/opencode-darwin-arm64.zip) · [Intel](https://opencode.ai/files/bin/2.0.6/opencode-darwin-x64.zip) · [Intel (baseline)](https://opencode.ai/files/bin/2.0.6/opencode-darwin-x64-baseline.zip)
- **Windows:** [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-windows-arm64.zip) · [x64](https://opencode.ai/files/bin/2.0.6/opencode-windows-x64.zip) · [x64 (baseline)](https://opencode.ai/files/bin/2.0.6/opencode-windows-x64-baseline.zip)
- **Linux (glibc):** [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-linux-arm64.tar.gz) · [x64](https://opencode.ai/files/bin/2.0.6/opencode-linux-x64.tar.gz) · [x64 (baseline)](https://opencode.ai/files/bin/2.0.6/opencode-linux-x64-baseline.tar.gz)
- **Linux (musl):** [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-linux-arm64-musl.tar.gz) · [x64](https://opencode.ai/files/bin/2.0.6/opencode-linux-x64-musl.tar.gz) · [x64 (baseline)](https://opencode.ai/files/bin/2.0.6/opencode-linux-x64-baseline-musl.tar.gz)
 
## Desktop
 
下载适用于你平台的最新 OpenCode Desktop 版本。
 
- **macOS:** [Apple silicon](https://opencode.ai/files/bin/2.0.6/opencode-desktop-mac-arm64.dmg) · [Intel](https://opencode.ai/files/bin/2.0.6/opencode-desktop-mac-x64.dmg)
- **Windows:** [x64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-win-x64.exe) · [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-win-arm64.exe)
- **Linux (.deb):** [x64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-amd64.deb) · [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-arm64.deb)
- **Linux (.rpm):** [x64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-x86_64.rpm) · [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-aarch64.rpm)
- **Linux (AppImage):** [x64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-x86_64.AppImage) · [ARM64](https://opencode.ai/files/bin/2.0.6/opencode-desktop-linux-arm64.AppImage)
 
 
## Web
 
使用 `opencode pair` 命令访问 OpenCode Web 界面。
 
```bash
$ opencode pair
 
  URLs      http://127.0.0.1:49374
  Username  opencode
  Password  ********
```
 
## Docker
 
Docker 镜像使用版本标签，例如 `ghcr.io/anomalyco/opencode:2.0.0`。
 
---
 
## 连接
 
OpenCode 内置支持许多 LLM 提供商 - 你可以直接 [在 TUI 中](/cli/providers) 使用 `/connect` 连接到它们。
 
查看 [提供商](/providers) 以配置自定义提供商。
 
如果你想轻松访问所有最佳的代码模型，可以尝试
[OpenCode Console](/console)。
 
你也可以尝试 [OpenCode Go](/console/go)，这是一个每月 $10 的订阅
计划，可让你访问最佳的开源模型。
 
---
 
## 自定义
 
通过编辑 [OpenCode 配置](/config)、[加载插件](/plugins)、[连接 MCP
服务器](/mcp-servers) 或 [创建命令](/commands) 来定制 OpenCode。有关终端界面的主题和快捷键，请参阅 [CLI
设置](/cli/config)。
