<path>D:\code\doc\opencodeDoc\migrate-v1.md</path>
<type>file</type>
<content>
1: # 从 V1 迁移
2: 
3: OpenCode 1 和 OpenCode 2 都使用 `opencode` 命令，且不再默认并排安装。在安装 V2 之前，请移除由包管理器管理的 V1 安装；V2 curl 安装程序将替换 V1 二进制文件。
4: 
5: ## 重大变更
6: 
7: 
8: V2 有三个预期的重大变更：
9: 
10: - [插件](/build/plugins/migrate-v1) 使用新的插件 API。
11: - [服务器 API 和客户端](#server-api-and-clients) 具有新的契约。
12: - [终端客户端配置](/cli/config) 从分层的 `tui.json(c)` 文件移至单一的全局 `cli.json` 文件（自动
13:   迁移）。
14: 
15: 在这些区域之外的受支持的 V1 功能旨在保持与 V1 兼容。V1 架构中接受的一些字段从未有 V2 对应项并会被有意忽略；这些字段在
16: [接受但不支持的字段](#accepted-but-unsupported-fields) 下列出。
17: 
18: 
19: 现有的受支持的服务器配置字段、智能体定义、命令定义、技能以及 `.opencode/` 中的其他文件
20: 应继续无需更改即可工作。如果本指南中描述的受支持行为在 V2 中停止工作，请将其视为
21: 兼容性错误，而非预期的迁移要求。
22: 
23: <Callout type="tip">
24:   如果受支持的 V1 功能在 V2 中不起作用，请遵循
25:   [故障排除](/troubleshooting) 中的问题报告指南并提交兼容性问题。
26: </Callout>
27: 
28: ## 安装 V2
29: 
30: 通过 [终端启动指南](/cli) 安装 V2 终端客户端。
31: 
32: 在启动 V2 之前，你不需要重写所有内容。实际的迁移步骤如下：
33: 
34: 1. 保留你现有的配置和基于文件的定义。
35: 2. 启动 V2 并验证模型、凭证、智能体、权限和 MCP 服务器。
36: 3. 迁移插件，因为 V1 插件实现无法在 V2 中运行。
37: 4. 迁移调用服务器 API 的集成。
38: 5. 当你准备好时，将配置转换为原生 V2 形状。此步骤是可选的。
39: 
40: 在验证迁移时，保留一份你的 V1 设置副本。V1 和 V2 使用相同的配置位置，因此在将文件转换为仅限原生 V2 的形状后，请不要
41: 指向 V1 使用这些文件。
42: 
43: ## 配置
44: 
45: 本节涵盖 `.opencode/` 下的 JSON/JSONC 配置和基于文件的定义。
46: 
47: ### 使用你现有的配置
48: 
49: V2 从与 V1 相同的位置读取现有的全局和项目配置：
50: 
51: ```text
52: ~/.config/opencode/opencode.json(c)
53: <project>/opencode.json(c)
54: <project>/.opencode/opencode.json(c)
55: ```
56: 
57: V2 读取这些相同的位置。它在内存中规范化受支持的 V1 字段和原生 V2 字段，而不会重写源
58: 文件。现有的受支持的 V1 配置旨在继续工作，因此你不需要将其转换为原生 V2 格式来尝试或采用
59: V2。
60: 
61: ### 让 OpenCode 进行迁移
62: 
63: V1 配置格式仍然受到支持。原生 V2 格式是可选的，并使多个设置更加明确且更符合人体工程学。
64: 
65: 
66: 推荐的迁移路径是让 OpenCode 为你更新配置：
67: 
68: 
69: ```text
70: 将我的 OpenCode 配置（包括基于文件的定义）从 V1 格式迁移到原生 V2 格式。
71: 保留其行为和所有无关的设置。
72: ```
73: 
74: OpenCode 可以检查完整文件，应用下方的相关更改，并避免重写不需要
75: 更改的设置。转换不需要一次性完成：受支持的 V1 字段和原生 V2 字段可以共存于顶层。
76: 当两种形式设置相同的规范值时，有效的原生 V2 值无论 JSON 键的顺序如何都优先。
77: 
78: 嵌套混合是被有意限制的。OpenCode 识别 `mcp`、`compaction` 和
79: `experimental` 内部混合的 V1 和 V2 成员，但它不会递归地推断单个智能体、提供程序、命令或模型内部的格式。请保持
80: 每个此类嵌套条目完全处于一种格式中。受支持的 V1 语法本身保持静默；格式错误的数值、
81: 不受支持的遗留字段以及冲突的 V1/V2 数值会产生警告，而无关的有效设置将继续加载。
82: 
83: 
84: ### 分享
85: 
86: 已弃用的 V1 `autoshare` 布尔值变为显式的 `share` 策略：
87: 
88: ```jsonc
89: // V1
90: { "autoshare": true }
91: 
92: // V2
93: { "share": "auto" }
94: ```
95: 
96: 使用 `"manual"`、`"auto"` 或 `"disabled"`。如果 V1 文件已经使用 `share`，则无需更改。
97: 
98: ### 权限和工具
99: 
100: V1 按工具对权限效果进行分组。V2 使用一个有序的 `permissions` 数组，使优先级和例外变得显式：
101: 
102: ```jsonc
103: // V1
104: {
105:   "permission": {
106:     "bash": {
107:       "git push *": "ask"
108:     },
109:     "edit": "allow"
110:   },
111:   "tools": {
112:     "websearch": false
113:   }
114: }
115: 
116: // V2
117: {
118:   "permissions": [
119:     { "action": "shell", "resource": "git push *", "effect": "ask" },
120:     { "action": "edit", "resource": "*", "effect": "allow" },
121:     { "action": "websearch", "resource": "*", "effect": "deny" }
122:   ]
123: }
124: ```
125: 
126: 权限操作也发生了变化：`bash` 现在是 `shell`，`task` 现在是 `subagent`，而 `write` 和 `patch` 现在是 `edit`。
127: 参阅 [Permissions](/permissions) 查看有序的 V2 规则格式。
128: 
129: ### 智能体和模式
130: 
131: 单一的 `agent` 和已弃用的 `mode` 映射变为 `agents`。智能体字段变得与其余
132: V2 配置更一致：
133: 
134: ```jsonc
135: // V1
136: {
137:   "agent": {
138:     "reviewer": {
139:       "prompt": "Review for correctness and missing tests.",
140:       "model": "anthropic/claude-sonnet-4-5",
141:       "variant": "high",
142:       "disable": false,
143:       "permission": {
144:         "edit": "deny"
145:       }
146:     }
147:   }
148: }
149: 
150: // V2
151: {
152:   "agents": {
153:     "reviewer": {
154:       "system": "Review for correctness and missing tests.",
155:       "model": "anthropic/claude-sonnet-4-5#high",
156:       "disabled": false,
157:       "permissions": [
158:         { "action": "edit", "resource": "*", "effect": "deny" }
159:       ]
160:     }
161:   }
162: }
163: ```
164: 
165: `prompt` 变为 `system`，`disable` 变为 `disabled`，并且独立的 `variant` joins 模型引用后的 `#`。
166: `temperature`、`top_p` 以及提供程序特定的 `options` 移至 `request.body` 下。`maxSteps` 变为 `steps`。从
167: 旧的 `mode` 映射中的条目变为主要智能体。
168: 
169: ### 快照
170: 
171: 将单一的 `snapshot` 字段重命名为 `snapshots`。其布尔值不改变：
172: 
173: ```jsonc
174: // V1
175: { "snapshot": false }
176: 
177: // V2
178: { "snapshots": false }
179: ```
180: 
181: ### 多媒体
182: 
183: 将单一的 `attachment` 对象重命名为 `media`。嵌套的图像设置保持相同的名称：
184: 
185: ```jsonc
186: // V1
187: { "attachment": { "image": { "auto_resize": true } } }
188: 
189: // V2
190: { "media": { "image": { "auto_resize": true } } }
191: ```
192: 
193: ### MCP 服务器
194: 
195: V2 在 `mcp.servers` 下对服务器进行分组，将 `enabled` 替换为反义词 `disabled`，并区分超时目的：
196: 
197: ```jsonc
198: // V1
199: {
200:   "mcp": {
201:     "playwright": {
202:       "type": "local",
203:       "command": ["npx", "@playwright/mcp"],
204:       "enabled": true,
205:       "timeout": 30000
206:     }
207:   }
208: }
209: 
210: // V2
211: {
212:   "mcp": {
213:     "servers": {
214:       "playwright": {
215:         "type": "local",
216:         "command": ["npx", "@playwright/mcp"],
217:         "disabled": false,
218:         "timeout": {
219:           "catalog": 30000,
220:           "execution": 30000
221:         }
222:       }
223:     }
224:   }
225: }
226: ```
227: 
228: 远程 OAuth 字段使用蛇形命名：`clientId` 变为 `client_id`，`clientSecret` 变为 `client_secret`，
229: `callbackPort` 变为 `callback_port`，且 `redirectUri` 变为 `redirect_uri`。V1 的 `experimental.mcp_timeout` 值
230: 也变为默认的 `mcp.timeout.catalog` 和 `mcp.timeout.execution` 值。参阅 [MCP servers](/mcp-servers)。
231: 
232: ### 压缩
233: 
234: V2 在 `keep` 下对保留上下文的 token 预算进行分组，并为预留空间提供更清晰的名称：
235: 
236: ```jsonc
237: // V1
238: {
239:   "compaction": {
240:     "preserve_recent_tokens": 8000,
241:     "reserved": 20000
242:   }
243: }
244: 
245: // V2
246: {
247:   "compaction": {
248:     "keep": {
249:       "tokens": 8000
250:     },
251:     "buffer": 20000
252:   }
253: }
254: ```
255: 
256: `auto` 保留其名称。V2 没有原生的 `tail_turns` 或 `prune` 字段；这两个遗留字段均在发出警告的同时被忽略。最近
257: 的上下文由 token 预算保留。参阅 [Compaction](/compaction)。
258: 
259: ### 技能
260: 
261: V1 分开额外的技能路径和 URL。V2 将两者合并为一个有序数组：
262: 
263: ```jsonc
264: // V1
265: {
266:   "skills": {
267:     "paths": ["./team-skills"],
268:     "urls": ["https://example.com/skills/"]
269:   }
270: }
271: 
272: // V2
273: {
274:   "skills": ["./team-skills", "https://example.com/skills/"]
275: }
276: ```
277: 
278: 现有的技能文件和自动 `.opencode/skills/` 发现机制保持不变。参阅 [Skills](/skills)。
279: 
280: ### 命令
281: 
282: 将单一的 `command` 映射重命名为 `commands`，并将 `subtask` 重命名为 `subagent`。将独立的模型 `variant` 加入模型引用：
283: 
284: ```jsonc
285: // V1
286: {
287:   "command": {
288:     "review": {
289:       "template": "Review the current changes.",
290:       "model": "anthropic/claude-sonnet-4-5",
291:       "variant": "high",
292:       "subtask": true
293:     }
294:   }
295: }
296: 
297: // V2
298: {
299:   "commands": {
300:     "review": {
301:       "template": "Review the current changes.",
302:       "model": "anthropic/claude-sonnet-4-5#high",
303:       "subagent": true
304:     }
305:   }
306: }
307: ```
308: 
309: `template`、`description` 和 `agent` 保留其名称。遗留的 `subtask` 仍被接受；委托的命令现在将
310: 在后台自动运行并向父会话报告其结果。现有的 Markdown 命令定义仍继续受到支持。
311: 参阅 [Commands](/commands)。
312: 
313: ### 引用
314: 
315: 将已弃用的单一 `reference` 映射重命名为 `references`：
316: 
317: ```jsonc
318: // V1
319: { "reference": { "docs": "../docs" } }
320: 
321: // V2
322: { "references": { "docs": "../docs" } }
323: ```
324: 
325: V1 已经接受 `references`，因此当文件使用它时无需更改。引用条目保持
326: 相同的形状。参阅 [References](/references)。
327: 
328: ### 提供程序
329: 
330: 将单一的 `provider` 映射重命名为 `providers`。V2 将运行时包、端点和请求设置分开：
331: 
332: ```jsonc
333: // V1
334: {
335:   "provider": {
336:     "acme": {
337:       "npm": "@ai-sdk/openai-compatible",
338:       "api": "https://llm.example.com/v1",
339:       "options": {
340:         "apiKey": "{env:ACME_API_KEY}"
341:       }
342:     }
343:   }
344: }
345: 
346: // V2
347: {
348:   "providers": {
349:     "acme": {
350:       "package": "aisdk:@ai-sdk/openai-compatible",
351:       "settings": {
352:         "baseURL": "https://llm.example.com/v1",
353:         "apiKey": "{env:ACME_API_KEY}"
354:       }
355:     }
356:   }
357: }
358: ```
359: 
360: V1 的 `npm` 变为 `package`，AI SDK 软件包将收到 `aisdk:` 前缀。`api` 变为 `settings.baseURL`。提供程序
361: `options` 根据其请求角色分为 `settings`、`headers` 和 `body`。参阅 [Providers](/providers)。
362: 
363: V2 合并了两个遗留的提供程序命名空间：
364: 
365: | V1 provider ID             | Canonical V2 provider ID |
366: | -------------------------- | ------------------------ |
367: | `azure-cognitive-services` | `azure`                  |
368: | `google-vertex-anthropic`  | `google-vertex`          |
369: 
370: 无歧义的 V1 提供程序、智能体、命令和提供程序过滤器迁移使用这些规范 ID。共享的
371: 顶层 `model` 字段保留其确切的提供程序 ID，因为相同的语法在原生 V2 配置中有效；在迁移遗留的内置提供程序时，请将该字段
372: 更新为规范 ID。
373: 
374: ### 模型和变体
375: 
376: 模型仍然嵌套在其提供程序下，但几个模型字段变得更加明确：
377: 
378: - `id` 变为 `modelID`。
379: - `tool_call` 和 `modalities` 变为 `capabilities.tools`、`capabilities.input` 和 `capabilities.output`。
380: - 状态为 `"deprecated"` 变为 `disabled: true`。
381: - 缓存成本从 `cache_read` 和 `cache_write` 移至 `cache.read` 和 `cache.write`。
382: - 提供程序特定的 `options` 变为 `settings`。
383: - V1 变体对象变为包含每个条目 `id` 的 V2 数组。
384: 
385: ```jsonc
386: // V1
387: {
388:   "variants": {
389:     "high": {
390:       "reasoningEffort": "high"
391:     }
392:   }
393: }
394: 
395: // V2
396: {
397:   "variants": [
398:     {
399:       "id": "high",
400:       "settings": {
401:         "reasoningEffort": "high"
402:       }
403:     }
404:   ]
405: }
406: ```
407: 
408: 参阅 [Models](/models) 获取完整的原生模型形状。
409: 
410: ### 没有直接原生对应项的受支持字段
411: 
412: 大多数保持相同形状的字段，包括 `shell`、`model`、`default_agent`、`watcher`、`formatter`、
413: `instructions`、`enterprise` 和 `tool_output`，不需要迁移。
414: 
415: V2 接受并保留 `lsp` 配置，但它不运行语言服务器，不暴露 LSP 工具，也不产生 LSP
416: 诊断。请用项目的 lint、typecheck 或编译器命令替换依赖于这些能力的流程。
417: 
418: V1 提供程序过滤器没有一一对应的原生 V2 配置字段，但其行为通过
419: [policies](/policies) 继续受到支持：
420: 
421: - `enabled_providers` 变为内部默认拒绝提供程序策略，随后对列出的提供程序进行允许。
422: - `disabled_providers` 变为对列出提供程序的内部拒绝策略。
423: - `autoupdate` 变为 `update`：`false` 映射到 `"disable"`，`"notify"` 映射到 `"notify"`，且 `true` 映射到 `"auto"`。
424: - `small_model` 变为内置 `title` 智能体的 `model` 选择。原生 V2 配置应使用
425:   `agents.title.model` 代之。
426: 
427: 你可以保留这些字段的 V1 语法。OpenCode 在不发出警告的情况下规范化它们。
428: 
429: ### 接受但不支持的字段
430: 
431: V1 架构还接受没有受支持的 V2 行为的字段。V2 忽略这些值并发出警告，以便
432: 它们不被误认为是活动配置：
433: 
434: - `logLevel`：启动 OpenCode 时使用 `OPENCODE_LOG_LEVEL`。
435: - `server`：使用 V2 服务和显式的服务器选项；服务器 API 是一个预期的重大变更。
436: - 顶层的 `subagent_depth`：应改用 `experimental.subagent_depth`。
437: - `compaction.tail_turns` 和 `compaction.prune`：V2 使用 `compaction.keep.tokens` 和基于检查点的压缩。
438: - V1 JSON 配置中的智能体 `name`。
439: - 一个没有 `type` 的仅启用 V1 MCP 条目。
440: - V1 实验性字段 `batch_tool`、`openTelemetry`、`primary_tools` 和 `continue_loop_on_deny`。
441: - V1 提供程序字段 `id`、`whitelist` 和 `blacklist`。
442: - V1 提供程序-模型字段 `release_date`、`attachment`、`reasoning`、`temperature`、`experimental`、非 `deprecated`
443:   的 `status` 以及布尔值 `interleaved`。
444: 
445: 忽略这些字段是预期的，并且不是兼容性倒退。如果 V2 没有保留本指南中其他地方标识为受支持的行为，请遵循
446: [故障排除](/troubleshooting) 中的问题报告指南。
447: 
448: ### 智能体文件
449: 
450: V1 智能体文件可使用 `agent/`、`agents/`、`mode/` 或 `modes/`。V2 仍会发现所有四个目录。首选的
451: V2 位置是：
452: 
453: ```text
454: .opencode/agents/<name>.md
455: ```
456: 
457: V1 `mode/` 或 `modes/` 目录下的文件代表主要智能体。当将其中一个移入 `agents/` 时，请在
458: 其前置内容中添加 `mode: primary`。`agent/` 下的文件可以移至 `agents/` 而无需更改其基于路径生成的 ID。
459: 
460: 在将前置内容转换为原生 V2 字段时：
461: 
462: - 将 Markdown 正文作为智能体的系统指令。
463: - 在 JSON 配置中出现时，将 `prompt` 重命名为 `system`；文件正文不需要 `system` 字段。
464: - 将 `disable` 重命名为 `disabled`，将 `permission` 重命名为 `permissions`。
465: - 将 `model` 和 `variant` 合并为 `provider/model#variant`。
466: - 将 `temperature`、`top_p` 和提供程序特定的选项移至 `request.body` 下。
467: 
468: V2 会自动转换遗留的智能体前置内容，因此这些编辑是可选的。参阅 [Agents](/agents)。
469: 
470: ### 命令文件
471: 
472: V1 命令文件可使用 `command/` 或 `commands/`。V2 都会发现。首选位置是：
473: 
474: ```text
475: .opencode/commands/<name>.md
476: ```
477: 
478: 将文件从 `command/` 移动到 `commands/` 下的相同相对路径，以保留命令名称。Markdown 正文保持
479: 为命令模板，且 `description` 和 `agent` 前置内容保持相同名称。将 `subtask` 重命名为 `subagent` 以使用
480: 用于后台委托的原始名称。如果前置内容具有独立的
481: `model` 和 `variant` 字段，请将变体追加到模型并移除 `variant`：
482: 
483: ```yaml
484: # V1
485: model: anthropic/claude-sonnet-4-5
486: variant: high
487: subtask: true
488: 
489: # V2
490: model: anthropic/claude-sonnet-4-5#high
491: subagent: true
492: ```
493: 
494: 参阅 [Commands](/commands)。
495: 
496: ### 技能文件
497: 
498: V2 从 `.opencode/skill/` 和 `.opencode/skills/` 两个位置发现技能。首选布局是：
499: 
500: ```text
501: .opencode/skills/<skill-id>/SKILL.md
502: ```
503: 
504: 移动完整的技能目录，而不仅仅是 `SKILL.md`，以便相对脚本、引用和其他支持文件保持
505: 可用。保持目录名称稳定以保留技能 ID。现有的技能前置内容和 Markdown 正文不需要
506: V2 重写。参阅 [Skills](/skills)。
507: 
508: ### 指令文件
509: 
510: 现有的 `AGENTS.md` 文件保持不变。V2 发现全局 `~/.config/opencode/AGENTS.md` 以及从当前目录到主目录
511: 发现的 `AGENTS.md` 文件。对于主目录之外的项目，发现将停止在项目根目录。
512: 
513: 如果 V1 设置依赖于 `CLAUDE.md` 回退，请将该指南移至适用的 `AGENTS.md`。V2 目前仅
514: 发现 `AGENTS.md`；由于非 API 的 V1 行为旨在保持兼容，请同时提交带有受影响项目详情的兼容性问题。参阅 [Instructions](/instructions)。
515: 
516: ## 终端客户端配置
517: 
518: V2 用一个全局终端客户端配置文件替换分层的 V1 `tui.json(c)` 文件：
519: 
520: ```text
521: ~/.config/opencode/cli.json
522: ```
523: 
524: 终端客户端拥有此文件；后台服务不加载它。当 `cli.json` 不存在时，第一个 V2
525: 终端客户端启动将迁移受支持的全局 `tui.json` 设置和持久化首选项，同时保持 V1 文件
526: 不变。项目本地客户端配置不会迁移，因为 V2 客户端配置是全局的。参阅
527: [CLI settings](/cli/config) 获取当前的客户端设置参考。
528: 
529: ## 插件
530: 
531: 将 `plugin` 重命名为 `plugins`。用对象替换“软件包和选项”元组：
532: 
533: ```jsonc
534: // V1
535: {
536:   "plugin": ["opencode-example-plugin", ["./plugin/local.ts", { "enabled": true }]]
537: }
538: 
539: // V2
540: {
541:   "plugins": [
542:     "opencode-example-plugin",
543:     {
544:       "package": "./plugin/local.ts",
545:       "options": { "enabled": true }
546:     }
547:   ]
548: }
549: ```
550: 
551: V2 从 `.opencode/plugin/` 和 `.opencode/plugins/` 两个位置发现本地插件；对于 V2 文件，请使用 `.opencode/plugins/`。
552: 
553: <Callout type="warning">
554:   V1 插件实现无法在 V2 中运行。仅移动文件或重命名其配置条目是不够的。
555: </Callout>
556: 
557: 使用专用的
558: [插件迁移指南](/build/plugins/migrate-v1) 迁移入口点、钩子、工具、事件和软件包导出。
559: 
560: ## 服务器 API 和客户端
561: 
562: OpenCode 2 拥有经过改进、更符合人体工程学的服务器 API 和一组新的客户端。调用 V1 服务器 API 的集成
563: 必须迁移到 V2 API。
564: 
565: 使用 `@opencode/client` 软件包访问发布的 V2 API。参阅生成的 [API 引用](/api) 查看其
566: 端点、请求类型和响应。
567: 
568: ## 验证你的设置
569: 
570: 在依赖 V2 进行常规工作之前，在项目中验证你的模型、提供程序凭证、智能体、权限、MCP 服务器和插件。保留你的 V1 设置直到你确认所需的 V2 行为，并且不要将 V1 指向
571: 你已转换为原生 V2 形状的配置。
```