# 你需要的唯一数据科学路线图，以获得一份工作

> 原文：[`towardsdatascience.com/the-only-data-science-roadmap-you-need-to-get-a-job/`](https://towardsdatascience.com/the-only-data-science-roadmap-you-need-to-get-a-job/)

<mdspan datatext="el1753973363296" class="mdspan-comment">你</mdspan>是否想成为一名数据科学家，却不知道从何开始？

在这篇文章中，我想为你提供一个简单直接、没有废话的学习路线图，你可以遵循它进入这个行业。

到最后，你将最终清楚地了解所需的内容和最佳资源，这应该有助于减少你可能有的任何困惑，并帮助你更快地获得数据科学工作！

# 统计学

我愿意为之献身的观点是，在我看来，统计学是作为数据科学家你应该知道的最重要领域。

新的机器学习趋势来来去去，技术经常被取代，但统计学已经经受住了几个世纪的考验。

根据[Wikipedia](https://en.wikipedia.org/wiki/Statistics):

> *统计学是关于数据的收集、组织、分析、解释和呈现的学科*。

由于标题是“数据”科学家，我认为统计学对我们领域的重要性是显而易见的。

幸运的是，你不需要拥有因果推断或随机微积分的博士学位来获得所需的统计学知识。基础是最重要的，实际上 90%的工作都依赖于它。

## 学习什么

你需要强烈掌握的领域包括：

+   **描述性统计** — 均值、中位数、众数、方差、相关性，任何允许你总结数据以得出有趣结论的东西。

+   **可视化** — 学会用条形图、折线图、饼图等图表来绘制数据。毕竟，一张图胜过千言万语。

+   **概率分布** — 学习最常见的一些，如正态分布、泊松分布、二项分布和伽马分布。这些是我最常用的。

+   **概率论** **—** 这个领域相当大，但主要要学习的是：随机变量、中心极限定理、抽样和最大似然估计。

+   **假设检验** — 如果你将要从事任何实验，你需要了解它们是如何在统计上运行的。这涉及到学习置信区间、显著性水平、z 检验、t 检验和检验统计量。你只需要知道如何进行假设检验。

+   **贝叶斯统计学** — 了解一些贝叶斯统计学是非常有价值的，因为我发现人们在领域中经常随意使用这个术语，但实际上并不真正理解。这是一个庞大的领域，但就像往常一样，学习基础，比如贝叶斯定理、共轭先验、可信区间和贝叶斯回归。

## 如何学习

正如我一开始提到的，我希望这个路线图简单明了，避免你可能遇到的任何分析瘫痪，因此为了学习上述几乎所有内容，我推荐获取[**数据科学实用统计学**](https://amzn.to/4m86nPk) **(联盟链接**)教科书。

然而，它没有涵盖贝叶斯统计，因此我推荐[**Think Bayes**](https://amzn.to/3J4AYih) **(联盟链接**)教科书。

这两本书就足够了，它们专为数据科学家设计，并且使用 Python 编写。

# 数学

统计学本质上是一个相当应用性的领域，其中一些概念需要纯数学知识才能完全理解。

此外，当涉及到机器学习等领域时，你需要对线性代数和微积分有很好的理解，才能完全理解底层发生的事情。

## 要学习什么

### 微积分

[**微积分**](https://en.wikipedia.org/wiki/Calculus)是机器学习算法实际“学习”的方式。它们的“学习”是通过数值连续优化完成的，你应该学习的领域包括：

+   导数是什么，它测量的是什么？

+   学习标准函数（如正弦、余弦、指数、正切等）的导数。

+   转折点、极大值和极小值是什么？

+   链式法则和乘积法则是神经网络工作得如此之好的原因，因为它们是反向传播的核心过程。

+   理解偏导数及其在多元微积分中的应用。

+   积分是什么，它在做什么？

+   分部积分和代换。

+   标准函数（如正弦、自然对数和其他多项式）的积分。

### 线性代数

[**线性代数**](https://en.wikipedia.org/wiki/Linear_algebra)是处理向量、矩阵及其变换的数学领域。

你应该学习：

+   向量，它们的幅度、方向和分量。此外，还包括点积和叉积规则。

+   矩阵及其运算，包括迹、逆、转置、点积和叉积规则。

+   学习如何通过消元法、行简化法和克拉默法则等技术求解线性方程组。

+   理解特征值和特征向量。这些是主成分分析等技术的基石，有助于降低数据集的维度。

## 如何学习

在之前的视频中，我推荐了一些教科书，虽然它们很有用，但内容相当密集，对于大多数人来说，在短短几个月内通过这些书是不切实际的。

正因如此，我现在建议在 Coursera 上学习[**机器学习与数据科学数学专项**](https://www.coursera.org/specializations/mathematics-for-machine-learning-and-data-science/)。

这门课程专门针对数据科学，包含 Python 练习，跳过了不必要的理论，专注于实际工作中真正需要的内容。

# 编程

你只需要两种编程语言：[**Python**](https://medium.com/data-science/how-i-would-learn-python-in-2024-from-zero-b1c5edcdec84?sk=7d25e0dc7d72decdd9636142bf6f1a77) 和 [**SQL**](https://www.youtube.com/watch?v=DfLELVaXgi0)。

## 学习内容

### **Python**

保持简单，学习基础知识：

+   变量和数据类型

+   布尔和比较运算符

+   控制流和条件

+   for 和 while 循环

+   函数和类

你还想要学习特定的科学计算库：

+   [**NumPy**](https://numpy.org/devdocs/user/quickstart.html) — 数值计算和数组。

+   [**Pandas**](https://pandas.pydata.org/) — 数据操作和分析。

+   [**Matplotlib**](https://matplotlib.org/stable/tutorials/index.html), [**Plotly**](https://plotly.com/) 和 [**Seaborn**](https://seaborn.pydata.org/) — 数据可视化。

+   [**scikit-learn**](https://scikit-learn.org/1.4/tutorial/index.html) — 实现经典机器学习算法。

### SQL

你需要学习 SQL 中分析所需的所有基本函数。这是一个相当小的语言，所以没有太多东西要学。

+   **SELECT * FROM** (标准查询)

+   **ALTER, INSERT, CREATE** (修改表)

+   **GROUP BY, ORDER BY**

+   **WHERE, AND, OR, BETWEEN, IN, HAVING** (过滤表)

+   **AVG, COUNT, MIN, MAX, SUM** (聚合函数)

+   **FULL JOIN, LEFT JOIN, RIGHT JOIN, INNER JOIN, UNION**

+   **CASE** (if 语句)

+   **DATEADD, DATEDIFF, DATEPART** (日期和时间函数)

## 如何学习

有很多 Python 和 SQL 的入门课程，它们都教授相同的内容。所以，选择一个开始吧。你实际上不会出错。

如果你需要推荐，那么查看[**W3Schools**](https://www.w3schools.com/)或[**freeCodeCamp 视频**](https://www.freecodecamp.org/news/tag/python/)。我都使用过，发现它们非常好。

# 技术工具

除了 Python 和 SQL，你还需要投入时间学习工作中使用的其他技术。

## 学习内容

工具很多，而且每家公司都不同，但以下这些是始终如一的：

+   [**Git 和 GitHub**](https://github.com/egorhowell) — 几乎每家公司都使用这个进行版本控制，所以你需要学习它；恐怕没有别的办法了。

+   [**Bash**](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29)**/Zsh** — 你将在终端中工作很多，而且大多数公司依赖于类 UNIX 系统，所以你需要熟悉命令行操作。

+   [**Poetry**](https://python-poetry.org/) **/** [**PyEnv**](https://github.com/pyenv/pyenv) / [**UV**](https://docs.astral.sh/uv/) — 管理包和 Python 版本在任何实际应用中都是至关重要的，所以熟悉这些工具是非常值得的。

## 如何学习

对于 git，我推荐来自 freeCodeCamp 的这门速成课程：

对于学习终端和 bash shell 脚本，我也推荐来自 freeCodeCamp 的这段视频。

而对于学习 PyEnv、Poetry 和 UV，请查看这些文章：

+   [使用 Pyenv 管理多个 Python 版本](https://realpython.com/intro-to-pyenv/)

+   [使用 Python Poetry 进行依赖管理](https://realpython.com/dependency-management-python-poetry/)

+   [使用 UV 管理 Python 项目](https://realpython.com/python-uv/)

# 机器学习

好的，现在是时候来点有趣的东西了！

机器学习是一个庞大的领域，即使我们倾尽全力也无法学习到所有内容。

就像我一直说的，要成为一名数据科学家，我们只需要了解基础知识以及一点深度学习。

忘记学习 LLMs、transformers、diffusion models 等等。这对大多数入门级职位来说不是必要的，坦白说，对于许多工作来说也是如此。

专注于掌握基础知识，因为它们超越了其他所有内容。时至今日，我仍然在使用基本的回归模型，和我一起工作的许多高级机器学习工程师也是如此。

这一切都关乎应用和理解你的问题，而不是在不必要的时候试图通过使用最新的最先进技术来显得花哨。

## 学习内容

你应该学习的核心算法和概念是：

+   线性、逻辑和多项式回归。

+   决策树、随机森林和梯度提升树。

+   支持向量机。

+   常规神经网络。

+   K-means 和 K-最近邻聚类。

+   正则化、偏差与方差权衡和交叉验证。

## 学习方法

以下两个资源就是你所需要的。所以，迭代地完成它们，你的机器学习知识将超越行业中的大多数从业者。相信我。

我第一次学习的机器学习课程是[**安德鲁·吴的机器学习专项课程**](https://www.coursera.org/learn/machine-learning)，我认为这可能是最好的一个。你可以单独完成这个课程，因为它真的很棒。

第二本可能是有史以来最好的机器学习书籍：[**使用 Scikit-Learn、Keras 和 TensorFlow 进行动手机器学习**](https://amzn.to/4iJxGyh) **(推广链接)。** 如果我只能推荐一本书来学习机器学习，那将是这本书！

# 深度学习

在我看来，这虽然是可选的，但我知道你们中的许多人对深度学习很感兴趣，所以我把它包括在这里以示完整。

我个人认为在这里不必浪费太多时间，因为很容易迷失在所有最新的发展中。

## 学习内容

这些深度学习概念经受了时间的考验，因此它们值得你投入学习：

+   [**卷积神经网络 (CNNs)**](https://medium.com/towards-data-science/convolution-explained-introduction-to-convolutional-neural-networks-5babc47fbcaa?sk=e6ce3db3f3c8a1a38798779beb719578)——这些被用于计算机视觉任务，如识别和分类图像。

+   [**循环神经网络（RNNs）**](https://towardsdatascience.com/recurrent-neural-networks-an-introduction-to-sequence-modelling-478e0e07c4ec?sk=3d8ff48072b608f38bf120f78a050718)**— **这些用于基于序列的数据，如时间序列和自然语言。

+   [**Transformers**](https://jalammar.github.io/illustrated-transformer/)** — **推动 AI 繁荣的最新技术水平算法。

## 如何学习

这些是我用来学习深度学习的资源，它们就是你需要的所有东西。

[**Andrew Ng 的深度学习专项课程**](https://www.coursera.org/specializations/deep-learning)——这是机器学习专项课程的后续课程，将教你所有关于深度学习、CNN 和 RNN 的知识。

再次强调，从第十四章开始的[**使用 Scikit-Learn、Keras 和 TensorFlow 的动手机器学习**](https://amzn.to/4iJxGyh) **（联盟链接）**教科书是一个优秀的深度学习部分。

最后，你们中的一些人可能听说过[**安德烈·卡帕西**](https://karpathy.ai/)，如果你还没有，他可能是目前最好的 AI 研究人员之一，曾在特斯拉和 OpenAI 工作过。

不管怎样，他的[**神经网络：从零到英雄**](https://www.youtube.com/playlist?list=PLAqhIrjkxbuWI23v9cThsA9GvCAUhRvKZ)YouTube 课程非常出色，教你从头开始构建自己的生成式预训练转换器（GPT）。

* * *

如果你通读这篇文章的每一部分，你将拥有进入数据科学领域的优秀知识。

然而，拥有这些知识是不够的；你需要构建一个稳固的简历来获得工作。

正因如此，我建议查看我之前发表的文章，我在那里解释了你需要构建的具体项目，以便尽快获得工作。

那里见！

[**停止构建无用的机器学习项目——真正有效的方法 | 向数据科学进发**]

*如何找到能让你获得工作的机器学习项目。*towardsdatascience.com](https://towardsdatascience.com/stop-building-useless-ml-projects-what-actually-works/)

# 另一件事！

我提供一对一辅导通话，我们可以聊任何你需要的事情——无论是项目、职业建议，还是只是确定你的下一步。我在这里帮助你前进！

[**与 Egor Howell 的一对一辅导通话**]

*职业指导、求职建议、项目帮助、简历审查*topmate.io](https://topmate.io/egorhowell/1203300)

# 与我联系

+   [**YouTube**](https://www.youtube.com/@egorhowell)

+   [**领英**](https://www.linkedin.com/in/egorhowell/)

+   [**Instagram**](https://www.instagram.com/egorhowell/)

+   [**网站**](https://egorhowell.com/)
