# 使用 Datapizza AI 快速构建 LLM 代理

> 原文：[`towardsdatascience.com/datapizza-the-ai-framework-made-in-italy/`](https://towardsdatascience.com/datapizza-the-ai-framework-made-in-italy/)

## <mdspan datatext="el1761766675960" class="mdspan-comment">简介</mdspan>

随着这些新工具在日常运营中被越来越多地采用，组织越来越投资于 AI。这股持续的创新浪潮正在推动对更高效和可靠的框架的需求。遵循这一趋势，[Datapizza](https://datapizza.tech/en/)（意大利科技社区的初创公司）刚刚发布了一个名为 ***Datapizza AI*** 的用于 GenAI 的开源框架。

当创建 LLM 驱动的代理时，您需要选择一个 **AI 栈**：

+   **语言模型** – 代理的大脑。第一个大的选择是开源（即 *Llama*，*DeepSeek*，*Phi*）与付费（即 *ChatGPT*，*Claude*，*Gemini*）。然后，根据用例，需要考虑 LLM 的知识：通用（了解一点关于所有事物的知识，如维基百科）与特定主题（例如，针对编码或金融进行微调）。

+   **LLM Engine** – 它是运行语言模型的部分，响应提示，推断意义，并创建文本。基本上，它生成智能。最常用的有 [OpenAI](https://platform.openai.com/) (*ChatGPT*)，[Anthropic](https://www.anthropic.com/) (*Claude*)，[Google](https://gemini.google.com/) (*Gemini*)，和 [Ollama](https://ollama.com/)（在本地运行开源模型）。

+   **AI 框架** – 它是构建和管理工作流程的编排层。换句话说，框架必须结构化 LLMs 创建的智能。目前，该领域由 [*LangChain*](https://www.langchain.com/)，[*LLamaIndex*](https://docs.llamaindex.ai/en/stable/)，和 *[CrewAI](https://www.crewai.com/)* 主导。新的库 [*Datapizza AI*](https://github.com/datapizza-labs/datapizza-ai) 属于这一类别，并希望成为其他主要框架的替代品。

在这篇文章中，我将展示如何使用新的 Datapizza 框架来构建 LLM 驱动的 AI 代理。我将展示一些可以轻松应用于其他类似情况的实用 Python 代码（只需复制、粘贴、运行），并逐行代码进行注释，以便您可以复制此示例。

## 设置

我将使用 **Ollama** 作为 LLM 引擎，因为我喜欢在我的电脑上本地托管模型。这是所有拥有敏感数据公司的标准做法。将一切保持在本地可以完全控制数据隐私、模型行为和成本。

首先，您需要从网站 [下载 Ollama](https://ollama.com/download)。然后，选择一个模型并运行页面上的指示命令来拉取 LLM。我选择 [阿里巴巴的 ***Qwen***](https://ollama.com/library/qwen3)，因为它既智能又轻量（`ollama run qwen3`）。

*Datapizza AI* 支持所有主要的 LLM 引擎。我们可以通过运行以下命令来完成设置：

```py
pip install datapizza-ai
pip install datapizza-ai-clients-openai-like
```

如 [官方文档](https://docs.datapizza.ai/0.0.2/Guides/Clients/local_model/) 所示，我们可以通过用简单的提示调用模型并提问来快速测试我们的 AI 栈。`OpenAILikeClient()` 对象是连接到 Ollama API 的方式，该 API 通常托管在同一本地主机 URL 上。

```py
from datapizza.clients.openai_like import OpenAILikeClient

llm = "qwen3"

prompt = '''
You are an intelligent assistant, provide the best possible answer to user's request. 
''' 

ollama = OpenAILikeClient(api_key="", model=llm, system_prompt=prompt, base_url="http://localhost:11434/v1")

q = '''
what time is it?
'''

llm_res = ollama.invoke(q)
print(llm_res.text)
```

![图片](img/283076c8e42d72b29ddd378a5f063669.png)

## 聊天机器人

另一种测试 LLM 能力的方法是构建一个简单的聊天机器人并进行一些对话。为此，在每次交互时，我们需要存储聊天历史并将其反馈给模型，指定是谁说了什么。Datapizza 框架已经内置了 **内存系统**。

```py
from datapizza.memory import Memory
from datapizza.type import TextBlock, ROLE

memory = Memory()
memory.add_turn(TextBlock(content=prompt), role=ROLE.SYSTEM)

while True:
    ## User
    q = input('🙂 >')
    if q == "quit":
        break

    ## LLM
    llm_res = ollama.invoke(q, memory=memory)
    res = llm_res.text
    print("🍕 >", f"\x1b1;30m{res}\x1b[0m")

    ## Update Memory
    memory.add_turn(TextBlock(content=q), role=ROLE.USER)
    memory.add_turn(TextBlock(content=res), role=ROLE.ASSISTANT)
```

![图片如果你想要 **检索聊天历史**，你只需访问内存。通常，AI 框架在 LLM 交互中使用了三个角色：“*系统*”（核心指令）、“*用户*”（人类所说的话）、“*助手*”（聊天机器人回复的话）。```pymemory.to_dict()```![图片](img/e7a4892a24eead018951caf8ef9f4a21.png)

显然，LLM 单独是非常有限的，它除了聊天之外几乎什么都不能做。因此，我们需要给它提供采取行动的可能性，换句话说，就是激活工具。

## 工具

工具是 **简单 LLM 和 AI 代理之间的主要区别**。当用户请求超出 LLM 知识库范围的内容时（例如，“*现在是什么时间？*”），代理应理解它不知道答案，激活一个工具来获取更多信息（例如检查时钟），通过 LLM 精炼结果，并生成答案。

Datapizza 框架允许你非常容易地从零开始创建工具。你只需要导入 [装饰器](https://www.w3schools.com/python/python_decorators.asp) `@tool`，任何函数都可以成为代理可执行的操作。

```py
from datapizza.tools import tool

@tool
def get_time() -> str:
    '''Get the current time.'''
    from datetime import datetime
    return datetime.now().strftime("%H:%M")

get_time()
```

然后，将指定的工具分配给 **代理**，你将拥有一个结合语言理解 + 自主决策 + 工具使用的 AI。

```py
from datapizza.agents import Agent
import os

os.environ["DATAPIZZA_AGENT_LOG_LEVEL"] = "DEBUG"  #max logging

agent = Agent(name="single-agent", client=ollama, system_prompt=prompt, 
              tools=[get_time], max_steps=2)

q = '''
what time is it?
'''

agent_res = agent.run(q)
```

![图片](img/877ed871a54906da0a482af83c7000ea.png)

一个由语言模型驱动的 AI 代理是一个围绕语言模型构建的智能系统，它不仅响应，还能推理、决策和行动。除了对话（即与通用知识库聊天）之外，代理可以执行的最常见操作是 **RAG**（与你的文档聊天）、**查询**（与数据库聊天）、**网络搜索**（与整个互联网聊天）。

例如，让我们尝试一个网络搜索工具。在 Python 中，最简单的方法是使用著名的私人浏览器 [DuckDuckGo](https://duckduckgo.com/)。你可以直接使用 [原始库](https://github.com/deedy5/ddgs) 或 Datapizza 框架包装器（`pip install datapizza-ai-tools-duckduckgo`）。

```py
from datapizza.tools.duckduckgo import DuckDuckGoSearchTool

DuckDuckGoSearchTool().search(query="powell")
```

![图片](img/72d5195771599d566dd13eb6509bf088.png)

让我们创建一个可以为我们搜索网络的代理。如果你想让它更加互动，你可以像我为聊天机器人所做的那样构建 AI。

```py
os.environ["DATAPIZZA_AGENT_LOG_LEVEL"] = "ERROR" #turn off logging

prompt = '''
You are a journalist. You must make assumptions, use your tool to research, make a guess, and formulate a final answer.
The final answer must contain facts, dates, evidences to support your guess.
'''

memory = Memory()

agent = Agent(name="single-agent", client=ollama, system_prompt=prompt, 
              tools=[DuckDuckGoSearchTool()], 
              memory=memory, max_steps=2)

while True:
    ## User
    q = input('🙂 >')
    if q == "quit":
        break

    ## Agent
    agent_res = agent.run(q)
    res = agent_res.text
    print("🍕 >", f"\x1b1;30m{res}\x1b[0m")

    ## Update Memory
    memory.add_turn(TextBlock(content=q), role=ROLE.USER)
    memory.add_turn(TextBlock(content=res), role=ROLE.ASSISTANT)
```

![图片

## 多智能体系统

代理的真实优势是**相互协作的能力**，就像人类一样。这些团队被称为[多智能体系统（MAS）](https://en.wikipedia.org/wiki/Multi-agent_system)，是一组在共享环境中共同工作以解决单个代理单独难以处理的复杂问题的 AI 代理。

这次，让我们创建一个更高级的工具：代码执行。请注意，LLM 通过接触大量代码和自然语言文本的语料库来学习编程语言的模式、语法和语义。但由于它们无法完成任何实际操作，它们创建的代码只是文本。简而言之，**LLM 可以生成 Python 代码，但不能执行它，代理可以**。

```py
import io
import contextlib

@tool
def code_exec(code:str) -> str:
    '''Execute python code. Use always the function print() to get the output'''
    output = io.StringIO()
    with contextlib.redirect_stdout(output):
        try:
            exec(code)
        except Exception as e:
            print(f"Error: {e}")
    return output.getvalue()

code_exec("from datetime import datetime; print(datetime.now().strftime('%H:%M'))")
```

MAS 有两种类型：**顺序过程**确保任务一个接一个地执行，遵循线性进度。另一方面，**层次结构**模拟传统的组织层次结构，以实现高效的任务委派和执行。我个人倾向于后者，因为其具有更多的并行性和灵活性。

使用 Datapizza 框架，您可以使用`can_call()`函数将两个或多个代理链接起来。这样，一个代理可以将当前任务传递给另一个代理。

```py
prompt_senior = '''
You are a senior Python coder. You check the code generated by the Junior, 
and use your tool to execute the code only if it's correct and safe.
'''
agent_senior = Agent(name="agent-senior", client=ollama, system_prompt=prompt_senior, 
                     tools=[code_exec])

prompt_junior = '''
You are a junior Python coder. You can generate code but you can't execute it. 
You receive a request from the Manager, and your final output must be Python code to pass on.
If you don't know some specific commands, you can use your tool and search the web for "how to ... with python?".
'''
agent_junior = Agent(name="agent-junior", client=ollama, system_prompt=prompt_junior, 
                     tools=[DuckDuckGoSearchTool()])
agent_junior.can_call([agent_senior])

prompt_manager = '''
You know nothing, you're just a manager. After you get a request from the user, 
first you ask the Junior to generate the code, then you ask the Senior to execute it.
'''
agent_manager = Agent(name="agent-manager", client=ollama, system_prompt=prompt_manager, 
                      tools=[])
agent_manager.can_call([agent_junior, agent_senior])

q = '''
Plot the Titanic dataframe. You can find the data here: 
https://raw.githubusercontent.com/mdipietro09/DataScience_ArtificialIntelligence_Utils/master/machine_learning/data_titanic.csv
'''

agent_res = agent_manager.run(q)
#print(agent_res.text)
```

![图片](img/de3db6586ac296dfa89e2927d1a7c4a0.png)![](img/0ea503e24d005059d5edbea05fb88e6b.png)

## 结论

本文是一篇教程，旨在**介绍*Datapizza AI***，这是一个全新的框架，用于构建由 LLM（大型语言模型）驱动的聊天机器人和 AI 代理。该库非常灵活且用户友好，可以覆盖不同的 GenAI 用例。我使用它与 Ollama 一起工作，但它可以与所有著名的引擎链接，如 OpenAI。

本文的完整代码：**[GitHub](https://github.com/mdipietro09/GenerativeAI/blob/main/DatapizzaAI/datapizza.ipynb)**

希望您喜欢！如有任何问题、反馈或只是想分享您有趣的项目，请随时联系我。

👉 [**让我们建立联系**](https://maurodp.carrd.co/) 👈

![图片](img/eb34262c85667d1d15c626dbed569023.png)

[^((所有图片均为作者所有，除非另有说明))]
