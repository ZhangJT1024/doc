> ## Documentation Index
> Fetch the complete documentation index at: https://docs.langchain.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Event streaming

> Stream subagents, messages, tool calls, and final output from Deep Agents.

This page covers streaming concerns specific to Deep Agents—most importantly, streaming from delegated subagents via `stream.subagents`. For general agent streaming (`stream.messages`, `stream.values`, tool calls, custom updates), see [LangChain Event Streaming](/oss/javascript/langchain/event-streaming).

## Stream subagents

Deep Agents add a subagent projection on top of LangGraph streaming. Use `stream.subagents` when you want one stream handle per delegated `task` call. The projection is lightweight: it discovers subagent tasks first, and message, tool-call, and value streams are opened only when you access them on a subagent handle.

Each handle's `name` is the sub-agent's configured name: the `subagent_type` the coordinator passes when it calls the `task` tool. Deep Agents binds that name to the delegated run, so the same label you defined in your subagent specs is what you filter and route on in the stream.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "Write me a haiku about the sea" }] },
  { version: "v3" },
);

const subagentNames: string[] = [];
for await (const subagent of stream.subagents) {
  console.log(subagent.name);
  console.log(await subagent.taskInput);

  for await (const message of subagent.messages) {
    console.log(await message.text);
  }

  subagentNames.push(subagent.name);
}

await stream.output;
```

## Subagent stream fields

Each subagent stream exposes the same kinds of projections as the parent run, such as messages, tool calls, nested subagents, and final output. For the general parent-run streaming model, see [LangChain Event Streaming](/oss/javascript/langchain/event-streaming).

TypeScript uses camelCase projection names such as `toolCalls` and `taskInput`. Each subagent stream can expose `.messages`, `.toolCalls`, `.values`, `.subagents`, and `.output`.

| Field       | Description                                                                                |
| ----------- | ------------------------------------------------------------------------------------------ |
| `name`      | Sub-agent name, taken from the `subagent_type` the coordinator selects in its `task` call. |
| `messages`  | Messages emitted by the subagent.                                                          |
| `subagents` | Nested subagent invocations.                                                               |
| `output`    | Final subagent state, or completion signal for the delegated task.                         |
| `taskInput` | Promise for the prompt passed to the task tool.                                            |
| `toolCalls` | Tool calls scoped to the subagent.                                                         |

## Track subagent lifecycle

Use `stream.subagents` when you only need to show which subagents started and finished. You do not need to subscribe to message or value streams unless you access those projections on an individual subagent.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

let running = 0;
let completed = 0;
let failed = 0;
const watchers: Promise<void>[] = [];

for await (const subagent of stream.subagents) {
  running += 1;
  console.log(`${subagent.name}: started`);

  watchers.push(
    subagent.output.then(
      () => {
        running -= 1;
        completed += 1;
        console.log(`${subagent.name}: completed`);
      },
      () => {
        running -= 1;
        failed += 1;
        console.log(`${subagent.name}: failed`);
      },
    ),
  );
}

await Promise.all(watchers);
console.log({ running, completed, failed });
```

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/1d3f1455-4a9c-4607-aa1d-115948f2e1fa/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

## Stream messages

Deep Agents can emit messages from the coordinator agent and from delegated subagents. Use `stream.messages` for top-level messages and `subagent.messages` for each delegated subagent.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

const coordinatorMessages: string[] = [];
for await (const message of stream.messages) {
  const text = await message.text;
  console.log("[coordinator]", text);
  coordinatorMessages.push(text);
}

for await (const subagent of stream.subagents) {
  for await (const message of subagent.messages) {
    console.log(`[${subagent.name}]`, await message.text);
  }
}

await stream.output;
```

## Stream tool calls

Deep Agents expose tool calls at each level of the agent tree. Use the top-level `stream.tool_calls` for coordinator tools and each `subagent.tool_calls` for delegated work.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

const coordinatorToolNames: string[] = [];
for await (const call of stream.toolCalls) {
  console.log("[coordinator tool]", call.name, call.input);
  console.log(await call.status);
  coordinatorToolNames.push(call.name);
}

for await (const subagent of stream.subagents) {
  for await (const call of subagent.toolCalls) {
    console.log(`[${subagent.name} tool]`, call.name, call.input);

    const status = await call.status;
    if (status === "finished") {
      console.log(await call.output);
    } else if (status === "error") {
      console.error(await call.error);
    }
  }
}
```

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/3fdda16a-c42c-4931-81f5-36e4a4ecfc5c/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

## Stream nested work

You can recurse into a subagent stream to observe nested subagents, messages, and tool calls.

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

const subagentNames: string[] = [];
for await (const subagent of stream.subagents) {
  console.log(`subagent ${subagent.name}: started`);

  for await (const toolCall of subagent.toolCalls) {
    console.log(`${toolCall.name}(${JSON.stringify(toolCall.input)})`);

    const status = await toolCall.status;
    if (status === "finished") {
      console.log(await toolCall.output);
    } else if (status === "error") {
      console.error(await toolCall.error);
    }
  }

  for await (const nested of subagent.subagents) {
    console.log(`nested subagent ${nested.name}: started`);
  }

  subagentNames.push(subagent.name);
}
```

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/0b947581-de0c-491e-96c1-ba37cc862b0e/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

## Consume concurrently

Coordinator and subagent output often interleave. Consume projections concurrently when you need live UI updates.

Use concurrent consumers in JavaScript:

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

await Promise.all([
  (async () => {
    for await (const message of stream.messages) {
      console.log("[coordinator]", await message.text);
    }
  })(),
  (async () => {
    for await (const subagent of stream.subagents) {
      void (async () => {
        for await (const message of subagent.messages) {
          console.log(`[${subagent.name}]`, await message.text);
        }
      })();
    }
  })(),
]);
```

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/467e4c96-80a0-42ba-9f5f-d869b74e5899/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

When you need exact arrival order across the coordinator and all subagents, iterate raw protocol events and use `namespace` to identify the source:

```ts theme={"theme":{"light":"catppuccin-latte","dark":"catppuccin-mocha"}}
const stream = await agent.streamEvents(input, { version: "v3" });

const textDeltas: string[] = [];
for await (const event of stream) {
  if (event.method !== "messages") continue;

  const data = event.params.data;
  if (data.event !== "content-block-delta") continue;

  const block = data.delta ?? {};
  if (block.type === "text-delta") {
    const isSubagent = event.params.namespace.some((seg) =>
      seg.startsWith("tools:"),
    );
    const source = isSubagent ? "subagent" : "coordinator";
    console.log(`[${source}] ${block.text}`);
    textDeltas.push(block.text);
  }
}
```

<Card title="View example trace" icon="chart-line" href="https://smith.langchain.com/public/4113a4d2-b6d8-4f2a-ad1d-2ab9e29859e6/r" arrow horizontal>
  Open a public LangSmith run for this example.
</Card>

## Subagents versus subgraphs

`stream.subgraphs` shows graph execution structure. `stream.subagents` shows product-level Deep Agents task delegations. Use `stream.subagents` for user-facing UI because it hides internal graph nodes and exposes the subagent concept directly.

## Related

* [LangChain Event Streaming](/oss/javascript/langchain/event-streaming) covers general agent message and tool-call streaming concepts.
* [Subagent frontend streaming](/oss/javascript/deepagents/frontend/subagent-streaming) shows UI patterns that separate coordinator messages from subagent cards.
* [LangGraph Event Streaming](/oss/javascript/langgraph/event-streaming) covers the underlying graph streaming model.

***

<div className="source-links">
  <Callout icon="terminal-2">
    [Connect these docs](/use-these-docs) to your agent of choice via MCP for real-time answers.
  </Callout>

  <Callout icon="edit">
    [Edit this page on GitHub](https://github.com/langchain-ai/docs/edit/main/src/oss/deepagents/event-streaming.mdx) or [file an issue](https://github.com/langchain-ai/docs/issues/new/choose).
  </Callout>
</div>
