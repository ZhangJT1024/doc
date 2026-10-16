# 结构化输出

> 原文：https://docs.langchain.com/oss/javascript/langchain/structured-output
> LangChain (TypeScript) 官方文档中文整理

结构化输出允许 agent 以特定、可预测的格式返回数据。这样你得到的就是类型化结构化数据，而无需解析自然语言响应。

> 【提示】本页介绍使用 `createAgent` 的 agent 结构化输出。若要直接在模型上使用结构化输出（在 agent 之外），请参阅[模型 - 结构化输出](/oss/javascript/langchain/models#structured-output)。

LangChain 预构建的 ReAct agent `createAgent` 会自动处理结构化输出。用户设置所需的结构化输出 schema，当模型生成结构化数据时，它会被捕获、验证，并返回到 agent 状态的 `structuredResponse` 键中。

```ts
type ResponseFormat = (
    | ZodSchema<StructuredResponseT> // a Zod schema
    | StandardSchema<StructuredResponseT> // any Standard Schema library
    | Record<string, unknown> // a JSON Schema
)

const agent = createAgent({
    // ...
    responseFormat: ResponseFormat | ResponseFormat[]
})
```

## 响应格式

控制 agent 如何返回结构化数据。你可以提供 Zod schema、任何兼容 [Standard Schema](https://standardschema.dev/) 的 schema，或一个 JSON Schema 对象。默认情况下，agent 使用工具调用策略，即输出通过一次额外的工具调用来生成。某些模型支持原生结构化输出，在这种情况下 agent 会改用该策略。

你可以通过将 `ResponseFormat` 包装在 `toolStrategy` 或 `providerStrategy` 函数调用来控制行为：

```ts
import { toolStrategy, providerStrategy } from "langchain";

const agent = createAgent({
    // use a provider strategy if supported by the model
    responseFormat: providerStrategy(z.object({ ... }))
    // or enforce a tool strategy
    responseFormat: toolStrategy(z.object({ ... }))
})
```

结构化响应会返回到 agent 最终状态的 `structuredResponse` 键中。

> 【提示】如果使用 `langchain>=1.1`，对原生结构化输出功能的支持会从模型的[配置文件数据](/oss/javascript/langchain/models#model-profiles)中动态读取。如果数据不可用，请使用其他条件或手动指定：
>
> ```typescript
> const customProfile: ModelProfile = {
>     structuredOutput: true,
>     // ...
> }
> const model = await initChatModel("...", { profile: customProfile });
> ```
>
> 如果指定了工具，模型必须支持同时使用工具和结构化输出。

## 提供商策略

某些模型提供商通过其 API 原生支持结构化输出（例如 OpenAI、xAI (Grok)、Gemini、Anthropic (Claude)）。在可用时，这是最可靠的方法。

要使用此策略，请配置 `ProviderStrategy`：

```ts
function providerStrategy<StructuredResponseT>(
    schema: ZodSchema<StructuredResponseT> | SerializableSchema | JsonSchemaFormat
): ProviderStrategy<StructuredResponseT>
```

**参数：`schema`（必填）**

定义结构化输出格式的 schema。支持：

* **Zod Schema**：一个 zod schema
* **Standard Schema**：任何实现了 [Standard Schema](https://standardschema.dev/) 规范的 schema
* **JSON Schema**：一个 JSON schema 对象

当你直接向 `createAgent.responseFormat` 传入 schema 类型且模型支持原生结构化输出时，LangChain 会自动使用 `ProviderStrategy`：

#### Zod Schema

```ts
import * as z from "zod";
import { createAgent, providerStrategy } from "langchain";

const ContactInfo = z.object({
    name: z.string().describe("The name of the person"),
    email: z.string().describe("The email address of the person"),
    phone: z.string().describe("The phone number of the person"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: providerStrategy(ContactInfo)
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Extract contact info from: John Doe, john@example.com, (555) 123-4567"}]
});

console.log(result.structuredResponse);
// { name: "John Doe", email: "john@example.com", phone: "(555) 123-4567" }
```

#### Standard Schema

```ts
import * as v from "valibot";
import { toStandardJsonSchema } from "@valibot/to-json-schema";
import { createAgent, providerStrategy } from "langchain";

const ContactInfo = toStandardJsonSchema(
    v.object({
        name: v.pipe(v.string(), v.description("The name of the person")),
        email: v.pipe(v.string(), v.description("The email address of the person")),
        phone: v.pipe(v.string(), v.description("The phone number of the person")),
    })
);

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: providerStrategy(ContactInfo)
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Extract contact info from: John Doe, john@example.com, (555) 123-4567"}]
});

console.log(result.structuredResponse);
// { name: "John Doe", email: "john@example.com", phone: "(555) 123-4567" }
```

#### JSON Schema

```ts
import { createAgent, providerStrategy } from "langchain";

const contactInfoSchema = {
    "type": "object",
    "description": "Contact information for a person.",
    "properties": {
        "name": {"type": "string", "description": "The name of the person"},
        "email": {"type": "string", "description": "The email address of the person"},
        "phone": {"type": "string", "description": "The phone number of the person"}
    },
    "required": ["name", "email", "phone"]
}

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: providerStrategy(contactInfoSchema)
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Extract contact info from: John Doe, john@example.com, (555) 123-4567"}]
});

console.log(result.structuredResponse);
// { name: "John Doe", email: "john@example.com", phone: "(555) 123-4567" }
```

提供商原生的结构化输出具有高可靠性和严格验证，因为模型提供商会强制执行该 schema。在可用时请使用它。

> 【注意】如果提供商对你所选的模型原生支持结构化输出，那么写 `responseFormat: contactInfoSchema` 与写 `responseFormat: providerStrategy(contactInfoSchema)` 在功能上是等价的。
>
> 无论哪种情况，如果不支持结构化输出，agent 都会回退到工具调用策略。

## 工具调用策略

对于不支持原生结构化输出的模型，LangChain 使用工具调用来达到相同的效果。这适用于所有支持工具调用的模型（大多数现代模型）。

要使用此策略，请配置 `ToolStrategy`：

```ts
function toolStrategy<StructuredResponseT>(
    responseFormat:
        | JsonSchemaFormat
        | ZodSchema<StructuredResponseT>
        | SerializableSchema
        | (ZodSchema<StructuredResponseT> | SerializableSchema | JsonSchemaFormat)[]
    options?: ToolStrategyOptions
): ToolStrategy<StructuredResponseT>
```

**参数：`schema`（必填）**

定义结构化输出格式的 schema。支持：

* **Zod Schema**：一个 zod schema
* **Standard Schema**：任何实现了 [Standard Schema](https://standardschema.dev/) 规范的 schema
* **JSON Schema**：一个 JSON schema 对象

**参数：`options.toolMessageContent`**

生成结构化输出时返回的工具消息的自定义内容。如果未提供，则默认为一条显示结构化响应数据的消息。

**参数：`options.handleError`**

选项参数，包含一个可选的 `handleError` 参数，用于自定义错误处理策略。

* **`true`**：使用默认错误模板捕获所有错误（默认）
* **`False`**：不重试，让异常向外传播
* **`(error: ToolStrategyError) => string | Promise<string>`**：使用提供的消息重试，或抛出错误

#### Zod Schema

```ts
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ProductReview = z.object({
    rating: z.number().min(1).max(5).optional(),
    sentiment: z.enum(["positive", "negative"]),
    keyPoints: z.array(z.string()).describe("The key points of the review. Lowercase, 1-3 words each."),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(ProductReview)
})

const result = await agent.invoke({
    "messages": [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})

console.log(result.structuredResponse);
// { "rating": 5, "sentiment": "positive", "keyPoints": ["fast shipping", "expensive"] }
```

#### Standard Schema

```ts
import * as v from "valibot";
import { toStandardJsonSchema } from "@valibot/to-json-schema";
import { createAgent, toolStrategy } from "langchain";

const ProductReview = toStandardJsonSchema(
    v.object({
        rating: v.optional(v.pipe(v.number(), v.minValue(1), v.maxValue(5))),
        sentiment: v.picklist(["positive", "negative"]),
        keyPoints: v.pipe(v.array(v.string()), v.description("The key points of the review. Lowercase, 1-3 words each.")),
    })
);

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(ProductReview)
})

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})

console.log(result.structuredResponse);
// { "rating": 5, "sentiment": "positive", "keyPoints": ["fast shipping", "expensive"] }
```

#### JSON Schema

```ts
import { createAgent, toolStrategy } from "langchain";

const productReviewSchema = {
    "type": "object",
    "description": "Analysis of a product review.",
    "properties": {
        "rating": {
            "type": ["integer", "null"],
            "description": "The rating of the product (1-5)",
            "minimum": 1,
            "maximum": 5
        },
        "sentiment": {
            "type": "string",
            "enum": ["positive", "negative"],
            "description": "The sentiment of the review"
        },
        "key_points": {
            "type": "array",
            "items": {"type": "string"},
            "description": "The key points of the review"
        }
    },
    "required": ["sentiment", "key_points"]
}

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(productReviewSchema)
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})

console.log(result.structuredResponse);
// { "rating": 5, "sentiment": "positive", "keyPoints": ["fast shipping", "expensive"] }
```

#### Union Types

```ts
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ProductReview = z.object({
    rating: z.number().min(1).max(5).optional(),
    sentiment: z.enum(["positive", "negative"]),
    keyPoints: z.array(z.string()).describe("The key points of the review. Lowercase, 1-3 words each."),
});

const CustomerComplaint = z.object({
    issueType: z.enum(["product", "service", "shipping", "billing"]),
    severity: z.enum(["low", "medium", "high"]),
    description: z.string().describe("Brief description of the complaint"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy([ProductReview, CustomerComplaint])
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})

console.log(result.structuredResponse);
// { "rating": 5, "sentiment": "positive", "keyPoints": ["fast shipping", "expensive"] }
```

### 自定义工具消息内容

`toolMessageContent` 参数允许你自定义在生成结构化输出时出现在对话历史中的消息：

```ts
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const MeetingAction = z.object({
    task: z.string().describe("The specific task to be completed"),
    assignee: z.string().describe("Person responsible for the task"),
    priority: z.enum(["low", "medium", "high"]).describe("Priority level"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(MeetingAction, {
        toolMessageContent: "Action item captured and added to meeting notes!"
    })
});

const result = await agent.invoke({
    messages: [{"role": "user", "content": "From our meeting: Sarah needs to update the project timeline as soon as possible"}]
});

console.log(result);
/**
 * {
 *   messages: [
 *     { role: "user", content: "From our meeting: Sarah needs to update the project timeline as soon as possible" },
 *     { role: "assistant", content: "Action item captured and added to meeting notes!", tool_calls: [ { name: "MeetingAction", args: { task: "update the project timeline", assignee: "Sarah", priority: "high" }, id: "call_456" } ] },
 *     { role: "tool", content: "Action item captured and added to meeting notes!", tool_call_id: "call_456", name: "MeetingAction" }
 *   ],
 *   structuredResponse: { task: "update the project timeline", assignee: "Sarah", priority: "high" }
 * }
 */
```

如果没有 `toolMessageContent`，我们会看到：

```ts
# console.log(result);
/**
 * {
 *   messages: [
 *     ...
 *     { role: "tool", content: "Returning structured response: {'task': 'update the project timeline', 'assignee': 'Sarah', 'priority': 'high'}", tool_call_id: "call_456", name: "MeetingAction" }
 *   ],
 *   structuredResponse: { task: "update the project timeline", assignee: "Sarah", priority: "high" }
 * }
 */
```

### 错误处理

模型在通过工具调用生成结构化输出时可能会犯错。LangChain 提供了智能重试机制来自动处理这些错误。

#### 多个结构化输出错误

当模型错误地调用了多个结构化输出工具时，agent 会在 [`ToolMessage`](https://reference.langchain.com/javascript/langchain-core/messages/ToolMessage) 中提供错误反馈，并提示模型重试：

```ts
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ContactInfo = z.object({
    name: z.string().describe("Person's name"),
    email: z.string().describe("Email address"),
});

const EventDetails = z.object({
    event_name: z.string().describe("Name of the event"),
    date: z.string().describe("Event date"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy([ContactInfo, EventDetails]),
});

const result = await agent.invoke({
    messages: [
        {
        role: "user",
        content:
            "Extract info: John Doe (john@email.com) is organizing Tech Conference on March 15th",
        },
    ],
});

console.log(result);

/**
 * {
 *   messages: [
 *     { role: "user", content: "Extract info: John Doe (john@email.com) is organizing Tech Conference on March 15th" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ContactInfo", args: { name: "John Doe", email: "john@email.com" }, id: "call_1" }, { name: "EventDetails", args: { event_name: "Tech Conference", date: "March 15th" }, id: "call_2" } ] },
 *     { role: "tool", content: "Error: Model incorrectly returned multiple structured responses (ContactInfo, EventDetails) when only one is expected.\n Please fix your mistakes.", tool_call_id: "call_1", name: "ContactInfo" },
 *     { role: "tool", content: "Error: Model incorrectly returned multiple structured responses (ContactInfo, EventDetails) when only one is expected.\n Please fix your mistakes.", tool_call_id: "call_2", name: "EventDetails" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ContactInfo", args: { name: "John Doe", email: "john@email.com" }, id: "call_3" } ] },
 *     { role: "tool", content: "Returning structured response: {'name': 'John Doe', 'email': 'john@email.com'}", tool_call_id: "call_3", name: "ContactInfo" }
 *   ],
 *   structuredResponse: { name: "John Doe", email: "john@email.com" }
 * }
 */
```

#### Schema 验证错误

当结构化输出与预期的 schema 不匹配时，agent 会提供具体的错误反馈：

```ts
import * as z from "zod";
import { createAgent, toolStrategy } from "langchain";

const ProductRating = z.object({
    rating: z.number().min(1).max(5).describe("Rating from 1-5"),
    comment: z.string().describe("Review comment"),
});

const agent = createAgent({
    model: "gpt-5.5",
    tools: [],
    responseFormat: toolStrategy(ProductRating),
});

const result = await agent.invoke({
    messages: [
        {
        role: "user",
        content: "Parse this: Amazing product, 10/10!",
        },
    ],
});

console.log(result);

/**
 * {
 *   messages: [
 *     { role: "user", content: "Parse this: Amazing product, 10/10!" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ProductRating", args: { rating: 10, comment: "Amazing product" }, id: "call_1" } ] },
 *     { role: "tool", content: "Error: Failed to parse structured output for tool 'ProductRating': 1 validation error for ProductRating\nrating\n  Input should be less than or equal to 5 [type=less_than_equal, input_value=10, input_type=int].\n Please fix your mistakes.", tool_call_id: "call_1", name: "ProductRating" },
 *     { role: "assistant", content: "", tool_calls: [ { name: "ProductRating", args: { rating: 5, comment: "Amazing product" }, id: "call_2" } ] },
 *     { role: "tool", content: "Returning structured response: {'rating': 5, 'comment': 'Amazing product'}", tool_call_id: "call_2", name: "ProductRating" }
 *   ],
 *   structuredResponse: { rating: 5, comment: "Amazing product" }
 * }
 */
```

#### 错误处理策略

你可以使用 `handleErrors` 参数自定义错误的处理方式：

**自定义错误消息：**

```ts
const responseFormat = toolStrategy(ProductRating, {
    handleError: "Please provide a valid rating between 1-5 and include a comment."
)

// Error message becomes:
// { role: "tool", content: "Please provide a valid rating between 1-5 and include a comment." }
```

**仅处理特定异常：**

```ts
import { ToolInputParsingException } from "@langchain/core/tools";

const responseFormat = toolStrategy(ProductRating, {
    handleError: (error: ToolStrategyError) => {
        if (error instanceof ToolInputParsingException) {
        return "Please provide a valid rating between 1-5 and include a comment.";
        }
        return error.message;
    }
)

// Only validation errors get retried with default message:
// { role: "tool", content: "Error: Failed to parse structured output for tool 'ProductRating': ...\n Please fix your mistakes." }
```

**处理多种异常类型：**

```ts
const responseFormat = toolStrategy(ProductRating, {
    handleError: (error: ToolStrategyError) => {
        if (error instanceof ToolInputParsingException) {
        return "Please provide a valid rating between 1-5 and include a comment.";
        }
        if (error instanceof CustomUserError) {
        return "This is a custom user error.";
        }
        return error.message;
    }
)
```

**不进行错误处理：**

```ts
const responseFormat = toolStrategy(ProductRating, {
    handleError: false  // All errors raised
)
```

***
