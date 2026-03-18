# 如何使用函数调用和 GPT-5 构建人工智能代理

> 原文：[`towardsdatascience.com/how-to-build-an-ai-agent-with-function-calling-and-gpt-5/`](https://towardsdatascience.com/how-to-build-an-ai-agent-with-function-calling-and-gpt-5/)

## <mdspan datatext="el1761003705735" class="mdspan-comment">人工智能代理</mdspan>和大型语言模型（LLM）

**大型语言模型（LLMs）**是建立在深度神经网络（如 transformers）之上的高级 AI 系统，在大量文本上进行训练以生成类似人类的语言。LLM 如 ChatGPT、Claude、Gemini 和 Grok 可以处理许多挑战性任务，并被用于科学、医疗保健、教育和金融等领域的多个方面。

人工智能代理扩展了 LLM 的能力，以解决超出其预训练知识的任务。LLM 可以从训练期间学到的内容编写 Python 教程。如果你要求它预订航班，这项任务需要访问你的日历、网络搜索以及执行操作的能力，这些都超出了 LLM 的预训练知识。一些常见的操作包括：

+   ***天气预报：***LLM 连接到网络搜索工具以获取最新的天气预报。

+   ***预订代理：***一种能够检查用户的日历，搜索网络以访问 Expedia 等预订网站以查找航班和酒店的可用选项，向用户展示以供确认，并代表用户完成预订的人工智能代理。

## 人工智能代理的工作原理

人工智能代理形成一个系统，该系统使用大型语言模型来规划、推理并采取步骤，使用模型推理中建议的工具与环境交互，以解决特定任务。

### 人工智能代理的基本结构

![](img/77d1923178fce7364d4de1c1b7974ba5.png)

由 Gemini 生成的图像

+   ***大型语言模型（LLM）：***LLM 是人工智能代理的**大脑**。它接收用户的提示，通过请求进行规划和推理，并将问题分解为步骤，以确定它应该使用哪些工具来完成任务。

+   ***工具：***这是代理根据大型语言模型的计划和推理执行动作的框架。如果你要求 LLM 为你预订餐厅的桌子，可能使用的工具包括日历来检查你的可用性，以及网络搜索工具来访问餐厅网站并为你预订。

### 预订人工智能代理的决策图示

![](img/728d143685fcf0c2ea22dd4b5eb11e39.png)

由 ChatGPT 生成的图像

> 人工智能代理可以根据任务访问不同的工具。一个工具可能是一个数据存储，例如数据库。例如，一个客户支持代理可以访问客户的账户详情和购买历史，并决定何时检索这些信息以帮助解决问题。

AI 代理用于解决各种任务，并且有许多强大的代理可供选择。编码代理，尤其是 Cursor、Windsurf 和 GitHub Copilot 等代理 IDE，可以帮助工程师更快地编写和调试代码，并快速构建项目。CLI 编码代理，如 Claude Code 和 Codex CLI，可以与用户的桌面和终端交互，执行编码任务。ChatGPT 支持代表用户进行预订等操作的代理。代理还集成到客户支持工作流程中，以与客户沟通并解决他们的问题。

## 函数调用

函数调用是一种将大型语言模型 (LLM) 连接到外部工具（如 API 或数据库）的技术。它在创建 AI 代理时用于将 LLM 连接到工具。在函数调用中，每个工具都被定义为一个代码函数（例如，用于获取最新预报的天气 API），以及一个 JSON Schema，该 Schema 指定函数的参数并指导 LLM 在何时以及如何针对特定任务调用该函数。

定义函数的类型取决于代理设计要执行的任务。例如，对于一个客户支持代理，我们可以定义一个可以从非结构化数据中提取信息的功能，例如包含有关企业产品详细信息的 PDF 文件。

在这篇文章中，我将演示如何使用函数调用构建一个简单的网络搜索代理，使用 GPT-5 作为大型语言模型。

## 网络搜索代理的基本结构

![](img/b04c68d07877f979963141f772014ae8.png)

Gemini 生成的图像

## 网络搜索代理背后的主要逻辑：

+   定义一个代码函数来处理网络搜索。

+   定义自定义指令，以指导大型语言模型确定何时根据查询调用网络搜索功能。例如，如果查询询问当前的天气，网络搜索代理将识别出需要搜索互联网以获取最新的天气报告的需求。然而，如果查询要求它编写关于 Python 等编程语言的教程，它可以从其预训练的知识中回答，它将不会调用网络搜索功能，而是直接回答。

## 前提

**创建 OpenAI 账户并生成 API 密钥**

1：如果您还没有，请创建一个 [OpenAI 账户](https://auth.openai.com/create-account)

2：[生成 API 密钥](https://platform.openai.com/account/api-keys)

**设置和激活环境**

```py
python3 -m venv env
source env/bin/activate
```

**导出 OpenAI API 密钥**

```py
export OPENAI_API_KEY="Your Openai API Key"
```

**为网络搜索设置 Tavily**

Tavily 是专为 AI 代理定制的网络搜索工具。在 [Tavily.com](https://www.tavily.com/) 上创建账户，一旦您的个人资料设置完成，就会生成一个 API 密钥，您可以将它复制到您的环境中。新账户将获得 1000 个免费积分，可用于最多 1000 次网络搜索。

**导出 TAVILY API 密钥**

```py
export TAVILY_API_KEY="Your Tavily API Key"
```

**安装包**

```py
pip3 install openai
pip3 install tavily-python
```

## 步步构建具有函数调用的网络搜索代理

### 步骤 1：使用 Tavily 创建网络搜索功能

使用 Tavily 实现了网络搜索功能，作为网络搜索代理中函数调用的工具。

```py
from tavily import TavilyClient
import os

tavily = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

def web_search(query: str, num_results: int = 10):
    try:
        result = tavily.search(
            query=query,
            search_depth="basic",
            max_results=num_results,
            include_answer=False,       
            include_raw_content=False,
            include_images=False
        )

        results = result.get("results", [])

        return {
            "query": query,
            "results": results, 
            "sources": [
                {"title": r.get("title", ""), "url": r.get("url", "")}
                for r in results
            ]
        }

    except Exception as e:
        return {
            "error": f"Search error: {e}",
            "query": query,
            "results": [],
            "sources": [],
        }
```

**网络函数代码分解**

使用 API 密钥初始化了 Tavily。在 `web_search` 函数中，执行以下步骤：

+   调用 Tavily 搜索函数以搜索互联网并检索前 10 个结果。

+   返回搜索结果及其对应来源。

此返回输出将作为网络搜索代理的相关上下文：我们将在本文的后面定义，以获取需要实时数据（如天气预报）的查询（提示）的最新信息。

### 第 2 步：创建工具模式

工具模式定义了 AI 模型何时应该调用工具的定制指令，在这种情况下是用于网络搜索功能的工具。它还指定了模型调用工具时应采取的条件和操作。以下是基于 [OpenAI 工具模式结构](https://platform.openai.com/docs/guides/structured-outputs?context=with_parse#supported-schemas) 定义的 json 工具模式。

```py
tool_schema = [
    {
        "type": "function",
        "name": "web_search",

        "description": """Execute a web search to fetch up to date information. Synthesize a concise, 
        self-contained answer from the content of the results of the visited pages.
        Fetch pages, extract text, and provide the best available result while citing 1-3 sources (title + URL). 
        If sources conflict, surface the uncertainty and prefer the most recent evidence.
        """,

        "strict": True,
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Query to be searched on the web.",
                },
            },
            "required": ["query"],
            "additionalProperties": False
        },
    },
] 
```

#### 工具模式的属性

+   ***type:*** 指定工具类型为函数。

+   ***name:*** 将用于工具调用的函数名称，即 ***web_search***。

+   ***description:*** 描述了当调用网络搜索工具时 AI 模型应该做什么。它指示模型使用 ***web_search*** 函数在互联网上搜索，以获取最新信息并提取相关细节以生成最佳响应。

+   ***strict:*** 被设置为 true，此属性指示 LLM 严格遵循工具模式的指令。

+   ***parameters:*** 定义了将传递给 ***web_search*** 函数的参数。在这种情况下，只有一个参数：***query***，它代表要在互联网上查找的搜索词。

+   ***required:*** 指示 LLM，查询是 ***web_search*** 函数的必填参数。

+   ***additionalProperties:*** 被设置为 false，意味着工具的 *arguments 对象* 不能包含除 ***parameters.properties*** 下定义的参数之外的其他参数。

### 第 3 步：使用 GPT-5 和函数调用创建网络搜索代理

最后，我将构建一个我们可以与之交谈的代理，当它需要最新的信息时可以搜索网络。我将使用 OpenAI 的快速且精确的模型 **GPT-5-mini**，并结合函数调用来调用已定义的 ***tool schema*** 和 ***web search function***。

```py
from datetime import datetime, timezone
import json
from openai import OpenAI
import os 

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# tracker for the last model's response id to maintain conversation's state 
prev_response_id = None

# a list for storing tool's results from the function call 
tool_results = []

while True:
    # if the tool results is empty prompt message 
    if len(tool_results) == 0:
        user_message = input("User: ")

        """ commands for exiting chat """
        if isinstance(user_message, str) and user_message.strip().lower() in {"exit", "q"}:
            print("Exiting chat. Goodbye!")
            break

    else:
        user_message = tool_results.copy()

        # clear the tool results for the next call 
        tool_results = []

    # obtain current's date to be passed into the model as an instruction to assist in decision making
    today_date = datetime.now(timezone.utc).date().isoformat()     

    response = client.responses.create(
        model = "gpt-5-mini",
        input = user_message,
        instructions=f"Current date is {today_date}.",
        tools = tool_schema,
        previous_response_id=prev_response_id,
        text = {"verbosity": "low"},
        reasoning={
            "effort": "low",
        },
        store=True,
        )

    prev_response_id = response.id

    # Handles model response's output 
    for output in response.output:

        if output.type == "reasoning":
            print("Assistant: ","Reasoning ....")

            for reasoning_summary in output.summary:
                print("Assistant: ",reasoning_summary)

        elif output.type == "message":
            for item in output.content:
                print("Assistant: ",item.text)

        elif output.type == "function_call":
            # obtain function name 
            function_name = globals().get(output.name)
            # loads function arguments 
            args = json.loads(output.arguments)
            function_response = function_name(**args)
            tool_results.append(
                {
                    "type": "function_call_output",
                    "call_id": output.call_id,
                    "output": json.dumps(function_response)
                }
            )
```

**逐步代码分解**

```py
from openai import OpenAI
import os 

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
prev_response_id = None
tool_results = []
```

+   使用 API 密钥初始化了 OpenAI 模型 API。

+   初始化了两个变量 ***prev_response_id*** 和 ***tool_results***。***prev_response_id*** 用于跟踪模型的响应以维护会话状态，而 ***tool_results*** 是一个存储从 ***web_search*** 函数调用返回的输出的列表。

聊天在***循环***内运行。用户输入一条消息，使用工具模式调用的模型接受该消息，对其进行推理，决定是否调用网络搜索工具，然后将工具的输出传递回模型。模型生成一个上下文感知的响应。这个过程会一直持续到用户退出聊天。

**循环的代码遍历**

```py
if len(tool_results) == 0:
    user_message = input("User: ")
    if isinstance(user_message, str) and user_message.strip().lower() in {"exit", "q"}:
        print("Exiting chat. Goodbye!")
        break

else:
    user_message = tool_results.copy()
    tool_results = []

today_date = datetime.now(timezone.utc).date().isoformat()     

response = client.responses.create(
    model = "gpt-5-mini",
    input = user_message,
    instructions=f"Current date is {today_date}.",
    tools = tool_schema,
    previous_response_id=prev_response_id,
    text = {"verbosity": "low"},
    reasoning={
        "effort": "low",
    },
    store=True,
    )

prev_response_id = response.id
```

+   检查***tool_results***是否为空。如果是空的，用户将被提示输入消息，可以选择使用***exit***或***q***退出。

+   如果***tool_results***不为空，***user_message***将被设置为收集到的工具输出，以便发送到模型。***tool_results***被清除，以避免在下一个循环迭代中重新发送相同的***tool outputs***。

+   获取当前日期（***today_date***）以供模型使用，以便做出时间感知的决策。

+   调用***client.responses.create***生成模型的响应，并接受以下参数：

    +   model：设置为***gpt-5-mini***。

    +   input：接受用户的消息。

    +   instructions：设置为当前日期（today_date）。

    +   tools：设置为之前定义的工具模式。

    +   previous_response_id：设置为前一个响应的 ID，以便模型可以维持对话状态。

    +   text：将详尽性设置为低，以保持模型响应的简洁性。

    +   reasoning：GPT-5-mini 是一个推理模型，将推理的努力程度设置为低以获得更快的响应。对于更复杂的任务，我们可以将其设置为高。

    +   store：告诉模型存储当前响应，以便以后可以检索，并有助于对话的连续性。

+   ***prev_response_id***设置为当前响应 ID，以便下一个函数调用可以继续同一对话。

```py
for output in response.output:
    if output.type == "reasoning":
        print("Assistant: ","Reasoning ....")

        for reasoning_summary in output.summary:
            print("Assistant: ",reasoning_summary)

    elif output.type == "message":
        for item in output.content:
            print("Assistant: ",item.text)

    elif output.type == "function_call":
        # obtain function name 
        function_name = globals().get(output.name)
        # loads function arguments 
        args = json.loads(output.arguments)
        function_response = function_name(**args)
        # append tool results list with the the function call's id and function's response 
        tool_results.append(
            {
                "type": "function_call_output",
                "call_id": output.call_id,
                "output": json.dumps(function_response)
            }
        )
```

这处理模型的响应输出，并执行以下操作；

+   如果输出类型是推理，打印推理摘要中的每个项。

+   如果输出类型是消息，遍历内容并打印每个文本项。

+   如果输出类型是函数调用，获取函数的名称，解析其参数，并将它们传递给函数（***web_search***）以生成响应。在这种情况下，网络搜索响应包含与用户消息相关的最新信息。最后将函数调用的响应和函数调用 ID 添加到***tool_results***中。这允许下一个循环将工具结果发送回模型。

### 网络搜索代理的完整代码

```py
from datetime import datetime, timezone
import json
from openai import OpenAI
import os 
from tavily import TavilyClient

tavily = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

def web_search(query: str, num_results: int = 10):
    try:
        result = tavily.search(
            query=query,
            search_depth="basic",
            max_results=num_results,
            include_answer=False,       
            include_raw_content=False,
            include_images=False
        )

        results = result.get("results", [])

        return {
            "query": query,
            "results": results, 
            "sources": [
                {"title": r.get("title", ""), "url": r.get("url", "")}
                for r in results
            ]
        }

    except Exception as e:
        return {
            "error": f"Search error: {e}",
            "query": query,
            "results": [],
            "sources": [],
        }

tool_schema = [
    {
        "type": "function",
        "name": "web_search",
        "description": """Execute a web search to fetch up to date information. Synthesize a concise, 
        self-contained answer from the content of the results of the visited pages.
        Fetch pages, extract text, and provide the best available result while citing 1-3 sources (title + URL). "
        If sources conflict, surface the uncertainty and prefer the most recent evidence.
        """,
        "strict": True,
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Query to be searched on the web.",
                },
            },
            "required": ["query"],
            "additionalProperties": False
        },
    },
]

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# tracker for the last model's response id to maintain conversation's state 
prev_response_id = None

# a list for storing tool's results from the function call 
tool_results = []

while True:
    # if the tool results is empty prompt message 
    if len(tool_results) == 0:
        user_message = input("User: ")

        """ commands for exiting chat """
        if isinstance(user_message, str) and user_message.strip().lower() in {"exit", "q"}:
            print("Exiting chat. Goodbye!")
            break

    else:
        # set the user's messages to the tool results to be sent to the model 
        user_message = tool_results.copy()

        # clear the tool results for the next call 
        tool_results = []

    # obtain current's date to be passed into the model as an instruction to assist in decision making
    today_date = datetime.now(timezone.utc).date().isoformat()     

    response = client.responses.create(
        model = "gpt-5-mini",
        input = user_message,
        instructions=f"Current date is {today_date}.",
        tools = tool_schema,
        previous_response_id=prev_response_id,
        text = {"verbosity": "low"},
        reasoning={
            "effort": "low",
        },
        store=True,
        )

    prev_response_id = response.id

    # Handles model response's output 
    for output in response.output:

        if output.type == "reasoning":
            print("Assistant: ","Reasoning ....")

            for reasoning_summary in output.summary:
                print("Assistant: ",reasoning_summary)

        elif output.type == "message":
            for item in output.content:
                print("Assistant: ",item.text)

        # checks if the output type is a function call and append the function call's results to the tool results list
        elif output.type == "function_call":
            # obtain function name 
            function_name = globals().get(output.name)
            # loads function arguments 
            args = json.loads(output.arguments)
            function_response = function_name(**args)
            # append tool results list with the the function call's id and function's response 
            tool_results.append(
                {
                    "type": "function_call_output",
                    "call_id": output.call_id,
                    "output": json.dumps(function_response)
                }
            )
```

当您运行代码时，您可以轻松地与代理聊天，询问需要最新信息的问题，例如当前天气或最新产品发布。代理会提供最新的信息，以及来自互联网的相关源代码。以下是从终端输出的示例。

```py
User: What is the weather like in London today?
Assistant:  Reasoning ....
Assistant:  Reasoning ....
Assistant:  Right now in London: overcast, about 18°C (64°F), humidity ~88%, light SW wind ~16 km/h, no precipitation reported. Source: WeatherAPI (current conditions) — https://www.weatherapi.com/

User: What is the latest iPhone model?
Assistant:  Reasoning ....
Assistant:  Reasoning ....
Assistant:  The latest iPhone models are the iPhone 17 lineup (including iPhone 17, iPhone 17 Pro, iPhone 17 Pro Max) and the new iPhone Air — announced by Apple on Sept 9, 2025\. Source: Apple Newsroom — https://www.apple.com/newsroom/2025/09/apple-debuts-iphone-17/

User: Multiply 500 by 12\.           
Assistant:  Reasoning ....
Assistant:  6000
User: exit   
Exiting chat. Goodbye!
```

您可以看到它们对应网页源代码的结果。当您要求它执行不需要最新信息的任务，例如数学计算或编写代码时，代理会直接响应，而无需进行任何网络搜索。

> 注意：网络搜索代理是一个简单、单工具的代理。高级代理系统协调多个专业工具，并使用高效的记忆来维持上下文、规划和解决更复杂的任务。

## 结论

在这篇帖子中，我解释了人工智能代理的工作原理以及它是如何通过使用工具来扩展大型语言模型与环境交互、执行动作和解决任务的能力。我还解释了函数调用以及它是如何使大型语言模型能够调用工具的。我演示了如何创建一个用于函数调用的工具模式，该模式定义了大型语言模型何时以及如何调用工具来执行动作。我使用 Tavily 定义了一个网络搜索函数，用于从网络中获取信息，然后逐步展示了如何使用函数调用和 GPT-5-mini 作为大型语言模型来构建网络搜索代理。最后，我们构建了一个能够从互联网检索最新信息以回答用户查询的网络搜索代理。

> 查看我的 GitHub 仓库*[GenAI-Courses](https://github.com/ayoolaolafenwa/GenAI-Courses/tree/main)*，我在那里发布了关于各种生成式人工智能主题的更多课程。它还包括一个关于使用函数调用构建[代理 RAG](https://github.com/ayoolaolafenwa/GenAI-Courses/blob/main/AI%20Agents/Agentic_RAG.md)的指南。

## 通过以下方式联系我：

> 邮箱：[[email protected]](https://mail.google.com/mail/u/0/#inbox)
> 
> [领英：https://www.linkedin.com/in/ayoola-olafenwa-003b901a9/](https://www.linkedin.com/in/ayoola-olafenwa-003b901a9/)

## 参考文献

[`platform.openai.com/docs/guides/function-calling?api-mode=responses`](https://platform.openai.com/docs/guides/function-calling?api-mode=responses)

[`docs.tavily.com/documentation/api-reference/endpoint/search`](https://docs.tavily.com/documentation/api-reference/endpoint/search)
