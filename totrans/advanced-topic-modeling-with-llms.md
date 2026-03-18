# 使用 LLMs 进行高级主题建模

> 原文：[`towardsdatascience.com/advanced-topic-modeling-with-llms/`](https://towardsdatascience.com/advanced-topic-modeling-with-llms/)

## <mdspan datatext="el1752776036552" class="mdspan-comment">简介</mdspan>

本文是关于从 OpenAlex API 开源情报（OSINT）主题建模的延续。在之前的一篇文章中，我介绍了主题建模、所使用的数据以及使用潜在狄利克雷分配（LDA）的传统自然语言处理（NLP）方法。

*参见之前的文章：*

> [使用 OpenAlex API 进行主题建模开源研究](https://towardsdatascience.com/topic-modeling-open-source-research-with-the-openalex-api-5191c7db9156/)

本文通过利用表示模型、生成式 AI 和其他高级技术，采用了一种更高级的主题建模方法。**我们利用 BERTopic 将几个模型整合到一个流程中，可视化我们的主题，并探索主题模型的变体。**

![图片](img/1416962c4acb8bec1f346a589887ca76.png)

图片由作者提供

*<mdspan datatext="el1753117843766" class="mdspan-comment">所使用的数据来自 OpenAlex API ([`openalex.org/`](https://openalex.org/))。这是一个来自世界各地的开源研究数据集/目录。数据在无保留权许可（CC0 许可）下免费提供。</mdspan>*

* * *

## BERTopic 流程

使用传统的主题建模方法可能很困难，需要构建自己的管道来清理数据、分词、词元化、创建特征等。传统的模型如 LDA 或 LSA 也计算成本高，并且通常结果不佳。

**BERTopic 通过嵌入模型利用 Transformer 架构，并整合了其他组件如降维和主题表示模型，以创建高性能的主题模型。** BERTopic 还提供了多种模型变体以适应各种数据和用例，可视化结果，以及更多功能。

![图片](img/92f02b547867d9aa262bc66880fda71e.png)

图片由作者提供

**BERTopic 的最大优势是其模块化。** 如上图所示，该流程由几个不同的模型组成：

1.  嵌入模型

1.  降维模型

1.  聚类模型

1.  分词器

1.  权重方案

1.  表示模型（可选）

因此，我们可以对每个组件中的不同模型进行实验，每个模型都有自己的参数。例如，我们可以尝试不同的嵌入模型，将降维从 PCA 切换到 UMAP，或者尝试微调我们的聚类模型的参数。这是允许我们将主题模型适应我们的数据和用例的一个巨大优势。

* * *

首先，我们需要导入必要的模块。其中大部分是为了构建我们的 BERTopic 模型的组件。

```py
#import packages for data management
import pickle

#import packages for topic modeling
from bertopic import BERTopic
from bertopic.representation import KeyBERTInspired
from bertopic.vectorizers import ClassTfidfTransformer
from sentence_transformers import SentenceTransformer
from umap.umap_ import UMAP
from hdbscan import HDBSCAN
from sklearn.feature_extraction.text import CountVectorizer

#import packages for data manipulation and visualization
import pandas as pd
import matplotlib.pyplot as plt
from scipy.cluster import hierarchy as sch
```

## 嵌入模型

BERTopic 模型的主要组成部分是嵌入模型。首先，我们使用 sentence transformer 初始化模型。然后，您可以指定您想要使用的嵌入模型。

在这个例子中，我使用了一个相对较小的模型（约 3000 万个参数）。虽然我们可能使用更大的嵌入模型可以得到更好的结果，但我决定使用较小的模型来强调这个流程中的速度。*您可以通过使用 Hugging Face 的 MTEB 排行榜（[*https://huggingface.co/spaces/mteb/leaderboard*](https://huggingface.co/spaces/mteb/leaderboard)）来查找和比较基于大小、性能、预期用途等不同嵌入模型。*

```py
#initalize embedding model
embedding_model = SentenceTransformer('thenlper/gte-small')

#calculate embeddings
embeddings = embedding_model.encode(data['all_text'].tolist(), show_progress_bar=True)
```

一旦我们运行了模型，我们可以使用.shape 函数来查看产生的向量的尺寸。下面，我们可以看到每个嵌入包含 384 个值，这些值构成了每个文档的意义。

```py
#invesigate shape and size of vectors
embeddings.shape

#output: (6102, 384)
```

## 降维模型

BERTopic 模型的下一个组成部分是降维模型。由于高维数据建模可能很麻烦，我们可以使用降维模型来以较低维度的表示来表示嵌入，同时不会丢失过多的信息。

![图片](img/766b0340fd7eae8e62898b6ca5759e31.png)

图片由作者提供

存在几种不同的降维模型，主成分分析（PCA）是最受欢迎的。在这种情况下，我们将使用统一流形近似和投影（UMAP）模型。**UMAP 模型是一个非线性模型，并且可能比 PCA 更好地处理我们数据中的复杂关系。**

```py
#initialize dimensionality reduction model and reduce embeddings
umap_model = UMAP(n_neighbors=5, min_dist=0.0, metric='cosine', random_state=42)
reduced_embeddings = umap_model.fit_transform(embeddings)
```

重要的是要注意，降维并不是解决高维数据的万能方法。降维在速度和准确性之间提供了一个权衡，因为信息会丢失。这些模型需要经过深思熟虑并经过实验，以避免在保持速度和可扩展性的同时丢失过多的信息。

## 聚类模型

第三步是使用降维嵌入并创建聚类。虽然聚类对于主题建模通常不是必需的，但我们**可以利用基于密度的聚类模型来隔离异常值并消除数据中的噪声**。下面，我们初始化了层次密度空间聚类应用噪声（HDBSCAN）模型并创建了我们的聚类。

```py
#initialize clustering model and cluster
hdbscan_model = HDBSCAN(min_cluster_size=30, metric='euclidean', cluster_selection_method='eom').fit(reduced_embeddings)
clusters = hdbscan_model.labels_
```

基于密度的方法给我们带来了一些优势。文档不会被强制放入它们不应该被分配到的聚类中，因此隔离异常值并减少数据中的噪声。此外，与基于质心的模型不同，我们不需要指定聚类数量，聚类更有可能被很好地定义。

*查看我的聚类算法指南：*

> [聚类算法指南](https://towardsdatascience.com/a-guide-to-clustering-algorithms-e28af85da0b7/)

查看下面的代码以可视化聚类模型的输出结果。

```py
#create dataframe of reduced embeddings and clusters
df = pd.DataFrame(reduced_embeddings, columns = ['x', 'y'])
df['Cluster'] = [str(c) for c in clusters]

#split between clusters and outliers
to_plot = df.loc[df.Cluster != '-1', :]
outliers = df.loc[df.Cluster == '-1', :]

#plot clusters
plt.scatter(outliers.x, outliers.y, alpha = 0.05, s = 2, c = 'grey')
plt.scatter(to_plot.x, to_plot.y, alpha = 0.6, s = 2, c = to_plot.Cluster.astype(int), cmap = 'tab20b')
plt.axis('off')
```

![图片](img/178b7ef86f0be4ca47c15ce1d38cef34.png)

图片由作者提供

我们可以看到定义良好的簇，它们之间没有重叠。我们还可以看到一些较小的簇聚集在一起，组成更高级的主题。最后，我们可以看到一些文档被灰色显示，并被识别为异常值。

* * *

## 创建 BERTopic 管道

我们现在拥有了构建 BERTopic 管道（嵌入模型、降维模型、聚类模型）所需的必要组件。我们可以使用我们已初始化并拟合数据的模型，使用 BERTopic 函数来调整这些模型。

```py
#use models above to BERTopic pipeline
topic_model = BERTopic(
  embedding_model=embedding_model,          # Step 1 - Extract embeddings
  umap_model=umap_model,                    # Step 2 - Reduce dimensionality
  hdbscan_model=hdbscan_model,              # Step 3 - Cluster reduced embeddings
  verbose = True).fit(data['all_text'].tolist(), embeddings)
```

由于我知道我已摄入了关于人机界面（增强现实、虚拟现实）的论文，让我们看看哪些主题与“增强现实”这一术语相一致。

```py
#topics most similar to 'augmented reality'
topic_model.find_topics("augmented reality")

#output: ([18, 3, 16, 24, 12], [0.9532771, 0.9498462, 0.94966936, 0.9451431, 0.9417263])
```

从上面的输出中，我们可以看到主题 18、3、16、24 和 12 与术语“增强现实”高度一致。**所有这些主题（希望）都将为增强现实这一更广泛的主题做出贡献，但每个主题都涵盖不同的方面。**

为了确认这一点，让我们调查一下主题表示。**主题表示是一个术语列表，旨在恰当地表示主题的潜在主题。**例如，“蛋糕”、“蜡烛”、“家庭”和“礼物”这些术语可能共同代表生日或生日派对的议题。

我们可以使用 get_topic()函数来调查主题 18 的表示。

```py
#investigate topic 18
topic_model.get_topic(18)
```

![图片](img/606854111652d5f35d4e32a275edf48e.png)

作者提供的图片

在上述表示中，我们可以看到一些有用的术语，如“现实”、“虚拟”、“增强”等。然而，作为一个整体，这并不有用，因为我们看到一些停用词，如“和”和“the”。这是因为 BERTopic 使用词袋作为表示主题的默认方式。这种表示也可能与其他关于增强现实的表示相匹配。

**接下来，我们将改进我们的 BERTopic 管道，以创建更有意义的主题表示，这能让我们对这些主题有更深入的了解。**

* * *

## 改进主题表示

我们可以通过添加一个加权方案来改进主题表示，这将突出显示最重要的术语，并更好地区分我们的主题。

这并不取代词袋模型，而是对其进行了改进。以下我们添加 TF-IDF 模型以更好地确定每个术语的重要性。我们使用 update_topics()函数更新我们的管道。

```py
#initialize tokenizer model
vectorizer_model = CountVectorizer(stop_words="english")

#initialize ctfidf model to weight terms
ctfidf_model = ClassTfidfTransformer()

#add tokenizer and ctfidf to pipeline
topic_model.update_topics(data['all_text'].tolist(), vectorizer_model=vectorizer_model, ctfidf_model=ctfidf_model)
```

```py
#investigate how topic representations have changed
topic_model.get_topic(18)
```

![图片](img/b60449fd3946e139ef5dd88bb3f29c1e.png)

作者提供的图片

使用 TF-IDF，这些主题表示变得更加有用。我们可以看到无意义的停用词已经消失，出现了其他有助于描述主题的术语，并且术语按照其重要性重新排序。

但我们不必止步于此。得益于 AI 和 NLP 领域无数新的发展，我们可以利用一些方法来微调这些表示。

为了微调，我们可以采取两种方法之一：

1.  表示模型

1.  生成模型

## 使用表示模型微调

首先，让我们添加 KeyBERTInspired 模型作为我们的表示模型。**这利用 BERT 来比较 TF-IDF 表示与文档本身的语义相似性，以更好地确定每个术语的相关性，而不是重要性。**

*所有表示模型选项请见此处：*[*https://maartengr.github.io/BERTopic/getting_started/representation/representation.html#keybertinspired*](https://maartengr.github.io/BERTopic/getting_started/representation/representation.html#keybertinspired)

```py
#initilzae representation model and add to pipeline
representation_model = KeyBERTInspired()
topic_model.update_topics(data['all_text'].tolist(), vectorizer_model=vectorizer_model, ctfidf_model=ctfidf_model, representation_model=representation_model)
```

![图片](img/fe094ce3404f3d974b81faaeb9f9717e.png)

图片由作者提供

在这里，我们可以看到术语发生了相当大的变化，增加了一些额外的术语和缩写。与 TF-IDF 表示进行比较，我们再次更好地理解了这个主题的内容。同时请注意，分数从 TF-IDF 权重（在没有上下文的情况下没有意义）变为 0-1 之间的分数。这些新分数代表语义相似性分数。

## 主题模型可视化

在我们转向用于微调的生成模型之前，让我们探索 BERTopic 提供的一些可视化方法。可视化主题模型对于理解你的数据和模型的工作方式至关重要。

首先，我们可以在二维空间中可视化我们的主题，这样我们可以看到主题的大小以及哪些主题是相似的。下面，我们可以看到我们有很多主题，主题集群构成了更大的主题。我们还可以看到一个大型且孤立的主题，这表明关于 crispr 的研究有很多相似之处。

![图片](img/5a51d0c4de027f7c15ece04b1bda9b85.png)

图片由作者提供

让我们放大这些主题集群，看看它们是如何分解成更高级主题的。下面，我们放大关于增强现实和虚拟现实的主题，看看一些主题是如何覆盖不同领域和应用的。

![图片](img/ef757f6991da50d2ce34aede148255cd.png)

图片由作者提供

![图片](img/1883b754eb457b60c47277fa5992610b.png)

图片由作者提供

我们还可以快速可视化每个主题中最重要或最相关的术语。这同样取决于你对主题表示的方法。

![图片](img/66ecd85405fb5b5ff8f9727b527776ae.png)

图片由作者提供

我们还可以使用热图来探索主题之间的相似性。

![图片](img/b16d6c5a01ff061e794e02e75ca32b35.png)

图片由作者提供

这些只是 BERTopic 提供的几种可视化方法。*完整列表请见此处：*[*https://maartengr.github.io/BERTopic/getting_started/visualization/visualization.html*](https://maartengr.github.io/BERTopic/getting_started/visualization/visualization.html)

## 利用生成模型

在我们的最后一步微调主题表示时，我们可以利用生成式 AI 来生成对主题的连贯描述。

**BERTopic 提供了一种简单的方法来利用 OpenAI 的 GPT 模型与主题模型进行交互。**我们首先创建一个提示，向模型展示数据和当前的主题表示。然后我们要求它为每个主题生成一个简短的标签。

然后我们初始化客户端和模型，并更新我们的管道。

```py
import openai
from bertopic.representation import OpenAI

#promt for GPT to create topic labels
prompt = """
I have a topic that contains the following documents:
[DOCUMENTS]

The topic is described by the following key words: [KEYWORDS]

Based on the information above, extract a short topic label in the following format:
topic: <short topic label>
"""

#import GPT
client = openai.OpenAI(api_key='API KEY')

#add GPT as representation model
representation_model = OpenAI(client, model = 'gpt-3.5-turbo', exponential_backoff=True, chat=True, prompt=prompt)
topic_model.update_topics(data['all_text'].tolist(), representation_model=representation_model)
```

现在，让我们回到增强现实主题。

```py
#investigate how topic representations have changed
topic_model.get_topic(18)

#output: [('Comparative analysis of virtual and augmented reality for immersive analytics',1)]
```

主题表示现在读作“虚拟和增强现实沉浸式分析的比较分析”。**主题现在更加清晰，因为我们可以看到包含在这些文档中的目标、技术和领域。**

下面是我们新的主题表示的完整列表。

![图片](img/f1a220b6dcaff4169d165f640f782474.png)

图片由作者提供

看看代码，我们就能看到生成式 AI 在支持我们的主题模型及其表示方面的强大功能。当然，在构建模型时深入挖掘并验证这些输出非常重要，同时进行大量的不同模型、参数和方法的实验。

## 利用主题模型变体

最后，BERTopic 提供了多种主题模型的变体，以提供针对不同数据和用例的解决方案。这些包括时间序列、分层、监督、半监督等。

*请在此处查看完整列表和文档：*[*https://maartengr.github.io/BERTopic/getting_started/topicsovertime/topicsovertime.html*](https://maartengr.github.io/BERTopic/getting_started/topicsovertime/topicsovertime.html)

让我们快速探索分层主题建模中的一种可能性。下面，我们使用 scipy 创建一个链接函数，确定我们主题之间的距离。我们可以轻松地将它拟合到我们的数据并可视化主题的层次结构。

```py
#create linkages between topics
linkage_function = lambda x: sch.linkage(x, 'single', optimal_ordering=True)
hierarchical_topics = topic_model.hierarchical_topics(data['all_text'], linkage_function=linkage_function)

#visualize topic model hierarchy
topic_model.visualize_hierarchy(hierarchical_topics=hierarchical_topics)
```

![图片](img/cd3a924b3f90753f73062d76ad32bd63.png)

图片由作者提供

在上面的可视化中，我们可以看到主题是如何组合在一起形成更广泛的主题的。例如，我们看到主题 25 和 30 结合形成“智能城市和可持续发展”。这个模型提供了强大的能力，能够放大和缩小，决定我们希望主题有多广泛或多狭窄。

* * *

## 结论

在这篇文章中，我们看到了 BERTopic 在主题建模方面的强大功能。BERTopic 使用变压器和嵌入模型显著提高了传统方法的结果。BERTopic 管道还提供了强大的功能和模块化，利用多个模型，并允许您插入其他模型以适应您的数据。所有这些模型都可以进行微调和组合，以创建一个强大的主题模型。

您还可以集成表示和生成模型来改进主题表示并提高可解释性。BERTopic 还提供了几个可视化工具，以真正探索您的数据并验证您的模型。最后，BERTopic 提供了多种主题建模的变体，如时间序列或分层主题建模，以更好地适应您的用例。

* * *

*希望您喜欢我的文章！请随时评论、提问或要求其他主题。*

*在 LinkedIn 上与我联系：*[*https://www.linkedin.com/in/alexdavis2020/*](https://www.linkedin.com/in/alexdavis2020/)
