[← 返回 README](../README_CN.md)

### 10. 小型专注的代理

而不是构建试图做所有事情的单体代理，而是构建小型专注的代理，做好一件事。代理只是更大的、大部分确定性系统中的一个构建块。

![1a0-small-focused-agents](../img/1a0-small-focused-agents.png)

这里的关键见解是关于 LLM 的限制：任务越大越复杂，需要的步骤就越多，这意味着更长的上下文窗口。随着上下文的增长，LLM 更容易迷失或失去焦点。通过保持代理专注于特定领域，最多 3-10，可能 20 个步骤，我们保持上下文窗口可管理且 LLM 性能高。

> #### 随着上下文的增长，LLM 更容易迷失或失去焦点

小型专注代理的好处：

1. **可管理的上下文**：较小的上下文窗口意味着更好的 LLM 性能
2. **清晰的职责**：每个代理都有明确定义的范围和目的
3. **更好的可靠性**：在复杂工作流中迷失的机会更少
4. **更轻松的测试**：更容易测试和验证特定功能
5. **改进的调试**：更容易在问题发生时识别和修复问题

### 如果 LLM 变得更聪明怎么办？

如果 LLM 变得足够智能来处理 100 步以上的工作流，我们还需要这个吗？

tl;dr 是的。随着代理和 LLM 的改进，它们**可能**自然会扩展到能够处理更长的上下文窗口。这意味着处理更大的 DAG 的更多部分。这种小型专注的方法确保你今天就能获得结果，同时为随着 LLM 上下文窗口变得更可靠而慢慢扩展代理范围做好准备。（如果你以前重构过大型确定性代码库，你现在可能会点头）。

[![gif](../img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/0cd3f52c-046e-4d5e-bab4-57657157c82f)

<details>
<summary><a href="../img/1a5-agent-scope-grow.gif">GIF 版本</a></summary>
![gif](../img/1a5-agent-scope-grow.gif)
</details>

有意识地考虑代理的大小/范围，并且只以允许你保持质量的方式增长，这是这里的关键。正如[构建 NotebookLM 的团队所说](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> 我觉得一致地，对我来说，AI 构建中最神奇的时刻出现在我真的、真的、真的非常接近模型能力边缘的时候

无论那个边界在哪里，如果你能找到那个边界并持续地正确，你将构建神奇的体验。这里有很多护城河可以建立，但像往常一样，它们需要一些工程严谨性。

[← 将错误压缩到上下文窗口](factor-09-compact-errors_CN.md) | [从任何地方触发 →](factor-11-trigger-from-anywhere_CN.md)