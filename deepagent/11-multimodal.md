# 多模态输入与输出

> 原文：Multimodal inputs and outputs
> 来源：https://docs.langchain.com/oss/javascript/deepagents/multimodal

---

> 当你的模型支持多模态输入和工具结果时，使用图像、音频、视频和文档配合 Deep Agents。

Deep Agents 在你使用能够接收多模态输入和工具结果、或返回多模态输出的 [Large Language Model](/oss/javascript/integrations/chat) 时支持多模态工作流。你可以将图像和其他媒体附加到用户消息中，使用内置的 `read_file` 工具读取非文本文件，并从自定义工具返回多模态内容。

内置的[上下文压缩](/oss/javascript/deepagents/context-engineering#context-compression)主要面向文本。请据此规划多模态工作负载：将大型媒体存储在 backend 中，并尽量传递引用。

## 多模态用户输入

在与发送给 agent 的 `messages` 中传入多模态内容，使用与 LangChain chat models 相同的[标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)：

```ts
const result = await agent.invoke({
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "What is in this screenshot?" },
        { type: "image", url: "https://example.com/screenshot.png" },
      ],
    },
  ],
});
```

关于块类型、各提供方的特定要求以及更多示例（PDF、音频、视频），请参阅[多模态消息](/oss/javascript/langchain/messages#multimodal)。

## 内置 `read_file` 工具

harness 的 `read_file` 工具会为支持的多模态文件返回[标准内容块](/oss/javascript/langchain/messages#standard-content-blocks)，而不是纯文本。当所选模型支持相应模态时，agent 可以检查存储在其[文件系统](/oss/javascript/deepagents/overview#virtual-filesystem-access)中的图像、文档和媒体。请查阅提供方的文档，了解你的模型支持哪些 MIME 类型。

#### 支持的多模态文件扩展名

| 类型                                                   | 扩展名                                                                |
| ------------------------------------------------------ | ------------------------------------------------------------------------- |
| [图像](/oss/javascript/langchain/messages#multimodal) | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.heic`, `.heif`                |
| [视频](/oss/javascript/langchain/messages#multimodal) | `.mp4`, `.mpeg`, `.mov`, `.avi`, `.flv`, `.mpg`, `.webm`, `.wmv`, `.3gpp` |
| [音频](/oss/javascript/langchain/messages#multimodal) | `.wav`, `.mp3`, `.aiff`, `.aac`, `.ogg`, `.flac`                          |
| [文件](/oss/javascript/langchain/messages#multimodal) | `.pdf`, `.ppt`, `.pptx`                                                   |

## 自定义工具输出

[自定义工具](/oss/javascript/deepagents/tools#custom-tools)可以包含多模态文件，例如图像：

```ts
import { tool } from "langchain";
import { z } from "zod";

const captureScreenshot = tool(
  async () => [
    { type: "text", text: "Screenshot of the current page:" },
    { type: "image", url: "https://example.com/page.png" },
  ],
  {
    name: "capture_screenshot",
    description: "Capture a screenshot of the current page.",
    schema: z.object({}),
  },
);
```

返回值会被转换为模型在下一轮读取的 `ToolMessage`。通过结果消息上的 `content_blocks` 访问规范化后的表示。

> **提示**：当工具生成图像或其他大型二进制数据时，请将产物保存到 [backend](/oss/javascript/deepagents/backends) 中，并返回简洁的文本描述以及路径或 URL。这样可以保持消息历史更小，也能更好地配合[上下文压缩](/oss/javascript/deepagents/context-engineering#context-compression)。

## 上下文压缩与多模态内容

内置的卸载（offloading）和摘要（summarization）针对文本和消息历史进行了优化：

* **卸载（Offloading）**只统计文本 token。非文本块（包括图像）会在替换消息中保留，而不会被压缩。仅包含图像的消息不会仅根据图像大小被卸载。
* **摘要（Summarization）**将较早的消息压缩为纯文本摘要。该范围内的图像、音频、视频和文件块不会被向前保留——模型只能看到摘要器对它们所写的内容。低于 keep 阈值的近期消息保持不变。

  当摘要运行时，较早轮次中的媒体块会从活动上下文中移除：

  ```ts
  // Before — model receives image blocks in older turns
  void {
    role: "user",
    content: [
      { type: "text", text: "What trends do you see in this chart?" },
      { type: "image", url: "https://example.com/chart.png" },
    ],
  };
  void {
    role: "tool",
    content: [
      { type: "text", text: "Updated chart:" },
      { type: "image", url: "https://example.com/chart-v2.png" },
    ],
  };

  // After — those turns collapse to text; image blocks are gone
  void {
    content:
      "User asked about trends in a chart screenshot. " +
      "Tool returned an updated chart. Agent identified Q3 revenue growth.",
  };
  ```

  原始对话仍会以文本形式写入文件系统。有关触发条件、keep 阈值和完整流程，请参阅[摘要](/oss/javascript/deepagents/context-engineering#summarization)。

对于多模态繁重的工作负载：

* 将图像、截图和图表存储在文件系统 backend 或外部对象存储中，然后通过消息传递文件路径或 URL。
* 在长时间运行的对话中，优先使用引用而不是 base64 编码的图像块。
* 对于图像密集的检查，使用 [subagents](/oss/javascript/deepagents/subagents)，以便主 agent 接收到紧凑的文本结果。
* 当你的提供方对图像收取较多 token 时，调整摘要阈值或提供自定义 token 计数器。

请参阅[上下文压缩](/oss/javascript/deepagents/context-engineering#context-compression)，了解卸载阈值、摘要触发条件和自定义选项。
