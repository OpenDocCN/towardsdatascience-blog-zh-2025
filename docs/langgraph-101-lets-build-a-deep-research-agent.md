# LangGraph 101：让我们构建一个深度研究代理

> 原文：[`towardsdatascience.com/langgraph-101-lets-build-a-deep-research-agent/`](https://towardsdatascience.com/langgraph-101-lets-build-a-deep-research-agent/)

<mdspan datatext="el1755215230074" class="mdspan-comment">构建真正在实践工作中起作用的 LLM 代理并不容易。

你需要考虑如何编排多步骤工作流程，跟踪代理的状态，实施必要的护栏，并监控决策过程。

幸运的是，**LangGraph**正好解决了你这些痛点。

最近，谷歌通过开源使用 LangGraph 和 Gemini（Apache-2.0 许可证）构建的**深度研究代理**的全栈实现，完美地展示了这一点。

这不是一个玩具实现：这个代理不仅能搜索，还能动态评估结果，决定是否需要通过进一步的搜索来获取更多信息。这种迭代工作流程正是 LangGraph 真正发光的地方。

因此，如果你想了解 LangGraph 在实际中的工作方式，还有什么比从这样一个真实、工作着的代理开始更好的地方呢？

**以下是本教程帖子的游戏计划**：我们将采用“**问题驱动**”的学习方法。我们不会从冗长、抽象的概念开始，而是直接进入代码，并检查谷歌的实现。然后，我们将每个部分都与 LangGraph 的核心概念联系起来。

到最后，你不仅将拥有一个工作研究代理，而且还将拥有足够的 LangGraph 知识来构建接下来的一切。

> *我们将在本帖中讨论的所有代码都来自官方谷歌 Gemini 仓库，你可以在这里找到[它](https://github.com/google-gemini/gemini-fullstack-langgraph-quickstart)。我们的重点是后端逻辑（backend/src/agent/目录），其中定义了研究代理。*

这是本帖的视觉路线图：

![](img/b650c207ac282ca5a34fe932f94a044c.png)

图 1. 本帖目录。（图片由作者提供）

* * *

## 1. 整体概念——使用图、节点和边建模工作流程

### 🎯 问题

在这个案例研究中，我们将构建一些令人兴奋的东西：一个**基于 LLM 的** **研究增强型**代理，这是你在 ChatGPT、Gemini、Claude 或 Perplexity 中已经看到的*深度研究*功能的最低限度的复制。这正是我们在这里的目标。

具体来说，我们的代理将这样工作：

> *它接受用户查询，自主搜索网络，检查它获得的结果，然后决定是否已经找到了足够的信息。如果是这样，它将继续创建一个精心制作的迷你报告，并附上适当的引用；否则，它将回到更深层次的搜索中。*

首先，让我们绘制一个高级流程图，以便我们清楚我们在这里要构建什么：

![](img/3dee0c602e2d64afd37ae504db9aa2e7.png)

图 2. 高级流程图（图片由作者提供）

### 💡LangGraph 的解决方案

现在，我们应该如何在 LangGraph 中建模这个工作流程呢？嗯，正如其名所示，LangGraph 使用 **图** 表示。好吧，但为什么使用图呢？

简短的回答是：图非常适合建模复杂的状态流，就像我们在这里旨在构建的应用程序一样。当你有分支决策、需要回环的循环，以及所有其他现实世界代理工作流程会向你抛出的其他混乱现实时，图为你提供了表示它们的最自然方式之一。

从技术上讲，一个图由 **节点** 和 **边** 组成。在 LangGraph 的世界中，**节点是工作流程中的单个处理步骤**，而 **边定义了步骤之间的转换**，即定义控制流和状态如何通过系统流动。

### </> 让我们看看一些代码！

在 LangGraph 中，从流程图到代码的转换是直接的。让我们看看来自 Google 仓库的 `agent/graph.py`，看看这是如何实现的。

第一步是创建图本身：

```py
from langgraph.graph import StateGraph
from agent.state import (
    OverallState,
    QueryGenerationState,
    ReflectionState,
    WebSearchState,
)
from agent.configuration import Configuration

# Create our Agent Graph
builder = StateGraph(OverallState, config_schema=Configuration)
```

这里，`StateGraph` 是 LangGraph 为 **状态感知图** 提供的构建器类。它接受一个 `OverallState` 类，该类定义了可以在节点之间移动的信息（这是我们将在下一节讨论的代理记忆部分），以及一个 `Configuration` 类，该类定义了运行时可调整的参数，例如在各个步骤中调用哪个 LLM、要生成的初始查询数量等。更多细节将在下一节中介绍。

一旦我们有了图容器，我们就可以向其中添加节点：

```py
# Define the nodes we will cycle between
builder.add_node("generate_query", generate_query)
builder.add_node("web_research", web_research)
builder.add_node("reflection", reflection)
builder.add_node("finalize_answer", finalize_answer)
```

`add_node()` 方法将第一个参数作为 **节点的名称**，第二个参数作为当节点运行时执行的 **可调用函数**。

通常，这个可调用函数可以是一个普通函数、一个异步函数、一个 LangChain `Runnable`，甚至另一个编译后的 StateGraph。

在我们的特定情况下：

+   `generate_query` 根据用户的问题生成搜索查询。

+   `web_search` 使用原生的 Google Search API 工具进行网络研究。

+   `reflection` 识别知识差距并生成潜在的后续查询。

+   `finalize_answer` 用于最终确定研究摘要。

我们将在稍后详细检查这些函数的实现。

好吧，现在我们已经定义了节点，下一步是添加边来连接它们并定义执行顺序：

```py
from langgraph.graph import START, END

# Set the entrypoint as `generate_query`
# This means that this node is the first one called
builder.add_edge(START, "generate_query")

# Add conditional edge to continue with search queries in a parallel branch
builder.add_conditional_edges(
    "generate_query", continue_to_web_research, ["web_research"]
)

# Reflect on the web research
builder.add_edge("web_research", "reflection")

# Evaluate the research
builder.add_conditional_edges(
    "reflection", evaluate_research, ["web_research", "finalize_answer"]
)

# Finalize the answer
builder.add_edge("finalize_answer", END)
```

这里有几个值得指出的事情：

+   注意我们之前定义的那些节点名称（例如，“generate_query”，“web_research”等）现在派上了用场——我们可以在我们的边定义中直接引用它们。

+   我们看到使用了两种类型的边，即 **静态边** 和 **条件边**。

+   当使用 `builder.add_edge()` 时，会在两个节点之间创建一个直接的无条件连接。在我们的情况下，`builder.add_edge("web_research", "reflection")`基本上意味着在完成网络研究后，流程将 *始终* 移动到反思步骤。

+   另一方面，当使用`builder.add_conditional_edges()`时，流程可能在运行时跳转到不同的分支。创建条件边时，我们需要三个关键参数：**源节点**、**一个**路由**函数**和**一个可能的目标节点列表**。路由函数检查当前状态，并返回要访问的下一个节点的名称。例如，`evaluate_research()`函数确定代理是否需要更多研究（在这种情况下，转到`"web_research"`节点）或者信息已经足够，代理可以最终确定答案（转到“finalize_answer”节点）。

> *但为什么我们需要在“generate_query”和“web_research”之间设置一个条件边？难道它不应该是一个静态边，因为我们总是希望在生成查询后进行搜索吗？这是一个很好的发现！这实际上与 LangGraph 如何实现并行化有关。我们将在稍后深入讨论。*

+   我们还注意到两个特殊节点：`START`和`END`。这些是 LangGraph 的内置入口和出口点。每个图恰好需要一个起点（执行开始的地方），但可以有多个终点（执行终止的地方）。

最后，是时候将所有内容组合起来，将图编译成一个可执行的代理：

```py
graph = builder.compile(name="pro-search-agent")
```

就这样！我们已经成功将我们的流程图转换成了 LangGraph 的实现。

### 🎁 奖励阅读：为什么图真正闪耀？

除了适合非线性工作流程之外，LangGraph 的节点/边/图表示法还带来了几个额外的实用好处，这使得在现实世界中构建和管理代理变得容易：

+   **细粒度控制与可观察性**。因为每个节点/边都有自己的标识，你可以轻松地检查点你的进度，并在出现意外情况时检查内部。这使得调试和评估变得简单。

+   **模块化与复用**。你可以将单个步骤捆绑成*可重用子图*，就像乐高积木一样。这就是软件最佳实践在行动中的体现。

+   **并行路径**。当你的工作流程中的部分是独立的，图可以轻松地让它们并行运行。显然，这有助于解决延迟问题，并使你的系统对故障更加健壮，这在你的管道复杂时尤其关键。

+   **易于可视化**。无论是调试还是展示方法，能够看到工作流程逻辑总是很棒的。图对于可视化来说很自然。

### 📌关键要点

让我们回顾一下在本节基础部分我们所学到的内容：

+   LangGraph 使用图来描述代理工作流程，因为图优雅地处理分支、循环和其他非线性过程。

+   在 LangGraph 中，节点表示处理步骤，边定义步骤之间的转换。

+   LangGraph 实现了两种类型的边：静态边和条件边。当你有节点之间的固定转换时，使用静态边。如果转换可能在运行时根据动态决策而改变，则使用条件边。

+   在 LangGraph 中构建图很简单。你首先创建一个 StateGraph，然后添加节点（及其函数），用边连接它们。最后，编译图。完成！

![](img/b3713124be64528ca479b01639cb6200.png)

图 3.在 LangGraph 中构建代理图。（图片由作者提供）

现在我们已经了解了基本结构，你可能想知道：这些节点之间是如何传递信息的？这把我们带到了 LangGraph 最重要的概念之一：**状态管理**。

让我们来看看。

* * *

## 2. 代理的记忆——节点如何通过状态共享信息

![](img/0aebf3b85d22e94f4b5664cba599d3a3.png)

图 4.当前进度。（图片由作者提供）

### 🎯 问题

当我们的代理遍历我们之前定义的图时，它需要跟踪它生成/学习的内容。例如：

+   用户提出的问题原文。

+   它生成的搜索查询列表。

+   它从网络上检索到的内容。

+   它对自己收集的信息是否足够充分的内部反思。

+   最终，经过打磨的答案。

那么，我们应该如何维护这些信息，以便我们的节点不是独立工作，而是协作并建立在彼此的工作之上？

### 💡 LangGraph 的解决方案

LangGraph 解决这个问题的方法是通过引入一个中央**状态对象**，一个每个图中的节点都可以查看并写入的共享白板。

这是它的工作方式：

+   当节点执行时，它接收图的当前状态。

+   节点使用状态中的信息执行其任务（例如，调用 LLM，运行工具）。

+   然后，节点返回一个只包含它想要**更新或添加**的状态部分的字典。

+   LangGraph 随后将这个输出自动合并到主状态对象中，然后再传递给下一个节点。

由于状态传递和合并由 LangGraph 在框架级别处理，因此单个节点不需要担心如何访问或更新共享数据。它们只需要专注于它们特定的任务逻辑。

此外，这种模式使你的代理工作流程高度模块化。你可以轻松地添加、删除或重新排序节点，而不会破坏状态流。

### </> 让我们看看一些代码！

记住上一节中的这句话？

```py
# Create our Agent Graph
builder = StateGraph(OverallState, config_schema=Configuration)
```

我们提到`OverallState`定义了代理的记忆，但尚未展示其具体实现方式。现在正是打开黑盒子的好时机。

在仓库中，`OverallState`在`agent/state.py`中定义：

```py
from typing import TypedDict, Annotated, List
from langgraph.graph.message import add_messages
import operator

class OverallState(TypedDict):
    messages: Annotated[list, add_messages]
    search_query: Annotated[list, operator.add]
    web_research_result: Annotated[list, operator.add]
    sources_gathered: Annotated[list, operator.add]
    initial_search_query_count: int
    max_research_loops: int
    research_loop_count: int
    reasoning_model: str
```

实质上，我们可以看到所谓的状态是一个`TypedDict`，它充当一个**合约**。它定义了你的工作流程关心的每个字段以及当多个节点写入这些字段时，这些字段应该如何合并。让我们来分解一下：

+   **字段用途**：`messages`存储对话历史，`search_query`、`web_search_result`和`source_gathered`跟踪代理的研究过程。其他字段通过设置限制和跟踪进度来控制代理的行为。

+   **注解模式**：我们看到一些字段使用`Annotated[list, add_messages]`或`Annotated[list, operator.add]`。这是为了告诉 LangGraph**如何进行合并更新**，当多个节点修改同一字段时。具体来说，`add_messages`是 LangGraph 的内置函数，用于智能合并对话消息，而`operator.add`在节点添加新项目时连接列表。

+   **合并行为**：例如`research_loop_count: int`这样的字段在更新时只是简单地替换旧值。另一方面，注解字段是*累积的*。它们随着时间的推移而积累，因为不同的节点将信息倒入其中。

虽然`OverallState`充当全局内存，但也许定义更小、针对特定节点的状态作为节点所需和产生的清晰“API 合约”会更好。毕竟，通常情况下，一个特定的节点不会需要整个`OverallState`中的所有信息，也不会修改`OverallState`中的所有内容。

这正是 LangGraph 所做的事情。

在`agent/state.py`中，除了定义`OverallState`外，还定义了三个其他状态：

```py
class ReflectionState(TypedDict):
    is_sufficient: bool
    knowledge_gap: str
    follow_up_queries: Annotated[list, operator.add]
    research_loop_count: int
    number_of_ran_queries: int

class QueryGenerationState(TypedDict):
    query_list: list[Query]

class WebSearchState(TypedDict):
    search_query: str
    id: str
```

这些状态是以下方式由节点使用的（`agent/graph.py`）：

```py
from agent.state import (
    OverallState,
    QueryGenerationState,
    ReflectionState,
    WebSearchState,
)

def generate_query(
    state: OverallState, 
    config: RunnableConfig
) -> QueryGenerationState:
    # ...Some logic to generate search queries...
    return {"query_list": result.query}

def continue_to_web_research(
    state: QueryGenerationState
):
    # ...Some logic to send out multiple search queries...

def web_research(
    state: WebSearchState, 
    config: RunnableConfig
) -> OverallState:
    # ...Some logic to performs web research...
    return {
        "sources_gathered": sources_gathered,
        "search_query": [state["search_query"]],
        "web_research_result": [modified_text],
    }

def reflection(
    state: OverallState, 
    config: RunnableConfig
) -> ReflectionState:
    # ...Some logic to reflect on the results...
    return {
        "is_sufficient": result.is_sufficient,
        "knowledge_gap": result.knowledge_gap,
        "follow_up_queries": result.follow_up_queries,
        "research_loop_count": state["research_loop_count"],
        "number_of_ran_queries": len(state["search_query"]),
    }

def evaluate_research(
    state: ReflectionState,
    config: RunnableConfig,
) -> OverallState:
    # ...Some logic to determine the next step in the research flow...

def finalize_answer(
    state: OverallState, 
    config: RunnableConfig) -> OverallState:
    # ...Some logic to finalize the research summary...

    return {
        "messages": [AIMessage(content=result.content)],
        "sources_gathered": unique_sources,
    }
```

以`reflection`节点为例：它从`OverallState`中读取，但返回一个与`ReflectionState`合约匹配的字典。之后，LangGraph 将处理将它们合并到主`OverallState`中的工作，使它们对图中的下一个节点可用。

### 🎁 奖励阅读：我的状态去哪了？

在使用 LangGraph 时，一个常见的混淆是`OverallState`和这些较小的、针对特定节点的状态如何交互。让我们在这里澄清这个混淆。

我们需要拥有的关键心智模型是：在运行时只有一个状态字典，即`OverallState`。

节点特定的`TypedDict`不是额外的运行时数据存储。相反，它们只是对单个基础字典（`OverallState`）的带类型“视图”，暂时放大节点应看到或产生的部分。它们存在的目的是让类型检查器和 LangGraph 运行时可以强制执行清晰的合约。

![](img/9670213883ae6f2165e25be7a0c5786c.png)

图 5.两种状态类型的快速比较。（图片由作者提供）

在节点运行之前，LangGraph 可以使用其类型提示来创建一个包含节点所需输入的“切片”，即`OverallState`的“切片”。

节点运行其逻辑并返回其小型、特定输出字典（例如，一个`ReflectionState`字典）。

LangGraph 接收返回的字典并运行`OverallState.update(return_dict)`。如果有任何键是用聚合器（如`operator.add`）定义的，则应用该逻辑。然后，更新的`OverallState`传递给下一个节点。

那么，为什么 LangGraph 采用了这种两级状态定义？除了为节点强制执行清晰的合约并使节点操作自我文档化之外，还有两个其他好处也值得提及：

+   **即插即用可重用性**：因为节点只宣传它需要和产生的状态的小部分，它成为一个模块化、即插即用的组件。例如，一个只需要从状态中获取`{user_query}`并输出`{queries}`的`generate_query`节点可以被放入另一个完全不同的图中，只要那个图的`OverallState`可以提供一个`user_query`。如果节点针对的是整个全局状态（即，输入和输出都使用`OverallState`类型），那么如果你重命名任何无关的键，你很容易破坏工作流程。这种模块化对于构建复杂系统至关重要。

+   **并行流中的效率**：想象一下，我们的智能体需要同时运行 10 个网络搜索。如果我们使用节点特定的状态作为一个小负载，那么我们只需要将搜索查询发送到每个并行分支。这比将整个智能体内存的副本（记住完整的聊天历史也存储在`OverallState`中！）发送到所有十个分支要高效得多。这样，我们可以显著减少内存和序列化开销。

那么这对我们在实际操作中意味着什么呢？

+   **✔** 在`OverallState`中声明所有需要持久化或对多个不同节点可见的关键键。

+   **✔** 尽可能使节点特定的状态保持最小。它们应该只包含节点负责生成的字段。

+   **✔** 你写入的每个键必须在某个状态模式中声明；否则，当节点尝试写入时，LangGraph 会引发`InvalidUpdateError`。

### 📌关键要点

让我们回顾一下本节中我们所学到的内容：

+   LangGraph 在两个级别上维护状态：在全局级别，有一个 OverallState 对象作为中央内存。在单个节点级别，基于 TypedDict 的小型对象存储节点特定的输入/输出。这保持了状态管理的整洁和组织。

+   在每一步之后，节点会返回最小的输出字典，然后将其合并回中央内存（`OverallState`）。这种合并是根据你的自定义规则（例如，`operator.add`用于列表）进行的。

+   节点是自包含和模块化的。你可以像积木一样轻松地重用它们来创建新的工作流程。

![](img/f10a853a5ea8e9dcfce7f5ff7f98884c.png)

图 6. LangGraph 状态管理中需要记住的关键点。（图片由作者提供）

现在我们已经了解了图的结构以及状态是如何通过它的，但每个节点内部发生了什么？现在让我们转向节点操作。

* * *

## 3. 节点操作 — 真正的工作发生的地方

![](img/4685c5d473831ba08dea399f145b00cc.png)

图 7. 当前进度。（图片由作者提供）

我们的可视化图可以路由消息并保持状态，但每个节点内部，我们仍然需要：

+   确保 LLM 输出正确的格式。

+   调用外部 API。

+   并行运行多个搜索。

+   决定何时停止循环。

幸运的是，LangGraph 有几种可靠的方法来应对这些挑战。让我们一一介绍，每个都通过我们工作代码库的一个片段。

### 3.1 结构化输出

### 🎯 问题

让一个 LLM 返回一个 JSON 对象很容易，但解析自由文本 JSON 在实践中并不可靠。一旦 LLM 使用不同的短语、添加意外的格式或更改键的顺序，我们的工作流程很容易脱轨。简而言之，我们需要在每个处理步骤中保证、可验证的输出结构。

### 💡 LangGraph 的解决方案

我们限制 LLM 生成符合预定义模式的输出。这可以通过使用 `llm.with_structured_output()` 将 **Pydantic** 模式附加到 LLM 调用来实现，这是一个由每个 LangChain 聊天模型包装器提供的辅助方法（例如，`ChatGoogleGenerativeAI`、`ChatOpenAI` 等）。

### </> 让我们看看一些代码！

让我们看看 `generate_query` 节点，其任务是创建一个搜索查询列表。由于我们需要这个列表是一个干净的 Python 对象，而不是一个混乱的字符串，以便下一个节点进行解析，因此强制执行输出模式，使用 `SearchQueryList`（在 `agent/tools_and_schemas.py` 中定义）是一个好主意：

```py
from typing import List
from pydantic import BaseModel, Field

class SearchQueryList(BaseModel):
    query: List[str] = Field(
        description="A list of search queries to be used for web research."
    )
    rationale: str = Field(
        description="A brief explanation of why these queries are relevant to the research topic."
    )
```

下面是如何在 `generate_query` 节点中使用此模式：

```py
from langchain_google_genai import ChatGoogleGenerativeAI
from agent.prompts import (
    get_current_date,
    query_writer_instructions,
)

def generate_query(
    state: OverallState, 
    config: RunnableConfig
) -> QueryGenerationState:
    """LangGraph node that generates a search queries 
       based on the User's question.

    Uses Gemini 2.0 Flash to create an optimized search 
    query for web research based on the User's question.

    Args:
        state: Current graph state containing the User's question
        config: Configuration for the runnable, including LLM 
                provider settings

    Returns:
        Dictionary with state update, including search_query key 
        containing the generated query
    """
    configurable = Configuration.from_runnable_config(config)

    # check for custom initial search query count
    if state.get("initial_search_query_count") is None:
        state["initial_search_query_count"] = configurable.number_of_initial_queries

    # init Gemini 2.0 Flash
    llm = ChatGoogleGenerativeAI(
        model=configurable.query_generator_model,
        temperature=1.0,
        max_retries=2,
        api_key=os.getenv("GEMINI_API_KEY"),
    )
    structured_llm = llm.with_structured_output(SearchQueryList)

    # Format the prompt
    current_date = get_current_date()
    formatted_prompt = query_writer_instructions.format(
        current_date=current_date,
        research_topic=get_research_topic(state["messages"]),
        number_queries=state["initial_search_query_count"],
    )
    # Generate the search queries
    result = structured_llm.invoke(formatted_prompt)
    return {"query_list": result.query}
```

这里，`llm.with_structured_output(SearchQueryList)` 使用 LangChain 的结构化输出辅助程序包装了 Gemini 模型。在底层，它使用模型首选的结构化输出功能（Gemini 2.0 Flash 的 JSON 模式）并自动将回复解析为 `SearchQueryList` Pydantic 实例，因此 `result` 已经是经过验证的 Python 数据。

也很有趣的是查看 Google 为这个节点使用的系统提示：

```py
query_writer_instructions = """Your goal is to generate sophisticated and 
diverse web search queries. These queries are intended for an advanced 
automated web research tool capable of analyzing complex results, following 
links, and synthesizing information.

Instructions:
- Always prefer a single search query, only add another query if the original 
  question requests multiple aspects or elements and one query is not enough.
- Each query should focus on one specific aspect of the original question.
- Don't produce more than {number_queries} queries.
- Queries should be diverse, if the topic is broad, generate more than 1 query.
- Don't generate multiple similar queries, 1 is enough.
- Query should ensure that the most current information is gathered. 
  The current date is {current_date}.

Format: 
- Format your response as a JSON object with ALL three of these exact keys:
   - "rationale": Brief explanation of why these queries are relevant
   - "query": A list of search queries

Example:

Topic: What revenue grew more last year apple stock or the number of people 
buying an iphone
```json

{{

    "rationale": "为了准确回答这个比较增长问题，

我们需要关于苹果股价表现和 iPhone 销售的具体数据点

指标。这些查询针对所需的精确财务信息：

公司收入趋势、特定产品的单位销售数据以及股价

在同一财期内进行直接比较的移动。",

    "query": ["2024 财年苹果总收入增长", "iPhone 单位

2024 财年销售增长", "2024 财年苹果股价增长"],

}}

```py

Context: {research_topic}"""
```

我们可以看到一些提示工程的最佳实践，例如定义模型的角色、指定约束、提供示例进行说明等。

### 3.2 工具调用

### 🎯 问题

为了我们的研究代理能够成功，它需要来自网络的最新信息。为了实现这一点，它需要一个“工具”来搜索网络。

### 💡 LangGraph 的解决方案

节点可以执行工具。这些可以是本地的 LLM 工具调用功能（如 Gemini）或通过 LangChain 的工具抽象集成。一旦收集到工具调用结果，它们可以放回代理的状态中。

### </> 让我们看看一些代码！

对于工具调用的使用模式，让我们看看`web_research`节点。这个节点使用 Gemini 的本地工具调用功能来执行 Google 搜索。注意工具是如何直接在模型的配置中指定的。

```py
from langchain_google_genai import ChatGoogleGenerativeAI
from agent.prompts import (
    web_searcher_instructions,
)
from agent.utils import (
    get_citations,
    insert_citation_markers,
    resolve_urls,
)

def web_research(
    state: WebSearchState, 
    config: RunnableConfig
) -> OverallState:
    """LangGraph node that performs web research using the native Google 
       Search API tool.

    Executes a web search using the native Google Search API tool in 
    combination with Gemini 2.0 Flash.

    Args:
        state: Current graph state containing the search query and 
               research loop count
        config: Configuration for the runnable, including search API settings

    Returns:
        Dictionary with state update, including sources_gathered, 
        research_loop_count, and web_research_results
    """
    # Configure
    configurable = Configuration.from_runnable_config(config)
    formatted_prompt = web_searcher_instructions.format(
        current_date=get_current_date(),
        research_topic=state["search_query"],
    )

    # Uses the google genai client as the langchain client doesn't 
    # return grounding metadata
    response = genai_client.models.generate_content(
        model=configurable.query_generator_model,
        contents=formatted_prompt,
        config={
            "tools": [{"google_search": {}}],
            "temperature": 0,
        },
    )
    # resolve the urls to short urls for saving tokens and time
    resolved_urls = resolve_urls(
        response.candidates[0].grounding_metadata.grounding_chunks, state["id"]
    )
    # Gets the citations and adds them to the generated text
    citations = get_citations(response, resolved_urls)
    modified_text = insert_citation_markers(response.text, citations)
    sources_gathered = [item for citation in citations for item in citation["segments"]]

    return {
        "sources_gathered": sources_gathered,
        "search_query": [state["search_query"]],
        "web_research_result": [modified_text],
    }
```

LLM 看到`Google Search`工具并理解它可以使用这个工具来满足提示。这种本地集成的一个关键好处是响应中返回的`grounding_metadata`。这些元数据包含*grounding chunks*——基本上，是与支持它们的 URL 配对的答案片段。这基本上为我们提供了免费的引用。

### 3.3 条件路由

### 🎯 问题

在初步研究之后，代理如何知道是否停止或继续？我们需要一个控制机制来创建一个可以自我终止的研究循环。

### 💡 LangGraph 的解决方案

条件路由由一种特殊的节点处理：而不是返回状态，这个节点返回要访问的*下一个*节点的名称。实际上，这个节点实现了一个路由功能，它检查当前状态并做出关于如何在图中引导流量的决定。

### </> 让我们看看一些代码！

`evaluate_research`节点是我们代理的决策者。它检查由`reflection`节点设置的`is_sufficient`标志，并将当前的`research_loop_count`值与预配置的最大阈值值进行比较。

```py
def evaluate_research(
    state: ReflectionState,
    config: RunnableConfig,
) -> OverallState:
    """LangGraph routing function that determines the next step in the 
       research flow.

    Controls the research loop by deciding whether to continue gathering 
    information or to finalize the summary based on the configured maximum 
    number of research loops.

    Args:
        state: Current graph state containing the research loop count
        config: Configuration for the runnable, including max_research_loops 
                setting

    Returns:
        String literal indicating the next node to visit 
        ("web_research" or "finalize_summary")
    """
    configurable = Configuration.from_runnable_config(config)
    max_research_loops = (
        state.get("max_research_loops")
        if state.get("max_research_loops") is not None
        else configurable.max_research_loops
    )
    if state["is_sufficient"] or state["research_loop_count"] >= max_research_loops:
        return "finalize_answer"
    else:
        return [
            Send(
                "web_research",
                {
                    "search_query": follow_up_query,
                    "id": state["number_of_ran_queries"] + int(idx),
                },
            )
            for idx, follow_up_query in enumerate(state["follow_up_queries"])
        ]
```

如果满足停止条件，它返回字符串`"finalize_answer"`，LangGraph 随后进入该节点。如果不满足，它返回一个包含`follow_up_queries`的新`Send`对象列表，这会启动另一波并行的网络研究，继续循环。

`Send`对象…那它是什么呢？

嗯，这是 LangGraph 触发并行执行的方式。现在让我们转向这一点。

### 3.4 并行处理

### 🎯 问题

为了尽可能全面地回答用户的查询，我们需要我们的`generate_query`节点生成多个搜索查询。然而，我们不想逐个运行这些搜索查询，因为这会非常慢且效率低下。我们想要的执行所有查询的并行网络搜索。

### 💡 LangGraph 的解决方案

要触发并行执行，一个节点可以返回一个`Send`对象列表。`Send`是一个特殊的指令，告诉 LangGraph 调度器并发地将这些任务调度到指定的节点（例如`"web_research"`），每个任务都有自己的状态。

### </> 让我们看看一些代码！

为了启用并行搜索，Google 的实现引入了`continue_to_web_research`节点作为调度器。它从状态中获取`query_list`并为每个查询创建一个单独的`Send`任务。

```py
from langgraph.types import Send

def continue_to_web_research(
    state: QueryGenerationState
):
    """LangGraph node that sends the search queries to the web research node.
    This is used to spawn n number of web research nodes, one for each 
    search query.
    """
    return [
        Send("web_research", {"search_query": search_query, "id": int(idx)})
        for idx, search_query in enumerate(state["query_list"])
    ]
```

那就是你需要的所有代码。魔法在于这个节点返回之后发生的事情。

当 LangGraph 接收到这个列表时，它足够聪明，不会简单地遍历它。实际上，它在幕后触发了复杂的扇出/扇入过程来并发处理事情：

首先，每个`Send`对象只携带你给它的小型有效负载（`{"search_query": ..., "id": ...}`），而不是整个`OverallState`。这里的目的是快速序列化。

然后，图调度器为列表中的每个项目启动一个`asyncio`任务。这种并发是自动发生的，作为工作流程构建者，你不需要担心编写`async def`或管理线程池。

最后，在所有并行`web_research`分支完成后，它们各自返回的字典会自动合并回主`OverallState`。还记得我们一开始讨论的`Annotated[list, operator.add]`吗？现在它变得至关重要：使用这种类型定义的字段，如`sources_gathered`，将它们的结果连接成一个单一的列表。

你可能想知道：如果并行搜索中的任何一个**失败或超时**会发生什么？这正是我们为每个`Send`有效负载添加自定义`id`的原因。这个 ID 直接流入跟踪日志，允许你定位和调试失败的分支。

如果你还记得之前的内容，我们在图定义中有以下一行：

```py
# Add conditional edge to continue with search queries in a parallel branch
builder.add_conditional_edges(
    "generate_query", continue_to_web_research, ["web_research"]
)
```

你可能会想知道：为什么我们需要将`continue_to_web_research`节点声明为条件边的一部分？

关键要认识到的是：`continue_to_web_research`不仅仅是在管道中的另一个步骤——它是一个**路由函数**。

`generate_query`节点可以返回*零*个查询（当用户询问一些琐碎的问题时）或者二十个。静态边会强制工作流程恰好调用一次`web_research`，即使没有任何事情要做。通过实现为*条件*边`continue_to_web_research`，在运行时决定是否分发，以及，多亏了`Send`，可以生成多少个并行分支。如果`continue_to_web_research`返回一个空列表，LangGraph 就简单地不会跟随这条边。这样就节省了往返搜索 API 的往返。

最后，这又是软件工程最佳实践的体现：`generate_query`关注*搜索什么*，`continue_to_web_research`关注*是否以及如何搜索*，而`web_research`关注*执行搜索*，这是一个清晰的关注点分离。

### 3.5 配置管理

### 🎯 问题

为了节点能够正确地完成它们的工作，它们需要知道，例如：

+   应该使用哪个 LLM 以及什么参数设置（例如，温度）？

+   应该生成多少个初始搜索查询？

+   研究循环的总数和每个运行并发数有什么限制？

+   以及许多其他内容…

简而言之，我们需要一种干净、集中的方式来管理这些设置，而不会使我们的核心逻辑变得杂乱。

### 💡 LangGraph 的解决方案

LangGraph 通过将单个、标准化的`config`传递给每个需要它的节点来解决此问题。这个对象充当运行特定设置的通用容器。

在节点内部，LangGraph 随后使用一个自定义的、有类型的辅助类来智能地解析这个`config`对象。这个辅助类实现了一个清晰的层次结构来获取值：

+   它首先查找当前运行中`config`对象中传递的覆盖值。

+   如果未找到，它将回退到检查环境变量。

+   如果仍然未找到，它将使用直接定义在这个辅助类中的默认值。

### </> 让我们看看一些代码！

让我们看看`反射`节点的实现，看看它是如何工作的。

```py
def reflection(
    state: OverallState, 
    config: RunnableConfig
) -> ReflectionState:
    """LangGraph node that identifies knowledge gaps and generates 
      potential follow-up queries.

    Analyzes the current summary to identify areas for further research 
    and generates potential follow-up queries. Uses structured output to 
    extract the follow-up query in JSON format.

    Args:
        state: Current graph state containing the running summary and 
               research topic
        config: Configuration for the runnable, including LLM provider 
                settings

    Returns:
        Dictionary with state update, including search_query key containing 
        the generated follow-up query
    """
    configurable = Configuration.from_runnable_config(config)
    # Increment the research loop count and get the reasoning model
    state["research_loop_count"] = state.get("research_loop_count", 0) + 1
    reasoning_model = state.get("reasoning_model") or configurable.reasoning_model

    # Format the prompt
    current_date = get_current_date()
    formatted_prompt = reflection_instructions.format(
        current_date=current_date,
        research_topic=get_research_topic(state["messages"]),
        summaries="\n\n---\n\n".join(state["web_research_result"]),
    )
    # init Reasoning Model
    llm = ChatGoogleGenerativeAI(
        model=reasoning_model,
        temperature=1.0,
        max_retries=2,
        api_key=os.getenv("GEMINI_API_KEY"),
    )
    result = llm.with_structured_output(Reflection).invoke(formatted_prompt)

    return {
        "is_sufficient": result.is_sufficient,
        "knowledge_gap": result.knowledge_gap,
        "follow_up_queries": result.follow_up_queries,
        "research_loop_count": state["research_loop_count"],
        "number_of_ran_queries": len(state["search_query"]),
    }
```

在节点中只需要一行样板代码：

```py
configurable = Configuration.from_runnable_config(config)
```

有很多“*config*-ish”术语在流传。让我们逐一解开它们，从`Configuration`开始：

```py
import os
from pydantic import BaseModel, Field
from typing import Any, Optional

from langchain_core.runnables import RunnableConfig

class Configuration(BaseModel):
    """The configuration for the agent."""

    query_generator_model: str = Field(
        default="gemini-2.0-flash",
        metadata={
            "description": "The name of the language model to use for the agent's query generation."
        },
    )

    reflection_model: str = Field(
        default="gemini-2.5-flash-preview-04-17",
        metadata={
            "description": "The name of the language model to use for the agent's reflection."
        },
    )

    answer_model: str = Field(
        default="gemini-2.5-pro-preview-05-06",
        metadata={
            "description": "The name of the language model to use for the agent's answer."
        },
    )

    number_of_initial_queries: int = Field(
        default=3,
        metadata={"description": "The number of initial search queries to generate."},
    )

    max_research_loops: int = Field(
        default=2,
        metadata={"description": "The maximum number of research loops to perform."},
    )

    @classmethod
    def from_runnable_config(
        cls, config: Optional[RunnableConfig] = None
    ) -> "Configuration":
        """Create a Configuration instance from a RunnableConfig."""
        configurable = (
            config["configurable"] if config and "configurable" in config else {}
        )

        # Get raw values from environment or config
        raw_values: dict[str, Any] = {
            name: os.environ.get(name.upper(), configurable.get(name))
            for name in cls.model_fields.keys()
        }

        # Filter out None values
        values = {k: v for k, v in raw_values.items() if v is not None}

        return cls(**values)
```

这是我们之前提到的自定义辅助类。您可以看到 Pydantic 被大量用于定义代理的所有参数。需要注意的是，这个类还定义了一个替代构造方法`from_runnable_config()`。这个构造方法通过从不同来源拉取值，同时强制执行我们在“💡 LangGraph 的解决方案”中讨论的覆盖层次结构，创建一个`Configuration`实例。

`config`是`from_runnable_config()`方法的输入。技术上，它是一个`RunnableConfig`类型，但实际上它只是一个带有可选元数据的字典。在 LangGraph 中，它主要用于以结构化的方式在图中传递上下文信息。例如，它可以携带诸如标签、跟踪选项，以及最重要的在`"configurable"`键下的嵌套覆盖字典。

最后，通过在每个节点中调用：

```py
configurable = Configuration.from_runnable_config(config)
```

我们通过结合三个来源的数据来创建`Configuration`类的实例：首先，`config["configurable"]`，然后是环境变量，最后是类默认值。因此，`configurable`是一个完全初始化、准备好使用的对象，它使节点能够访问所有相关设置，例如`configurable.reflection_model`。

> *Google 原始代码（在反射节点和最终答案节点中）存在一个错误：*

```py
reasoning_model = state.get("reasoning_model") or configurable.reasoning_model
```

> *然而，`reasoning_model`在 configuration.py 中从未定义过。相反，应该根据 configuration.py 的定义使用`reflect_model`和`answer_model`。详细信息请见 PR #46。*

总结一下：`配置`是定义，`config`是运行时输入，而`configurable`是结果，即节点使用的解析后的配置对象。

## 🎁 奖励阅读：我们没有涵盖什么？

LangGraph 提供的功能远不止我们在这个教程中能涵盖的。随着您构建更复杂的代理，您可能会发现自己提出像这些问题：

1\. **我能否使我的应用程序更响应？**

LangGraph 支持**流式传输**，因此您可以逐个输出结果，以提供实时用户体验。

2\. **当 API 调用失败时会发生什么？**

LangGraph 实现了**重试和回退机制**来处理错误。

3\. **如何避免重新运行昂贵的计算？**

如果你的一些节点需要进行昂贵的处理，你可以使用 LangGraph 的**缓存**机制来缓存节点输出。此外，LangGraph 支持**检查点**。此功能允许你保存图的状态并在你停止的地方继续。这对于你有长时间运行的过程并且想要暂停并在以后恢复尤为重要。

4. **我可以实现**人机交互**工作流程吗？**

是的。LangGraph 内置了对**人机交互**工作流程的支持。这使你能够在继续之前暂停图并等待用户输入或批准。

5. **我如何跟踪我的代理行为？**

LangGraph 与**LangSmith**原生集成，它提供了详细的跟踪和可观察性，以最小的设置深入了解你的代理行为。

6. **我的代理如何自动发现并使用新工具？**

LangGraph 支持**MCP**（模型上下文协议）集成。这允许它自动发现并使用遵循此开放标准的工具。

查看 LangGraph[官方文档](https://langchain-ai.github.io/langgraph/concepts/why-langgraph/)获取更多详细信息。

## 📌关键要点

让我们回顾一下本节中我们涵盖的内容：

+   **结构化输出**：使用`.with_structured_output`强制 AI 的响应符合你定义的特定结构。这确保你总是得到干净、可靠的数据，下游步骤可以轻松解析。

+   **工具调用**：你可以在模型调用中嵌入工具，使代理能够与外部世界交互。

+   **条件路由**：这是构建“选择你的冒险”逻辑的方法。一个节点可以通过简单地返回下一个节点的名称来决定下一步去哪里。这样，你可以动态地创建循环和决策点，使你的代理工作流程变得更加智能。

+   **并行处理**：LangGraph 允许你同时触发多个步骤运行。所有将工作扇出和收集结果的繁重工作都由 LangGraph 自动处理。

+   **配置管理**：你不需要在代码中分散设置，可以使用专门的配置类来管理运行时设置、环境变量、默认值等，在一个干净、集中的地方。

![](img/58e3a166631d7a6b1d09a6e828f640ab.png)

图 8.增强 LLM 代理能力各个方面。（图片由作者提供）

* * *

## 4. 结论

在这篇文章中，我们覆盖了很多内容！现在我们已经看到 LangGraph 的核心概念是如何结合在一起构建一个现实世界的研究代理，让我们通过一些关键要点来结束我们的旅程：

+   **图自然描述代理工作流程**。现实世界的工作流程涉及循环、分支和动态决策。LangGraph 基于图架构（节点、边和状态）提供了一个干净直观的方式来表示和管理这种复杂性。

+   **状态是代理的记忆**。中心的`OverallState`对象是一个共享的白板，图中的每个节点都可以查看并写入。与节点特定的状态模式一起，它们构成了代理的记忆系统。

+   **节点是可重用的模块化组件**。在 LangGraph 中，你应该构建具有明确职责的节点，例如生成查询、调用工具或路由逻辑。这使得代理系统更容易测试、维护和扩展。

+   **控制权在你手中**。在 LangGraph 中，你可以通过**条件边**来引导逻辑流程，通过**结构化输出**来确保数据可靠性，使用**集中配置**来全局调整参数，或者使用`Send`来实现任务的并行执行。它们的组合赋予你构建智能、高效和可靠的代理的能力。

现在你已经了解了关于 LangGraph 的所有知识，接下来你想要构建什么？
