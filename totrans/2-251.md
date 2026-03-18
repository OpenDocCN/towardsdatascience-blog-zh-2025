# 连接点，以获得更好的电影推荐

> 原文：[`towardsdatascience.com/connecting-the-dots-for-better-movie-recommendations/`](https://towardsdatascience.com/connecting-the-dots-for-better-movie-recommendations/)

<mdspan datatext="el1749774191081" class="mdspan-comment">RAG（检索增强生成）的一个承诺是它允许 AI 系统使用最新或特定领域的知识来回答问题，而无需重新训练模型。但大多数 RAG 管道仍然将文档和信息视为扁平且不连贯的——基于向量相似性检索孤立的片段，没有任何关于这些片段之间关系的概念。

为了弥补 RAG 对文档和片段之间——通常是显而易见——的连接的忽视，开发者们转向了图 RAG 方法，但往往发现图 RAG 的好处[不值得实现它的额外复杂性](https://medium.com/data-science/the-quest-for-production-quality-graph-rag-easy-to-start-hard-to-finish-46ca404cee3d)。

在我们最近关于[开源 Graph RAG 项目和 GraphRetriever](https://www.datastax.com/blog/introducing-graph-rag-project-and-graphretriever)的文章中，我们介绍了一种新的、更简单的方法，它将现有的向量搜索与基于轻量级元数据的图遍历相结合，这不需要图构建或存储。图连接可以在运行时——甚至查询时——通过指定您希望用于定义图“边”的文档元数据值来定义，这些连接在图 RAG 的检索过程中被遍历。

在本文中，我们扩展了 Graph RAG 项目文档中的一个用例——[演示笔记本可以在这里找到](https://github.com/datastax/graph-rag/blob/main/docs/examples/movie-reviews-graph-rag.ipynb)，这是一个简单但具有说明性的例子：从 Rotten Tomatoes 数据集中搜索电影评论，自动将每个评论与其相关的本地子图连接起来，然后将具有完整上下文和电影、评论、评论者以及其他数据和元数据属性之间关系的查询响应组合起来。

## 数据集：Rotten Tomatoes 评论和电影元数据

本案例研究使用的数据集来自一个名为“Massive Rotten Tomatoes Movies and Reviews”的公共 Kaggle 数据集[“大量 Rotten Tomatoes 电影和评论”](https://www.kaggle.com/datasets/andrezaza/clapper-massive-rotten-tomatoes-movies-and-reviews)。它包括两个主要的 CSV 文件：

+   rotten_tomatoes_movies.csv ——包含超过 20 万部电影的结构化信息，包括标题、演员、导演、类型、语言、上映日期、时长和票房收入。

+   rotten_tomatoes_movie_reviews.csv ——近 200 万条用户提交的电影评论的集合，包括评论文本、评分（例如，3/5）、情感分类、评论日期以及关联电影的引用。

每个评论都通过共享的 movie_id 与电影相关联，从而在非结构化评论内容和结构化电影元数据之间建立了自然的关系。这使得它成为展示 GraphRetriever 仅使用元数据遍历文档关系的完美候选者——无需手动构建或存储单独的图。

通过将元数据字段如 movie_id、类型，甚至共享的演员和导演视为图边，我们可以构建一个连接的检索流程，该流程可以自动丰富每个查询的相关上下文。

## 挑战：将电影评论置于上下文中

在 AI 驱动的搜索和推荐系统中，一个常见的目标是让用户提出自然、开放的问题，并获得有意义的、上下文相关的结果。在拥有大量电影评论和元数据的大型数据集中，我们希望支持对如下提示的完整上下文响应：

+   “有哪些适合家庭的好电影？”

+   “有哪些激动人心的动作电影的推荐？”

+   “有哪些经典电影拥有令人惊叹的摄影技术？”

对每个这些提示的精彩回答都需要主观的评论内容以及一些半结构化属性，如类型、观众或视觉风格。为了给出一个具有完整上下文的良好回答，系统需要：

1.  根据用户的查询，使用基于向量的语义相似度检索最相关的评论

1.  为每个评论添加完整的电影详情——标题、上映年份、类型、导演等——以便模型可以呈现完整、有根据的推荐

1.  将这些信息与其他评论或电影连接起来，以提供更广泛的上下文，例如：其他评论者说了什么？同类型的其他电影如何比较？

传统的 RAG 管道可能很好地处理第一步——提取相关的文本片段。但是，如果没有了解检索到的片段与数据集中其他信息的关系，模型的响应可能会缺乏上下文、深度或准确性。

## 图 RAG 如何应对挑战

给定一个用户的查询，一个普通的 RAG 系统可能会根据一小部分直接语义相关的评论推荐一部电影。但是，图 RAG 和 GraphRetriever 可以轻松地拉入相关的上下文——例如，同一电影的评论或其他同类型的电影——在做出推荐之前进行比较和对比。

从实现的角度来看，图 RAG 提供了一个干净、两步的解决方案：

### 第一步：构建标准的 RAG 系统

首先，就像任何 RAG 系统一样，我们使用语言模型嵌入文档文本并将嵌入存储在向量数据库中。每个嵌入的评论可能包括结构化元数据，如 reviewed_movie_id、评分和情感——我们将使用这些信息来定义后续的关系。每个嵌入的电影描述包括元数据，如 movie_id、类型、上映年份、导演等。

这允许我们处理典型的基于向量的检索：当用户输入查询“有哪些好的家庭电影？”时，我们可以快速从数据集中检索与家庭电影语义相关的评论。将这些评论与更广泛的上下文连接发生在下一步。

### 第 2 步：使用 GraphRetriever 添加图遍历

在第一步中使用向量搜索检索到语义相关的评论后，我们可以使用 GraphRetriever 来遍历评论及其相关电影记录之间的连接。

具体来说，GraphRetriever：

+   通过语义搜索（RAG）获取相关评论

+   通过基于元数据的边（如 reviewed_movie_id）检索与每个评论直接相关的更多信息，例如电影描述和属性、评论者信息等。

+   将内容合并到单个上下文窗口中，供语言模型在生成答案时使用

一个关键点：不需要预构建的知识图谱。图完全根据元数据定义，并在查询时动态遍历。如果您想扩展连接以包括共享演员、类型或时期，只需更新检索器配置中的边定义即可——无需重新处理或重塑数据。

因此，当用户询问具有某些特定品质的精彩动作电影时，系统可以引入如电影上映年份、类型和演员等数据点，从而提高相关性和可读性。当有人询问具有惊人摄影的经典电影时，系统可以借鉴老电影的评论，并将它们与类型或时代等元数据配对，提供既主观又基于事实的回复。

简而言之，GraphRetriever 在非结构化意见（主观文本）和结构化上下文（连接的元数据）之间架起桥梁——产生更智能、更可信、更完整的查询响应。

## GraphRetriever 的实际应用

为了展示 GraphRetriever 如何将非结构化的评论内容与结构化的电影元数据连接起来，我们通过 Rotten Tomatoes 数据集的一个样本来演示一个基本的设置。这包括三个主要步骤：创建向量存储，将原始数据转换为 LangChain 文档，以及配置图遍历策略。

请参阅[Graph RAG 项目中的示例笔记本](https://github.com/datastax/graph-rag/blob/main/docs/examples/movie-reviews-graph-rag.ipynb)以获取完整的、可工作的代码。

### 创建向量存储和嵌入

我们首先嵌入并存储文档，就像在任何一个 RAG 系统中做的那样。在这里，我们使用 OpenAIEmbeddings 和 Astra DB 向量存储：

```py
from langchain_astradb import AstraDBVectorStore
from langchain_openai import OpenAIEmbeddings

COLLECTION = "movie_reviews_rotten_tomatoes"
vectorstore = AstraDBVectorStore(
    embedding=OpenAIEmbeddings(),
    collection_name=COLLECTION,
)
```

### 数据和元数据的结构

我们像通常为任何 RAG 系统那样存储和嵌入文档内容，但我们还保留了用于图遍历的结构化元数据。文档内容保持最小（评论文本、电影标题、描述），而丰富的结构化数据存储在存储文档对象的“元数据”字段中。

这是向量存储中一个电影文档的示例 JSON：

```py
> pprint(documents[0].metadata)

{'audienceScore': '66',
 'boxOffice': '$111.3M',
 'director': 'Barry Sonnenfeld',
 'distributor': 'Paramount Pictures',
 'doc_type': 'movie_info',
 'genre': 'Comedy',
 'movie_id': 'addams_family',
 'originalLanguage': 'English',
 'rating': '',
 'ratingContents': '',
 'releaseDateStreaming': '2005-08-18',
 'releaseDateTheaters': '1991-11-22',
 'runtimeMinutes': '99',
 'soundMix': 'Surround, Dolby SR',
 'title': 'The Addams Family',
 'tomatoMeter': '67.0',
 'writer': 'Charles Addams,Caroline Thompson,Larry Wilson'}
```

注意，使用 GraphRetriever 进行图遍历仅使用此元数据字段中的属性，不需要专门的图数据库，也不使用任何 LLM 调用或其他昂贵的操作

### 配置和运行 GraphRetriever

GraphRetriever 遍历由元数据连接定义的简单图。在这种情况下，我们使用**reviewed_movie_id**（在评论中）和**movie_id**（在电影描述中）之间的方向关系，从每个评论到其对应的电影定义一个边。

我们使用“贪婪”遍历策略，这是最简单的遍历策略之一。有关策略的更多详细信息，请参阅[Graph RAG 项目的文档](https://datastax.github.io/graph-rag/reference/graph_retriever/strategies/)。

```py
from graph_retriever.strategies import Eager
from langchain_graph_retriever import GraphRetriever

retriever = GraphRetriever(
    store=vectorstore,
    edges=[("reviewed_movie_id", "movie_id")],
    strategy=Eager(start_k=10, adjacent_k=10, select_k=100, max_depth=1),
)
```

在此配置下：

+   `start_k=10`: 使用语义搜索检索 10 个评论文档

+   `adjacent_k=10`: 允许在图遍历的每一步中检索最多 10 个相邻文档

+   `select_k=100`: 可以返回最多 100 个总文档

+   `max_depth=1`: 图仅遍历一个级别，从评论到电影

注意，由于每个评论仅链接到一部特定的电影，在这个简单的例子中，无论此参数如何，图遍历深度都会停止在 1。有关更复杂的遍历示例，请参阅[Graph RAG 项目中的更多示例](https://datastax.github.io/graph-rag/examples/)。

### 调用查询

您现在可以运行自然语言查询，例如：

```py
INITIAL_PROMPT_TEXT = "What are some good family movies?"

query_results = retriever.invoke(INITIAL_PROMPT_TEXT)
```

通过一点排序和文本重新格式化——请参阅笔记本以获取详细信息——我们可以打印出检索到的电影和评论的基本列表，例如：

```py
 Movie Title: The Addams Family
 Movie ID: addams_family
 Review: A witty family comedy that has enough sly humour to keep adults chuckling throughout.

 Movie Title: The Addams Family
 Movie ID: the_addams_family_2019
 Review: ...The film's simplistic and episodic plot put a major dampener on what could have been a welcome breath of fresh air for family animation.

 Movie Title: The Addams Family 2
 Movie ID: the_addams_family_2
 Review: This serviceable animated sequel focuses on Wednesday's feelings of alienation and benefits from the family's kid-friendly jokes and road trip adventures.
 Review: The Addams Family 2 repeats what the first movie accomplished by taking the popular family and turning them into one of the most boringly generic kids films in recent years.

 Movie Title: Addams Family Values
 Movie ID: addams_family_values
 Review: The title is apt. Using those morbidly sensual cartoon characters as pawns, the new movie Addams Family Values launches a witty assault on those with fixed ideas about what constitutes a loving family. 
 Review: Addams Family Values has its moments -- rather a lot of them, in fact. You knew that just from the title, which is a nice way of turning Charles Addams' family of ghouls, monsters and vampires loose on Dan Quayle.
```

然后，我们可以将上述输出传递给 LLM 以生成最终响应，使用评论中的完整信息以及链接的电影。

设置最终提示和 LLM 调用如下所示：

```py
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from pprint import pprint

MODEL = ChatOpenAI(model="gpt-4o", temperature=0)

VECTOR_ANSWER_PROMPT = PromptTemplate.from_template("""

A list of Movie Reviews appears below. Please answer the Initial Prompt text
(below) using only the listed Movie Reviews.

Please include all movies that might be helpful to someone looking for movie
recommendations.

Initial Prompt:
{initial_prompt}

Movie Reviews:
{movie_reviews}
""")

formatted_prompt = VECTOR_ANSWER_PROMPT.format(
    initial_prompt=INITIAL_PROMPT_TEXT,
    movie_reviews=formatted_text,
)

result = MODEL.invoke(formatted_prompt)

print(result.content)
```

图 RAG 系统的最终响应可能如下所示：

> `基于提供的评论，“阿达家庭”和“阿达家庭的价值”被推荐为优秀的家庭电影。“阿达家庭”被描述为一部机智的家庭喜剧，有足够的幽默来娱乐成年人，而“阿达家庭的价值”则因其对家庭动态的巧妙处理和其娱乐时刻而著称。`

请记住，此最终响应是针对提及家庭电影的评论的初始语义搜索的结果——以及与这些评论直接相关的文档的扩展上下文。通过将相关上下文的窗口扩展到简单的语义搜索之外，LLM 和整个图 RAG 系统能够提供更完整、更有帮助的响应。

## 尝试自己操作

本文中的案例研究展示了如何：

+   在您的 RAG 管道中混合非结构化和结构化数据

+   将元数据用作动态知识图，而无需构建或存储

+   通过呈现相关上下文来提高 AI 生成响应的深度和相关性

简而言之，这就是 Graph RAG 的实际应用：为 LLMs 添加结构和关系，使其不仅能够检索，还能更有效地构建上下文和推理。如果你已经在你的文档旁边存储了丰富的元数据，GraphRetriever 为你提供了一种将那些元数据付诸实践的实际方法——无需额外的基础设施。

我们希望这能激发你尝试在自己的数据上使用 GraphRetriever——它全部是开源的——特别是如果你已经在处理通过共享属性、链接或引用隐式连接的文档。

你可以在这里探索完整的笔记本和实现细节：[Graph RAG 在烂番茄电影评论中的应用](https://github.com/datastax/graph-rag/blob/main/docs/examples/movie-reviews-graph-rag.ipynb)。
