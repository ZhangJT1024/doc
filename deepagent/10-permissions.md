# 权限（Permissions）

> 原文：Permissions
> 来源：https://docs.langchain.com/oss/javascript/deepagents/permissions

---

> 通过声明式权限规则控制 Deep Agents 的文件系统访问

使用声明式权限规则控制 agent 可以读取或写入哪些文件和目录。将一组规则传给 `permissions=`，agent 内置的文件系统 tools 就会遵守它们。

> **注意**：Permissions 需要 `deepagents>=1.9.1`。

Permissions 仅适用于内置的文件系统 tools（`ls`、`read_file`、`glob`、`grep`、`write_file`、`edit_file`）。访问文件系统的自定义 tools 和 MCP tools 不在覆盖范围内。Permissions 也不适用于 [sandbox backends](/oss/javascript/deepagents/sandboxes)，后者通过 `execute` tool 支持任意命令执行。

> **提示**：当你需要对内置文件系统 tools 施加**基于路径的允许/拒绝规则**时，使用 `permissions`。当你需要自定义校验逻辑（速率限制、审计日志、内容检查）或需要控制自定义 tools 时，使用 [backend policy hooks](/oss/javascript/deepagents/backends#add-policy-hooks)。

## 基本用法

将一组 `FilesystemPermission` 规则传给 `createDeepAgent`。规则按声明顺序求值。第一个匹配的规则生效。如果没有规则匹配，则允许该操作。

```ts
const agent = createDeepAgent({
  model,
  backend,
  permissions: [
    {
      operations: ["write"],
      paths: ["/**"],
      mode: "deny",
    },
  ],
});
if (!agent) throw new Error("basic: agent not created");
```

## 规则结构

每个 `FilesystemPermission` 有三个字段：

| Field        | Type                    | Description                                                                                                                          |
| ------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `operations` | `("read" \| "write")[]` | 此规则适用的操作。`"read"` 涵盖 `ls`、`read_file`、`glob`、`grep`。`"write"` 涵盖 `write_file`、`edit_file`。      |
| `paths`      | `string[]`              | 用于匹配文件路径的 glob 模式（例如 `["/workspace/**"]`）。支持用于递归匹配的 `**` 和用于二选一的 `{a,b}`。 |
| `mode`       | `"allow" \| "deny"`     | 是允许还是拒绝匹配的操作。默认为 `"allow"`。                                                                 |

规则采用首次匹配即胜出的求值方式：其 `operations` 和 `paths` 匹配当前调用的第一条规则决定结果。如果没有规则匹配，则调用被**允许**（宽松的默认值）。

路径必须是绝对路径（以 `/` 开头），且不能包含 `..` 或 `~`。无效路径会在构造 agent 时抛出。

## 示例

### 隔离到某个工作目录

只允许在 `/workspace/` 下读写，拒绝其它所有内容：

```ts
const agent = createDeepAgent({
  model,
  backend,
  permissions: [
    {
      operations: ["read", "write"],
      paths: ["/workspace/**"],
      mode: "allow",
    },
    {
      operations: ["read", "write"],
      paths: ["/**"],
      mode: "deny",
    },
  ],
});
if (!agent) throw new Error("isolate-workspace: agent not created");
```

### 保护特定文件

```ts
const agent = createDeepAgent({
  model,
  backend,
  permissions: [
    {
      operations: ["read", "write"],
      paths: ["/workspace/.env", "/workspace/examples/**"],
      mode: "deny",
    },
    {
      operations: ["read", "write"],
      paths: ["/workspace/**"],
      mode: "allow",
    },
    {
      operations: ["read", "write"],
      paths: ["/**"],
      mode: "deny",
    },
  ],
});
if (!agent) throw new Error("protect-files: agent not created");
```

### 只读 memory

允许 agent 读取 memory 文件但阻止其修改它们。这对于只应由应用代码更新的组织级策略或共享知识库很有用。更多背景请参阅[只读 vs 可写 memory](/oss/javascript/deepagents/memory#read-only-vs-writable-memory)。

```ts
const store = new InMemoryStore();
const agent = createDeepAgent({
  model,
  backend: new CompositeBackend(new StateBackend(), {
    "/memories/": new StoreBackend({
      namespace: (rt) => [rt.serverInfo.user.identity],
    }),
    "/policies/": new StoreBackend({
      namespace: (rt) => [rt.context.orgId],
    }),
  }),
  permissions: [
    {
      operations: ["write"],
      paths: ["/memories/**", "/policies/**"],
      mode: "deny",
    },
  ],
  store,
});
if (!agent) throw new Error("read-only-memory: agent not created");
```

### 拒绝所有访问

阻止所有读写。这是一个限制性的基线，你可以在其上叠加更具体的允许规则：

```ts
const agent = createDeepAgent({
  model,
  backend,
  permissions: [
    {
      operations: ["read", "write"],
      paths: ["/**"],
      mode: "deny",
    },
  ],
});
if (!agent) throw new Error("deny-all: agent not created");
```

### 规则顺序

由于采用首次匹配即胜出，规则顺序很重要。将更具体的规则放在更宽泛的规则之前：

```ts
const correctPermissions: FilesystemPermission[] = [
  { operations: ["read", "write"], paths: ["/workspace/.env"], mode: "deny" },
  {
    operations: ["read", "write"],
    paths: ["/workspace/**"],
    mode: "allow",
  },
  { operations: ["read", "write"], paths: ["/**"], mode: "deny" },
];

const incorrectPermissions: FilesystemPermission[] = [
  {
    operations: ["read", "write"],
    paths: ["/workspace/**"],
    mode: "allow",
  },
  {
    operations: ["read", "write"],
    paths: ["/workspace/.env"],
    mode: "deny",
  },
  { operations: ["read", "write"], paths: ["/**"], mode: "deny" },
];
```

## Subagent permissions

[Subagents](/oss/javascript/deepagents/subagents) 默认继承父 agent 的 permissions。要给 subagent 不同的 permissions，请在其 spec 中设置 `permissions` 字段。这会**完全替换**父级的规则。

```ts
const agent = createDeepAgent({
  model,
  backend,
  permissions: [
    {
      operations: ["read", "write"],
      paths: ["/workspace/**"],
      mode: "allow",
    },
    { operations: ["read", "write"], paths: ["/**"], mode: "deny" },
  ],
  subagents: [
    {
      name: "auditor",
      description: "Read-only code reviewer",
      systemPrompt: "Review the code for issues.",
      permissions: [
        { operations: ["write"], paths: ["/**"], mode: "deny" },
        { operations: ["read"], paths: ["/workspace/**"], mode: "allow" },
        { operations: ["read"], paths: ["/**"], mode: "deny" },
      ],
    },
  ],
});
if (!agent) throw new Error("subagent: agent not created");
```

要显式授予 subagent 不受限制的访问权限，请设置 `permissions: []`。空数组会以无限制覆盖父级规则。省略 `permissions` 则从父级继承。

## Composite backends

当使用带 sandbox 默认值的 `CompositeBackend` 时，每个 permission 路径都必须限定在已知的路由前缀之下。Sandboxes 支持任意命令执行，因此仅靠基于路径的限制无法阻止通过 shell 命令访问文件系统。将 permissions 限定到特定路由的 [backends](/oss/javascript/deepagents/backends) 可以避免这一冲突。

```ts
const sandbox = new StateBackend();
const memoriesBackend = new StateBackend();
const composite = new CompositeBackend(sandbox, {
  "/memories/": memoriesBackend,
});
const agent = createDeepAgent({
  model,
  backend: composite,
  permissions: [
    { operations: ["write"], paths: ["/memories/**"], mode: "deny" },
  ],
});
if (!agent) throw new Error("composite-backend: agent not created");
```

包含任何路由之外路径的 permissions 会在构造时抛出：

```ts
const sandbox = new StateBackend();
const memoriesBackend = new StateBackend();
const composite = new CompositeBackend(sandbox, {
  "/memories/": memoriesBackend,
});

createDeepAgent({
  model,
  backend: composite,
  permissions: [
    { operations: ["write"], paths: ["/workspace/**"], mode: "deny" },
  ],
});

createDeepAgent({
  model,
  backend: composite,
  permissions: [{ operations: ["read"], paths: ["/**"], mode: "deny" }],
});
```

***
