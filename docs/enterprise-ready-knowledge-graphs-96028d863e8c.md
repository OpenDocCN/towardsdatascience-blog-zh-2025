# 如何在几分钟内构建知识图谱（并使其企业级就绪）

> 原文：[`towardsdatascience.com/enterprise-ready-knowledge-graphs-96028d863e8c/`](https://towardsdatascience.com/enterprise-ready-knowledge-graphs-96028d863e8c/)

![由 detait 在 Unsplash 上的图片](img/8903bdb9f41bdaf228ae52490ae133f6.png)

图片由[detait](https://unsplash.com/@detait?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)提供

起初，知识图谱（KGs）听起来令人畏惧——不是概念，而是构建的过程。

我之前尝试构建一个知识图谱，但失败了。

图表无疑是表示复杂关系最好的方式之一。它们有很多用途，比如推荐系统和欺诈检测。然而，我最感兴趣的是信息检索。

我开始使用知识图谱来构建更好的 RAGs。

RAGs（关系抽取模型）并不一定需要知识图谱。它们甚至不需要数据库。只要你能从大量信息中提取相关内容并将其传递给 LLM（大型语言模型）的上下文，RAGs 就能工作。

你可以用网络搜索作为其信息检索策略来构建一个 RAG，或者使用向量存储来利用其语义文本搜索功能。

如果你使用图数据库来检索上下文信息，我们称之为 GraphRAG。

*这不是一篇关于 GraphRAGs 的文章（也许在未来的文章中会提到）。这是关于使用 LLM 构建知识图谱本身的文章。* 但值得一提的是，作为内容存储的 KG 如何改进 RAGs。所以，这就是它。

## 为什么 RAG 需要知识图谱？

知识图谱提供了检索更多相关信息的巧妙技术。尽管向量存储通常能发挥作用，但有时会不足。

从向量存储中的主要检索技术是编码文本的语义相似度。以下是它是如何工作的。

我们使用向量嵌入模型，例如[OpenAI 的 text-embedding-3](https://platform.openai.com/docs/guides/embeddings)，来创建文本的向量表示。这使得 Apple 和 Appam（一种印度食物）在它们的向量表示中非常不同，尽管它们共享许多标准字母。然后这些向量被存储在像[Chroma](https://www.trychroma.com/)这样的向量数据库中。

在检索阶段，我们使用与之前相同的嵌入模型对用户输入进行编码。然后我们可以使用距离矩阵，如余弦相似度，从向量存储中检索信息。

这种方法是我们从向量存储中检索信息的唯一方式。正如你可能已经猜到的，我们使用的嵌入模型在检索过程的准确性中起着关键作用。就准确性而言，数据库的类型很少是问题（，但存在并发、速度等问题）。

**以下是一个例子：**

想象一下，你有一份关于各个公司领导团队的庞大文档。

向量嵌入系统有效地处理简单的事实性问题，例如“*约翰·多先生担任 CEO 的是哪个组织？*”，因为答案直接表示在嵌入的文档块中。

然而，更广泛的或分析性查询，例如“*谁与约翰·多先生一起担任多个董事会的董事？*”是具有挑战性的。

> 向量相似性搜索依赖于知识库中的明确提及。知识图谱允许在全局数据集级别进行推理。

最近邻搜索依赖于知识库中的明确提及。没有这样的摘要，基于嵌入的系统难以跨多个来源进行推理或综合信息。

知识图谱允许在全局数据集级别进行推理。它们可以有国家节点和策略节点更紧密地聚集。我们可以运行一个简单的查询来获取我们想要的信息。

既然我们已经了解了为什么知识图谱很重要，那么让我们来谈谈构建知识图谱的挑战。

## 构建知识图谱在过去太复杂了（那时。）

几年前，一位同事向我介绍了知识图谱。他希望为我们的所有项目创建一个统一、可搜索的 KG。

在周末学习了[Neo4J](https://neo4j.com/)之后，它看起来更有希望。

缺少的部分是从大量的 PDF、PowerPoint 幻灯片和 Word 文档中提取节点和边（实体和关系）。

答案并不丰富，因为我们只能手动将非结构化文档转移到图数据模型中。

我们可以使用 PyPDF2 来读取 PDF 文件，并对已知的节点和边进行关键词搜索。然而，这种方法并不成功，所以我们不得不放弃这个想法，并将其标记为“不值得努力”。

随着 LLM 成为我们日常生活的一部分，这种情况已经发生了变化。

在下一节中，我们将借助 LLM 构建一个小型（可能是最简单的）知识图谱。

## 几分钟内构建知识图谱

与过去不同，从文本和图像中提取信息并不那么具有挑战性。

我同意处理非结构化数据需要改进。然而，过去几年，主要是使用 LLM 的发展，开辟了新的可能性。

本节将探讨使用 LLM 构建知识图谱的过程，并讨论如何改进它并使其适用于企业。

因此，我们将使用 Langchain 的一个实验性功能[LLMGraphTransformer](https://python.langchain.com/docs/tutorials/graph/)。我们还使用[Neo4J Auro](https://neo4j.com/product/auradb/)，一个云托管解决方案，作为图数据存储。

如果你正在使用 LlamaIndex，请查看[KnowledgeGraphIndex](https://docs.llamaindex.ai/en/stable/examples/index_structs/knowledge_graph/KnowledgeGraphDemo/)，这是一个与我们这里将要使用的类似 API。你也可以使用各种其他图数据库代替 Neo4J。

让我们先安装所需的包。

```py
pip install neo4j langchain-openai langchain-community langchain-experimental
```

在这个例子中，我们将一组商业领袖、他们所属的组织等映射到图数据库中。如果你想跟上，请找到我使用的样本数据[这里](https://raw.githubusercontent.com/thuwarakeshm/knowledge-graph-example/refs/heads/main/sample.txt)。这是一个我创建的虚拟数据集（当然，使用 AI。）

从非结构化文档中创建知识图谱的出奇简单的代码如下：

```py
import os

from langchain_neo4j import Neo4jGraph
from langchain_openai import ChatOpenAI
from langchain_community.document_loaders import TextLoader
from langchain_experimental.graph_transformers import LLMGraphTransformer

graph = Neo4jGraph(
    url=os.getenv("NEO4J_URL"),
    username=os.getenv("NEO4J_USERNAME", "neo4j"),
    password=os.getenv("NEO4J_PASSWORD"),
)

# ------------------- Important --------------------
llm_transformer = LLMGraphTransformer(
    llm= ChatOpenAI(temperature=0, model_name="gpt-4-turbo")
)
# ------------------- Important --------------------

document = TextLoader("data/sample.txt").load()

graph_documents = llm_transformer.convert_to_graph_documents(document)

graph.add_graph_documents(graph_documents)
```

上述代码非常直接。

我已经突出显示了代码中最关键的部分：构建图数据库。LLMGraphTransformer 类使用我们传递给它的 LLM，并从文档中提取图。

现在，你可以将任何 Langchain Document 类型传递给`convert_to_graph_documents`方法以提取知识图谱。源文件可以是文本、Markdown、网页、另一个数据库查询的响应等。

*如果这是一项手动任务，可能需要几个月的时间，因为就在几年前.*

你可以访问 Aura db 控制台来可视化图。它可能看起来像下面这样：

![由 LLMGraphTransformer 模块在 Neo4J Aura 云上生成的图 - 作者截图](img/7e787c0c0b02245d8f9c616a68945cbd.png)

由 LLMGraphTransformer 模块在 Neo4J Aura 云上生成的图 – 作者截图

在底层，API 使用 LLM 提取相关信息，并为节点和边构建[Neo4J Python 对象](https://neo4j.com/docs/python-manual/current/)。

如我们所知，使用提取 API 构建知识图谱是无力的；让我们讨论一下我们可以做什么来使其适用于企业级应用。

## 如何使知识图谱企业化

我们放弃了构建知识图谱的想法，因为它太复杂了。确实，我们现在可以以显著更少的努力更快地构建 KGs。然而，我们刚刚创建的这个还不适合可靠的应用。

我已经识别出了一些缺陷，我也解决了它们。其中两个值得在这里讨论。

### **1. 获取更多对图提取过程的控制**

如果你正在跟随示例，你会注意到生成的图中只有“Person”和“Organization”类型的节点。本质上，提取只针对了作为董事服务的人和公司。

*注意：使用 LLM 提取图是一种概率技术。你的结果可能与我的不同。*

然而，我们可以从文本文件中提取更多信息。例如，我们可以确定每位高管就读过的大学以及他们的先前经验。

我们能否指定需要提取的实体和关系类型？幸运的是，LLMGraphTransformer 类支持这一点。

这里是初始化实例的另一种方式：

```py
llm_transformer = LLMGraphTransformer(
    llm=llm,
    allowed_nodes=["Person", "Company", "University"],
    allowed_relationships=[
        ("Person", "CEO_OF", "Company"),
        ("Person", "CFO_OF", "Company"),
        ("Person", "CTO_OF", "Company"),
        ("Person", "STUDIED_AT", "University"),
    ],
    node_properties=True,
)
```

在上述版本中，我们明确告诉转换器寻找人、公司和大学类型的实体。我们还教育了处理过程，了解实体之间可以存在的各种关系。这对于 LLM 提取实体来说是一块非常重要的信息。

注意，第三个参数 `node_properties` 帮助转换器收集所有那些彼此之间没有关系的实体的属性。

明确指定节点和关系的知识图谱通常更完整、更准确。但我们不能保证所有关键信息都被正确提取。为此，以下技术可能有所帮助。

### 2. 在图转换之前进行命题

文本是复杂的数据形式。编写这些文本的大脑甚至更奇怪。

有时候，我们不会直接谈论所有事情。即使在正式和技术的写作中，我们也会在不同的地方谈论同一件事。注意，在这篇文章中，我在几个地方使用了“知识图谱”和“KG”这两个词。

这使得 LLM 解释上下文变得困难。

在底层，图转换器仍然对文本进行分块，并在每个分块内独立工作。因此，对其他分块的引用不计。

例如，在文本的开头，你提到了某人的职位是首席投资官（CIO）；然而，在文本分割过程中，定义在后续的分块中丢失了。在后一个分块中，LLM 不知道 CIO 中的 I 代表投资、信息还是其他什么。

为了解决这个问题，我们使用命题法。在分块或处理任何其他文本数据之前，我们对其进行命题，以确保每个分块都能正确解释。

我在我的几篇文章中多次提到了命题法。

> [**如何在 RAG 的块分割中实现接近人类水平的性能**](https://towardsdatascience.com/agentic-chunking-for-rags-091beccd94b1)

尽管如此，以下是操作方法。

```py
obj = hub.pull("wfh/proposal-indexing")

# You can explore the prompt template behind this by running the following:
# obj.get_prompts()[0].messages[0].prompt.template

llm = ChatOpenAI(model="gpt-4o")

# A Pydantic model to extract sentences from the passage
class Sentences(BaseModel):
    sentences: List[str]

extraction_llm = llm.with_structured_output(Sentences)

# Create the sentence extraction chain
extraction_chain = obj | extraction_llm

# Test it out
sentences = extraction_chain.invoke(
    """
    On July 20, 1969, astronaut Neil Armstrong walked on the moon . 
    He was leading the NASA's Apollo 11 mission. 
    Armstrong famously said, "That's one small step for man, one giant leap for mankind" as he stepped onto the lunar surface.
    """
)

>>['On July 20, 1969, astronaut Neil Armstrong walked on the moon.',
 "Neil Armstrong was leading NASA's Apollo 11 mission.",
 'Neil Armstrong famously said, "That's one small step for man, one giant leap for mankind" as he stepped onto the lunar surface.']
```

这段代码使用了来自 Langchain 提示中心的[提示](https://smith.langchain.com/hub/wfh/proposal-indexing?organizationId=65e2223e-316a-5256-b012-5033801a97fa)。正如你在结果中看到的那样，每个分块都是自我解释的，即使没有引用其他分块。

在创建图之前这样做可以帮助 LLM 避免由于引用丢失而丢失节点或关系。

## 最后的想法

我之前尝试构建一个知识图谱并失败了。对于组织来说，拥有它的好处远不及所需的努力。但这是在 LLM 出现之前。

当我了解到 LLM 可以从普通文本中提取图信息并将其放入像 Neo4J 这样的数据库时，我感到非常惊讶。但实验功能并不完美。

我认为它们还没有准备好投入生产——除非进行一些额外的调整。

在这篇文章中，我列举了两种帮助我提高我所构建的知识图谱质量的技术。

* * *

*感谢阅读，朋友！除了 **[Medium](https://thuwarakesh.medium.com/)**，* 我还在 **[LinkedIn](https://www.linkedin.com/in/thuwarakesh/)** 和 **[X,](https://twitter.com/Thuwarakesh)** 上！*
