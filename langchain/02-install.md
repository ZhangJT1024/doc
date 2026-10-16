# 安装 LangChain

> 原文：https://docs.langchain.com/oss/javascript/langchain/install
> LangChain (TypeScript) 官方文档中文整理

> ## 文档索引
> 获取完整文档索引请访问：https://docs.langchain.com/llms.txt
> 使用该文件可以在进一步浏览之前发现所有可用的页面。

# 安装 LangChain

安装 LangChain 包：

#### npm
```bash
npm install langchain @langchain/core
# Requires Node.js 22+
```

#### pnpm
```bash
pnpm add langchain @langchain/core
# Requires Node.js 22+
```

#### yarn
```bash
yarn add langchain @langchain/core
# Requires Node.js 22+
```

#### bun
```bash
bun add langchain @langchain/core
# Requires Bun v1.0.0+
```

LangChain 提供对数百个 LLM 和数千个其他集成的支持。这些集成位于各自独立的提供商包中。

#### npm
```bash
# Installing the OpenAI integration
npm install @langchain/openai
# Installing the Anthropic integration
npm install @langchain/anthropic
```

#### pnpm
```bash
# Installing the OpenAI integration
pnpm install @langchain/openai
# Installing the Anthropic integration
pnpm install @langchain/anthropic
```

#### yarn
```bash
# Installing the OpenAI integration
yarn add @langchain/openai
# Installing the Anthropic integration
yarn add @langchain/anthropic
```

#### bun
```bash
# Installing the OpenAI integration
bun add @langchain/openai
# Installing the Anthropic integration
bun add @langchain/anthropic
```

> **【提示】** 查看 [集成（Integrations）标签页](/oss/javascript/integrations/providers/overview) 获取可用集成的完整列表。

现在你已经安装了 LangChain，可以按照 [快速开始指南](/oss/javascript/langchain/quickstart) 开始使用。

> **【提示】** 设置 [LangSmith](https://smith.langchain.com?utm_source=docs\&utm_medium=cta\&utm_campaign=langsmith-signup\&utm_content=oss-langchain-install) 追踪来调试你的第一个 LangChain 应用。按照 [追踪快速开始](/langsmith/trace-with-langchain) 进行设置。我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的追踪记录、检测问题并提出修复建议。

***
