[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README_CN.md)

### 7. 通过工具调用联系人类

默认情况下，LLM API 依赖于一个基本的高风险令牌选择：我们是返回纯文本内容，还是返回结构化数据？

![170-contact-humans-with-tools](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

你给第一个令牌的选择带来了很大的权重，在 `the weather in tokyo` 的情况下，它是

> "the"

但在 `fetch_weather` 的情况下，它是某个特殊令牌来表示 JSON 对象的开始。

> |JSON>

通过让 LLM*总是*输出 json，然后使用一些自然语言令牌如 `request_human_input` 或 `done_for_now`（而不是像 `check_weather_in_city` 这样的"正确"工具）来声明其意图，你可能会获得更好的结果。

同样，你可能不会从中获得任何性能提升，但你应该实验，并确保你可以自由地尝试奇怪的东西来获得最佳结果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# 人类交互的工具定义
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# 在代理循环中的示例用法
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # 中断循环并等待响应与线程 ID 一起返回
else:
  # ... 其他情况
```

稍后，你可能会收到来自处理 slack、电子邮件、短信或其他事件的系统的 webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... 为简洁起见进行了简化，你可能不想在这里阻塞 web 工作线程
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - 循环或中断或任何你想要的

  return {"status": "ok"}
```

以上包括了来自[因素 5 - 统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state_CN.md)、[因素 8 - 拥有你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow_CN.md)、[因素 3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window_CN.md)和[因素 4 - 工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs_CN.md)等几个的模式。

如果我们使用来自[因素 3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window_CN.md)的 XML 格式，我们几次轮转后的上下文窗口可能看起来像这样：

```xml

（为简洁起见已截断）

<slack_message>
    From: @alex
    Channel: #deployments
    Text: 你能将后端 v1.2.3 部署到生产环境吗？
    Thread: []
</slack_message>

<request_human_input>
    intent: "request_human_input"
    question: "你想继续将 v1.2.3 部署到生产环境吗？"
    context: "这是一个生产部署，将影响实时用户。"
    options: {
        urgency: "high"
        format: "yes_no"
    }
</request_human_input>

<human_response>
    response: "是的，请继续"
    approved: true
    timestamp: "2024-03-15T10:30:00Z"
    user: "alex@company.com"
</human_response>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<deploy_backend_result>
    status: "success"
    message: "v1.2.3 部署到生产环境成功完成。"
    timestamp: "2024-03-15T10:30:00Z"
</deploy_backend_result>
```


好处：

1. **清晰的指令**：用于不同类型人类联系的工具允许 LLM 更具体
2. **内循环与外循环**：使代理工作流能够在传统的 chatGPT 风格界面之外，其中控制流和上下文初始化可能是 `Agent->Human` 而不是 `Human->Agent`（想想，由 cron 或事件启动的代理）
3. **多人访问**：可以通过结构化事件轻松跟踪和协调来自不同人类的输入
4. **多代理**：简单的抽象可以轻松扩展以支持 `Agent->Agent` 请求和响应
5. **持久性**：与[因素 6 - 使用简单的 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume_CN.md)结合，这使得持久、可靠和可检查的多玩家工作流成为可能

[关于外循环代理的更多信息在这里](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

![175-outer-loop-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

与[因素 11 - 从任何地方触发，在用户所在的地方与他们见面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)配合使用效果很好

[← 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume_CN.md) | [拥有你的控制流 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)