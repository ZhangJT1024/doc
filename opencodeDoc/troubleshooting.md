<path>D:\code\doc\opencodeDoc\troubleshooting.md</path>
<type>file</type>
<content>
1: # 故障排除
2: 
3: <Callout type="tip">
4:   你可以要求 OpenCode 调试自身。描述问题并要求它使用此故障排除页面；它可以阅读
5:   下方的步骤，检查其服务和日志，并帮助识别问题。
6: </Callout>
7: 
8: OpenCode 使用客户端-服务器架构。后台服务器拥有会话、插件、权限和其他应用程序
9: 状态。首先确定问题是在客户端、共享服务器还是特定项目中。
10: 
11: ## 检查后台服务
12: 
13: 显示当前的服务器信息：
14: 
15: ```bash
16: opencode service status
17: ```
18: 
19: 验证其 API 是否健康：
20: 
21: ```bash
22: opencode api get /api/info
23: ```
24: 
25: 如果服务卡住或不健康，重启它：
26: 
27: ```bash
28: opencode service restart
29: ```
30: 
31: 你也可以显式地停止和启动它：
32: 
33: ```bash
34: opencode service stop
35: opencode service start
36: ```
37: 
38: <Callout type="note">
39:   OpenCode 通常会自动发现或启动共享后台服务。仅在诊断其生命周期时才需要服务命令。
40: </Callout>
41: 
42: ## 允许浏览器源
43: 
44: 如果另一个源的浏览器客户端由于 CORS 无法连接，请将客户端的源添加到服务配置中：
45: 
46: ```bash
47: opencode service set cors http://192.168.1.10:3001
48: opencode service get cors
49: ```
50: 
51: 使用准确的 HTTP 或 HTTPS 源，必要时包含端口，不带路径或末尾斜杠。要允许多个
52: 源，请作为单个参数传递逗号分隔的列表；每个源周围的空格将被修剪：
53: 
54: ```bash
55: opencode service set cors "http://192.168.1.10:3001, https://app.example.com"
56: ```
57: 
58: `service get cors` 打印 JSON 数组。使用以下命令移除配置的列表：
59: 
60: ```bash
61: opencode service unset cors
62: ```
63: 
64: 设置或取消设置服务配置会停止后台服务。它的下一次启动将加载新配置；
65: 使用 `opencode service start` 显式启动它。
66: 
67: 对于前台服务器，为每个额外的允许源重复 `--cors`：
68: 
69: ```bash
70: opencode serve --cors http://192.168.1.10:3001 --cors https://app.example.com
71: ```
72: 
73: 使用 `serve --service` 时，提供的 `--cors` 标志将覆盖该进程的持久化列表。如果没有这些标志，
74: 服务模式将使用持久化列表。CORS 不会更改监听地址或绕过服务器身份验证。
75: 
76: ## 检查 API
77: 
78: `api` 命令使用本地服务发现和身份验证流程。它接受 HTTP 方法和路径，或
79: OpenAPI 操作 ID。
80: 
81: 参阅 [API reference](/api) 查看所有端点和操作 ID。
82: 
83: 使用 `--data` 或 `-d` 传递 JSON 请求体，并使用 `--header` 或 `-H` 添加标头。
84: 
85: <Callout type="warning">
86:   运行 `opencode api` 时，如果不存在兼容的健康服务，可能会启动后台服务。
87: </Callout>
88: 
89: ## 读取日志
90: 
91: 已安装的构建将日志写入：
92: 
93: ```text
94: ~/.local/share/opencode/log/opencode.log
95: ```
96: 
97: 在重现问题时遵循日志：
98: 
99: ```bash
100: tail -f ~/.local/share/opencode/log/opencode.log
101: ```
102: 
103: 每行都包括一个进程 `run` ID 和一个 `role` 字段。使用 `role=server` 查看会话、提供程序、插件、权限和
104: 工具活动。
105: 
106: ```bash
107: grep 'role=server' ~/.local/share/opencode/log/opencode.log
108: grep 'run=8fc3b1d5' ~/.local/share/opencode/log/opencode.log
109: ```
110: 
111: ## 捕获 CPU 和内存配置
112: 
113: 在 macOS 和 Linux 上，你可以发出信号给正在运行的 OpenCode 进程以捕获诊断数据。从健康端点获取后台服务器 PID：
114: 
115: ```bash
116: opencode api get /api/info
117: ```
118: 
119: 使用来自响应的 `pid` 以及以下信号之一：
120: 
121: - `SIGPROF` 捕获十秒的 CPU 配置：
122: 
123:   ```bash
124:   kill -SIGPROF <pid>
125:   ```
126: 
127:   结果将作为 `cpu-<pid>-<timestamp>.cpuprofile` 写入日志目录。
128: 
129: - `SIGUSR1` 捕获内存（堆）快照：
130: 
131:   ```bash
132:   kill -SIGUSR1 <pid>
133:   ```
134: 
135:   结果将作为 `heap-<pid>-<timestamp>.heapsnapshot` 写入日志目录。
136: 
137: 在打开文件之前，请等待 `opencode.log` 中出现 `CPU profile written` 或 `heap snapshot written`。相应的
138: 日志条目包含其完整路径。你可以在 Chrome DevTools 中查看这两种文件类型。
139: 
140: <Callout type="note">
141:   信号触发的配置在 Windows 上不可用。写入堆快照可能会暂停进程并暂时
142:   增加其内存使用情况。
143: </Callout>
144: 
145: ## 服务文件
146: 
147: 共享服务器在以下位置注册：
148: 
149: ```text
150: ~/.local/state/opencode/service.json
151: ```
152: 
153: 其私有服务配置单独存储在：
154: 
155: ```text
156: ~/.config/opencode/service.json
157: ```
158: 
159: 数据库通常位于：
160: 
161: ```text
162: ~/.local/share/opencode/opencode.db
163: ```
164: 
165: `OPENCODE_DB` 可以覆盖数据库位置。
166: 
167: <Callout type="warning">
168:   在故障排除期间，请勿删除或编辑服务文件或数据库。使用服务命令来管理
169: 守护进程，并在使用外部工具检查持久数据之前进行备份。
170: </Callout>
171: 
172: ## 报告问题
173: 
174: 在报告可重现的问题时，请包含以下内容：
175: 
176: - `opencode --version` 的输出
177: - `opencode service status` 的输出
178: - 重现问题的最小步骤序列
179: - 问题是否影响共享服务、特定客户端或单个项目
180: - 相关日志行，包括其 `run` 和 `role` 字段
181: 
182: 在分享日志之前，请移除 API 密钥、授权标头、提示词、文件内容和其他敏感数据。
183: 
184: 在 [GitHub Issues](https://github.com/anomalyco/opencode/issues) 中提交可重现的问题。
```