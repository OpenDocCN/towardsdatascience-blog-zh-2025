# 从零开始构建 AI 代理：多代理系统

> 原文：[`towardsdatascience.com/ai-agents-from-zero-to-hero-part-3/`](https://towardsdatascience.com/ai-agents-from-zero-to-hero-part-3/)

## **<mdspan datatext="el1743121445633" class="mdspan-comment">Intro</mdspan>**

在本教程系列的[第一部分](https://towardsdatascience.com/ai-agents-from-zero-to-hero-part-1/)中，我们介绍了**AI 代理**，这些是执行任务、做出决策并与他人通信的自主程序。

在本教程系列的[第二部分](https://towardsdatascience.com/ai-agents-from-zero-to-hero-part-2/)中，我们了解了如何通过**迭代和链**使代理尝试并重试直到任务完成。

单个代理通常可以使用工具有效地操作，但在同时使用多个工具时可能效果较差。解决复杂任务的一种方法是通过“分而治之”的方法：为每个任务创建一个专门的代理，并让他们作为一个**多代理系统（MAS**）一起工作。

在 MAS 中，多个代理协作以实现共同目标，通常解决单个代理单独难以应对的挑战。它们可以以两种主要方式交互：

+   **顺序流程** **–** 代理按特定顺序执行工作，一个接一个。例如，代理 1 完成其任务，然后代理 2 使用结果来完成其任务。当任务相互依赖且必须按步骤完成时，这很有用。

+   **分层流程** **–** 通常，一个高级代理管理整个流程并向专注于特定任务的低级代理下达指令。当最终输出需要来回沟通时，这很有用。

在这个教程中，我将展示如何从头开始构建不同类型的**多代理系统**，从简单到更高级。我将提供一些易于在其他类似情况下应用的 Python 代码（只需复制、粘贴、运行），并逐行带注释地解释代码，以便您可以复制此示例（文章末尾有完整代码的链接）。

## **设置**

请参阅[第一部分](https://towardsdatascience.com/ai-agents-from-zero-to-hero-part-1/)以了解[***Ollama***](https://ollama.com/)和主要 LLM 的设置。

```py
import ollama
llm = "qwen2.5" 
```

在这个例子中，我将要求模型处理图像，因此我还需要一个**视觉 LLM**。这是一个专门的大型语言模型版本，它将 NLP 与 CV 集成，旨在理解视觉输入，如图像和视频，以及文本。

微软的[*LLaVa*](https://github.com/haotian-liu/LLaVA)是一个高效的选择，因为它也可以在没有 GPU 的情况下运行。

![](img/3216444cd8f885e5ac090a90058c2447.png)

下载完成后，您可以继续使用 Python 并开始编写代码。让我们加载一张图片，以便我们可以尝试视觉 LLM。

```py
from matplotlib import image as pltimg, pyplot as plt

image_file = "draghi.jpeg"

plt.imshow(pltimg.imread(image_file))
plt.show()
```

为了测试视觉 LLM，你只需将图像作为输入传递：

```py
import ollama

ollama.generate(model="llava",
 prompt="describe the image",                
 images=[image_file])["response"]
```

![](img/ab595a74b27144c50617e0db18852bcb.png)

## ****顺序多代理系统****

我将构建两个代理，它们将以 **顺序流程**工作，一个接一个，第二个代理将第一个代理的输出作为输入，就像一个链一样。

![](img/f744d3503623fd924a035ae9d75c84e3.png)

+   **第一个代理**必须处理用户提供的图像，并返回其看到的口头描述。

+   **第二个代理**将根据第一个代理提供的描述在互联网上搜索，并尝试了解图片是在哪里和什么时候拍摄的。

两个代理各应使用一个 **工具**。第一个代理将使用视觉 LLM 作为工具。请记住，在使用 *Ollama* 时，为了使用工具，必须在字典中描述其功能。

```py
def process_image(path: str) -> str:
    return ollama.generate(model="llava", prompt="describe the image", images=[path])["response"]

tool_process_image = {'type':'function', 'function':{
  'name': 'process_image',
  'description': 'Load an image for a given path and describe what you see',
  'parameters': {'type': 'object',
                'required': ['path'],
                'properties': {
                    'path': {'type':'str', 'description':'the path of the image'},
}}}}
```

第二个代理应该有一个网络搜索工具。在本教程系列的先前文章中，我展示了如何利用 *DuckDuckGo* 包进行网络搜索。所以这次，我们可以使用一个新的工具：[*维基百科*](https://pypi.org/project/wikipedia/) (`pip install wikipedia==1.4.0`)。你可以直接使用原始库或导入 *LangChain* 包装器。

```py
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

def search_wikipedia(query:str) -> str:
    return WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper()).run(query)

tool_search_wikipedia = {'type':'function', 'function':{
  'name': 'search_wikipedia',
  'description': 'Search on Wikipedia by passing some keywords',
  'parameters': {'type': 'object',
                'required': ['query'],
                'properties': {
                    'query': {'type':'str', 'description':'The input must be short keywords, not a long text'},
}}}}
## test
search_wikipedia(query="draghi")
```

![](img/24f28e2a0b47f4454d6f6635ce487230.png)

首先，你需要编写一个提示来描述每个代理的任务（越详细越好），这将是与 LLM 的聊天历史中的第一条消息。

```py
prompt = '''
You are a photographer that analyzes and describes images in details.
'''
messages_1 = [{"role":"system", "content":prompt}]
```

在构建 MAS 时，一个重要的决定是代理是否应该共享聊天历史。**聊天历史的管理**取决于系统的设计和目标：

+   **共享聊天历史** – 代理可以访问一个公共的对话日志，允许他们看到其他代理在之前的交互中说过或做过什么。这可以增强协作和对整体上下文的理解。

+   **单独的聊天历史** – 代理只能访问他们自己的交互，专注于他们自己的沟通。这种设计通常用于独立决策很重要的情况。

我建议除非有必要，否则保持聊天内容分开。由于 LLMs 可能有一个有限的范围窗口，因此最好尽可能使历史记录保持轻量。

```py
prompt = '''
You are a detective. You read the image description provided by the photographer, and you search Wikipedia to understand when and where the picture was taken.
'''

messages_2 = [{"role":"system", "content":prompt}]
```

为了方便起见，我将使用之前文章中定义的函数来处理模型的响应。

```py
def use_tool(agent_res:dict, dic_tools:dict) -> dict:
    ## use tool
    if "tool_calls" in agent_res["message"].keys():
        for tool in agent_res["message"]["tool_calls"]:
            t_name, t_inputs = tool["function"]["name"], tool["function"]["arguments"]
            if f := dic_tools.get(t_name):
                ### calling tool
                print('🔧 >', f"\x1b[1;31m{t_name} -> Inputs: {t_inputs}\x1b[0m")
                ### tool output
                t_output = f(**tool["function"]["arguments"])
                print(t_output)
                ### final res
                res = t_output
            else:
                print('🤬 >', f"\x1b[1;31m{t_name} -> NotFound\x1b[0m")
    ## don't use tool
    if agent_res['message']['content'] != '':
        res = agent_res["message"]["content"]
        t_name, t_inputs = '', ''
    return {'res':res, 'tool_used':t_name, 'inputs_used':t_inputs}
```

正如我们在之前的教程中所做的那样，可以通过一个 *while 循环* 来启动与代理的交互。用户被要求提供一个图像，第一个代理将对其进行处理。

```py
dic_tools = {'process_image':process_image, 
 'search_wikipedia':search_wikipedia}

while True:
    ## user input
    try:
        q = input('📷 > give me the image to analyze:')
    except EOFError:
        break
    if q == "quit":
        break
    if q.strip() == "":
        continue
    messages_1.append( {"role":"user", "content":q} )

    plt.imshow(pltimg.imread(q))
    plt.show()

    ## Agent 1
    agent_res = ollama.chat(model=llm,
 tools=[tool_process_image],
 messages=messages_1)
    dic_res = use_tool(agent_res, dic_tools)    
 res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]

    print("👽📷 >", f"\x1b1;30m{res}\x1b[0m")
    messages_1.append( {"role":"assistant", "content":res} )
```

![

第一个代理使用了视觉 LLM 工具并在图像中识别了文本。现在，描述将被传递给第二个代理，该代理将提取一些关键词以搜索 *维基百科*。

```py
    ## Agent 2
    messages_2.append( {"role":"system", "content":"-Picture: "+res} )

    agent_res = ollama.chat(model=llm,
 tools=[tool_search_wikipedia],
 messages=messages_2)
    dic_res = use_tool(agent_res, dic_tools)    
 res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]   
```

![](img/8fd55642c0a2c05d181f4455be7f2cbc.png)

第二个代理使用了工具，并根据第一个代理提供的描述从网络上提取信息。现在，它可以处理所有信息并给出最终答案。

```py
    if tool_used == "search_wikipedia":
        messages_2.append( {"role":"system", "content":"-Wikipedia: "+res} )
        agent_res = ollama.chat(model=llm, tools=[], messages=messages_2)
        dic_res = use_tool(agent_res, dic_tools)        
 res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"] 
    else:
        messages_2.append( {"role":"assistant", "content":res} )

    print("👽📖 >", f"\x1b1;30m{res}\x1b[0m")
```

![这简直是完美的！让我们继续下一个例子。## ****分层多代理系统**想象一下拥有一支由代理组成的队伍，它们以**分层流程**运作，就像人类团队一样，具有不同的角色以确保顺畅的协作和高效的问题解决。在顶部，经理监督整体战略，与客户（用户）交谈，做出高级决策，并引导团队朝着目标前进。同时，其他团队成员处理操作任务。就像人类一样，代理可以一起工作并适当分配任务。![](img/902fcf98fac54a13fa8601b31a4436ff.png)

我将建立一个由 3 个代理组成的科技团队，目标是根据用户的请求查询 SQL 数据库。他们必须在分层流程中工作：

+   **高级代理**与用户交谈并理解请求。然后，它决定哪个团队成员最适合这项任务。

+   **初级代理**负责探索数据库并构建 SQL 查询。

+   **高级代理**应审查 SQL 代码，如有必要，进行修正并执行。

LLMs 通过接触大量代码和自然语言文本的语料库来学习如何编码，它们学习编程语言的模式、语法和语义。模型通过预测序列中的下一个标记来学习代码不同部分之间的关系。简而言之，LLMs 可以生成 SQL 代码，但不能执行它，代理可以。

首先，我将创建一个数据库并连接到它，然后我将准备一系列**用于执行 SQL 代码的工具**。

```py
## Read dataset
import pandas as pd

dtf = pd.read_csv('http://bit.ly/kaggletrain')
dtf.head(3)

## Create dbimport sqlite3
dtf.to_sql(index=False, name="titanic",
 con=sqlite3.connect("database.db"),            
 if_exists="replace")

## Connect db
from langchain_community.utilities.sql_database import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///database.db")
```

![](img/97869535248efaa9e927db63e949eb6a.png)

让我们从**初级代理**开始。LLMs 不需要工具来生成 SQL 代码，但代理不知道表名和结构。因此，我们需要提供工具来调查数据库。

```py
from langchain_community.tools.sql_database.tool import ListSQLDatabaseTool

def get_tables() -> str:
    return ListSQLDatabaseTool(db=db).invoke("")

tool_get_tables = {'type':'function', 'function':{
  'name': 'get_tables',
  'description': 'Returns the name of the tables in the database.',
  'parameters': {'type': 'object',
                'required': [],
                'properties': {}
}}}

## test
get_tables()
```

这将显示数据库中的可用表，这将打印出表中的列。

```py
from langchain_community.tools.sql_database.tool import InfoSQLDatabaseTool

def get_schema(tables: str) -> str:
    tool = InfoSQLDatabaseTool(db=db)
    return tool.invoke(tables)

tool_get_schema = {'type':'function', 'function':{
  'name': 'get_schema',
  'description': 'Returns the name of the columns in the table.',
  'parameters': {'type': 'object',
                'required': ['tables'],
                'properties': {'tables': {'type':'str', 'description':'table name. Example Input: table1, table2, table3'}}
}}}

## test
get_schema(tables='titanic')
```

由于此代理必须使用多个可能失败的工具，我将编写一个坚实的提示，遵循前一篇文章的结构。

```py
prompt_junior = '''
[GOAL] You are a data engineer who builds efficient SQL queries to get data from the database.

[RETURN] You must return a final SQL query based on user's instructions.

[WARNINGS] Use your tools only once.

[CONTEXT] In order to generate the perfect SQL query, you need to know the name of the table and the schema.
First ALWAYS use the tool 'get_tables' to find the name of the table.
Then, you MUST use the tool 'get_schema' to get the columns in the table.
Finally, based on the information you got, generate an SQL query to answer user question.
'''
```

转到**高级代理**。代码检查不需要任何特殊技巧，只需使用 LLM 即可。

```py
def sql_check(sql: str) -> str:
    p = f'''Double check if the SQL query is correct: {sql}. You MUST just SQL code without comments'''
    res = ollama.generate(model=llm, prompt=p)["response"]
    return res.replace('sql','').replace('```','').replace('\n',' ').strip()

tool_sql_check = {'type':'function', 'function':{

'name': 'sql_check',

'description': '在执行查询之前，始终审查 SQL 查询，并在必要时修正代码',

'parameters': {'type': 'object',

'required': ['sql'],

'properties': {'sql': {'type':'str', 'description':'SQL 代码'}}

}}}

## test

sql_check(sql='SELECT * FROM titanic TOP 3')

```py

![](img/10c5530d53c160cec71f1b1505af8e80.png)

Executing code on the database is a different story: LLMs can’t do that alone.

```

from langchain_community.tools.sql_database.tool import QuerySQLDataBaseTool

def sql_exec(sql: str) -> str:

return QuerySQLDataBaseTool(db=db).invoke(sql)

tool_sql_exec = {'type':'function', 'function':{

'name': 'sql_exec',

'description': '执行 SQL 查询',

'parameters': {'type': 'object',

'required': ['sql'],

'properties': {'sql': {'type':'str', 'description':'SQL 代码'}}

}}}

## test

sql_exec(sql='SELECT * FROM titanic LIMIT 3')

```py

![](img/3539d8f4421864f8cfdec7677d0590f9.png)

And of course, a good prompt.

```

prompt_senior = '''[GOAL] 您是一位高级数据工程师，负责审查和执行他人编写的 SQL 查询。

[RETURN] 您必须从数据库返回数据。

[WARNINGS] 只使用一次您的工具。

[CONTEXT] 在数据库上执行之前，始终检查 SQL 代码。首先始终使用工具'sql_check'来审查查询。该工具的输出是正确的 SQL 查询。您必须仅使用正确的 SQL 查询来使用工具'sql_exec'。'''

```py

Finally, we shall create the **Lead Agent**. It has the most important job: invoking other Agents and telling them what to do. There are many ways to achieve that, but I find creating a simple Tool the most accurate one.

```

def invoke_agent(agent:str, instructions:str) -> str:

return agent+" - "+instructions if agent in ['junior','senior'] else f"Agent '{agent}' Not Found"

tool_invoke_agent = {'type':'function', 'function':{

'name': 'invoke_agent',

'description': '调用另一个 Agent 为您工作。',

'parameters': {'type': 'object',

'required': ['agent', 'instructions'],

'properties': {

'agent': {'type':'str', 'description':'Agent 名称，可以是"junior"或"senior"。'},

'instructions': {'type':'str', 'description':'Agent 的详细说明。'}

}

}}}

## test

invoke_agent(agent="intern", instructions="构建查询")

```py

Describe in the prompt what kind of behavior you’re expecting. Try to be as detailed as possible, for hierarchical Multi-Agent Systems can get very confusing.

```

prompt_lead = '''

[GOAL] 您是一位技术负责人。

您有一个团队，包括一个名为'junior'的初级数据工程师和一个名为'senior'的高级数据工程师。

[RETURN] 您必须根据用户请求从数据库返回数据。

[WARNINGS] 您是唯一与用户交谈并接收用户请求的人。

'junior'数据工程师仅构建查询。

'senior'数据工程师检查查询并执行它们。

[CONTEXT] 首先始终询问用户他们想要什么。

然后，您必须使用工具'invoke_agent'将指令传递给'junior'以构建查询。

最后，您必须使用工具'invoke_agent'将指令传递给'senior'以从数据库检索数据。

'''

```py

I shall keep chat history separate so each Agent will know only a specific part of the whole process.

```

dic_tools = {'get_tables':get_tables,

'get_schema':get_schema,

'sql_exec':sql_exec,

'sql_check':sql_check,

'Invoke_agent':invoke_agent}

messages_junior = [{"role":"system", "content":prompt_junior}]

messages_senior = [{"role":"system", "content":prompt_senior}]

messages_lead   = [{"role":"system", "content":prompt_lead}]

```py

Everything is ready to **start the workflow**. After the user begins the chat, the first to respond is the Leader, which is the only one that directly interacts with the human.

```

while True:

## 用户输入

q = input('🙂 >')

if q == "quit":

break

messages_lead.append( {"role":"user", "content":q} )

## 领导 Agent

agent_res = ollama.chat(model=llm, messages=messages_lead, tools=[tool_invoke_agent])

dic_res = use_tool(agent_res, dic_tools)

res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]

agent_invoked = res.split("-")[0].strip() if len(res.split("-")) > 1 else ''

instructions = res.split("-")[1].strip() if len(res.split("-")) > 1 else ''

###-->在此处调用其他 Agent 的代码<--###

## 领导 Agent 最终响应    print("👩‍💼 >", f"\x1b1;30m{res}\x1b[0m")    messages_lead.append( {"role":"assistant", "content":res} )

```py

![

The Lead Agent decided to invoke the Junior Agent giving it some instruction, based on the interaction with the user. Now the Junior Agent shall start working on the query.

```

## 调用初级 Agent

if agent_invoked == "junior":

print("😎 >", f"\x1b[1;32mReceived instructions: {instructions}\x1b[0m")

messages_junior.append( {"role":"user", "content":instructions} )

### 使用工具

available_tools = {"get_tables":tool_get_tables, "get_schema":tool_get_schema}

context = ''

while available_tools:

agent_res = ollama.chat(model=llm, messages=messages_junior,

tools=[v for v in available_tools.values()])

dic_res = use_tool(agent_res, dic_tools)

res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]

if tool_used:

available_tools.pop(tool_used)

context = context + f"\n 使用的工具: {tool_used}. 输出: {res}" #->添加工具使用上下文

messages_junior.append( {"role":"user", "content":context} )

### 响应

agent_res = ollama.chat(model=llm, messages=messages_junior)

dic_res = use_tool(agent_res, dic_tools)

res = dic_res["res"]

print("😎 >", f"\x1b1;32m{res}\x1b[0m")

messages_junior.append( {"role":"assistant", "content":res} )

```py

![

The Junior Agent activated all its Tools to explore the database and collected the necessary information to generate some SQL code. Now, it must report back to the Lead.

```

## 更新主代理

context = "初级代理已经写下了此查询: "+res+ "\n 现在调用高级代理进行审查和执行代码。"

print("👩‍💼 >", f"\x1b[1;30m{context}\x1b[0m")

messages_lead.append( {"role":"user", "content":context} )

agent_res = ollama.chat(model=llm, messages=messages_lead, tools=[tool_invoke_agent])

dic_res = use_tool(agent_res, dic_tools)

res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]

agent_invoked = res.split("-")[0].strip() if len(res.split("-")) > 1 else ''

instructions = res.split("-")[1].strip() if len(res.split("-")) > 1 else ''

```py

![](img/7345bedd102c9d9802362e40d4d6d6be.png)

The Lead Agent received the output from the Junior and asked the Senior Agent to review and execute the SQL query.

```

## 调用高级代理

if agent_invoked == "senior":

print("🧓 >", f"\x1b[1;34m 收到指令: {instructions}\x1b[0m")

messages_senior.append( {"role":"user", "content":instructions} )

### 使用工具

available_tools = {"sql_check":tool_sql_check, "sql_exec":tool_sql_exec}

context = ''

while available_tools:

agent_res = ollama.chat(model=llm, messages=messages_senior,

tools=[v for v in available_tools.values()])

dic_res = use_tool(agent_res, dic_tools)

res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]

if tool_used:

available_tools.pop(tool_used)

context = context + f"\n 使用的工具: {tool_used}. 输出: {res}" #->添加工具使用上下文

messages_senior.append( {"role":"user", "content":context} )

### 响应

print("🧓 >", f"\x1b1;34m{res}\x1b[0m")

messages_senior.append( {"role":"assistant", "content":res} )

```py

![

The Senior Agent executed the query on the db and got an answer. Finally, it can report back to the Lead which will give the final answer to the user.

```

### 更新主代理

context = "高级代理返回了此输出: "+res

print("👩‍💼 >", f"\x1b1;30m{context}\x1b[0m")

messages_lead.append( {"role":"user", "content":context} )

```

![

## **结论**

本文已涵盖使用 *Ollama* 从零开始创建多代理系统的基本步骤。有了这些构建块，您已经准备好开始为不同的用例开发自己的 MAS。

**敬请期待第四部分**，我们将深入探讨更多高级示例。

本文章的全代码：[**GitHub**](https://github.com/mdipietro09/GenerativeAI/blob/main/Agents_ZeroToHero/notebook.ipynb)

我希望您喜欢！如有任何疑问和反馈，请随时联系我，或者只是分享您有趣的项目。

👉 [**让我们联系**](https://maurodp.carrd.co/) 👈

![](img/f8402e36b94999e5758de93fa3bcbd19.png)

所有图片，除非另有说明，均为作者所有
