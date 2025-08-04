# 12-Factor Agents - 构建可靠 LLM 应用的原则

<div align="center">
<a href="https://www.apache.org/licenses/LICENSE-2.0">
        <img src="https://img.shields.io/badge/Code-Apache%202.0-blue.svg" alt="代码许可证: Apache 2.0"></a>
<a href="https://creativecommons.org/licenses/by-sa/4.0/">
        <img src="https://img.shields.io/badge/Content-CC%20BY--SA%204.0-lightgrey.svg" alt="内容许可证: CC BY-SA 4.0"></a>
<a href="https://humanlayer.dev/discord">
    <img src="https://img.shields.io/badge/chat-discord-5865F2" alt="Discord 服务器"></a>
<a href="https://www.youtube.com/watch?v=8kMaTybvDUw">
    <img src="https://img.shields.io/badge/aidotengineer-conf_talk_(17m)-white" alt="YouTube
深度解析"></a>
<a href="https://www.youtube.com/watch?v=yxJDyQ8v6P0">
    <img src="https://img.shields.io/badge/youtube-deep_dive-crimson" alt="YouTube
深度解析"></a>
    
</div>

<p></p>

*秉承 [12 Factor Apps](https://12factor.net/) 的精神*。*本项目的源代码公开在 https://github.com/humanlayer/12-factor-agents，我欢迎您的反馈和贡献。让我们一起探索这个问题！*

> [!TIP]
> 错过了 AI Engineer World's Fair？[在此观看演讲](https://www.youtube.com/watch?v=8kMaTybvDUw)
>
> 寻找上下文工程？[直接跳转到因素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
>
> 想为 `npx/uvx create-12-factor-agent` 做贡献 - 查看[讨论线程](https://github.com/humanlayer/12-factor-agents/discussions/61)


<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=2acad99a-c2d9-48df-86f5-9ca8061b7bf9" />

<a href="#visual-nav"><img width="907" alt="Screenshot 2025-04-03 at 2 49 07 PM" src="https://github.com/user-attachments/assets/23286ad8-7bef-4902-b371-88ff6a22e998" /></a>


你好，我是 Dex。我在[AI agents](https://theouterloop.substack.com)领域已经[钻研](https://youtu.be/8bIHcttkOTE)了[一段时间](https://humanlayer.dev)。


**我尝试了市面上的所有代理框架**，从即插即用的 crew/langchains 到"极简主义"的 smolagents，再到"生产级"的 langraph、griptape 等。

**我与很多非常优秀的创始人交流过**，无论是否来自 YC，他们都在用 AI 构建令人印象深刻的东西。他们中的大多数都在自己构建技术栈。我在生产环境面向客户的代理中看到的框架并不多。

**我惊讶地发现**，大多数标榜为"AI Agents"的产品实际上并不那么具有代理性。它们大多是确定性代码，只是在恰到好处的地方加入了 LLM 步骤，使体验真正神奇。

好的代理，至少是好的代理，并不遵循["这是你的提示，这是一袋工具，循环直到达到目标"](https://www.anthropic.com/engineering/building-effective-agents#agents)的模式。相反，它们主要由软件组成。

因此，我开始探索回答这个问题：

> ### **我们可以使用什么原则来构建真正足够好、能够交给生产客户的 LLM 驱动软件？**

欢迎来到 12-factor agents。正如自 Daley 以来每位芝加哥市长一直在城市主要机场上张贴的那样，我们很高兴您来到这里。

*特别感谢 [@iantbutler01](https://github.com/iantbutler01)、[@tnm](https://github.com/tnm)、[@hellovai](https://www.github.com/hellovai)、[@stantonk](https://www.github.com/stantonk)、[@balanceiskey](https://www.github.com/balanceiskey)、[@AdjectiveAllison](https://www.github.com/AdjectiveAllison)、[@pfbyjy](https://www.github.com/pfbyjy)、[@a-churchill](https://www.github.com/a-churchill) 和 SF MLOps 社区对本指南的早期反馈。*

## 简短版本：12 个因素

即使 LLM [继续呈指数级变得更强大](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)，仍然有一些核心技术可以使 LLM 驱动的软件更可靠、更可扩展、更易于维护。

- [我们是如何走到今天的：软件简史](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [因素 1：自然语言到工具调用](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [因素 2：拥有你的提示](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [因素 3：拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [因素 4：工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [因素 5：统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [因素 6：使用简单的 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [因素 7：通过工具调用联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [因素 8：拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [因素 9：将错误压缩到上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [因素 10：小型专注的代理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [因素 11：从任何地方触发，在用户所在的地方与他们见面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [因素 12：使你的代理成为无状态归约器](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)

### 可视化导航

|    |    |    |
|----|----|-----|
|[![因素 1](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [![因素 2](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [![因素 3](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) |
|[![因素 4](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [![因素 5](https://github.com/humanlayer/12-factor-agents/blob/main/img/150-unify-state.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [![因素 6](https://github.com/humanlayer/12-factor-agents/blob/main/img/160-pause-resume-with-simple-apis.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) |
| [![因素 7](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [![因素 8](https://github.com/humanlayer/12-factor-agents/blob/main/img/180-control-flow.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [![因素 9](https://github.com/humanlayer/12-factor-agents/blob/main/img/190-factor-9-errors-static.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md) |
| [![因素 10](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [![因素 11](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md) | [![因素 12](https://github.com/humanlayer/12-factor-agents/blob/main/img/1c0-stateless-reducer.png)](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) |

## 我们是如何走到今天的

关于我的代理之旅以及我们是如何走到今天的更深入探讨，请查看[软件简史](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md) - 这里是简要总结：

### 代理的承诺

我们将要大量讨论有向图（DGs）和它们的无环朋友，DAGs。我首先要指出的是...嗯...软件是一个有向图。这就是为什么我们过去用流程图来表示程序。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 从代码到 DAG

大约 20 年前，我们开始看到 DAG 编排器变得流行。我们谈论的是像 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/) 这样经典的，还有一些前辈，以及一些更新的比如 ([dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/))。它们遵循相同的图模式，增加了可观察性、模块化、重试、管理等优势。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 代理的承诺

我不是第一个[这么说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但当我开始学习代理时，我最大的收获是你可以抛弃 DAG。你不再需要软件工程师编写每个步骤和边缘情况，你可以给代理一个目标和一组转换：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

让 LLM 实时决策来找出路径

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是你写的软件更少，你只需给 LLM 图的"边"，让它找出节点。你可以从错误中恢复，你可以写更少的代码，你可能会发现 LLM 找到解决问题的新方法。

### 作为循环的代理

正如我们稍后将看到的，这种方法并不完全有效。

让我们深入一步 - 使用代理，你有这个包含 3 个步骤的循环：

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

我们的初始上下文只是起始事件（可能是用户消息、可能是 cron 触发、可能是 webhook 等），我们要求 llm 选择下一步（工具）或确定我们已完成。

这是一个多步骤示例：

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF 版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)]

</details>

## 为什么是 12-factor agents？

归根结底，这种方法并不如我们期望的那样有效。

在构建 HumanLayer 的过程中，我与至少 100 个 SaaS 构建者（主要是技术创始人）交流，他们希望使其现有产品更具代理性。旅程通常是这样的：

1. 决定要构建一个代理
2. 产品设计、UX 映射、要解决的问题
3. 希望快速行动，所以抓取 $FRAMEWORK 并*开始构建*
4. 达到 70-80% 的质量标准
5. 意识到 80% 对大多数面向客户的功能来说不够好
6. 意识到要超越 80% 需要逆向工程框架、提示、流程等
7. 从头开始

<details>
<summary>随机免责声明</summary>

**免责声明**：我不确定在这里说的确切位置是否合适，但这里似乎和其他地方一样好：**这绝不是对众多框架或工作的聪明人的贬低**。它们实现了令人难以置信的事情，并加速了 AI 生态系统。

我希望这篇文章的一个成果是代理框架构建者可以从我自己和其他人的旅程中学习，并使框架变得更好。

特别是对于希望快速行动但需要深度控制的构建者。

**免责声明 2**：我不会谈论 MCP。我相信你能看到它适合的位置。

**免责声明 3**：我主要使用 typescript，有[原因](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30)，但所有这些内容在 python 或你喜欢的任何其他语言中都有效。

无论如何回到正题...

</details>

### 优秀 LLM 应用的设计模式

在研究了数百个 AI 库并与数十位创始人合作后，我的直觉是：

1. 有一些核心的东西使代理变得优秀
2. 全力投入框架并构建本质上是从头开始的 rewriting 可能会适得其反
3. 有一些核心原则使代理变得优秀，如果你引入框架，你会获得其中的大部分/全部
4. **但是**，我见过的构建者将高质量 AI 软件交到客户手中的最快方法是采用代理构建中的小型模块化概念，并将它们整合到他们现有的产品中
5. 这些来自代理的模块化概念可以被大多数熟练的软件工程师定义和应用，即使他们没有 AI 背景

> #### 我见过的构建者将优秀 AI 软件交到客户手中的最快方法是采用代理构建中的小型模块化概念，并将它们整合到他们现有的产品中

## 12 个因素（再次）

- [我们是如何走到今天的：软件简史](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [因素 1：自然语言到工具调用](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [因素 2：拥有你的提示](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [因素 3：拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [因素 4：工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [因素 5：统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [因素 6：使用简单的 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [因素 7：通过工具调用联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [因素 8：拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [因素 9：将错误压缩到上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [因素 10：小型专注的代理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [因素 11：从任何地方触发，在用户所在的地方与他们见面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [因素 12：使你的代理成为无状态归约器](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)

## 荣誉提及 / 其他建议

- [因素 13：预取你可能需要的所有上下文](https://github.com/humanlayer/12-factor-agents/blob/main/content/appendix-13-pre-fetch.md)

## 相关资源

- 在此为指南做贡献 [here](https://github.com/humanlayer/12-factor-agents)
- [我在 2025 年 3 月的 Tool Use 播客的一集中谈到了很多这个内容](https://youtu.be/8bIHcttkOTE)
- 我在 [The Outer Loop](https://theouterloop.substack.com) 写一些这方面的内容
- 我与 [@hellovai](https://github.com/hellovai) 一起做[关于最大化 LLM 性能的网络研讨会](https://github.com/hellovai/ai-that-works/tree/main)
- 我们在 [got-agents/agents](https://github.com/got-agents/agents) 下使用这种方法构建 OSS 代理
- 我们忽略了自己的所有建议，构建了一个[在 kubernetes 中运行分布式代理的框架](https://github.com/humanlayer/kubechain)
- 本指南中的其他链接：
  - [12 Factor Apps](https://12factor.net)
  - [构建有效的代理 (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [提示是函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/ )
  - [库模式：为什么框架是邪恶的](https://tomasp.net/blog/2015/library-frameworks/)
  - [错误的抽象](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew 演示视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit 演示](https://x.com/chainlit_io/status/1858613325921480922)
  - [TypeScript for LLMs](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [模式对齐解析](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [函数调用 vs 结构化输出 vs JSON 模式](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [GitHub 上的 BAML](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs 函数调用](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [外循环代理](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [AI 代理指数 (MIT)](https://aiagentindex.mit.edu/)
  - [NotebookLM 关于发现模型能力边界](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

## 贡献者

感谢所有为 12-factor agents 做出贡献的人！

[<img src="https://avatars.githubusercontent.com/u/3730605?v=4&s=80" width="80px" alt="dexhorthy" />](https://github.com/dexhorthy) [<img src="https://avatars.githubusercontent.com/u/50557586?v=4&s=80" width="80px" alt="Sypherd" />](https://github.com/Sypherd) [<img src="https://avatars.githubusercontent.com/u/66259401?v=4&s=80" width="80px" alt="tofaramususa" />](https://github.com/tofaramususa) [<img src="https://avatars.githubusercontent.com/u/18105223?v=4&s=80" width="80px" alt="a-churchill" />](https://github.com/a-churchill) [<img src="https://avatars.githubusercontent.com/u/4084885?v=4&s=80" width="80px" alt="Elijas" />](https://github.com/Elijas) [<img src="https://avatars.githubusercontent.com/u/39267118?v=4&s=80" width="80px" alt="hugolmn" />](https://github.com/hugolmn) [<img src="https://avatars.githubusercontent.com/u/1882972?v=4&s=80" width="80px" alt="jeremypeters" />](https://github.com/jeremypeters)

[<img src="https://avatars.githubusercontent.com/u/380402?v=4&s=80" width="80px" alt="kndl" />](https://github.com/kndl) [<img src="https://avatars.githubusercontent.com/u/16674643?v=4&s=80" width="80px" alt="maciejkos" />](https://github.com/maciejkos) [<img src="https://avatars.githubusercontent.com/u/85041180?v=4&s=80" width="80px" alt="pfbyjy" />](https://github.com/pfbyjy) [<img src="https://avatars.githubusercontent.com/u/36044389?v=4&s=80" width="80px" alt="0xRaduan" />](https://github.com/0xRaduan) [<img src="https://avatars.githubusercontent.com/u/7169731?v=4&s=80" width="80px" alt="zyuanlim" />](https://github.com/zyuanlim) [<img src="https://avatars.githubusercontent.com/u/15862501?v=4&s=80" width="80px" alt="lombardo-chcg" />](https://github.com/lombardo-chcg) [<img src="https://avatars.githubusercontent.com/u/160066852?v=4&s=80" width="80px" alt="sahanatvessel" />](https://github.com/sahanatvessel)

## 版本

这是 12-factor agents 的当前版本，版本 1.0。在 [v1.1 分支](https://github.com/humanlayer/12-factor-agents/tree/v1.1) 上有版本 1.1 的草稿。有一些[问题来跟踪 v1.1 的工作](https://github.com/humanlayer/12-factor-agents/issues?q=is%3Aissue%20state%3Aopen%20label%3Aversion%3A%3A1.1)。

## 许可证

所有内容和图像都根据 <a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0 许可证</a> 授权

代码根据 <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 许可证</a> 授权