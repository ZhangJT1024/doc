# LangChain 概览

> 原文：https://docs.langchain.com/oss/javascript/langchain/overview
> LangChain (TypeScript) 官方文档中文整理

> ## 文档索引
> 获取完整文档索引请访问：https://docs.langchain.com/llms.txt
> 使用该文件可以在进一步浏览之前发现所有可用的页面。

# LangChain 概览

> LangChain 提供了 `create_agent`：一个极简、高度可配置的智能体运行框架（agent harness）。你可以根据用例需求，用模型、工具、提示词和中间件精确组合出所需的智能体。

**智能体 = 模型 + 运行框架（Agent = Model + Harness）。** LangChain 提供 `create_agent`：一个极简、高度可配置的运行框架。运行框架是围绕模型循环的一切：提示词、工具，以及任何塑造行为的中间件。从基础原语开始，精确组合你的用例所需的内容。支持 [OpenAI、Anthropic、Google 等](/oss/javascript/integrations/providers/overview)。

> **【提示】LangChain vs. LangGraph vs. Deep Agents**
>
> 如果需要一个“开箱即用”的智能体，具备自动上下文压缩、虚拟文件系统、子智能体生成等功能，请从 [Deep Agents](/oss/javascript/deepagents/overview/) 开始。Deep Agents 构建在 LangChain [智能体](/oss/javascript/langchain/agents/) 之上，你也可以直接使用后者。
>
> 如需高度可定制、能轻松适配你的用例和数据的运行框架，请使用 [LangChain](/oss/javascript/langchain/agents)（`create_agent`）。
>
> 对于结合确定性工作流与智能体工作流的高级需求，请使用我们的底层编排框架 [LangGraph](/oss/javascript/langgraph/overview)。
>
> 使用 [LangSmith](/langsmith/observability) 可以追踪、调试和评估用上述任一框架构建的智能体。按照 [追踪快速开始](/langsmith/trace-with-langchain) 进行设置。我们还建议你设置 [LangSmith Engine](/langsmith/engine)，它会监控你的追踪记录、检测问题并提出修复建议。

## 创建智能体

以下示例演示如何用自定义工具创建一个简单的 LangChain 智能体：

#### OpenAI
```ts
// First install: npm install langchain zod @langchain/openai
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Google Gemini
```ts
// First install: npm install langchain zod @langchain/google-genai
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "google-genai:gemini-2.5-flash-lite",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Claude (Anthropic)
```ts
// First install: npm install langchain zod @langchain/anthropic
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "claude-sonnet-4-6",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### OpenRouter
```ts
// First install: npm install langchain zod @langchain/openrouter
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "openrouter:anthropic/claude-sonnet-4-6",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Fireworks
```ts
// First install: npm install langchain zod
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "fireworks:accounts/fireworks/models/qwen3p5-397b-a17b",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Baseten
```ts
// First install: npm install langchain zod
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "baseten:zai-org/GLM-5.2",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Ollama
```ts
// First install: npm install langchain zod @langchain/ollama
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "ollama:devstral-2",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### Azure
```ts
// First install: npm install langchain zod @langchain/openai
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "azure_openai:gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

#### AWS Bedrock
```ts
// First install: npm install langchain zod @langchain/aws
import { createAgent, tool } from "langchain";
import * as z from "zod";

const getWeather = tool(
  (input) => `It's always sunny in ${input.city}!`,
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);

const agent = createAgent({
  model: "bedrock:gpt-5.5",
  tools: [getWeather],
});

console.log(
  await agent.invoke({
    messages: [{ role: "user", content: "What's the weather in San Francisco?" }],
  })
);
```

请参阅 [安装说明](/oss/javascript/langchain/install) 和 [快速开始指南](/oss/javascript/langchain/quickstart)，开始用 LangChain 构建你自己的智能体和应用程序。

> **【提示】** 使用 [LangSmith](/langsmith/observability) 追踪请求、调试智能体行为并评估输出。设置 `LANGSMITH_TRACING=true` 和你的 API 密钥即可开始。

## 核心优势

- **[标准化的模型接口](/oss/javascript/langchain/models)**：跨提供商使用统一的接口来调用聊天模型、嵌入模型等。以最少的代码改动切换模型，让应用在需求变化时依然保持可移植性。
- **[高度可配置的运行框架](/oss/javascript/langchain/agents)**：从 `create_agent` 这个极简运行框架开始，通过中间件逐步添加能力。只组合你的用例所需的内容，从防护栏、重试到路由和自定义工具策略。
- **[构建在 LangGraph 之上](/oss/javascript/langgraph/overview)**：LangChain 的智能体构建在 LangGraph 之上。这让我们能够利用 LangGraph 的持久化执行、人类介入（human-in-the-loop）支持、持久化等能力。
- **[用 LangSmith 调试](/langsmith/observability)**：在一处检查追踪记录、工具调用、状态转换和延迟。发现故障模式、评估质量，并用执行数据改进智能体行为。

***
