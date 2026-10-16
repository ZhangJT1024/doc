# LangChain JS API 参考 · 人在环路（Human-in-the-Loop）

> 对应官方参考：https://reference.langchain.com/javascript/langchain/...
> 整理自官方 API 参考 v1.5.15；说明文字为简体中文，代码与签名保持英文。

本章涵盖 LangChain JS 中用于处理“人在环路（Human-in-the-Loop）”模式的各种接口、类型和配置。它提供了模型在执行某些动作（如工具调用）之前需要人工审批、编辑或拒绝的机制，包括请求结构、响应数据模型以及拦截策略。

## 目录
- [ApproveDecision](#approvedecision)
- [Decision](#decision)
- [DecisionType](#decisiontype)
- [DescriptionFactory](#descriptionfactory)
- [EditDecision](#editdecision)
- [HITLRequest](#hitlrequest)
- [HITLResponse](#hitlresponse)
- [Interrupt](#interrupt)
- [InterruptOnConfig](#interruptonconfig)
- [RejectDecision](#rejectdecision)
- [ReviewConfig](#reviewconfig)
- [WhenPredicate](#whenpredicate)

## 接口
### `ApproveDecision`
<a id="ApproveDecision"></a>
```ts
interface ApproveDecision
```
当人类批准操作时，`ApproveDecision` 的响应。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| approve | `string` | |
| type | `"approve"` | |

### `EditDecision`
<a id="EditDecision"></a>
```ts
interface EditDecision
```
当人类编辑操作时，`EditDecision` 的响应。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| editedAction | `Action` | 代理需要执行的已编辑操作。 |
| type | `"edit"` | |

### `HITLRequest`
<a id="HITLRequest"></a>
```ts
interface HITLRequest
```
模型请求的动作序列中的人类反馈请求。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| actionRequests | `ActionRequest[]` | 待人类审核的代理动作列表。 |
| reviewConfigs | `ReviewConfig[]` | 所有可能动作的审核配置。 |

### `HITLResponse`
<a id="HITLResponse"></a>
```ts
interface HITLResponse
```
`HITLRequest` 的响应负载。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| decisions | `Decision[]` | 人类做出的决策。 |

### `Interrupt`
<a id="Interrupt"></a>
```ts
interface Interrupt
```
表示中断的信息。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| id | `string` | 中断的 ID。 |
| value | `TValue` | 人类输入请求。 |

### `RejectDecision`
<a id="RejectDecision"></a>
```ts
interface RejectDecision
```
当人类拒绝操作时，`RejectDecision` 的响应。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| message | `string` | 向模型解释操作被拒绝原因的消息。 |
| type | `"reject"` | |

### `ReviewConfig`
<a id="ReviewConfig"></a>
```ts
interface ReviewConfig
```
审核 `HITL` 请求的策略。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| actionName | `string` | 与此审核配置相关的动作名称。 |
| allowedDecisions | `"approve" | "edit" | "reject"[]` | 此请求允许的决策。 |
| argsSchema | `Record<string, any>` | 如果允许编辑，则为动作相关参数的 JSON Schema。 |

## 类型
### `Decision`
<a id="Decision"></a>
```ts
Decision : ApproveDecision | EditDecision | RejectDecision
```
所有可能决策类型的联合类型。

### `DecisionType`
<a id="DecisionType"></a>
```ts
z . infer < typeof DecisionType >
```

### `DescriptionFactory`
<a id="DescriptionFactory"></a>
```ts
z . infer < typeof DescriptionFunctionSchema >
```
动态为工具调用审批请求生成描述的函数类型。

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| toolCall | `ToolCall` | 是 | |
| state | `any` | 是 | |
| runtime | `any` | 是 | |

| 返回 | 类型 | 说明 |
| --- | --- | --- |
| `string` | | |

**示例**
```ts
import { type DescriptionFactory, type ToolCall } from "langchain" ; 
const descriptionFactory : DescriptionFactory = ( toolCall , state , runtime ) => { 
  return `Please review: ${ toolCall . name }(${ JSON . stringify ( toolCall . args ) })` ; 
  };
```

### `InterruptOnConfig`
<a id="InterruptOnConfig"></a>
```ts
InterruptOnConfig : z . input < typeof InterruptOnConfigSchema >
```

### `WhenPredicate`
<a id="WhenPredicate"></a>
```ts
z . infer < typeof WhenFunctionSchema >
```
控制给定工具调用是否触发中断的谓词。接收 `ToolCallRequest`，如果返回 `true` 则中断，如果返回 `false` 则自动批准该工具调用。请求中 `tool` 设置为 `undefined`，`runtime` 设置为节点级 `Runtime`，因此反映的是批处理（`afterModel`）上下文，而非单个调用工具的执行上下文。

**示例**
```ts
import { type WhenPredicate } from "langchain" ; 
// Only interrupt delete_file calls targeting /etc 
const when : WhenPredicate = ( request ) => 
String (request.toolCall.args.path ?? "" ). startsWith ( "/etc" );
```
