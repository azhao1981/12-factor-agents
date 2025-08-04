[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README_CN.md)

## 更长版本：我们是如何走到今天的

### 你不必听我的

无论你是代理的新手还是像我这样的固执老手，我都会试图说服你抛弃大部分关于 AI Agents 的想法，退后一步，从第一原理重新思考它们。（如果你没有注意到几周前的 OpenAI 响应发布，剧透警告：将更多代理逻辑推到 API 后面并不是答案）

## 代理是软件，及其简史

让我们谈谈我们是如何走到今天的

### 60 年前

我们将要大量讨论有向图（DGs）和它们的无环朋友，DAGs。我首先要指出的是...嗯...软件是一个有向图。这就是为什么我们过去用流程图来表示程序。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 年前

大约 20 年前，我们开始看到 DAG 编排器变得流行。我们谈论的是像 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/) 这样经典的，还有一些前辈，以及一些更新的比如 ([dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/))。它们遵循相同的图模式，增加了可观察性、模块化、重试、管理等优势。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 年前

当 ML 模型开始变得足够好用时，我们开始看到 DAG 中加入了 ML 模型。你可以想象像"将此列中的文本总结到新列中"或"按严重性或情绪分类支持问题"这样的步骤。

![020-dags-with-ml](https://github.com/humanlayer/12-factor-agents/blob/main/img/020-dags-with-ml.png)

但归根结底，它仍然大部分是相同的旧确定性软件。

### 代理的承诺

我不是第一个[这么说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但当我开始学习代理时，我最大的收获是你可以抛弃 DAG。你不再需要软件工程师编写每个步骤和边缘情况，你可以给代理一个目标和一组转换：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

让 LLM 实时决策来找出路径

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是你写的软件更少，你只需给 LLM 图的"边"，让它找出节点。你可以从错误中恢复，你可以写更少的代码，你可能会发现 LLM 找到解决问题的新方法。

### 作为循环的代理

换句话说，你有这个包含 3 个步骤的循环：

1. LLM 确定工作流程中的下一步，输出结构化 json（"工具调用"）
2. 确定性代码执行工具调用
3. 结果附加到上下文窗口
4. 重复直到下一步确定为"完成"

```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

我们的初始上下文只是起始事件（可能是用户消息、可能是 cron 触发、可能是 webhook 等），
我们要求 llm 选择下一步（工具）或确定我们已完成。

这是一个多步骤示例：

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF 版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)]

</details>

生成的"具体化"DAG 看起来像这样：

![027-agent-loop-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### 这种"循环直到解决"模式的问题

这种模式的最大问题：

- 当上下文窗口变得太长时代理会迷失方向 - 它们会一遍又一遍地尝试相同的错误方法
- 字面上就是这样，但这足以使这种方法失去作用

即使你没有手动构建代理，你可能在使用代理编码工具时也看到过这个长上下文问题。它们一会儿就会迷失方向，你需要开始新的聊天。

我甚至可能提出一个我经常听到的说法，而你可能已经对此形成了自己的直觉：

> ### **即使模型支持越来越长的上下文窗口，使用小而专注的提示和上下文你总是会获得更好的结果**

我交谈过的大多数构建者在意识到任何超过 10-20 轮的对话都会变成 LLM 无法恢复的混乱时，都**将"工具调用循环"想法推到了一边**。即使代理 90% 的时间都是正确的，这与"足够好到可以交给客户"还有很长的距离。你能想象一个在 10% 的页面加载时崩溃的网络应用吗？

**2025-06-09 更新** - 我很喜欢 [@swyx](https://x.com/swyx/status/1932125643384455237) 的这个说法：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="Screenshot 2025-07-02 at 11 50 50 AM" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### 什么真正有效 - 微代理

我在实际中确实看到的一件事是将代理模式融入到更广泛的确定性 DAG 中。

![micro-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

你可能会问 - "在这种情况下为什么还要使用代理？" - 我们会很快讨论这个问题，但基本上，让语言模型管理范围明确的任务集使得整合实时人类反馈变得容易，将其转换为工作流程步骤而不会陷入上下文错误循环。（[因素 1](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)，[因素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) [因素 7](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)）。

> #### 让语言模型管理范围明确的任务集使得整合实时人类反馈变得容易...而不会陷入上下文错误循环

### 真实的微代理示例

这是一个确定性代码如何运行一个负责处理部署中人工循环步骤的微代理的示例。

![029-deploybot-high-level](https://github.com/humanlayer/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **人类** 将 PR 合并到 GitHub main 分支
* **确定性代码** 部署到暂存环境
* **确定性代码** 对暂存环境运行端到端（e2e）测试
* **确定性代码** 交给代理进行生产部署，初始上下文："将 SHA 4af9ec0 部署到生产环境"
* **代理** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **确定性代码** 请求人类批准此操作
* **人类** 拒绝操作并反馈"你能先部署后端吗？"
* **代理** 调用 `deploy_backend_to_prod(4af9ec0)`
* **确定性代码** 请求人类批准此操作
* **人类** 批准操作
* **确定性代码** 执行后端部署
* **代理** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **确定性代码** 请求人类批准此操作
* **人类** 批准操作
* **确定性代码** 执行前端部署
* **代理** 确定任务成功完成，我们完成了！
* **确定性代码** 对生产环境运行端到端测试
* **确定性代码** 任务完成，或传递给回滚代理以检查失败并可能回滚

[![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif">GIF 版本</a></summary>

![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)]

</details>

这个示例基于我们实际发布的[用于管理 HumanLayer 部署的 OSS 代理](https://github.com/got-agents/agents/tree/main/deploybot-ts) - 这是我上周与它的真实对话：

![035-deploybot-conversation](https://github.com/humanlayer/12-factor-agents/blob/main/img/035-deploybot-conversation.png)

我们没有给这个代理大量的工具或任务。LLM 的主要价值是解析人类的纯文本反馈并提出更新的行动方案。我们尽可能隔离任务和上下文，以保持 LLM 专注于小型的 5-10 步工作流程。

这是另一个[更经典的支持/聊天机器人演示](https://x.com/chainlit_io/status/1858613325921480922)。

### 那么代理到底是什么？

- **提示** - 告诉 LLM 如何行为，以及它有哪些可用的"工具"。提示的输出是一个描述工作流程中下一步的 JSON 对象（"工具调用"或"函数调用"）。（[因素 2](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)）
- **switch 语句** - 基于 LLM 返回的 JSON，决定如何处理它。（[因素 8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 的一部分）
- **累积的上下文** - 存储已发生的步骤及其结果列表（[因素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)）
- **for 循环** - 直到 LLM 发出某种"终端"工具调用（或纯文本响应），将 switch 语句的结果添加到上下文窗口并要求 LLM 选择下一步。（[因素 8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)）

![040-4-components](https://github.com/humanlayer/12-factor-agents/blob/main/img/040-4-components.png)

在"deploybot"示例中，我们从拥有控制流和上下文累积中获得了一些好处：

- 在我们的 **switch 语句** 和 **for 循环** 中，我们可以劫持控制流以暂停等待人类输入或等待长时间运行的任务完成
- 我们可以轻松地序列化 **上下文** 窗口以进行暂停+恢复
- 在我们的 **提示** 中，我们可以优化如何向 LLM 传递指令和"到目前为止发生了什么"

[第二部分](https://github.com/humanlayer/12-factor-agents/blob/main/README_CN.md#12-个因素) 将**使这些模式正式化**，以便它们可以应用于为任何软件项目添加令人印象深刻的 AI 功能，而无需完全投入"AI agent"的传统实现/定义。

[因素 1 - 自然语言到工具调用 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)