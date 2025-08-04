[← 返回 README](../README_CN.md)

### 2. 拥有你的提示词

不要将你的提示词工程外包给框架。

![120-own-your-prompts](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)

顺便说一句，[这远不是新颖的建议：](https://hamel.dev/blog/posts/prompt/)

![image](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

一些框架提供这样的"黑盒"方法：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

这对于引入一些顶级的提示词工程来让你开始很有帮助，但通常很难调整和/或逆向工程以将正确的令牌放入模型中。

相反，拥有你的提示词并将它们视为一流的代码：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}
    
    你是一个有帮助的助手，管理前端和后端系统的部署。
    你勤奋工作，通过遵循最佳实践和适当的部署程序确保安全成功的部署。
    
    在部署任何系统之前，你应该检查：
    - 部署环境（暂存 vs 生产）
    - 要部署的正确标签/版本
    - 当前系统状态
    
    你可以使用 deploy_backend、deploy_frontend 和 check_deployment_status 等工具
    来管理部署。对于敏感部署，使用 request_approval 获取
    人工验证。
    
    始终先思考要做什么，比如：
    - 检查当前部署状态
    - 验证部署标签是否存在
    - 如果需要请求批准
    - 在生产之前部署到暂存环境
    - 监控部署进度
    
    {{ _.role("user") }}

    {{ thread }}
    
    下一步应该是什么？
  "#
}
```

（上面的示例使用 [BAML](https://github.com/boundaryml/baml) 生成提示，但你可以使用任何你想要的提示词工程工具来做到这一点，甚至只是手动模板化）

如果签名看起来有点有趣，我们将在[因素 4 - 工具只是结构化输出](factor-04-tools-are-structured-outputs_CN.md) 中讨论

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

拥有你的提示词的关键好处：

1. **完全控制**：准确编写你的代理需要的指令，没有黑盒抽象
2. **测试和评估**：为你的提示词构建测试和评估，就像你对任何其他代码所做的那样
3. **迭代**：根据实际性能快速修改提示
4. **透明度**：确切知道你的代理正在使用什么指令
5. **角色黑客**：利用支持用户/助手角色非标准使用的 API - 例如，现在已弃用的 OpenAI "completions" API 的非聊天风格。这包括一些所谓的"模型煤气灯"技术

记住：你的提示词是你的应用程序逻辑和 LLM 之间的主要接口。

完全控制你的提示词为你提供了生产级代理所需的灵活性和提示控制。

我不知道什么是最好的提示，但我知道你想要能够尝试一切的灵活性。

[← 自然语言到工具调用](factor-01-natural-language-to-tool-calls_CN.md) | [拥有你的上下文窗口 →](factor-03-own-your-context-window_CN.md)