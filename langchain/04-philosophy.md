# 设计理念

> 原文：https://docs.langchain.com/oss/javascript/langchain/philosophy
> LangChain (TypeScript) 官方文档中文整理

> ## 文档索引
> 获取完整文档索引请访问：https://docs.langchain.com/llms.txt
> 使用该文件可以在进一步浏览之前发现所有可用的页面。

# 设计理念

> LangChain 存在的意义是成为使用 LLM 构建应用最容易上手的地方，同时保持灵活并可用于生产环境。

LangChain 由几个核心理念驱动：

* 大型语言模型（LLM）是伟大且强大的新技术。
* 当 LLM 与外部数据源结合时，它们会变得更强大。
* LLM 将改变未来应用的面貌。具体来说，未来的应用将越来越具备智能体（agentic）特性。
* 这一转变仍处于非常早期的阶段。
* 虽然构建这类智能体应用的原型很容易，但要构建足够可靠、能投入生产的智能体仍然非常困难。

如今，开发者可以选择如何构建智能体：使用 [LangChain](/oss/javascript/langchain/overview) 获得最大的灵活性和控制力，或使用 [Deep Agents](/oss/javascript/langchain/overview)，它提供类似的灵活性和控制力，但自带精心设计的规划、文件系统工具、子智能体和上下文管理。两者都构建在 [LangGraph](/oss/javascript/langgraph/overview) 之上。

对于 LangChain，我们有两个核心关注点：

#### 我们希望让开发者能够使用最好的模型进行构建。

### 我们希望让开发者能够使用最好的模型进行构建

    不同的提供商暴露不同的 API，具有不同的模型参数和不同的消息格式。
    标准化这些模型输入和输出是一个核心关注点，让开发者能够轻松切换到最新的最先进模型，避免被锁定。

#### 我们希望让使用模型来编排与其他数据和计算交互的更复杂流程变得容易。

### 我们希望让使用模型来编排与其他数据和计算交互的更复杂流程变得容易

    模型不应该只用于 *文本生成* ——它们还应该被用来编排与其他数据交互的更复杂流程。LangChain 让定义 LLM 可以动态使用的 [工具（tools）](/oss/javascript/langchain/tools) 变得容易，同时也有助于解析和访问非结构化数据。

## 历史

鉴于该领域持续不断的变化速度，LangChain 也随时间不断演进。以下是 LangChain 多年来如何变化的简要时间线，它与“使用 LLM 进行构建”含义的演变同步发展：

#### 2022-10-24（v0.0.1）

### 2022-10-24（v0.0.1）

    在 ChatGPT 出现的一个月前，**LangChain 作为 Python 包发布**。它由两个主要组件组成：

    * LLM 抽象
    * “Chains（链）”，即为常见用例预先确定的计算步骤。例如 - RAG：先运行检索步骤，再运行生成步骤。

    LangChain 这个名字来自 “Language”（如语言模型）和 “Chains”。

#### 2022-12

### 2022-12

    第一批通用智能体被添加到 LangChain。

    这些通用智能体基于 [ReAct 论文](https://arxiv.org/abs/2210.03629)（ReAct 代表 Reasoning and Acting，即推理与行动）。它们使用 LLM 生成表示工具调用的 JSON，然后解析该 JSON 以确定要调用哪些工具。

#### 2023-01

### 2023-01（Chat Completion API 发布）

    OpenAI 发布 “Chat Completion” API。

    此前，模型接收字符串并返回字符串。在 ChatCompletions API 中，它们演进为接收消息列表并返回一条消息。其他模型提供商纷纷效仿，LangChain 也更新为支持消息列表。

#### 2023-01

### 2023-01（JavaScript 版本发布）

    LangChain 发布 JavaScript 版本。

    LLM 和智能体将改变应用的构建方式，而 JavaScript 是应用开发者的语言。

#### 2023-02

### 2023-02

    **LangChain Inc. 作为公司成立**，围绕开源 LangChain 项目。

    主要目标是“让智能体无处不在”。团队认识到，虽然 LangChain 是关键的一部分（LangChain 让开始使用 LLM 变得简单），但还需要其他组件。

#### 2023-03

### 2023-03

    OpenAI 在其 API 中发布 “function calling（函数调用）”。

    这使 API 能够明确生成表示工具调用的负载。其他模型提供商纷纷效仿，LangChain 也更新为将其用作工具调用的首选方式（而不是解析 JSON）。

#### 2023-06

### 2023-06

    **LangSmith 发布**，作为 LangChain Inc. 的闭源平台，提供可观测性和评估（evals）。

    构建智能体的主要问题是让它们变得可靠，而 LangSmith 提供的可观测性和评估正是为解决这一需求而构建。LangChain 也更新为与 LangSmith 无缝集成。

#### 2024-01（v0.1.0）

### 2024-01（v0.1.0）

    **LangChain 发布 0.1.0**，这是它第一个非 0.0.x 版本。

    行业从原型走向生产，因此 LangChain 更加注重稳定性。

#### 2024-02

### 2024-02

    **LangGraph 作为开源库发布**。

    最初的 LangChain 有两个关注点：LLM 抽象，以及用于开始构建常见应用的高层接口；然而，它缺少一个允许开发者控制智能体确切流程的底层编排层。于是有了：LangGraph。

    在构建 LangGraph 时，我们吸取了构建 LangChain 的经验教训，并添加了我们发现所需的功能：流式传输、持久化执行、短期记忆、人类介入（human-in-the-loop）等。

#### 2024-06

### 2024-06

    **LangChain 拥有超过 700 个集成。**

    集成被从核心 LangChain 包中拆分出来，要么移入它们自己的独立包（用于核心集成），要么移入 `@langchain/community`。

#### 2024-10

### 2024-10

    LangGraph 成为构建任何不止一次 LLM 调用的 AI 应用的首选方式。

    随着开发者尝试提高其应用的可靠性，他们需要比高层接口所提供的更多控制。LangGraph 提供了这种底层灵活性。LangChain 中的大多数链和智能体被标记为已弃用，并附有关于如何将它们迁移到 LangGraph 的指南。LangGraph 中仍然创建了一个高层抽象：智能体抽象。它构建在底层 LangGraph 之上，并与 LangChain 中 ReAct 智能体具有相同的接口。

#### 2025-04

### 2025-04

    模型 API 变得更加多模态。

    模型开始接受文件、图像、视频等。我们相应地更新了 `@langchain/core` 消息格式，允许开发者以标准方式指定这些多模态输入。

#### 2025-10-20（v1.0.0）

### 2025-10-20（v1.0.0）

    **LangChain 发布 1.0**，带来两项重大变化：

    1. 完全重构了 `langchain` 中所有的链和智能体。所有链和智能体现在都被替换为唯一一个高层抽象：构建在 LangGraph 之上的智能体抽象。这正是最初在 LangGraph 中创建、但现在只是移入 LangChain 的高层抽象。

       对于仍在使用旧版 LangChain 链/智能体、且不打算升级的用户（注意：我们建议你升级），你可以通过安装 `@langchain/classic` 包继续使用旧版 LangChain。

    2. 标准化的消息内容格式：模型 API 从返回带有简单内容字符串的消息，演进为更复杂的输出类型——推理块、引用、服务端工具调用等。LangChain 演进其消息格式，以跨提供商标准化这些内容。

#### 2026-03-15（v0.5.3）

### 2026-03-15（v0.5.3）

    **Deep Agents 发布**，作为构建在 LangGraph 之上的开源智能体运行框架（agent harness）。

    虽然 LangChain 为自定义智能体架构提供灵活的构建块，但 [Deep Agents](/oss/javascript/langchain/overview) 为复杂、长时间运行的任务（如研究和编码）提供了一个“开箱即用”的选项。它添加了内置的规划工具、带有可插拔后端（内存、磁盘、LangGraph store、沙箱）的虚拟文件系统，以及用于上下文隔离的子智能体生成。当你需要更自主、带有预定义工具的智能体时使用 Deep Agents；当你需要完全控制智能体架构时使用 LangChain。

***
