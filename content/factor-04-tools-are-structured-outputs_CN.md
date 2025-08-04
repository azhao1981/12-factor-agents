[← 返回 README](../README_CN.md)

### 4. 工具只是结构化输出

工具不需要很复杂。从本质上讲，它们只是来自 LLM 的结构化输出，触发确定性代码执行。

![140-tools-are-just-structured-outputs](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)

例如，假设你有两个工具 `CreateIssue` 和 `SearchIssues`。要求 LLM"使用几个工具中的一个"就是要求它输出我们可以解析为代表这些工具的对象的 JSON。

```python

class Issue:
  title: str
  description: str
  team_id: str
  assignee_id: str

class CreateIssue:
  intent: "create_issue"
  issue: Issue

class SearchIssues:
  intent: "search_issues"
  query: str
  what_youre_looking_for: str
```

这个模式很简单：
1. LLM 输出结构化 JSON
2. 确定性代码执行适当的操作（比如调用外部 API）
3. 结果被捕获并反馈到上下文中

这在 LLM 的决策和你的应用程序操作之间创建了清晰的分离。LLM 决定做什么，但你的代码控制如何做。仅仅因为 LLM"调用了工具"并不意味着你每次都必须以相同的方式去执行特定的相应函数。

如果你记得我们上面的 switch 语句

```python
if nextStep.intent == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return # 或者你想要的任何东西，见下文
elif nextStep.intent == 'wait_for_a_while': 
    # 做一些 monadic 的事情，我不知道
else: #... 模型没有调用我们知道的工具
    # 做其他事情
```

**注意**：关于"纯提示"与"工具调用"与"JSON 模式"的好处以及每种方法的性能权衡已经有很多讨论。我们很快会链接一些相关资源，但这里不会深入讨论。参见[提示 vs JSON 模式 vs 函数调用 vs 约束生成 vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing)，[我什么时候应该使用函数调用、结构化输出或 JSON 模式？](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead) 和 [OpenAI JSON vs 函数调用](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)。

"下一步"可能不像"运行纯函数并返回结果"那样原子化。当你将"工具调用"视为模型输出描述确定性代码应该做什么的 JSON 时，你就解锁了很多灵活性。将这一点与[因素 8 拥有你的控制流](factor-08-own-your-control-flow_CN.md)结合起来。

[← 拥有你的上下文窗口](factor-03-own-your-context-window_CN.md) | [统一执行状态 →](factor-05-unify-execution-state_CN.md)