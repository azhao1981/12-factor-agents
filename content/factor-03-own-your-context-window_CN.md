[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README_CN.md)

### 3. 拥有你的上下文窗口

你不需要 necessarily 使用标准的基于消息的格式来向 LLM 传递上下文。

> #### 在任何给定点，你在代理中对 LLM 的输入是"这是到目前为止发生的事情，下一步是什么"

<!-- todo syntax highlighting -->
<!-- ![130-own-your-context-building](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

一切都是上下文工程。[LLM 是无状态函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，将输入转换为输出。为了获得最佳输出，你需要给它们最佳输入。

创建优秀的上下文意味着：

- 你给模型的提示和指令
- 你检索的任何文档或外部数据（例如 RAG）
- 任何过去的状态、工具调用、结果或其他历史记录
- 来自相关但独立的历史/对话的任何过去消息或事件（记忆）
- 关于输出什么类型结构化数据的指令

![image](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)

### 关于上下文工程

本指南是关于充分利用当今模型的。值得注意的是没有提到的：

- 对模型参数的更改，如温度、top_p、frequency_penalty、presence_penalty 等
- 训练你自己的完成或嵌入模型
- 微调现有模型

再次，我不知道什么是向 LLM 提供上下文的最佳方式，但我知道你想要能够尝试一切的灵活性。

#### 标准与自定义上下文格式

大多数 LLM 客户端使用这样的标准基于消息的格式：

```yaml
[
  {
    "role": "system",
    "content": "你是一个有帮助的助手..."
  },
  {
    "role": "user",
    "content": "你能部署后端吗？"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

虽然这对大多数用例都很好用，但如果你想真正充分利用当今的 LLM，你需要以最令牌和注意力高效的方式将上下文传递给 LLM。

作为标准基于消息格式的替代方案，你可以构建自己的针对用例优化的上下文格式。例如，你可以使用自定义对象并将它们打包/扩展到一个或多个用户、系统、助手或工具消息中，只要合理即可。

这是将整个上下文窗口放入单个用户消息的示例：
```yaml

[
  {
    "role": "system",
    "content": "你是一个有帮助的助手..."
  },
  {
    "role": "user",
    "content": |
            这是到目前为止发生的一切：
        
        <slack_message>
            From: @alex
            Channel: #deployments
            Text: 你能部署后端吗？
        </slack_message>
        
        <list_git_tags>
            intent: "list_git_tags"
        </list_git_tags>
        
        <list_git_tags_result>
            tags:
              - name: "v1.2.3"
                commit: "abc123"
                date: "2024-03-15T10:00:00Z"
              - name: "v1.2.2"
                commit: "def456"
                date: "2024-03-14T15:30:00Z"
              - name: "v1.2.1"
                commit: "ghi789"
                date: "2024-03-13T09:15:00Z"
        </list_git_tags_result>
        
        下一步是什么？
    }
]
```

模型可能会通过你提供的工具模式推断你在问它"下一步是什么"，但将其滚动到你的提示模板中永远不会有害。

### 代码示例

我们可以用这样的东西构建：

```python

class Thread:
  events: List[Event]

class Event:
  # 可以只使用字符串，或者可以是显式的 - 由你决定
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |  
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

拥有你的上下文窗口的关键好处：

1. **信息密度**：以最大化 LLM 理解的方式构建信息
2. **错误处理**：以帮助 LLM 恢复的格式包含错误信息。考虑在错误和失败调用解决后将其从上下文窗口中隐藏。
3. **安全性**：控制传递给 LLM 的信息，过滤掉敏感数据
4. **灵活性**：随着你了解什么对你的用例最有效而调整格式
5. **令牌效率**：为令牌效率和 LLM 理解优化上下文格式

上下文包括：提示、指令、RAG 文档、历史记录、工具调用、记忆

记住：上下文窗口是你与 LLM 的主要接口。控制你构建和呈现信息的方式可以显著改善你的代理的性能。

[← 拥有你的提示](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts_CN.md) | [工具只是结构化输出 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)