# 人在回路（Human-in-the-loop）

> 原文：https://docs.langchain.com/oss/javascript/langchain/human-in-the-loop
> LangChain (TypeScript) 官方文档中文整理

> **文档索引**
> 获取完整文档索引：https://docs.langchain.com/llms.txt
> 使用该文件可在进一步探索前发现所有可用页面。

人在回路（HITL）[中间件](/oss/javascript/langchain/middleware/built-in#human-in-the-loop)允许你为智能体工具调用添加人工监督。
当模型提出可能需要审查的操作时——例如写入文件或执行 SQL——中间件可以暂停执行并等待决定。

它通过根据可配置的策略检查每个工具调用来实现这一点。如果需要干预，中间件会发出一个 [interrupt](https://reference.langchain.com/javascript/langchain-langgraph/index/interrupt)，从而停止执行。图状态会使用 LangGraph 的[持久化层](/oss/javascript/langgraph/persistence)保存，因此执行可以安全地暂停并在稍后恢复。

然后由人工决定接下来发生什么：操作可以按原样批准（`approve`）、在运行前修改（`edit`）、附带反馈拒绝（`reject`），或直接回复（`respond`）用于"询问用户"风格的工具。

## 中断决策类型

[中间件](/oss/javascript/langchain/middleware/built-in#human-in-the-loop)定义了四种内置的人工响应中断的方式：

| 决策类型 | 描述                                                                                                     | 示例用例                                  |
| ------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| ✅ `approve`   | 按智能体提出的原始参数执行工具。                                          | 按原样发送电子邮件草稿            |
| ✏️ `edit`     | 在执行前修改工具参数。                                                                     | 在发送电子邮件前更改收件人      |
| ❌ `reject`    | 完全跳过执行此工具调用，并向智能体返回拒绝反馈。                              | 拒绝删除文件并说明原因                |
| 💬 `respond`  | 直接将人工的消息作为合成的工具结果返回，跳过执行，用于"询问用户"风格的工具。 | 用直接回复回答 `"ask_user"` 提示 |

各工具可用的决策类型取决于你在 `interrupt_on` 中配置的策略。
当多个工具调用同时暂停时，每个操作都需要单独的决定。
决策必须按照操作在中断请求中出现的相同顺序提供。

当人工拒绝所请求的操作时，使用 `reject`。仅当人工充当工具本身（例如回答 `ask_user` 提示）时，才使用 `respond`。不要使用 `respond` 来拒绝有副作用的工具，因为它的消息会被视为成功的工具结果。

> **【提示】** **编辑**工具参数时，请保守地修改。对原始参数进行重大修改可能会导致模型重新评估其方法，并可能多次执行工具或采取意外操作。

## 配置中断

要使用 HITL，在创建智能体时将[中间件](/oss/javascript/langchain/middleware/built-in#human-in-the-loop)添加到智能体的 `middleware` 列表中。

你通过工具操作到每个操作允许的决策类型的映射来配置它。当工具调用与映射中的某个操作匹配时，中间件将中断执行。

```ts
import { createAgent, humanInTheLoopMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
    model: "gpt-5.5",
    tools: [writeFileTool, executeSQLTool, readDataTool],
    middleware: [
        humanInTheLoopMiddleware({
            interruptOn: {
                write_file: true, // All decisions (approve, edit, reject, respond) allowed
                execute_sql: {
                    allowedDecisions: ["approve", "reject"],
                    // No editing allowed
                    description: "🚨 SQL execution requires DBA approval",
                },
                // Safe operation, no approval needed
                read_data: false,
            },
            // Prefix for interrupt messages - combined with tool name and args to form the full message
            // e.g., "Tool execution pending approval: execute_sql with query='DELETE FROM...'"
            // Individual tools can override this by specifying a "description" in their interrupt config
            descriptionPrefix: "Tool execution pending approval",
        }),
    ],
    // Human-in-the-loop requires checkpointing to handle interrupts.
    // In production, use a persistent checkpointer like AsyncPostgresSaver or MongoDBSaver.
    checkpointer: new MemorySaver(),
});
```

> **【信息】** 你必须配置一个检查点保存器（checkpointer）以在中断之间持久化图状态。
>
> 在生产环境中，使用持久化检查点保存器，如 [`AsyncPostgresSaver`](https://reference.langchain.com/javascript/classes/_langchain_langgraph-checkpoint-postgres.AsyncPostgresSaver.html) 或 [`MongoDBSaver`](https://reference.langchain.com/javascript/langchain-langgraph-checkpoint-mongodb/MongoDBSaver)。对于测试或原型设计，使用 [`InMemorySaver`](https://reference.langchain.com/javascript/classes/_langchain_langgraph-checkpoint.MemorySaver.html)。
>
> 调用智能体时，传入一个包含**线程 ID** 的 `config`，以将执行与对话线程关联。
> 有关详细信息，请参阅 [LangGraph 中断文档](/oss/javascript/langgraph/interrupts)。

#### 配置选项

- **`interruptOn`**（`object`，必填）：工具名称到审批配置的映射

**工具审批配置选项：**

- **`allowAccept`**（`boolean`，默认 `false`）：是否允许批准
- **`allowEdit`**（`boolean`，默认 `false`）：是否允许编辑
- **`allowRespond`**（`boolean`，默认 `false`）：是否允许响应/拒绝

## 条件中断

默认情况下，`interrupt_on` 中列出的每个工具调用都会暂停以进行审查。要仅暂停某些调用，请为工具的 `InterruptOnConfig` 添加 `when` 谓词。该谓词接收一个 `ToolCallRequest`，返回 `True` 时中断，返回 `False` 时自动批准，因此你可以根据工具的参数进行控制。

条件中断目前仅在 Python 中可用。

## 响应中断

当你调用智能体时，它会一直运行，直到完成或引发中断。当工具调用与你配置在 `interrupt_on` 中的策略匹配时，会触发中断。使用 `version="v2"` 时，结果是一个带有 `interrupts` 属性的 `GraphOutput`，其中包含需要审查的操作。然后你可以将这些操作呈现给审查者，并在提供决策后恢复执行。

```typescript
import { HumanMessage } from "@langchain/core/messages";
import { Command } from "@langchain/langgraph";

// You must provide a thread ID to associate the execution with a conversation thread,
// so the conversation can be paused and resumed (as is needed for human review).
const config = { configurable: { thread_id: "some_id" } };

// Run the graph until the interrupt is hit.
const result = await agent.invoke(
    {
        messages: [new HumanMessage("Delete old records from the database")],
    },
    config
);


// The interrupt contains the full HITL request with action_requests and review_configs
console.log(result.__interrupt__);
// > [
// >    Interrupt(
// >       value: {
// >          actionRequests: [
// >             {
// >                name: 'execute_sql',
// >                arguments: { query: 'DELETE FROM records WHERE created_at < NOW() - INTERVAL \'30 days\';' },
// >                description: 'Tool execution pending approval\n\nTool: execute_sql\nArgs: {...}'
// >             }
// >          ],
// >          reviewConfigs: [
// >             {
// >                actionName: 'execute_sql',
// >                allowedDecisions: ['approve', 'reject']
// >             }
// >          ]
// >       }
// >    )
// > ]

// Resume with approval decision
await agent.invoke(
    new Command({
        resume: { decisions: [{ type: "approve" }] }, // or "reject"
    }),
    config // Same thread ID to resume the paused conversation
);
```

### 决策类型

#### ✅ approve

使用 `approve` 按原样批准工具调用，并在不做更改的情况下执行它。

```typescript
await agent.invoke(
    new Command({
        // Decisions are provided as a list, one per action under review.
        // The order of decisions must match the order of actions
        // in the interrupt request.
        resume: {
            decisions: [
                {
                    type: "approve",
                }
            ]
        }
    }),
    config  // Same thread ID to resume the paused conversation
);
```

#### ✏️ edit

使用 `edit` 在执行前修改工具调用。
提供带有新工具名称和参数的已编辑操作。

```typescript
await agent.invoke(
    new Command({
        // Decisions are provided as a list, one per action under review.
        // The order of decisions must match the order of actions
        // in the interrupt request.
        resume: {
            decisions: [
                {
                    type: "edit",
                    // Edited action with tool name and args
                    editedAction: {
                        // Tool name to call.
                        // Will usually be the same as the original action.
                        name: "new_tool_name",
                        // Arguments to pass to the tool.
                        args: { key1: "new_value", key2: "original_value" },
                    }
                }
            ]
        }
    }),
    config  // Same thread ID to resume the paused conversation
);
```

> **【提示】** **编辑**工具参数时，请保守地修改。对原始参数进行重大修改可能会导致模型重新评估其方法，并可能多次执行工具或采取意外操作。

#### ❌ reject

使用 `reject` 拒绝工具调用，并提供反馈而不是执行。该工具不会被执行。

```typescript
await agent.invoke(
    new Command({
        // Decisions are provided as a list, one per action under review.
        // The order of decisions must match the order of actions
        // in the interrupt request.
        resume: {
            decisions: [
                {
                    type: "reject",
                    // Optional: explain why the action was rejected
                    // and whether the agent should retry a different approach.
                    message: "User rejected this action. Do not retry this tool call.",
                }
            ]
        }
    }),
    config  // Same thread ID to resume the paused conversation
);
```

`message` 会作为反馈添加到对话中，以帮助智能体理解操作被拒绝的原因以及它应该做什么。当你省略 `message` 时，中间件会使用默认的拒绝消息，告诉模型工具未被执行，并且除非用户要求，否则不要重试相同的工具调用。对于有副作用的工具，请提供特定领域的消息，明确说明智能体应该放弃该操作、提出后续问题，还是尝试更安全的替代方案。

#### 💬 respond

对于工具的真实实现就是人工回复的"询问用户"风格的工具，使用 `respond`。`message` 内容会直接作为工具结果返回；工具本身不会被执行。

```typescript
await agent.invoke(
    new Command({
        // Decisions are provided as a list, one per action under review.
        // The order of decisions must match the order of actions
        // in the interrupt request.
        resume: {
            decisions: [
                {
                    type: "respond",
                    // The human's reply, returned directly as the tool result
                    message: "Blue.",
                }
            ]
        }
    }),
    config  // Same thread ID to resume the paused conversation
);
```

`message` 会作为成功的 `ToolMessage` 返回给智能体。当工具有意作为人工输入的占位符时（例如提示澄清的 `ask_user` 工具），使用 `respond`。不要使用 `respond` 来拒绝提议的操作，因为它会告诉模型工具已成功完成。

***

### 多个决策

当有多个操作需要审查时，按照它们在中断中出现的相同顺序为每个操作提供一个决策：

```typescript
{
    decisions: [
        { type: "approve" },
        {
            type: "edit",
            editedAction: {
                name: "tool_name",
                args: { param: "new_value" }
            }
        },
        {
            type: "reject",
            message: "This action is not allowed"
        }
    ]
}
```

## 带人在回路的流式传输

你可以在智能体运行并处理中断时使用 `stream_events()` 流式传输实时更新。使用 `stream.messages` 流式传输 LLM token，使用 `stream.values` 检查智能体状态快照以获取中断。

```typescript
import { Command } from "@langchain/langgraph";

const config = { configurable: { thread_id: "some_id" } };

// Stream agent progress and LLM tokens until interrupt
const stream = await agent.streamEvents(
    { messages: [{ role: "user", content: "Delete old records from the database" }] },
    { ...config, version: "v3" }
);
for await (const message of stream.messages) {
    for await (const token of message.text) {
        process.stdout.write(token);
    }
}

// Check whether the run paused for human input
if (stream.interrupted) {
    console.log(`\n\nInterrupt: ${JSON.stringify(stream.interrupts)}`);
}

// Resume with streaming after human decision
const resumeStream = await agent.streamEvents(
    new Command({ resume: { decisions: [{ type: "approve" }] } }),
    { ...config, version: "v3" }
);
for await (const message of resumeStream.messages) {
    for await (const token of message.text) {
        process.stdout.write(token);
    }
}
```

有关更多详细信息，请参阅[流式传输](/oss/javascript/langchain/streaming)指南。

## 执行生命周期

中间件定义了一个 `after_model` 钩子，它在模型生成响应之后、但在任何工具调用执行之前运行：

1. 智能体调用模型生成响应。
2. 中间件检查响应中的工具调用。
3. 如果任何调用需要人工输入，中间件会构建一个带有 `action_requests` 和 `review_configs` 的 [HITLRequest](https://reference.langchain.com/javascript/langchain/index/HITLRequest)，并调用 [interrupt](https://reference.langchain.com/javascript/langchain-langgraph/index/interrupt)。
4. 智能体等待人工决策。
5. 根据 `HITLResponse` 决策，中间件执行已批准或已编辑的调用，为被拒绝的调用合成 [ToolMessage](https://reference.langchain.com/javascript/langchain-core/messages/ToolMessage)，对于 `respond` 决策直接返回人工回复作为 [ToolMessage](https://reference.langchain.com/javascript/langchain-core/messages/ToolMessage)，并恢复执行。

## 自定义 HITL 逻辑

对于更专业的工作流，你可以直接使用 [interrupt](https://reference.langchain.com/javascript/langchain-langgraph/index/interrupt) 原语和[中间件](/oss/javascript/langchain/middleware)抽象来构建自定义 HITL 逻辑。

请查看上面的[执行生命周期](#execution-lifecycle)，了解如何将中断集成到智能体的操作中。

***

> **【信息】** [通过 MCP 将这些文档连接](/use-these-docs)到 Claude、VSCode 等，以获得实时答案。
>
> **【信息】** [在 GitHub 上编辑此页面](https://github.com/langchain-ai/docs/edit/main/src/oss/langchain/human-in-the-loop.mdx)或[提交问题](https://github.com/langchain-ai/docs/issues/new/choose)。
