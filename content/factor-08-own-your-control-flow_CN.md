[← 返回 README](../README_CN.md)

### 8. 拥有你的控制流

如果你拥有你的控制流，你可以做很多有趣的事情。

![180-control-flow](../img/180-control-flow.png)


构建对你特定用例有意义的控制结构。具体来说，某些类型的工具调用可能是中断循环并等待来自人类或其他长时间运行任务（如训练管道）响应的原因。你可能还想合并自定义实现：

- 工具调用结果的总结或缓存
- LLM 作为结构化输出的判断者
- 上下文窗口压缩或其他[内存管理](factor-03-own-your-context-window_CN.md)
- 日志记录、跟踪和指标
- 客户端速率限制
- 持久化睡眠/暂停/"等待事件"


下面的示例显示了三种可能的控制流模式：

- request_clarification：模型要求更多信息，中断循环并等待来自人类的响应
- fetch_git_tags：模型要求 git 标签列表，获取标签，附加到上下文窗口，并直接传回模型
- deploy_backend：模型要求部署后端，这是一个高风险的事情，所以中断循环并等待人工批准

```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))
    
    # 为清晰起见内联 - 实际上你可以把它放在方法中，使用异常进行控制流，或者任何你想要的
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # 异步步骤 - 中断循环，我们稍后会收到 webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # 同步步骤 - 将新的上下文传递给 LLM 以确定下一个下一步
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # 异步步骤 - 中断循环，我们稍后会收到 webhook
      break
```

这种模式允许你根据需要中断和恢复代理的流程，创建更自然的对话和工作流。

**示例** - 我对每个 AI 框架的首要功能请求是我们需要能够中断工作代理并在以后恢复，特别是在工具**选择**时刻和工具**调用**时刻之间。

没有这种级别的可恢复性/粒度，就无法在工具调用运行之前审查/批准它，这意味着你被迫：

1. 在内存中暂停任务等待长时间运行的事情完成（想想 `while...sleep`），如果进程中断则从头重新开始
2. 将代理限制为仅低风险、低风险的调用，如研究和总结
3. 给予代理访问做更大、更有用的事情的能力，只是希望它不会搞砸


你可能注意到这与[因素 5 - 统一执行状态和业务状态](factor-05-unify-execution-state_CN.md)和[因素 6 - 使用简单的 API 启动/暂停/恢复](factor-06-launch-pause-resume_CN.md)密切相关，但可以独立实现。

[← 通过工具调用联系人类](factor-07-contact-humans-with-tools_CN.md) | [将错误压缩到上下文窗口 →](factor-09-compact-errors_CN.md)