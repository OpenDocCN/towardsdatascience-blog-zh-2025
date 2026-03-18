# 使用 ACP 的 AI 代理通信的未来

> 原文：[`towardsdatascience.com/the-future-of-ai-agent-communication-with-acp/`](https://towardsdatascience.com/the-future-of-ai-agent-communication-with-acp/)

<mdspan datatext="el1752595818813" class="mdspan-comment">令人兴奋的是</mdspan>看到通用人工智能（GenAI）行业开始向标准化迈进。我们可能正在见证类似于互联网早期 HTTP（超文本传输协议）首次出现的情景。当蒂姆·伯纳斯-李在 1990 年开发 HTTP 时，它提供了一个简单且可扩展的协议，将互联网从专业研究网络转变为全球可访问的万维网。到 1993 年，像 Mosaic 这样的网络浏览器使 HTTP 变得如此流行，以至于网络流量迅速超过了其他系统。

在这个方向上，一个有希望的步骤是由 Anthropic 开发的 MCP（模型上下文协议）。MCP 通过尝试标准化 LLM（大型语言模型）与外部工具或数据源之间的交互而越来越受欢迎。最近（[*第一次提交*](https://github.com/i-am-bee/acp/commit/19e4c708153e23b529d09cdc6e23f0e95ef8a933) *日期为 2025 年 4 月*），出现了一种名为 ACP（代理通信协议）的新协议。它通过定义代理之间如何相互通信来补充 MCP。

![图片](img/f47c02bebd2e9067ff2f85195455f3f0.png)

架构示例 | 图片由作者提供

在这篇文章中，我想讨论 ACP 是什么，为什么它可能很有帮助，以及如何在实践中使用它。我们将构建一个多代理人工智能系统来与数据交互。

## ACP 概述

在进入实践之前，让我们花一点时间来了解 ACP 背后的理论以及它是如何在幕后工作的。

ACP（代理通信协议）是一个开放协议，旨在解决连接 AI 代理、应用程序和人类日益增长的挑战。当前的通用人工智能（GenAI）行业相当分散，不同的团队使用各种、通常不兼容的框架和技术独立构建代理。这种分散减缓了创新，并使得代理难以有效协作。

为了应对这一挑战，ACP 旨在通过 RESTful API 标准化代理之间的通信。该协议是框架和技术无关的，这意味着它可以与任何代理框架一起使用，例如 LangChain、CrewAI、smolagents 或其他框架。这种灵活性使得构建可互操作的系统变得更容易，其中代理可以无缝地协同工作，无论它们最初是如何开发的。

该协议是在 Linux Foundation 的支持下，与 BeeAI（其参考实现）一起作为开放标准开发的。团队强调的一个关键点是 ACP 是公开治理并由社区塑造的，而不是由一组供应商主导。

ACP 能带来哪些好处？

+   **易于替换的代理**。在 GenAI 领域创新的速度下，新的尖端技术不断涌现。ACP 使得代理能够在生产中无缝替换，降低维护成本，并在最先进的技术可用时更容易采用。

+   **在基于不同框架的多个代理之间实现协作** **。正如我们所知，从人员管理来看，专业化往往能带来更好的结果。同样的情况也适用于代理系统。专注于特定任务（如编写 Python 代码或网络研究）的代理群体，通常能比试图做所有事情的单一代理表现得更好。ACP 使得这样的专业化代理能够通过不同的框架或技术进行通信和协作**。

+   **新的合作机会**。有了代理通信的统一标准，代理之间的协作将变得更加容易，从而在公司内部的不同团队之间甚至不同公司之间激发新的合作关系。想象一下，如果您的智能家居代理注意到温度异常下降，确定供暖系统已故障，并检查您的公用事业提供商的代理以确认没有计划中的停电。最后，它通过您的 Google 日历代理预订技术人员，以确保您在家。这可能听起来很未来派，但有了 ACP，这可以非常接近现实**。

我们已经介绍了 ACP 是什么以及为什么它很重要。该协议看起来非常有前景。那么，让我们来测试一下它在实际中的表现如何。

## ACP 的实践应用

让我们在经典的“与数据对话”用例中尝试使用 ACP。为了利用 ACP 无框架的优势，我们将使用不同的框架来构建 ACP 代理：

+   **使用 CrewAI 的 SQL 代理** 来编写 SQL 查询

+   **使用 HuggingFace smolagents 的 DB 代理** 来执行这些查询。

> *我不会在这里详细介绍每个框架的细节，但如果您感兴趣，我已经为它们各自撰写了深入的文章：
> 
> – [“多智能体系统 101”](https://towardsdatascience.com/multi-ai-agent-systems-101-bac58e3bcc47/) 关于 CrewAI，
> 
> – [“代码代理：代理人工智能的未来”](https://towardsdatascience.com/code-agents-the-future-of-agentic-ai/) 关于 smolagents.*

### 构建 DB 代理

让我们从 DB 代理开始。正如我之前提到的，ACP 可以与 MCP 相辅相成。因此，我将通过 MCP 服务器使用我的分析工具箱中的工具。您可以在 [GitHub](https://github.com/miptgirl/mcp-analyst-toolkit) 上找到 MCP 服务器的实现。要深入了解和获取逐步说明，请查看我的上一篇文章 [我的个人分析工具箱](https://towardsdatascience.com/your-personal-analytics-toolbox/)，其中我详细介绍了 MCP。

代码本身相当简单：我们初始化 ACP 服务器，并使用 `@server.agent()` 装饰器来定义我们的代理函数。这个函数期望接收一个消息列表作为输入，并返回一个生成器。

```py
from collections.abc import AsyncGenerator
from acp_sdk.models import Message, MessagePart
from acp_sdk.server import Context, RunYield, RunYieldResume, Server
from smolagents import LiteLLMModel,ToolCallingAgent, ToolCollection
import logging 
from dotenv import load_dotenv
from mcp import StdioServerParameters

load_dotenv() 

# initialise ACP server
server = Server()

# initialise LLM
model = LiteLLMModel(
  model_id="openai/gpt-4o-mini",  
  max_tokens=2048
)

# define config for MCP server to connect
server_parameters = StdioServerParameters(
  command="uv",
  args=[
      "--directory",
      "/Users/marie/Documents/github/mcp-analyst-toolkit/src/mcp_server",
      "run",
      "server.py"
  ],
  env=None
)

@server.agent()
async def db_agent(input: list[Message], context: Context) -> AsyncGenerator[RunYield, RunYieldResume]:
  "This is a CodeAgent can execute SQL queries against ClickHouse database."
  with ToolCollection.from_mcp(server_parameters, trust_remote_code=True) as tool_collection:
    agent = ToolCallingAgent(tools=[*tool_collection.tools], model=model)
    question = input[0].parts[0].content
    response = agent.run(question)

  yield Message(parts=[MessagePart(content=str(response))])

if __name__ == "__main__":
  server.run(port=8001)
```

我们还需要设置一个 Python 环境。我将使用 `uv` 包管理器来完成这项工作。

```py
uv init --name acp-sql-agent
uv venv
source .venv/bin/activate
uv add acp-sdk "smolagents[litellm]" python-dotenv mcp "smolagents[mcp]" ipykernel
```

然后，我们可以使用以下命令运行代理。

```py
uv run db_agent.py
```

如果一切设置正确，你将看到在 8001 端口上运行的服务器。我们需要一个 ACP 客户端来验证它是否按预期工作。请稍等，我们很快就会测试它。

### 构建 SQL 代理

在此之前，让我们构建一个 SQL 代理来构建查询。我们将为此使用 CrewAI 框架。我们的代理将参考问题和查询的知识库来生成答案。因此，我们将配备一个 RAG（检索增强生成）工具。

首先，我们将初始化 RAG 工具并加载参考文件 `clickhouse_queries.txt`。接下来，我们将通过指定其角色、目标和背景故事来创建一个 CrewAI 代理。最后，我们将创建一个任务并将所有内容捆绑成一个 Crew 对象。

```py
from crewai import Crew, Task, Agent, LLM
from crewai.tools import BaseTool
from crewai_tools import RagTool
from collections.abc import AsyncGenerator
from acp_sdk.models import Message, MessagePart
from acp_sdk.server import RunYield, RunYieldResume, Server
import json
import os
from datetime import datetime
from typing import Type
from pydantic import BaseModel, Field

import nest_asyncio
nest_asyncio.apply()

# config for RAG tool
config = {
  "llm": {
    "provider": "openai",
    "config": {
      "model": "gpt-4o-mini",
    }
  },
  "embedding_model": {
    "provider": "openai",
    "config": {
      "model": "text-embedding-ada-002"
    }
  }
}

# initialise tool
rag_tool = RagTool(
  config=config,  
  chunk_size=1200,       
  chunk_overlap=200)

rag_tool.add("clickhouse_queries.txt")

# initialise ACP server
server = Server()

# initialise LLM
llm = LLM(model="openai/gpt-4o-mini", max_tokens=2048)

@server.agent()
async def sql_agent(input: list[Message]) -> AsyncGenerator[RunYield, RunYieldResume]:
  "This agent knows the database schema and can return SQL queries to answer questions about the data."

  # create agent
  sql_agent = Agent(
    role="Senior SQL analyst", 
    goal="Write SQL queries to answer questions about the e-commerce analytics database.",
    backstory="""
You are an expert in ClickHouse SQL queries with over 10 years of experience. You are familiar with the e-commerce analytics database schema and can write optimized queries to extract insights.

## Database Schema

You are working with an e-commerce analytics database containing the following tables:

### Table: ecommerce.users 
**Description:** Customer information for the online shop
**Primary Key:** user_id
**Fields:** 
- user_id (Int64) - Unique customer identifier (e.g., 1000004, 3000004)
- country (String) - Customer's country of residence (e.g., "Netherlands", "United Kingdom")
- is_active (Int8) - Customer status: 1 = active, 0 = inactive
- age (Int32) - Customer age in full years (e.g., 31, 72)

### Table: ecommerce.sessions 
**Description:** User session data and transaction records
**Primary Key:** session_id
**Foreign Key:** user_id (references ecommerce.users.user_id)
**Fields:** 
- user_id (Int64) - Customer identifier linking to users table (e.g., 1000004, 3000004)
- session_id (Int64) - Unique session identifier (e.g., 106, 1023)
- action_date (Date) - Session start date (e.g., "2021-01-03", "2024-12-02")
- session_duration (Int32) - Session duration in seconds (e.g., 125, 49)
- os (String) - Operating system used (e.g., "Windows", "Android", "iOS", "MacOS")
- browser (String) - Browser used (e.g., "Chrome", "Safari", "Firefox", "Edge")
- is_fraud (Int8) - Fraud indicator: 1 = fraudulent session, 0 = legitimate
- revenue (Float64) - Purchase amount in USD (0.0 for non-purchase sessions, >0 for purchases)

## ClickHouse-Specific Guidelines

1\. **Use ClickHouse-optimized functions:**
   - uniqExact() for precise unique counts
   - uniqExactIf() for conditional unique counts
   - quantile() functions for percentiles
   - Date functions: toStartOfMonth(), toStartOfYear(), today()

2\. **Query formatting requirements:**
   - Always end queries with "format TabSeparatedWithNames"
   - Use meaningful column aliases
   - Use proper JOIN syntax when combining tables
   - Wrap date literals in quotes (e.g., '2024-01-01')

3\. **Performance considerations:**
   - Use appropriate WHERE clauses to filter data
   - Consider using HAVING for post-aggregation filtering
   - Use LIMIT when finding top/bottom results

4\. **Data interpretation:**
   - revenue > 0 indicates a purchase session
   - revenue = 0 indicates a browsing session without purchase
   - is_fraud = 1 sessions should typically be excluded from business metrics unless specifically analyzing fraud

## Response Format
Provide only the SQL query as your answer. Include brief reasoning in comments if the query logic is complex. 
        """,

    verbose=True,
    allow_delegation=False,
    llm=llm,
    tools=[rag_tool], 
    max_retry_limit=5
  )

  # create task
  task1 = Task(
    description=input[0].parts[0].content,
    expected_output = "Reliable SQL query that answers the question based on the e-commerce analytics database schema.",
    agent=sql_agent
  )

  # create crew
  crew = Crew(agents=[sql_agent], tasks=[task1], verbose=True)

  # execute agent
  task_output = await crew.kickoff_async()
  yield Message(parts=[MessagePart(content=str(task_output))])

if __name__ == "__main__":
  server.run(port=8002)
```

在运行服务器之前，我们还需要将任何缺失的包添加到 `uv` 中。

```py
uv add crewai crewai_tools nest-asyncio
uv run sql_agent.py
```

现在，第二个代理正在 8002 端口上运行。随着两个服务器都已启动并运行，是时候检查它们是否正常工作了。

### 使用客户端调用 ACP 代理

现在我们准备测试我们的代理，我们将使用 ACP 客户端来同步运行它们。为此，我们需要使用服务器 URL 初始化一个 Client，并使用 `run_sync` 函数指定代理的名称和输入。

```py
import os
import nest_asyncio
nest_asyncio.apply()
from acp_sdk.client import Client
import asyncio

# Set your OpenAI API key here (or use environment variable)
# os.environ["OPENAI_API_KEY"] = "your-api-key-here"

async def example() -> None:
  async with Client(base_url="http://localhost:8001") as client1:
    run1 = await client1.run_sync(
      agent="db_agent", input="select 1 as test"
    )
    print('<TEST> DB agent response:')
    print(run1.output[0].parts[0].content)

  async with Client(base_url="http://localhost:8002") as client2:
    run2 = await client2.run_sync(
      agent="sql_agent", input="How many customers did we have in May 2024?" 
    )
    print('<TEST> SQL agent response:')
    print(run2.output[0].parts[0].content)

if __name__ == "__main__":
  asyncio.run(example())

# <TEST> DB agent response:
# 1
# <TEST> SQL agent response:
# ```

# SELECT COUNT(DISTINCT user_id) AS total_customers

# FROM ecommerce.users

# WHERE is_active = 1

# AND user_id IN (

# SELECT DISTINCT user_id

# FROM ecommerce.sessions

# WHERE action_date >= '2024-05-01' AND action_date < '2024-06-01'

# )

# 格式 TabSeparatedWithNames

```py

We received expected results from both servers, so it looks like everything is working as intended. 

> **💡*Tip****: You can check the full execution logs in the terminal where each server is running.*

### Chaining agents sequentially 

To answer actual questions from customers, we need both agents to work together. Let’s chain them one after the other. So, we will first call the SQL agent and then pass the generated SQL query to the DB agent for execution.

![](img/2a8d512ca9614bfd3664663f7cfc4c56.png)

Image by author

Here’s the code to chain the agents. It’s quite similar to what we used earlier to test each server individually. The main difference is that we now pass the output from the SQL agent directly into the DB agent.

```

async def example() -> None:

async with Client(base_url="http://localhost:8001") as db_agent, Client(base_url="http://localhost:8002") as sql_agent:

    question = '我们在 2024 年 5 月有多少客户？'

    sql_query = await sql_agent.run_sync(

    agent="sql_agent", input=question

    )

    打印('SQL 代理生成的查询：')

    打印(sql_query.output[0].parts[0].content)

    answer = await db_agent.run_sync(

    agent="db_agent", input=sql_query.output[0].parts[0].content

    )

    打印('数据库代理的答案：')

    打印(answer.output[0].parts[0].content)

asyncio.run(example())

```py

Everything worked smoothly, and we received the expected output. 

```

SQL 代理生成的查询：

思考：我需要编写一个 SQL 查询来计算唯一客户的数量

根据他们的会话在 2024 年 5 月活跃的用户。

```pysql
SELECT COUNT(DISTINCT u.user_id) AS active_customers
FROM ecommerce.users AS u
JOIN ecommerce.sessions AS s ON u.user_id = s.user_id
WHERE u.is_active = 1
AND s.action_date >= '2024-05-01' 
AND s.action_date < '2024-06-01'
FORMAT TabSeparatedWithNames
```

数据库代理的答案：

234544

```py

### Router pattern

In some use cases, the path is static and well-defined, and we can chain agents directly as we did earlier. However, more often we expect LLM agents to reason independently and decide which tools or agents to use to achieve a goal. To solve for such cases, we will implement a router pattern using ACP. We will create a new agent  (the orchestrator ) that can delegate tasks to DB and SQL agents.

![](img/3cf31e181f503a50b7d0b325203f852d.png)

Image by author

We will start by adding a reference implementation `beeai_framework` to the package manager.

```

添加 beeai_framework

```py

To enable our orchestrator to call the SQL and DB agents, we will wrap them as tools. This way, the orchestrator can treat them like any other tool and invoke them when needed. 

Let’s start with the SQL agent. It’s primarily boilerplate code: we define the input and output fields using Pydantic and then call the agent in the `_run` function.

```

从 pydantic 导入 BaseModel, Field

从 acp_sdk 导入 Message

从 acp_sdk.client 导入 Client

从 acp_sdk.models 导入 MessagePart

从 beeai_framework.tools.tool 导入 Tool

从 beeai_framework.tools.types 导入 ToolRunOptions

从 beeai_framework.context 导入 RunContext

从 beeai_framework.emitter 导入 Emitter

从 beeai_framework.tools 导入 ToolOutput

从 beeai_framework.utils.strings 导入 to_json

# 辅助函数

async def run_agent(agent: str, input: str) -> list[Message]:

async with Client(base_url="http://localhost:8002") as client:

    run = await client.run_sync(

    agent=agent, input=[Message(parts=[MessagePart(content=input, content_type="text/plain")])]

    )

return run.output

class SqlQueryToolInput(BaseModel):

question: str = Field(description="使用 SQL 查询回答的电子商务分析数据库问题")

class SqlQueryToolResult(BaseModel):

sql_query: str = Field(description="回答问题的 SQL 查询")

class SqlQueryToolOutput(ToolOutput):

result: SqlQueryToolResult = Field(description="SQL 查询结果")

def get_text_content(self) -> str:

    return to_json(self.result)

def is_empty(self) -> bool:

    return self.result.sql_query.strip() == ""

def __init__(self, result: SqlQueryToolResult) -> None:

    super().__init__()

    self.result = result

class SqlQueryTool(Tool[SqlQueryToolInput, ToolRunOptions, SqlQueryToolOutput]):

name = "SQL 查询生成器"

description = "生成用于回答电子商务分析数据库问题的 SQL 查询"

input_schema = SqlQueryToolInput

def _create_emitter(self) -> Emitter:

    return Emitter.root().child(

        namespace=["tool", "sql_query"],

        creator=self,

    )

async def _run(self, input: SqlQueryToolInput, options: ToolRunOptions | None, context: RunContext) -> SqlQueryToolOutput:

    result = await run_agent("sql_agent", input.question)

    return SqlQueryToolOutput(result=SqlQueryToolResult(sql_query=str(result[0])))

```py

Let’s follow the same approach with the DB agent.

```

from pydantic import BaseModel, Field

from acp_sdk import Message

from acp_sdk.client import Client

from acp_sdk.models import MessagePart

from beeai_framework.tools.tool import Tool

from beeai_framework.tools.types import ToolRunOptions

from beeai_framework.context import RunContext

from beeai_framework.emitter import Emitter

from beeai_framework.tools import ToolOutput

from beeai_framework.utils.strings import to_json

async def run_agent(agent: str, input: str) -> list[Message]:

async with Client(base_url="http://localhost:8001") as client:

    run = await client.run_sync(

    agent=agent, input=[Message(parts=[MessagePart(content=input, content_type="text/plain")])]

    )

return run.output

class DatabaseQueryToolInput(BaseModel):

query: str = Field(description="要执行的 SQL 查询或问题")

class DatabaseQueryToolResult(BaseModel):

result: str = Field(description="数据库查询执行结果")

class DatabaseQueryToolOutput(ToolOutput):

result: DatabaseQueryToolResult = Field(description="数据库查询执行结果")

def get_text_content(self) -> str:

    return to_json(self.result)

def is_empty(self) -> bool:

    return self.result.result.strip() == ""

def __init__(self, result: DatabaseQueryToolResult) -> None:

    super().__init__()

    self.result = result

class DatabaseQueryTool(Tool[DatabaseQueryToolInput, ToolRunOptions, DatabaseQueryToolOutput]):

name = "数据库查询执行器"

description = "执行对 ClickHouse 数据库的 SQL 查询和问题"

input_schema = DatabaseQueryToolInput

def _create_emitter(self) -> Emitter:

    return Emitter.root().child(

    namespace=["tool", "database_query"],

    creator=self,

    )

async def _run(self, input: DatabaseQueryToolInput, options: ToolRunOptions | None, context: RunContext) -> DatabaseQueryToolOutput:

    result = await run_agent("db_agent", input.query)

    return DatabaseQueryToolOutput(result=DatabaseQueryToolResult(result=str(result[0])))

```py

Now let’s put together the main agent that will be orchestrating the others as tools. We will use the ReAct agent implementation from the BeeAI framework for the orchestrator. I’ve also added some extra logging to the tool wrappers around our DB and SQL agents, so that we can see all the information about the calls.

```

from collections.abc import AsyncGenerator

from acp_sdk import Message

from acp_sdk.models import MessagePart

from acp_sdk.server import Context, Server

from beeai_framework.backend.chat import ChatModel

from beeai_framework.agents.react import ReActAgent

from beeai_framework.memory import TokenMemory

from beeai_framework.utils.dicts import exclude_none

from sql_tool import SqlQueryTool

from db_tool import DatabaseQueryTool

import os

import logging

# 配置日志记录

logger = logging.getLogger(__name__)

logger.setLevel(logging.INFO)

# 只在不存在处理器时添加处理器

if not logger.handlers:

handler = logging.StreamHandler()

handler.setLevel(logging.INFO)

formatter = logging.Formatter('ORCHESTRATOR - %(levelname)s - %(message)s')

handler.setFormatter(formatter)

logger.addHandler(handler)

# 防止消息传播以避免重复消息

logger.propagate = False

# 使用额外的日志记录来包装我们的工具以提高可追溯性

class LoggingSqlQueryTool(SqlQueryTool):

async def _run(self, input, options, context):

    logger.info(f"🔍 SQL 工具请求: {input.question}")

    result = await super()._run(input, options, context)

    logger.info(f"📝 SQL 工具响应: {result.result.sql_query}")

    return result

class LoggingDatabaseQueryTool(DatabaseQueryTool):

async def _run(self, input, options, context):

    logger.info(f"🗄️ 数据库工具请求: {input.query}")

    result = await super()._run(input, options, context)

    logger.info(f"📊 数据库工具响应: {result.result.result}...")

    return result

server = Server()

@server.agent(name="orchestrator")

async def orchestrator(input: list[Message], context: Context) -> AsyncGenerator:

logger.info(f"🚀 Orchestrator 启动，输入: {input[0].parts[0].content}")

llm = ChatModel.from_name("openai:gpt-4o-mini")

agent = ReActAgent(

    llm=llm,

    tools=[LoggingSqlQueryTool(), LoggingDatabaseQueryTool()],

    templates={

        "system": lambda template: template.update(

        defaults=exclude_none({

            "instructions": """

                您是一位专家数据分析助手，帮助用户分析电子商务数据。

                您可以访问两个工具：

                1\. SqlQueryTool - 使用此工具从关于电子商务数据库的自然语言问题生成 SQL 查询

                2\. 数据库查询工具 - 使用此工具直接在 ClickHouse 数据库中执行 SQL 查询

                数据库包含两个主要表：

                - ecommerce.users (客户信息)

                - ecommerce.sessions (用户会话和交易)

                当用户提出问题时：

                1\. 首先，使用 SqlQueryTool 生成适当的 SQL 查询

                2\. 然后，使用 DatabaseQueryTool 执行该查询并获取结果

                3\. 以清晰、易懂的格式呈现结果

                总是提供关于数据展示和可以得出的见解的背景信息。

            """,

            "role": "system"

        })

    )

    }, memory=TokenMemory(llm))

prompt = (str(input[0]))

logger.info(f"🤖 正在运行 ReAct 代理，提示：{prompt}")

response = await agent.run(prompt)

logger.info(f"✅ Orchestrator 完成。响应长度：{len(response.result.text)} 个字符")

logger.info(f"📤 最终响应：{response.result.text}...")

yield Message(parts=[MessagePart(content=response.result.text)])

if __name__ == "__main__":

server.run(port=8003)

```py

Now, just like before, we can run the orchestrator agent using the ACP client to see the result.

```

async def router_example() -> None:

async with Client(base_url="http://localhost:8003") as orchestrator_client:

    question = '我们在 2024 年 5 月有多少客户？'

    response = await orchestrator_client.run_sync(

    agent="orchestrator", input=question

    )

    print('Orchestrator 响应：')

    # 调试：打印响应结构

    print(f"响应类型：{type(response)}")

    print(f"响应输出长度：{len(response.output) if hasattr(response, 'output') else 'No output attribute'}")

    if response.output and len(response.output) > 0:

    print(response.output[0].parts[0].content)

    else:

    print("未从 orchestrator 收到响应")

    print(f"完整响应：{response}")

asyncio.run(router_example())

# 在 2024 年 5 月，我们拥有 234,544 位独特的活跃客户。

```py

Our system worked well, and we got the expected result. Good job!

Let’s see how it worked under the hood by checking the logs from the orchestrator server. The router first invoked the SQL agent as a SQL tool. Then, it used the returned query to call the DB agent. Finally, it produced the final answer. 

```

ORCHESTRATOR - INFO - 🚀 Orchestrator 启动，输入：我们在 2024 年 5 月有多少客户？

ORCHESTRATOR - INFO - 🤖 正在运行 ReAct 代理，提示：我们在 2024 年 5 月有多少客户？

ORCHESTRATOR - INFO - 🔍 SQL 工具请求：我们在 2024 年 5 月有多少客户？

ORCHESTRATOR - INFO - 📝 SQL 工具响应：

SELECT COUNT(uniqExact(u.user_id)) AS active_customers

FROM ecommerce.users AS u

JOIN ecommerce.sessions AS s ON u.user_id = s.user_id

WHERE u.is_active = 1

AND s.action_date >= '2024-05-01'

AND s.action_date < '2024-06-01'

格式：TabSeparatedWithNames

ORCHESTRATOR - INFO - 🗄️ 数据库工具请求：

SELECT COUNT(uniqExact(u.user_id)) AS active_customers

FROM ecommerce.users AS u

JOIN ecommerce.sessions AS s ON u.user_id = s.user_id

WHERE u.is_active = 1

AND s.action_date >= '2024-05-01'

AND s.action_date < '2024-06-01'

格式：TabSeparatedWithNames

ORCHESTRATOR - INFO - 📊 数据库工具响应：234544...

ORCHESTRATOR - INFO - ✅ Orchestrator 完成。响应长度：52 个字符

ORCHESTRATOR - INFO - 📤 最终响应：2024 年 5 月，我们拥有 234,544 位独特的活跃客户....

```

多亏了我们添加的额外日志，我们现在可以追踪 orchestrator 所做的所有调用。

> *您可以在 [GitHub](https://github.com/miptgirl/acp-kpis-explainer) 上找到完整的代码。*

## 摘要

在本文中，我们探讨了 ACP 协议及其功能。以下是关键点的快速回顾：

+   ACP（代理通信协议）是一个旨在标准化代理之间通信的开放协议。它补充了 MCP，后者处理代理与外部工具和数据源之间的交互。

+   ACP 采用客户端-服务器架构并使用 RESTful API。

+   该协议不依赖于特定技术和框架，允许你构建可互操作的系统，并在代理之间无缝创建新的协作。

+   使用 ACP，你可以实现各种代理交互，从在定义良好的工作流程中的简单链式操作到路由模式，其中协调器可以动态地将任务委派给其他代理。

> *感谢阅读。希望这篇文章能给你带来启发。记住爱因斯坦的建议：“重要的是不要停止提问。好奇心有其存在的理由。”愿你的好奇心引导你发现下一个伟大的洞察。*

## 参考文献

这篇文章灵感来源于 *DeepLearning.AI* 的短课程[*“ACP: Agent Communication Protocol”*](https://www.deeplearning.ai/short-courses/acp-agent-communication-protocol/)。
