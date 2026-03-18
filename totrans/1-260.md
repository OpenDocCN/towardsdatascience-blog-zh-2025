# 从零开始创建和部署 MCP 服务器

> 原文：[`towardsdatascience.com/creating-and-deploying-an-mcp-server-from-scratch/`](https://towardsdatascience.com/creating-and-deploying-an-mcp-server-from-scratch/)

## 简介

<mdspan datatext="el1758563591407" class="mdspan-comment">在 2025 年 9 月的一个周末</mdspan>，我参加了在巴黎由 Mistral 组织的一场黑客马拉松。所有团队都必须创建一个 MCP 服务器并将其集成到 Mistral 中。

虽然我的团队没有赢得任何东西，但这是一次极好的个人经历！此外，我以前从未创建过 MCP 服务器，因此这让我有机会直接接触新技术。

因此，我们创建了[**Prédictif**](https://www.youtube.com/watch?v=5tNjjvV4B6g)——一个允许直接在聊天中训练和测试机器学习模型，并在不同对话中持久保存数据集、结果和模型的 MCP 服务器。

![](img/0a19ec0301c0d8a105486ff6ca59f42c.png)

Prédictif 标志

> *鉴于我真的很喜欢这个活动，我决定更进一步，写这篇文章，为其他工程师提供一个简单的 MCP 介绍，并提供从零开始创建 MCP 服务器的指南*。
> 
> *如果你好奇，所有团队的解决方案都在这里[这里](https://cerebralvalley.ai/e/mistral-mcp-hackathon/hackathon/gallery)*。

## MCP

AI 代理和 MCP 服务器是相对较新的技术，目前在机器学习领域需求很高。

**MCP**代表“模型上下文协议”，最初于 2024 年由 Anthropic 开发，然后开源。创建 MCP 的动机是不同 LLM 供应商（OpenAI、Google、Mistral 等）为他们的 LLM 提供了不同的 API 来创建外部工具（连接器）。

因此，如果一个开发者为 OpenAI 创建了一个连接器，那么如果他们想要将其插入 Mistral 等，他们就必须进行另一个集成。这种方法不允许简单重用连接器。这就是 MCP 介入的地方。

使用 MCP，开发者可以创建一个工具，并在多个 MCP 兼容的 LLM（大型语言模型）之间重用它。这为开发者带来了更简单的流程，因为他们不再需要执行额外的集成。这与许多 LLM 兼容。

![](img/0030e2a08ebabde3eaaeb9efd66bd7c7.png)

两个图表说明了在没有 MCP（左侧）和 MCP 引入后（右侧）的开发者工作流程。正如我们所见，MCP 统一了集成过程；因此，相同的工具可以无差别地连接到多个供应商，而无需额外步骤。

> *为了信息，MCP 使用**JSON-RPC**协议*。

## 示例

### 第 1 步

我们将构建一个非常简单的 MCP 服务器，它将只有一个工具，其目标将是问候用户。为此，我们将使用 FastMCP——一个允许我们以 Pythonic 方式构建 MCP 服务器的库。

首先，我们需要设置环境：

```py
uv init hello-mcp
cd hello-mcp
```

添加一个 fastmcp 依赖项（这将更新 *pyproject.toml* 文件）：

```py
uv add fastmcp
```

创建一个 main.py 文件并将以下代码放在那里：

```py
from mcp.server.fastmcp import FastMCP
from pydantic import Field

mcp = FastMCP(
    name="Hello MCP Server",
    host="0.0.0.0",
    port=3000,
    stateless_http=True,
    debug=False,
)

@mcp.tool(
    title="Welcome a user",
    description="Return a friendly welcome message for the user.",
)
def welcome(
    name: str = Field(description="Name of the user")
) -> str:
    return f"Welcome {name} from this amazing application!"

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

太好了！现在 MCP 服务器已经完成，甚至可以本地部署：

```py
uv run python main.py
```

从现在开始，创建一个 GitHub 仓库并将本地项目目录推送到那里。

### 第 2 步

我们的自定义 MCP 服务器已经准备好了，但尚未部署。为了部署，我们将使用 [**Alpic**](https://alpic.ai) —— 一个允许我们在几个点击中部署 MCP 服务器的平台。为此，创建一个账户并登录到 Alpic。

在菜单中，选择一个选项来创建一个新的项目。Alpic 提议导入一个现有的 Git 仓库。如果你将 GitHub 账户连接到 Alpic，你应该能够看到可用于部署的可用仓库列表。选择与 MCP 服务器对应的选项，然后点击“导入”。

![图片](img/4720a29d987a0d3d935e92af970d5486.png)

在下一个窗口中，Alpic 提供了几个配置环境的选项。对于我们的例子，你可以保留这些选项的默认设置，然后点击 *“部署”*。

![图片](img/9a527eba4c5e794e304072905e67f44d.png)

之后，Alpic 将使用导入的仓库构建一个 Docker 容器。根据复杂度，部署可能需要一些时间。如果一切顺利，你将看到带有绿色圆圈的 *“已部署”* 状态。

![图片](img/c410d352f6feb2b897eeb322f76a8435.png)

在标签 *“域”* 下，有一个已部署服务器的 JSON-RPC 地址。现在先复制它，因为我们将在下一步连接它。

### 第 3 步

MCP 服务器已经构建。现在我们需要将其连接到 LLM 提供商，以便我们可以在对话中使用它。在我们的例子中，我们将使用 Mistral，但对于其他 LLM 提供商，连接过程应该是类似的。

在左侧菜单中，选择“连接器”选项，这将打开一个新窗口，显示可用的连接器。连接器使 LLM 能够连接到 MCP 服务器。例如，如果你向 Mistral 添加 GitHub 连接器，那么在聊天中，如果需要，LLM 将能够搜索你的仓库中的代码，以提供对给定提示的答案。

在我们的例子中，我们想要导入我们刚刚构建的自定义 MCP 服务器，因此我们点击“添加连接器”按钮。

![图片](img/f3750c8f5bc464b5ccb8a4be4ff7fa00.png)

在模态窗口中，导航到“自定义 MVP 连接器”并填写以下截图所示的信息。对于连接器服务器，使用第 2 步中部署的 MCP 服务器的 HTTPS 地址。

在连接器添加后，你可以在连接器菜单中看到它：

![图片](img/09878216e15ecf7802cb62dc50f25555.png)

如果你点击 MCP 连接器，在 *“函数”* 子窗口中，你将看到 MCP 服务器中实现的工具列表。在我们的例子中，我们只实现了一个工具 *“欢迎”*，因此这是我们在这里看到的唯一功能。

![图片](img/7c731c5a40c8f435fb76f25e1578f18d.png)

### 第 4 步

现在，返回聊天并点击*“启用工具”*按钮，这允许您指定 LLM 被允许使用的工具或 MCP 服务器。

![图片](img/90fb60bd37f5320b56b3198adeb5119c.png)

点击对应于我们连接器的复选框。

![图片](img/5d9cc266b1314b64b9a0bc90675c22c3.png)

现在是时候测试连接器了。我们可以要求 LLM 使用“欢迎”工具来问候用户。在 Mistral 聊天中，如果 LLM 意识到它需要使用外部工具，将出现一个模态窗口，显示工具名称（*“欢迎”*）和它将接受的参数（*name = “Francisco”*）。

![图片](img/a009bf7bb4756debd102048b32023182.png)

为了确认选择，请点击*“继续”*。在这种情况下，我们将得到一个响应：

![图片](img/64b457cf5a40043d91bfc50928a36d6a.png)

太棒了！我们的 MCP 服务器正在正常运行。同样，我们可以创建更复杂的工具。

## 结论

在本文中，我们介绍 MCP 作为一种高效机制，用于与 LLM 供应商创建连接器。其简洁性和可重用性使得 MCP 在当今非常受欢迎，允许开发者减少实现 LLM 插件所需的时间。

此外，我们检查了一个简单的示例，展示了如何创建 MCP 服务器。实际上，没有任何东西阻止开发者构建更高级的 MCP 应用程序并利用 LLM 提供商的附加功能。

例如，在 Mistral 的情况下，MCP 服务器可以利用库和文档的功能，使工具不仅可以接受文本提示作为输入，还可以接受上传的文件。这些结果可以保存在聊天中，并在不同的对话中保持持久性。

## 资源

+   [介绍模型上下文协议 | Anthropic](https://www.anthropic.com/news/model-context-protocol)

*除非另有说明，所有图片均为作者所有。*
