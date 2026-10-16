# Web
 
OpenCode 自带一个 Web UI，由与 TUI 相同的服务器提供服务。它默认可用并受密码保护。
 
## 访问
 
```bash
$ opencode pair
 
  打开一个连接链接。链接仅可使用一次，并将在 5 分钟后过期。
 
  http://127.0.0.1:49374/auth/connect/...
 
  █▀▀▀▀▀█ ...
```
 
在浏览器中打开链接将使用会话 cookie 进行登录并加载 Web UI。扫描 OpenCode 应用中的 QR 码，或将链接粘贴到其服务器地址字段中，也可以以相同方式连接应用。会话有效期为 30 天；更改服务器密码将退出所有会话。
 
### 通过 SSH
 
当服务器仅在 localhost 上监听时，请从您的机器转发其端口，并在本地打开打印的链接：
 
```bash
$ ssh -L 49374:127.0.0.1:49374 my-server
```
 
默认情况下，服务器运行在端口 49374 上并仅在 localhost 上监听。您可以使用 `opencode service` 命令更改此配置。
 
## 配置
 
使用 `opencode service set` 设置任何选项：
 
```bash
# 在每个网络接口上监听
$ opencode service set hostname 0.0.0.0
 
# 使用固定端口而不是通道默认端口
$ opencode service set port 49374
 
# 替换生成的密码
$ opencode service set password "a-long-secret"
 
# 允许来自另一个源提供的 Web 客户端
$ opencode service set cors https://app.example.com,https://other.example.com
 
# 向服务器进程传递环境变量
$ opencode service set env OPENCODE_LOG_LEVEL DEBUG
```
 
更改设置会停止后台服务器。要应用新配置：
 
```bash
$ opencode service start
```
 
## 独立运行
 
`opencode serve` 在前台运行相同的服务器，而不是通过共享的后台服务运行。
 
```bash
$ opencode serve --hostname 0.0.0.0 --port 4096
server listening on http://0.0.0.0:4096
server password <password>
```
 
当您想要：
 
- 在共享、常开或远程主机上运行 OpenCode，然后通过 `opencode --server <url>` 连接客户端时。
- 控制单个进程的主机名、端口和 CORS 源时。
- 在 systemd、Docker 或其他期望前台进程的环境下运行 supervisor 时。
- 保留专用服务器而不是共享后台服务时。
 
使用它，并使用 `--server` 连接客户端：
 
```bash
$ opencode --server http://127.0.0.1:4096
```
