# AI 代理处理时间序列和大型数据框

> 原文：[`towardsdatascience.com/ai-agents-processing-timeseries-and-large-dataframes/`](https://towardsdatascience.com/ai-agents-processing-timeseries-and-large-dataframes/)

## <mdspan datatext="el1745296097126" class="mdspan-comment">简介</mdspan>

代理是受 LLM 驱动的 AI 系统，可以对其目标进行推理并采取行动以实现最终目标。它们不仅被设计来响应用户查询，还旨在编排一系列操作，包括处理数据（即数据框和时间序列）。这种能力解锁了众多现实世界的应用，例如自动化报告、无代码查询、数据清洗和操作支持。

可以以两种不同方式与数据框交互的代理：

+   使用 **自然语言** —— LLM 将表格作为字符串读取，并基于其知识库尝试理解其含义

+   通过 **生成和执行代码** —— 代理激活工具以对象的形式处理数据集。

![](img/855b5b5c3620bba73cf85eec6edc45a4.png)因此，通过结合自然语言处理的力量和代码执行的精确性，AI 代理使更广泛的用户能够与复杂的数据集交互并得出见解。

在本教程中，我将展示如何使用 AI 代理 **处理数据框和时间序列**。我将展示一些易于在其他类似情况下应用的 Python 代码（只需复制、粘贴、运行），并逐行带注释地解释代码，以便您可以复制此示例（文章末尾有完整代码的链接）。

## 设置

让我们从设置 [***Ollama***](https://ollama.com/)(`pip install ollama==0.4.7`) 开始，这是一个允许用户在本地运行开源 LLM 的库，无需云服务，从而在数据隐私和性能方面提供更多控制。由于它是在本地运行的，任何对话数据都不会离开您的机器。

首先，您需要从网站下载 *Ollama*。

![](img/d90f38a72bde7bd1406eacedafae8f4e.png)

然后，在您的笔记本电脑的提示壳中，使用命令下载所选的 LLM。我选择阿里巴巴的 ***Qwen***，因为它既智能又轻便。

![](img/81f5386d8c3805c35d9fa511fd494ab4.png)

下载完成后，您可以继续使用 Python 并开始编写代码。

```py
import ollama
llm = "qwen2.5"
```

让我们测试 LLM：

```py
stream = ollama.generate(model=llm, prompt='''what time is it?''', stream=True)
for chunk in stream:
    print(chunk['response'], end='', flush=True)
```

![](img/6959b9689c74fa5ceca40bdad86f38cb.png)

## 时间序列

时间序列是一系列随时间测量的数据点，通常用于分析和预测。它使我们能够看到变量随时间的变化，并用于识别趋势和季节性模式。

我将生成一个假的时间序列数据集作为示例。

```py
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

## create data
np.random.seed(1) #<--for reproducibility
length = 30
ts = pd.DataFrame(data=np.random.randint(low=0, high=15, size=length),
                  columns=['y'],
                  index=pd.date_range(start='2023-01-01', freq='MS', periods=length).strftime('%Y-%m'))

## plot
ts.plot(kind="bar", figsize=(10,3), legend=False, color="black").grid(axis='y')
```

![](img/93843a94dc8353f99ebc9ba3a856f0bc.png)

通常，时间序列数据集具有非常简单的结构，主要变量作为列，时间作为索引。

![](img/8b36161dfdbc8b81dcccf4d811605261.png)

在将其转换为字符串之前，我想确保所有内容都放在一个列下，这样我们就不会丢失任何信息片段。

```py
dtf = ts.reset_index().rename(columns={"index":"date"})
dtf.head()
```

![图片](img/b9f39156e1a55341792d4d5394ee8d2e.png)

然后，我将数据类型**从 dataframe 转换为 dictionary**。

```py
data = dtf.to_dict(orient='records')
data[0:5]
```

![图片](img/ecead947902115628cec97e5eaa18cf2.png)

最后，**从 dictionary 转换为 string**。

```py
str_data = "\n".join([str(row) for row in data])
str_data
```

![图片](img/4cdce3278aa35542fd802f5ff8ff3c18.png)

现在我们有了字符串，它可以被包含在一个任何语言模型都能处理的提示中。当你将数据集粘贴到提示中时，LLM 将数据作为纯文本读取，但仍然可以根据训练期间看到的模式理解结构和意义。

```py
prompt = f'''
Analyze this dataset, it contains monthly sales data of an online retail product:
{str_data}
'''
```

我们可以轻松地与 LLM 开始聊天。请注意，目前这还不是智能体，因为它没有任何工具，我们只是在使用语言模型。虽然它不像计算机那样处理数字，但 LLM 可以识别列名、基于时间的模式、趋势和异常值，尤其是在较小的数据集上。它可以模拟分析和解释发现，但不会独立执行精确的计算，因为它不像智能体那样执行代码。

```py
messages = [{"role":"system", "content":prompt}]

while True:
    ## User
    q = input('🙂 >')
    if q == "quit":
        break
    messages.append( {"role":"user", "content":q} )

    ## Model
    agent_res = ollama.chat(model=llm, messages=messages, tools=[])
    res = agent_res["message"]["content"]

    ## Response
    print("👽 >", f"\x1b1;30m{res}\x1b[0m")
    messages.append( {"role":"assistant", "content":res} )
```

![图片 LLM 识别数字并理解一般上下文，就像它可能理解一个食谱或一行代码一样。![图片](img/7f8cab95728c2ac73305053b5f417300.png)

正如你所见，使用 LLM 分析时间序列对于快速和对话式的洞察力来说是非常好的。

## 智能体

LLMs 适合头脑风暴和轻量级探索，而智能体可以运行代码。因此，它可以处理更复杂的任务，如绘图、预测和异常检测。所以，让我们创建工具。

有时，将**“最终答案”视为一个工具**可能更有效。例如，如果智能体执行多个动作以生成中间结果，最终答案可以被视为将所有这些信息整合成一个连贯响应的工具。通过这种方式设计，你可以对结果有更多的定制和控制。

```py
def final_answer(text:str) -> str:
    return text

tool_final_answer = {'type':'function', 'function':{
  'name': 'final_answer',
  'description': 'Returns a natural language response to the user',
  'parameters': {'type': 'object',
                'required': ['text'],
                'properties': {'text': {'type':'str', 'description':'natural language response'}}
}}}

final_answer(text="hi")
```

然后，是**编码工具**。

```py
import io
import contextlib

def code_exec(code:str) -> str:
    output = io.StringIO()
    with contextlib.redirect_stdout(output):
        try:
            exec(code)
        except Exception as e:
            print(f"Error: {e}")
    return output.getvalue()

tool_code_exec = {'type':'function', 'function':{
  'name': 'code_exec',
  'description': 'Execute python code. Use always the function print() to get the output.',
  'parameters': {'type': 'object',
                'required': ['code'],
                'properties': {
                    'code': {'type':'str', 'description':'code to execute'},
}}}}

code_exec("from datetime import datetime; print(datetime.now().strftime('%H:%M'))")
```

此外，我将添加几个**实用函数**用于工具使用和运行智能体。

```py
dic_tools = {"final_answer":final_answer, "code_exec":code_exec}

# Utils
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

当智能体尝试解决一个任务时，我希望它能记录下已使用的工具、尝试的输入以及得到的结果。迭代只有在模型准备好给出最终答案时才应停止。

```py
def run_agent(llm, messages, available_tools):
    tool_used, local_memory = '', ''
    while tool_used != 'final_answer':
        ### use tools
        try:
            agent_res = ollama.chat(model=llm,
                                    messages=messages,                                                                                                              tools=[v for v in available_tools.values()])
            dic_res = use_tool(agent_res, dic_tools)
            res, tool_used, inputs_used = dic_res["res"], dic_res["tool_used"], dic_res["inputs_used"]
        ### error
        except Exception as e:
            print("⚠️ >", e)
            res = f"I tried to use {tool_used} but didn't work. I will try something else."
            print("👽 >", f"\x1b[1;30m{res}\x1b[0m")
            messages.append( {"role":"assistant", "content":res} )
        ### update memory
        if tool_used not in ['','final_answer']:
            local_memory += f"\nTool used: {tool_used}.\nInput used: {inputs_used}.\nOutput: {res}"
            messages.append( {"role":"assistant", "content":local_memory} )
            available_tools.pop(tool_used)
            if len(available_tools) == 1:
                messages.append( {"role":"user", "content":"now activate the tool final_answer."} )
        ### tools not used
        if tool_used == '':
            break
    return res
```

关于编码工具，我注意到智能体倾向于在每一步都重新创建 dataframe。因此，我将使用**记忆强化**来提醒模型数据集已经存在。这是一个常用的技巧，以获得期望的行为。最终，记忆强化有助于你获得更有意义和有效的交互。

```py
# Start a chat
messages = [{"role":"system", "content":prompt}]
memory = '''
The dataset already exists and it's called 'dtf', don't create a new one.
'''
while True:
    ## User
    q = input('🙂 >')
    if q == "quit":
        break
    messages.append( {"role":"user", "content":q} )

    ## Memory
    messages.append( {"role":"user", "content":memory} )     

    ## Model
    available_tools = {"final_answer":tool_final_answer, "code_exec":tool_code_exec}
    res = run_agent(llm, messages, available_tools)

    ## Response
    print("👽 >", f"\x1b1;30m{res}\x1b[0m")
    messages.append( {"role":"assistant", "content":res} )
```

![图片创建图表是 LLM 单独无法做到的事情。但请记住，即使代理可以创建图像，它们也无法看到它们，因为毕竟，引擎仍然是一个语言模型。所以，用户是唯一一个可视化图表的人。![图片](img/c0bd32f43292d204353db907a483776e.png)

代理正在使用[*statsmodels*](https://www.statsmodels.org/stable/index.html)库来训练模型并预测时间序列。

![图片](img/531b9d7f6eae0c95379b91d5c9c69b17.png)

## 大型数据框

LLMs 的内存有限，这限制了它们一次可以处理多少信息，即使是最高级的模型也有令牌限制（几百页的文本）。此外，除非集成检索系统，否则 LLMs 不会在会话之间保留记忆。在实践中，为了有效地处理大型数据框，开发者通常会使用分块、RAG、向量数据库和总结内容等策略，在将其输入模型之前。

让我们创建一个大型数据集来玩耍。

```py
import random
import string

length = 1000

dtf = pd.DataFrame(data={
    'Id': [''.join(random.choices(string.ascii_letters, k=5)) for _ in range(length)],
    'Age': np.random.randint(low=18, high=80, size=length),
    'Score': np.random.uniform(low=50, high=100, size=length).round(1),
    'Status': np.random.choice(['Active','Inactive','Pending'], size=length)
})

dtf.tail()
```

![图片](img/3076b1c72a05502320cae42679ed64f9.png)

我将添加一个**网络搜索工具**，这样，具备执行 Python 代码和搜索互联网的能力，通用人工智能就能访问所有可用知识，并做出数据驱动的决策。

在 Python 中，创建网络搜索工具最简单的方法是使用著名的私有浏览器[*DuckDuckGo*](https://pypi.org/project/duckduckgo-search/)(`pip install duckduckgo-search==6.3.5`)。您可以直接使用原始库或导入[*LangChain*](https://www.langchain.com/)包装器(`pip install langchain-community==0.3.17`)。

```py
from langchain_community.tools import DuckDuckGoSearchResults

def search_web(query:str) -> str:
  return DuckDuckGoSearchResults(backend="news").run(query)

tool_search_web = {'type':'function', 'function':{
  'name': 'search_web',
  'description': 'Search the web',
  'parameters': {'type': 'object',
                'required': ['query'],
                'properties': {
                    'query': {'type':'str', 'description':'the topic or subject to search on the web'},
}}}}

search_web(query="nvidia")
```

![图片](img/ae8a3c1805de80a214dbe6d60a119f24.png)

总的来说，代理现在有 3 个工具。

```py
dic_tools = {'final_answer':final_answer,
             'search_web':search_web,
             'code_exec':code_exec}
```

由于我无法在提示中添加完整的数据框，我将只提供前 10 行，以便 LLM 可以理解数据集的一般背景。此外，我将指定如何找到完整的数据集。

```py
str_data = "\n".join([str(row) for row in dtf.head(10).to_dict(orient='records')])

prompt = f'''
You are a Data Analyst, you will be given a task to solve as best you can.
You have access to the following tools:
- tool 'final_answer' to return a text response.
- tool 'code_exec' to execute Python code.
- tool 'search_web' to search for information on the internet.

If you use the 'code_exec' tool, remember to always use the function print() to get the output.
The dataset already exists and it's called 'dtf', don't create a new one.

This dataset contains credit score for each customer of the bank. Here's the first rows:
{str_data}
'''
```

最后，我们可以运行代理。

```py
messages = [{"role":"system", "content":prompt}]
memory = '''
The dataset already exists and it's called 'dtf', don't create a new one.
'''
while True:
    ## User
    q = input('🙂 >')
    if q == "quit":
        break
    messages.append( {"role":"user", "content":q} )

    ## Memory
    messages.append( {"role":"user", "content":memory} )     

    ## Model
    available_tools = {"final_answer":tool_final_answer, "code_exec":tool_code_exec, "search_web":tool_search_web}
    res = run_agent(llm, messages, available_tools)

    ## Response
    print("👽 >", f"\x1b1;30m{res}\x1b[0m")
    messages.append( {"role":"assistant", "content":res} )
```

![图片在这次交互中，代理正确地使用了编码工具。现在，我想让它也利用其他工具。![图片](img/be760b347309a8e7a814f32032dfc5d6.png)

最后，我需要代理将到目前为止从这次聊天中获得的所有信息整合在一起。

## 结论

这篇文章是一个教程，展示了**如何从头开始构建处理时间序列和大型数据框的代理**。我们涵盖了模型与数据交互的两种方式：通过自然语言，其中 LLM 使用其知识库将表格解释为字符串，以及通过生成和执行代码，利用工具将数据集作为对象处理。

本文的完整代码：[**GitHub**](https://github.com/mdipietro09/GenerativeAI/blob/main/Agents_ZeroToHero/notebook.ipynb)

我希望您喜欢它！如果您有任何问题或反馈，请随时联系我，或者只是分享您有趣的项目。

👉 [**让我们连接**](https://maurodp.carrd.co/) 👈

![图片](img/ba1b725596a2319415fcdfe06bcba8b2.png)
