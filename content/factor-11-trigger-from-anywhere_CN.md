[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README_CN.md)

### 11. 从任何地方触发，在用户所在的地方与他们见面

如果你在等待 [humanlayer](https://humanlayer.dev) 的宣传，你做到了。如果你在做[因素 6 - 使用简单的 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume_CN.md)和[因素 7 - 通过工具调用联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools_CN.md)，你准备好整合这个因素。

![1b0-trigger-from-anywhere](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

使用户能够从 slack、电子邮件、短信或他们想要的任何其他渠道触发代理。使代理能够通过相同的渠道响应。

好处：

- **在用户所在的地方与他们见面**：这有助于你构建感觉像真实人类的 AI 应用程序，或者至少是数字同事
- **外循环代理**：使代理能够由非人类触发，例如事件、定时任务、中断等。它们可能工作 5、20、90 分钟，但当它们到达关键点时，它们可以联系人类寻求帮助、反馈或批准
- **高风险工具**：如果你能够快速联系各种人类，你可以给代理访问高风险操作的能力，如发送外部电子邮件、更新生产数据等。保持清晰标准让你获得代理的可审计性和信心，这些代理能够[执行更大更好的事情](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents_CN.md#if-llms-get-smarter)

[← 小型专注的代理](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents_CN.md) | [无状态归约器 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)