# LangGraph + SciPy：构建一个阅读文档并做出决策的 AI

> 原文：[`towardsdatascience.com/langgraph-scipy-building-an-ai-that-reads-documentation-and-makes-decisions/`](https://towardsdatascience.com/langgraph-scipy-building-an-ai-that-reads-documentation-and-makes-decisions/)

## 简介

<mdspan datatext="el1754934717839" class="mdspan-comment">统计学和数据科学</mdspan>一直并肩而行，手牵手。

我记得当我开始学习数据科学时，听到过这样的说法：“学习统计学，了解算法背后的东西”。虽然这一切对我来说都很有趣，但也真的很令人不知所措。

事实上，有太多的统计概念、测试和分布需要跟踪。如果你不知道我在说什么，只需访问[Scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html)页面，你就会明白了。

如果你在这个数据科学领域有足够的经验，你可能已经将那些统计测试速查表书签了（甚至打印了）。它们曾经很受欢迎。但现在，大型语言模型正成为我们的“第二大脑”，帮助我们快速查阅我们喜欢的几乎所有信息，并且额外的好处是它会被总结并适应我们的需求。

考虑到这一点，我认为选择正确的统计测试可能会让人困惑，因为它取决于变量类型、假设等。

因此，我想我可以找到一个助手来帮助我。然后，我的项目就形成了。

+   我使用 LangGraph 构建了一个多步骤代理

+   前端是用 Streamlit 构建的

+   代理可以快速查阅 SciPy Stats 文档，并为每种特定情况检索正确的代码。

+   然后，它给我们提供了一个示例 Python 代码

+   它部署在 Streamlit Apps 上，以防你想尝试它。

+   应用链接：[`ai-statistical-advisor.streamlit.app/`](https://ai-statistical-advisor.streamlit.app/)

太棒了！

让我们深入探讨，学习如何构建这个代理。

## LangGraph

LangGraph 是一个库，它通过将大型语言模型（LLMs）表示为图来帮助构建复杂的多步骤应用程序。这种图架构使开发者能够创建条件、循环，这使得它对于创建能够根据前一步的结果做出下一步决策的复杂代理和聊天机器人非常有用。

它本质上将一系列固定的动作转换成一个灵活的、动态的决策过程。在 LangGraph 中，每个节点都是一个函数或工具。

接下来，让我们更深入地了解我们将在本文中创建的代理。

## 统计顾问代理

这个代理是一个统计顾问。所以，主要思想是：

1.  机器人接收一个与统计学相关的问题，例如“*如何比较两组的平均值*”。

1.  它检查问题，并确定是否需要咨询 SciPy 的文档或直接给出答案。

1.  如果需要，代理会在嵌入的 SciPy 文档上使用 RAG 工具

1.  返回一个答案。

1.  如果适用，它将返回一个示例 Python 代码，说明如何执行统计测试。

让我们快速查看 LangGraph 生成的图，以展示这个代理。

![图片](img/ecfde6e40ab5358ba2f146c1669e582b.png)

使用 LangGraph 创建的代理。图片由作者提供。

太好了。现在，让我们直奔主题，开始编码！

## 代码

为了使事情变得简单，我将开发分解成模块。首先，让我们安装我们将需要的包。

```py
pip install chromadb langchain-chroma langchain-community langchain-openai 
langchain langgraph openai streamlit
```

### 块和嵌入

接下来，我们将创建脚本来获取我们的文档并创建文本块，以及嵌入这些块。我们这样做是为了使向量数据库如`ChromaDB`更容易搜索和检索信息。

因此，我创建了此函数`embed_docs()`，您可以在 GitHub 仓库中查看，链接如下[这里](https://github.com/gurezende/AI-Statistical-Advisor/blob/main/rag/embedder.py)。

+   该函数使用 Scipy 的文档（在 BSD 许可下开源）

+   将其分成 500 个标记的块和 50 个标记的重叠。

+   使用`OpenAIEmbedding`进行嵌入（将文本转换为数值，以优化向量数据库搜索）

+   将嵌入保存到`ChromaDB`实例中

现在，数据已准备好作为检索增强生成（RAG）的知识库。但它需要一个可以搜索和找到数据的检索器。这正是`retriever`的作用。

### 检索器

`get_doc_answer()`函数将：

+   加载之前创建的 ChromaDB 实例。

+   创建一个`OpenAI GPT 4o`实例

+   创建一个`retriever`对象

+   在`retrieval_chain`中将一切粘合在一起，该链从用户那里获取问题，将其发送到 LLM

+   该模型使用`retriever`访问 ChromaDB 实例，获取有关统计测试的相关数据，并将答案返回给用户。

现在，我们已经完成了 RAG，文档已嵌入，检索器已就绪。让我们继续到代理节点。

### 代理节点

LangGraph 具有一个有趣的架构，它将每个节点视为一个函数。因此，现在我们必须创建处理代理每个部分的功能。

我们将遵循流程，从`classify_intent`节点开始。由于一些节点需要与 LLM 交互，我们需要生成一个客户端。

```py
from rag.retriever import get_doc_answer
from openai import OpenAI
import os
from dotenv import load_dotenv
load_dotenv()

# Instance of OpenAI
client = OpenAI()
```

一旦我们启动代理，它将接收用户的查询。因此，此节点将检查问题并决定下一个节点将是一个简单响应还是需要搜索 Scipy 的文档。

```py
def classify_intent(state):
    """Check if the user question needs a doc search or can be answered directly."""
    question = state["question"]

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "You are an assistant that decides if a question about statistical tests needs document lookup or not. If it is about definitions or choosing the right test, return 'search'. Otherwise return 'simple'."},
            {"role": "user", "content": f"Question: {question}"}
        ]
    )
    decision = response.choices[0].message.content.strip().lower()

    return {"intent": decision}  # "search" or "simple"
```

如果提出有关统计概念或测试的问题，则激活`retrieve_info()`节点。它在文档中执行 RAG。

```py
def retrieve_info(state):
    """Use the RAG tool to answer from embedded docs."""
    question = state["question"]
    answer = get_doc_answer(question=question)
    return {"rag_answer": answer}
```

一旦从 ChromaDB 检索到适当的文本块，代理将前往下一个节点以生成答案。

```py
def respond(state):
    """Build the final answer."""
    if state.get("rag_answer"):
        return {"final_answer": state["rag_answer"]}
    else:
        return {"final_answer": "I'm not sure how to help with that yet."}
```

最后，最后一个节点是生成代码，如果适用。这意味着，如果有一个可以通过 Scipy 执行的测试的答案，将会有一个示例代码。

```py
def generate_code(state):
    """Generate Python code to perform the recommended statistical test."""
    question = state["question"]
    suggested_test = state.get("rag_answer") or "a statistical test"

    prompt = f"""
    You are a Python tutor. 
    Based on the following user question, generate a short Python code snippet using scipy.stats that performs the appropriate statistical test.

    User question:
    {question}

    Answer given:
    {suggested_test}

    Only output code. Don't include explanations.
    """

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )

    return {"code_snippet": response.choices[0].message.content.strip()}
```

注意这里的一个重要事项：我们节点中的所有函数总是有 `state` 作为参数，因为 **状态是整个工作流程的唯一真相来源**。**图中的每个函数，或“节点”，都从这个中心状态对象中读取并写入。**

例如：

+   `classify_intent` 函数从状态中读取 **问题** 并添加一个 *intent* 键。

+   `retrieve_info` 函数可以读取相同的 **问题** 并添加一个 *rag_answer*，这个 *rag_answer* 最终由响应函数读取以构建 *final_answer*。这个共享的状态字典是不同步骤在代理推理和行动过程中保持连接的方式。

接下来，让我们把所有东西放在一起，构建我们的图！

## 构建图

图就是代理本身。所以，我们在这里基本上是在告诉 LangGraph 我们有哪些节点以及它们是如何相互连接的，这样框架就可以根据这个流程运行信息。

让我们导入模块。

```py
from langgraph.graph import StateGraph, END
from typing_extensions import TypedDict
from langgraph_agent.nodes import classify_intent, retrieve_info, respond, generate_code
```

定义我们的状态模式。记住代理用来连接过程步骤的那个字典？就是它。

```py
# Define the state schema (just a dictionary for now)
class TypedDictState(TypedDict):
    question: str
    intent: str
    rag_answer: str
    code_snippet: str
    final_answer: str
```

在这里，我们将创建一个构建图的函数。

+   为了告诉 LangGraph 过程中的步骤（函数），我们使用 `add_node`

+   一旦我们列出了所有函数，我们就开始创建边，即节点之间的连接。

+   我们通过 `set_entry_point` 开始这个过程。这是第一个要使用的函数。

+   我们使用 `add_edge` 来连接一个节点到另一个节点，第一个参数是从哪个函数获取信息，第二个参数是信息去向。

+   如果我们有条件要遵循，我们使用 `add_conditional_edges`

+   我们使用 `END` 来完成图，并使用 `compile` 来构建它。

```py
def build_graph():
    # Build the LangGraph flow
    builder = StateGraph(TypedDictState)

    # Add nodes
    builder.add_node("classify_intent", classify_intent)
    builder.add_node("retrieve_info", retrieve_info)
    builder.add_node("respond", respond)
    builder.add_node("generate_code", generate_code)

    # Define flow
    builder.set_entry_point("classify_intent")

    builder.add_conditional_edges(
        "classify_intent",
        lambda state: state["intent"],
        {
            "search": "retrieve_info",
            "simple": "respond"
        }
    )

    builder.add_edge("retrieve_info", "respond")
    builder.add_edge("respond", "generate_code")
    builder.add_edge("generate_code", END)

    return builder.compile()
```

我们的图构建函数准备好了，现在我们只需要创建一个美丽的前端，我们可以在这个前端与这个代理进行交互。

现在让我们来做这件事。

## Streamlit 前端

前端是拼图中的最后一部分，我们在这里创建一个用户界面，允许我们轻松地在合适的文本框中输入问题，并正确地看到答案。

我选择了 Streamlit，因为它非常容易原型化和部署。让我们从导入开始。

```py
import os
import time
import streamlit as st
```

然后，我们配置页面的外观。

```py
# Config page
st.set_page_config(page_title="Stats Advisor Agent",
                   page_icon='🤖',
                   layout="wide",
                   initial_sidebar_state="expanded")
```

创建一个侧边栏，用户可以在其中输入他们的 OpenAI API 密钥，以及一个“清除”会话按钮。

```py
# Add a place to enter the API key
with st.sidebar:
    api_key = st.text_input("OPENAI_API_KEY", type="password")

    # Save the API key to the environment variable
    if api_key:
        os.environ["OPENAI_API_KEY"] = api_key

    # Clear
    if st.button('Clear'):
        st.rerun()
```

接下来，我们设置页面标题和说明，并为用户添加一个输入问题的文本框。

```py
# Title and Instructions
if not api_key:
    st.warning("Please enter your OpenAI API key in the sidebar.")

st.title('Statistical Advisor Agent | 🤖')
st.caption('This AI Agent is trained to answer questions about statistical tests from the [Scipy](https://docs.scipy.org/doc/scipy/reference/stats.html) package.')
st.caption('Ask questions like: "What is the best statistical test to compare two means".')
st.divider()

# User question
question = st.text_input(label="Ask me something:",
                         placeholder= "e.g. What is the best test to compare 3 groups means?") 
```

最后，我们可以运行图构建器并在屏幕上显示答案。

```py
# Run the graph
if st.button('Search'):

    # Progress bar
    progress_bar = st.progress(0)

    with st.spinner("Thinking..", show_time=True):

        from langgraph_agent.graph import build_graph
        progress_bar.progress(10)
        # Build the graph
        graph = build_graph()
        result = graph.invoke({"question": question})

        # Progress bar
        progress_bar.progress(50)

        # Print the result
        st.subheader("📖 Answer:")

        # Progress bar
        progress_bar.progress(100)

        st.write(result["final_answer"])

        if "code_snippet" in result:
            st.subheader("💻 Suggested Python Code:")
            st.write(result["code_snippet"]) 
```

让我们看看现在的结果。

![](img/bca2c55267703aba76d0d5c36374266f.png)

哇，结果令人印象深刻！

+   我问：*比较两组均值最好的测试是什么？*

+   答案：*要比较两组数据的均值，通常最合适的测试是独立双样本 t 检验，如果两组数据是独立的且数据呈正态分布。如果数据不是正态分布，则可能更适合使用非参数检验，如曼-惠特尼 U 检验。如果两组数据是配对的或相关的，则配对样本 t 检验是合适的。*

我们提出的创建目标已经完成

### 试试看吧

你想尝试这个代理吗？

现在就试试部署的版本吧！

[`ai-statistical-advisor.streamlit.app`](https://ai-statistical-advisor.streamlit.app)

## 在你离开之前

我知道这是一篇很长的帖子。但我希望它值得你看到最后。我们学到了很多关于 LangGraph 的知识。它让我们以不同的方式思考创建 AI 代理。

该框架迫使我们思考信息的每一步，从问题被提示给 LLM，直到将要显示的答案。在开发过程中，这些问题开始出现在你的脑海中：

+   *用户提问后会发生什么？*

+   *代理在继续之前需要验证某些内容吗？*

+   *在交互过程中需要考虑哪些条件？*

这种架构成为优势，因为它使整个过程更干净、可扩展，因为添加新功能可以像添加新函数（节点）一样简单。

另一方面，LangGraph 不像 Agno 或 CrewAI 这样的框架那样用户友好，后者将这些抽象封装在更简单的方法中，使得学习和开发过程变得更容易，但也降低了灵活性。

最后，这完全取决于要解决的问题以及你需要多灵活。

### GitHub 仓库

[`github.com/gurezende/AI-Statistical-Advisor`](https://github.com/gurezende/AI-Statistical-Advisor)

### 关于我

如果你喜欢这个内容，并想了解更多关于我的工作，请访问我的网站，在那里你还可以找到所有我的联系方式。

[`gustavorsantos.me`](https://gustavorsantos.me)

# 参考资料

[1. LangGraph 文档](https://langchain-ai.github.io/langgraph/concepts/why-langgraph/)

[2. Scipy 统计](https://docs.scipy.org/doc/scipy/reference/stats.html)

[3. Streamlit 文档](https://docs.streamlit.io/)

[4. 统计顾问应用](https://ai-statistical-advisor.streamlit.app/)
