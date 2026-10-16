# 翻译与排版规范（内部使用）

你的任务：把官方 Deep Agents（TypeScript）文档的某个 `.md` 源文件，翻译整理成一份**完整、忠实、中文**的 Markdown 章节文档。

## 硬性要求

1. **全文使用简体中文**。但以下内容必须**保留英文原文，不要翻译**：
   - 所有代码（代码块内的一切，包括变量名、注释、字符串、报错信息）。
   - 专有名词、产品名、库名、API/类/函数/参数/字段名、枚举值、命令、路径、URL、文件名。例如：`createDeepAgent`、`agent`、`model`、`backend`、`tools`、`StateBackend`、`subagents`、`permissions`、`LangChain`、`LangGraph`、`LangSmith`、`MCP`、`npm install`、`/workspace/**`、`AGENTS.md`、`SKILL.md`。
   - 表格中若是 API 名称/字段名/枚举，保留英文；仅翻译描述性文字。
2. **忠实、完整**：不得删减正文要点、不得摘要、不得自行发挥。逐段翻译，保持信息量与原文档一致。
3. **不丢代码块**：原文档中出现的每一个代码块都要保留（语言标签也要保留），只去掉形如 `theme={...}` 的属性。
4. **保持章节结构**：`#` / `##` / `###` 的层级与顺序要和原文一致。标题翻译成中文（专有名词保留英文），例如 `## Core capabilities` → `## 核心能力`。

## MDX 组件处理规则（原文是 Mintlify MDX）

- 删除文件开头的 `> ## Documentation Index ...` 引用块。
- 删除所有 JSX/TS 导出块（例如 `export const PatternEmbed = ({...}) => {...}` 这种整段脚本），以及 `<PatternEmbed ... />` 这类纯交互组件（可替换成一行中文说明：`> 官方原页面此处为交互式 Playground 演示，请在官网查看。`）。
- 删除结尾的 `<div className="source-links"> ... </div>` 整块。
- 组件转换为 Markdown：
  - `<Note>...</Note>` → `> **注意**：...`
  - `<Tip>...</Tip>` / `icon="lightbulb"` → `> **提示**：...`
  - `<Info>...</Info>` → `> **说明**：...`
  - `<Warning>...</Warning>` → `> **警告**：...`
  - `<Card title="X" href="Y">正文</Card>` → `> **X**（[链接](Y)）：正文`
  - `<Accordion title="X">正文</Accordion>` → `#### X` + 正文（或保留标题下正文）
  - `<Tabs>` / `<Tab title="X">` → 用 `#### X` 小标题分隔各 Tab 内容（语言 Tab 如 TypeScript/Python 保留 `#### TypeScript` / `#### Python`）。
  - `<CodeGroup>` 内多个代码块：全部保留，可按语言加注释或 `#### LanguageName`。
  - `<Steps>` / `<Step title="X">` → 有序列表，步骤标题加粗。
  - `<Columns>` / `<Column>` / `<Frame>` / `<div>` 等布局标签：去掉标签，保留内部内容。
- 链接处理：保留 Markdown 链接语法 `[中文文字](URL)`。站内相对链接（以 `/oss/...` 开头）原样保留 URL。

## 每个输出文件的固定开头模板

```markdown
# <中文标题>

> 原文：<English Title>
> 来源：<官方页面 URL>

---

<正文...>
```

- `<English Title>` 用源文件 H1 的英文标题。
- `<官方页面 URL>` 用 `https://docs.langchain.com/oss/javascript/deepagents/<slug>`。

## 常用术语对照（供参考，专有名词本身仍保留英文）

- harness → 运行框架 / 载体（可保留 harness）
- execution environment → 执行环境
- context management → 上下文管理
- delegation → 任务委派
- steering → 人工干预 / 引导
- subagent → 子代理（subagent 保留）
- tool / tool call → 工具 / 工具调用
- backend → 后端（backend 保留）
- filesystem / virtual filesystem → 文件系统 / 虚拟文件系统
- permission → 权限
- skill → 技能（skill 保留）
- memory → 记忆（memory 保留）
- summarization → 摘要
- context offloading → 上下文卸载
- reflection → 反思
- human-in-the-loop → Human-in-the-loop（保留英文）
