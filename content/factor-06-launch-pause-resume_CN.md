[← 返回 README](../README_CN.md)

### 6. 使用简单的 API 启动/暂停/恢复

代理只是程序，我们对如何启动、查询、恢复和停止它们有一些期望。

[![pause-resume animation](../img/165-pause-resume-animation.gif)](https://github.com/user-attachments/assets/feb1a425-cb96-4009-a133-8bd29480f21f)

<details>
<summary><a href="../img/165-pause-resume-animation.gif">GIF 版本</a></summary>

![pause-resume animation](../img/165-pause-resume-animation.gif)]

</details>


对于用户、应用程序、管道和其他代理来说，应该能够通过简单的 API 启动代理。

代理及其编排的确定性代码应该能够在需要长时间运行的操作时暂停代理。

像 webhook 这样的外部触发器应该能够让代理从停止的地方恢复，而无需与代理编排器深度集成。

与[因素 5 - 统一执行状态和业务状态](factor-05-unify-execution-state_CN.md)和[因素 8 - 拥有你的控制流](factor-08-own-your-control-flow_CN.md)密切相关，但可以独立实现。



**注意** - 通常 AI 编排器允许暂停和恢复，但不能在工具选择和工具执行之间进行。另见[因素 7 - 通过工具调用联系人类](factor-07-contact-humans-with-tools_CN.md)和[因素 11 - 从任何地方触发，在用户所在的地方与他们见面](factor-11-trigger-from-anywhere_CN.md)。

[← 统一执行状态](factor-05-unify-execution-state_CN.md) | [通过工具调用联系人类 →](factor-07-contact-humans-with-tools_CN.md)