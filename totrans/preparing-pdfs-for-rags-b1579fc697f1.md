# 为 RAG 准备 PDF

> 原文：[`towardsdatascience.com/preparing-pdfs-for-rags-b1579fc697f1/`](https://towardsdatascience.com/preparing-pdfs-for-rags-b1579fc697f1/)

![照片由 Annual Report Design Agency - Report Yak 在 Unsplash 提供](img/5b7c58ef2b4d405575bb6d841ef0981f.png)

照片由 [Annual Report Design Agency – Report Yak](https://unsplash.com/@reportsyak?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上提供

> 将 PDF 转换为文本是可能的，但从未如此简单。

我最近创建了一个用于 RAG 的图数据存储。换句话说，我们构建了一个 GraphRAG。

> [**如何在几分钟内构建知识图谱（并使其企业级就绪）**](https://towardsdatascience.com/enterprise-ready-knowledge-graphs-96028d863e8c)

图 RAG 是其他 RAG 应用程序（如广泛使用的基于向量存储的 RAG）的绝佳替代品。它们带来了推理能力。例如，使用语义相似度搜索（在向量存储中用于检索信息的技术），你可以问 XYZ 公司去年谁是首席财务官。因为 XYZ 公司去年的年度报告会明确提到其首席财务官。但考虑这样一个问题：XYZ 公司的哪两位董事曾在同一所学校学习？如果没有提到学校名称，检索过程将无法获取相关信息。但图 RAG 可以做到。

然而，这里的关键问题是我们的检索图是如何构建的。我最近在一篇单独的文章中解决了这个问题。再退一步思考，我们如何准备年度报告，使其更容易创建图表？

这正是本文的重点。

我们所有工作的第一步工程是将数据从 PDF 转换为文本。然而，年度报告是复杂的文档。不会只有文本。还会有图表、表格等。每一项都提供了关于公司的关键信息。

那么，让我们从这里开始。

### 如何将 PDF 转换为富文本

大多数 Python 程序员在某个时候都使用过 PDF 阅读器——至少是为了跟随教程。最受欢迎的是 PyPDF2。

大多数这些库都能完成任务。但信息的实用性并不大。

我已经了解多年的 **PyPDF2** 库，它可以提取所有 PDF 内容作为文本，而不进行任何格式化。提取后，你不知道哪个是标题，哪个是列表。

然后是 **[PyMuPDF4LLM](https://pymupdf.readthedocs.io/en/latest/index.html)**。这个包可以将 PDF 文件直接转换为 Markdown 格式。Markdown 也包含大量关于文本结构的关键信息。像 Langchain 这样的框架支持 Markdown。它们使用文本格式中的额外信息来更好地分割和存储数据在数据存储中。这反过来使得检索相关数据变得更加容易。

PyMuPDF4LLM 的问题在于它处理表格不太好。提取的表格与原始表格相差甚远。（*不要放弃 PyMuPDF4LLM。它仍然在我们的最终解决方案中发挥了不可思议的作用*）

最后，我们还尝试了几款其他现代工具。一个是[Docling](https://ds4sd.github.io/docling/)，由[IBM Deep Search](https://github.com/DS4SD)开发和开源的库，另一个是[Marker](https://github.com/VikParuchuri/marker)，同样是一个出色的库。

这里展示了我们讨论过的四个包转换同一 PDF 页面的输出结果。

**PyPDF2:**

![使用 PyPDF2 从 PDF 中提取的文本信息——作者截图](img/0146cb32da0c03b9f3c0da6a2fe580b2.png)

使用 PyPDF2 从 PDF 中提取的文本信息——作者截图

**PyMuPDF4LLM:**

![使用 PyMuPDF4LLM 从 PDF 中提取的 Markdown 信息——作者截图](img/8d0fa9781044bf6b85201f50c8746bc5.png)

使用 PyMuPDF4LLM 从 PDF 中提取的 Markdown 信息——作者截图

**Docling:**

![使用 Docling 从 PDF 中提取的 Markdown 信息——作者截图](img/23e0f467e86c9cef3ff5ff372be43885.png)

使用**Docling**从 PDF 中提取的 Markdown 信息——作者截图

**Marker:**

![使用 Marker 从 PDF 中提取的 Markdown 信息——作者截图](img/c950097525fa62d19b19a66343c22b99.png)

使用 Marker 从 PDF 中提取的 Markdown 信息——作者截图

Docling 版本比其他两个版本更整洁，并且包含更多的结构信息。它提取了包含所有相关信息的表格。它为图像留下了占位符，并保留了层次标题结构。所有这些信息都有助于创建块、图数据库、更好的检索，最终是一个更好的 RAG 系统。

## 哪些没有工作得很好？

Marker 和对接都是很有潜力的。然而，如果我们花时间去将 PDF 转换为 Markdown，我并不觉得有什么特别印象深刻的地方。

为了正确测试，我将年度报告中包含文本、表格和图像的片段分割出来。然后我创建了几个不同页数的副本。

我然后用这些不同大小的 PDF 文件对每个工具进行了转换，并测量了它们所需的时间。以下是最终结果。

![](img/8a4c125ebf4c4d2111dca4bfb4871e7b.png)

结果清楚地显示，尽管 Docling 和 PyPDF2 在转换表格方面表现出色，但与其他两种选项相比，它们的速度慢得多。

Docling 处理单个 PDF 页面大约需要 4 秒，而 Marker 则需要大约双倍的时间。

我做了什么？

## 如何选择最佳的 PDF 转 Markdown 工具

我们的项目有几十份年度报告。幸运的是，它们不是几百份。

如果每份年度报告大约有 300 页长，并且有 50 份报告使用对接，我们需要 17 小时（每份报告 300 页 x 50 份报告 x 每页 4 秒 / 每小时 3600 秒）。

因此，我们可以承担这个时间。

但我们仍在原型设计阶段。我们需要考虑项目的扩展。如果这个项目扩展到所有 S&P500 公司和它们的 30 年历史报告，会怎么样呢？

除非我们**分配负载并并行处理**，否则将需要 208 天，这正是我们之前所做的事情。

我们创建了一个在云中处理转换并更新 Graph DB 的服务。这可以并行运行并很好地扩展。

> 如果速度是你的关注点，PyPDF2 是最好的选择——前提是你不能并行运行 Docling。

## 最后的想法

PDF 到 Markdown 的工具已经变得更好。但将 PDF 中的表格转换为 Markdown 表格仍然不是很好。

我们比较了四个免费和开源的工具，用于将 PDF 转换为 Markdown。然后，我们将 Markdown 分块并转换成 RAG 应用中的图表。

在我们比较的工具中，Docling 产生了出色的结果。然而，其性能却是一个缺点。它比广泛使用的 PyPDF2 慢得多。

因此，我们转向了一个扩展性良好且并行进行转换的云服务。

* * *

*感谢阅读，朋友！除了**[Medium](https://thuwarakesh.medium.com/)**，*我还在**[LinkedIn](https://www.linkedin.com/in/thuwarakesh/)**和**[X,](https://twitter.com/Thuwarakesh)**上！*
