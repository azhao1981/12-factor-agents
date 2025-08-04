[← 返回 README](../README_CN.md)

### 1. 自然语言到工具调用

代理构建中最常见的模式之一是将自然语言转换为结构化工具调用。这是一个强大的模式，允许你构建能够推理任务并执行任务的代理。

![110-natural-language-tool-calls](../img/110-natural-language-tool-calls.png)

当原子化应用时，这个模式是简单地将这样的短语

> 你能为 Terri 创建一个 750 美元的支付链接，用于赞助二月 AI 爱好者聚会吗？

转换为描述 Stripe API 调用的结构化对象，例如

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "Hey Jeff - see below for the payment link for the february ai tinkerers meetup"
    }
  }
}
```

**注意**：实际上 Stripe API 更复杂一些，[执行此操作的真实代理](https://github.com/dexhorthy/mailcrew)（[视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)）会列出客户、产品、价格等，以使用正确的 ID 构建此负载，或者将这些 ID 包含在提示词/上下文窗口中（我们将在下面看到这些实际上是一样的东西！）

从那里，确定性代码可以接收负载并对其进行处理。（更多内容见[因素 3](factor-03-own-your-context-window_CN.md)）

```python
# LLM 接收自然语言并返回结构化对象
nextStep = await llm.determineNextStep(
  """
  为 Jeff 创建一个 750 美元的支付链接
  用于赞助二月 AI 爱好者聚会
  """
  )

# 根据其函数处理结构化输出
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # 或者你想要的任何东西，见下文
elif nextStep.function == 'something_else':
    # ... 更多情况
    pass
else:  # 模型没有调用我们知道的工具
    # 做其他事情
    pass
```

**注意**：虽然完整的代理会接收 API 调用结果并循环处理，最终返回类似这样的内容

> 我已成功为 Terri 创建了一个 750 美元的支付链接，用于赞助二月 AI 爱好者聚会。链接是：https://buy.stripe.com/test_1234567890

**相反**，我们实际上将在这里跳过这一步，并将其保存到另一个因素，你可能也可能想要 incorporate（由你决定！）

[← 我们是如何走到今天的](brief-history-of-software_CN.md) | [拥有你的提示词 →](factor-02-own-your-prompts_CN.md)