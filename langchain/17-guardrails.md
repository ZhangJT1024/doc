# 护栏（Guardrails）

> 原文：https://docs.langchain.com/oss/javascript/langchain/guardrails
> LangChain (TypeScript) 官方文档中文整理

# 护栏

> 为你的智能体实现安全检查与内容过滤

护栏通过在智能体执行的关键节点对内容进行验证和过滤，帮助你构建安全、合规的 AI 应用。它们可以检测敏感信息、执行内容策略、验证输出，并在不安全行为造成问题之前予以阻止。

常见用例包括：

* 防止 PII（个人身份信息）泄露
* 检测并阻止提示注入攻击
* 阻止不当或有害内容
* 执行业务规则与合规要求
* 验证输出质量与准确性

你可以使用[中间件](/oss/javascript/langchain/middleware)在策略性节点拦截执行来实现护栏——在智能体启动之前、完成之后，或围绕模型与工具调用。

![Middleware flow diagram](https://mintcdn.com/langchain-5e9cc07a/RAP6mjwE5G00xYsA/oss/images/middleware_final.png?fit=max&auto=format&n=RAP6mjwE5G00xYsA&q=85&s=eb4404b137edec6f6f0c8ccb8323eaf1)

护栏可以通过两种互补的方式实现：

* **确定性护栏（Deterministic guardrails）**：使用基于规则的逻辑，如正则表达式模式、关键词匹配或显式检查。快速、可预测且经济高效，但可能会遗漏细微的违规行为。
* **基于模型的护栏（Model-based guardrails）**：使用 LLM 或分类器以语义理解来评估内容。能够捕捉规则遗漏的细微问题，但更慢且更昂贵。

LangChain 既提供内置护栏（例如 [PII 检测](#pii-detection)、[人在回路](#human-in-the-loop)），也提供灵活的中间件系统，用于以任一方式构建自定义护栏。

## 内置护栏

### PII 检测

LangChain 提供内置中间件，用于检测和处理对话中的个人身份信息（PII）。该中间件可以检测常见的 PII 类型，如电子邮件、信用卡、IP 地址等。

PII 检测中间件适用于有合规要求的医疗和金融应用、需要清理日志的客服智能体，以及任何处理敏感用户数据的应用。

PII 中间件支持多种处理检测到的 PII 的策略：

| 策略     | 描述                                     | 示例                  |
| -------- | ---------------------------------------- | --------------------- |
| `redact` | 替换为 `[REDACTED_{PII_TYPE}]`           | `[REDACTED_EMAIL]`    |
| `mask`   | 部分遮蔽（例如保留最后 4 位数字）        | `****-****-****-1234` |
| `hash`   | 替换为确定性哈希值                       | `a8f5f167...`         |
| `block`  | 检测到时抛出异常                         | 抛出错误              |

```typescript
import { createAgent, piiRedactionMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [customerServiceTool, emailTool],
  middleware: [
    // Redact emails in user input before sending to model
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToInput: true,
    }),
    // Mask credit cards in user input
    piiRedactionMiddleware({
      piiType: "credit_card",
      strategy: "mask",
      applyToInput: true,
    }),
    // Block API keys - raise error if detected
    piiRedactionMiddleware({
      piiType: "api_key",
      detector: /sk-[a-zA-Z0-9]{32}/,
      strategy: "block",
      applyToInput: true,
    }),
  ],
});

// When user provides PII, it will be handled according to the strategy
const result = await agent.invoke({
  messages: [{
    role: "user",
    content: "My email is john.doe@example.com and card is 5105-1051-0510-5100"
  }]
});
```

**内置 PII 类型与配置**

**内置 PII 类型：**

* `email` - 电子邮件地址
* `credit_card` - 信用卡号（经 Luhn 校验）
* `ip` - IP 地址
* `mac_address` - MAC 地址
* `url` - URL

**配置选项：**

| 参数                 | 描述                                                              | 默认值                        |
| -------------------- | ----------------------------------------------------------------- | ----------------------------- |
| `piiType`            | 要检测的 PII 类型（内置或自定义）                                 | 必填                          |
| `strategy`           | 如何处理检测到的 PII（`"block"`、`"redact"`、`"mask"`、`"hash"`） | `"redact"`                    |
| `detector`           | 自定义检测器正则表达式模式                                        | `undefined`（使用内置检测器） |
| `applyToInput`       | 在模型调用前检查用户消息                                          | `true`                        |
| `applyToOutput`      | 在模型调用后检查 AI 消息                                          | `false`                       |
| `applyToToolResults` | 在工具执行后检查工具结果消息                                      | `false`                       |

有关 PII 检测能力的完整细节，请参阅[中间件文档](/oss/javascript/langchain/middleware#pii-detection)。

### 人在回路（Human-in-the-loop）

LangChain 提供内置中间件，用于在执行敏感操作之前要求人工审批。这是高风险决策中最有效的护栏之一。

人在回路中间件适用于金融交易与转账、删除或修改生产数据、向外部各方发送通信，以及任何具有重大业务影响的操作。

```typescript
import { createAgent, humanInTheLoopMiddleware } from "langchain";
import { MemorySaver, Command } from "@langchain/langgraph";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, sendEmailTool, deleteDatabaseTool],
  middleware: [
    humanInTheLoopMiddleware({
      interruptOn: {
        // Require approval for sensitive operations
        send_email: { allowAccept: true, allowEdit: true, allowRespond: true },
        delete_database: { allowAccept: true, allowEdit: true, allowRespond: true },
        // Auto-approve safe operations
        search: false,
      }
    }),
  ],
  checkpointer: new MemorySaver(),
});

// Human-in-the-loop requires a thread ID for persistence
const config = { configurable: { thread_id: "some_id" } };

// Agent will pause and wait for approval before executing sensitive tools
let result = await agent.invoke(
  { messages: [{ role: "user", content: "Send an email to the team" }] },
  config
);

result = await agent.invoke(
  new Command({ resume: { decisions: [{ type: "approve" }] } }),
  config  // Same thread ID to resume the paused conversation
);
```

> **【提示】** 有关实现审批工作流的完整细节，请参阅[人在回路文档](/oss/javascript/langchain/human-in-the-loop)。

## 自定义护栏

对于更复杂的护栏，你可以创建自定义中间件，在智能体执行之前或之后运行。这样你就能完全掌控验证逻辑、内容过滤和安全检查。

### 智能体之前的护栏

使用“智能体之前”（before agent）钩子，在每次调用的开始处对请求验证一次。这对于会话级检查很有用，例如身份验证、速率限制，或在任何处理开始之前阻止不当请求。

```typescript
import { createMiddleware, AIMessage } from "langchain";

const contentFilterMiddleware = (bannedKeywords: string[]) => {
  const keywords = bannedKeywords.map(kw => kw.toLowerCase());

  return createMiddleware({
    name: "ContentFilterMiddleware",
    beforeAgent: {
      hook: (state) => {
        // Get the first user message
        if (!state.messages || state.messages.length === 0) {
          return;
        }

        const firstMessage = state.messages[0];
        if (firstMessage._getType() !== "human") {
          return;
        }

        const content = firstMessage.content.toString().toLowerCase();

        // Check for banned keywords
        for (const keyword of keywords) {
          if (content.includes(keyword)) {
            // Block execution before any processing
            return {
              messages: [
                new AIMessage(
                  "I cannot process requests containing inappropriate content. Please rephrase your request."
                )
              ],
              jumpTo: "end",
            };
          }
        }

        return;
      },
      canJumpTo: ['end']
    }
  });
};

// Use the custom guardrail
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, calculatorTool],
  middleware: [
    contentFilterMiddleware(["hack", "exploit", "malware"]),
  ],
});

// This request will be blocked before any processing
const result = await agent.invoke({
  messages: [{ role: "user", content: "How do I hack into a database?" }]
});
```

### 智能体之后的护栏

使用“智能体之后”（after agent）钩子，在返回给用户之前对最终输出验证一次。这对于基于模型的安全检查、质量验证，或对完整的智能体响应进行最终合规扫描很有用。

```typescript
import { createMiddleware, AIMessage, initChatModel } from "langchain";

const safetyGuardrailMiddleware = () => {
  const safetyModel = initChatModel("gpt-5.4-mini");

  return createMiddleware({
    name: "SafetyGuardrailMiddleware",
    afterAgent: {
      hook: async (state) => {
        // Get the final AI response
        if (!state.messages || state.messages.length === 0) {
          return;
        }

        const lastMessage = state.messages[state.messages.length - 1];
        if (lastMessage._getType() !== "ai") {
          return;
        }

        // Use a model to evaluate safety
        const safetyPrompt = `Evaluate if this response is safe and appropriate.
        Respond with only 'SAFE' or 'UNSAFE'.

        Response: ${lastMessage.content.toString()}`;

        const result = await safetyModel.invoke([
          { role: "user", content: safetyPrompt }
        ]);

        if (result.content.toString().includes("UNSAFE")) {
          return {
            messages: [
              new AIMessage(
                "I cannot provide that response. Please rephrase your request."
              )
            ],
            jumpTo: "end",
          };
        }

        return;
      },
      canJumpTo: ['end']
    }
  });
};

// Use the safety guardrail
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, calculatorTool],
  middleware: [safetyGuardrailMiddleware()],
});

const result = await agent.invoke({
  messages: [{ role: "user", content: "How do I make explosives?" }]
});
```

### 组合多个护栏

你可以通过将它们添加到中间件数组来叠加多个护栏。它们按顺序执行，让你能够构建分层防护：

```typescript
import { createAgent, piiRedactionMiddleware, humanInTheLoopMiddleware } from "langchain";

const agent = createAgent({
  model: "gpt-5.5",
  tools: [searchTool, sendEmailTool],
  middleware: [
    // Layer 1: Deterministic input filter (before agent)
    contentFilterMiddleware(["hack", "exploit"]),

    // Layer 2: PII protection (before and after model)
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToInput: true,
    }),
    piiRedactionMiddleware({
      piiType: "email",
      strategy: "redact",
      applyToOutput: true,
    }),

    // Layer 3: Human approval for sensitive tools
    humanInTheLoopMiddleware({
      interruptOn: {
        send_email: { allowAccept: true, allowEdit: true, allowRespond: true },
      }
    }),

    // Layer 4: Model-based safety check (after agent)
    safetyGuardrailMiddleware(),
  ],
});
```

## 其他资源

* [中间件文档](/oss/javascript/langchain/middleware) - 自定义中间件完整指南
* [中间件 API 参考](https://reference.langchain.com/python/langchain/middleware/) - 自定义中间件完整指南
* [人在回路](/oss/javascript/langchain/human-in-the-loop) - 为敏感操作添加人工审核
* [测试智能体](/oss/javascript/langchain/test/) - 测试安全机制的策略
