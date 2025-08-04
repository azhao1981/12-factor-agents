[← 返回 README](../README_CN.md)

### 9. 将错误压缩到上下文窗口

这个有点短但值得一提。代理的好处之一是"自愈" - 对于短任务，LLM 可能调用一个失败的工具。好的 LLM 有相当好的机会读取错误消息或堆栈跟踪并弄清楚在后续工具调用中要更改什么。


大多数框架都实现了这一点，但你可以只做这一点而不做其他 11 个因素。这是一个例子：

```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # 我们的 switch 语句
  except Exception as e:
    # 如果我们得到错误，我们可以将其添加到上下文窗口并重试
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # 循环，或在这里做任何其他事情来尝试恢复
```

你可能想为特定的工具调用实现一个 errorCounter，以限制单个工具的约 3 次尝试，或者对你的用例有意义的任何其他逻辑。

```python
consecutive_errors = 0

while True:

  # ... 现有代码 ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # 成功！重置错误计数器
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # 做循环并重试
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # 中断循环，重置上下文窗口的部分，升级给人类，或任何你想做的事情
      break
  }
}
```
达到某个连续错误阈值可能是[升级给人类](factor-07-contact-humans-with-tools_CN.md)的好地方，无论是通过模型决策还是通过确定性接管控制流。

[![195-factor-09-errors](../img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)


<details>
<summary>[GIF 版本](../img/195-factor-09-errors.gif)</summary>

![195-factor-09-errors](../img/195-factor-09-errors.gif)

</details>

好处：

1. **自愈**：LLM 可以读取错误消息并弄清楚在后续工具调用中要更改什么
2. **持久性**：即使一个工具调用失败，代理也可以继续运行

我相信你会发现如果你做得太多，你的代理会开始失控并可能一遍又一遍地重复同样的错误。

这就是[因素 8 - 拥有你的控制流](factor-08-own-your-control-flow_CN.md)和[因素 3 - 拥有你的上下文构建](factor-03-own-your-context-window_CN.md)的用武之地 - 你不需要只是将原始错误放回去，你可以完全重新结构化它的表示方式，从上下文窗口中删除先前的事件，或者任何你发现能让代理回到正轨的确定性事情。

但防止错误失控的第一种方法是拥抱[因素 10 - 小型专注的代理](factor-10-small-focused-agents_CN.md)。

[← 拥有你的控制流](factor-08-own-your-control-flow_CN.md) | [小型专注的代理 →](factor-10-small-focused-agents_CN.md)