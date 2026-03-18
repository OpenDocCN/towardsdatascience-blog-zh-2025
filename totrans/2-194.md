# 停止构建无用的机器学习项目 – 什么是真正有效的

> 原文：[`towardsdatascience.com/stop-building-useless-ml-projects-what-actually-works/`](https://towardsdatascience.com/stop-building-useless-ml-projects-what-actually-works/)

<mdspan datatext="el1751308065686" class="mdspan-comment">我经常被问到：</mdspan>

> *“我应该做什么项目才能在数据科学或机器学习领域找到工作？”*

这个问题从一开始就是有缺陷的。

一个伟大的项目对你来说很个人化，这意味着我提出的任何项目都会自动成为一个“糟糕”的选择。

在这篇文章中，我的目标是分析那些真正有助于你获得工作的项目类型，以及你可以遵循的框架来寻找这些项目。

## 4-5 个简单的项目

首先通过构建 4-5 个较小的项目来给你的作品集增加一些初始的分量。

这里的主要目标主要是为了“外观”和确保你的简历/CV、GitHub 和 LinkedIn 个人资料看起来活跃且内容丰富。

请花几周时间构建这些较小的项目，确保它们的质量足够高，并且不是你匆忙用 ChatGPT 生成的。

努力构建各种项目，每个项目使用不同的工具、数据集和机器学习算法。

### 算法和机器学习模型

我建议你的项目包含以下算法：

+   [**梯度提升树**](https://xgboost.readthedocs.io/en/stable/) — 表格数据的黄金标准算法，所以这将是你在工作中一定会使用的。

+   [**神经网络** ](https://medium.com/@egorhowell/list/neural-networks-616db722dbbb)— 对深度学习框架如 [**TensorFlow**](https://www.tensorflow.org/) 或 [**PyTorch**](https://pytorch.org/) 的良好理解很有价值，尤其是如果你想从事计算机视觉、自然语言处理或人工智能工作。

+   **聚类算法** — 例如 [**K-Means**](https://en.wikipedia.org/wiki/K-means_clustering) 和 [**DBSCAN**](https://www.datacamp.com/tutorial/dbscan-clustering-algorithm) 这样的模型展示了你对无监督学习的掌握，这对于某些职位是必需的。

### 获取令人兴奋和新颖的数据

获得更混乱、更真实的数据库集，这些数据库集反映了你在现实世界中会遇到的数据。这将给雇主和面试官留下更深刻的印象，直接展示你作为数据科学家的能力。

在选择项目数据集时，避免使用过度使用的数据集，如 **[MNIST](https://www.kaggle.com/datasets/hojjatk/mnist-dataset)**、**[泰坦尼克号](https://www.kaggle.com/datasets/yasserh/titanic-dataset)** 或 **[鸢尾花](https://www.kaggle.com/datasets/uciml/iris)**。如果我看到这些，将会是立即拒绝，或者至少会让我感到很不舒服。

一些获取数据的好地方：

+   使用公共和免费的 API — 你可以查看 [**free-apis**](https://free-apis.github.io/#/) 网站获取一些想法。

+   从相关网站抓取数据（确保你首先被允许这样做！）——[**这里**](https://www.clay.com/blog/websites-that-allow-web-scraping#which-websites-allow-web-scraping)是一个允许网络抓取的网站列表。

+   公共政府数据来源——[**data.gov**](https://data.gov/)是你可以使用的例子。

+   通过调查和问卷收集自己的数据。

要决定你的项目应该是什么，最好的办法是先回答你认为从数据中发现的有趣的具体问题。

我建议使用**[Streamlit](https://streamlit.io/)**或通过**[GitHub Actions](https://github.com/features/actions)**部署一个简单的模型来展示你的结果。

然而，不要过于担心使用**[AWS](https://aws.amazon.com/?nc2=h_lg)**或其服务（如**[EC2](https://aws.amazon.com/ec2/)**或**[ECS](https://aws.amazon.com/ecs/)**）构建一个完整的端到端生产系统。在这个阶段，如果你不知道如何做，是完全正常的，这也不是这些小型项目的目标。

## 一个大项目

这是你真正需要集中精力并花时间的地方。

在你构建了较小的项目之后，是时候做一个大项目了。如果你每天花一两个小时来做，这个项目可能需要几个月的时间。

这可能会让你感到害怕，但如果你想有一个与众不同的项目，你就需要付出努力。

*问题是，你应该构建什么？*

如我之前提到的，我不能为你选择这个项目，但我可以提供一个框架供你遵循，让你自己找到一个出色的项目。

### 示例项目

*让我给你举一个出色项目的例子。*

在我之前的公司，我们正在招聘一名初级数据科学家，负责处理[**优化**](https://medium.com/@egorhowell/list/optimisation-algorithms-069bf9c6c8d5)和[**运筹学**](https://en.wikipedia.org/wiki/Operations_research)问题。

我们雇佣的候选人之所以突出，主要原因之一是他们有一个高度相关且非常个人化的项目，这与角色非常匹配。

他们对 NFL 梦幻足球充满热情，并希望改进他们每周组建阵容的方式（这类似于英国的梦幻英超联赛）。

因此，他们开发了自己的优化引擎，在程序的约束条件下更有效地分配玩家。

这不仅仅是引擎本身；他们阅读了关于优化策略的学术论文，并研究了其他人如何处理相同的问题。

*你看到为什么这是一个如此强大的项目了吗？*

+   他们感兴趣的是一个个人问题。

+   它是独一无二的，我们之前和之后都没有见过类似的东西。

+   这显示了他们对优化和运筹学的热情和兴趣。

+   这与他们申请的工作直接相关。

### 我的框架

这里有一个简单的框架，供你参考，以产生出色的项目想法：

1.  列出至少五个你在工作之外以及数据科学或机器学习领域感兴趣的事物。

1.  对于每一项，提出你想要得到答案的问题，或者其他人可能会感兴趣的问题。

1.  考虑机器学习如何帮助回答这些问题。如果问题看起来不可能，不要担心；尽可能发挥创意。

1.  选择一个你最感兴趣的问题。理想情况下，选择一个感觉稍微超出你能力范围的问题；这样你才能真正学习和挑战自己。

### 构建复杂性和规模

为了使这个项目脱颖而出，我们需要给它增加一些复杂性和规模。这意味着不同的事情，有各种方法可以融入。

如果你目标是成为一名机器学习工程师，从头到尾构建和部署项目尤其有价值。

你的项目理想上应包括以下内容：

+   数据收集和存储。

+   数据预处理。

+   模型训练和评估。

+   模型部署（通过 API、Web 应用等）。

+   结果的分析和展示。

要做到这一点，你需要学习以下内容：

+   通过诸如 [**Docker**](https://medium.com/data-science/from-chaos-to-consistency-docker-for-data-scientists-240372adff18?sk=68898c01460f59a75617244d19f101d1) 和 **[Kubernetes](https://kubernetes.io/)** 等方式实现容器化。

+   CI/CD 流水线和平台，例如 **[CircleCI](https://circleci.com/)**。

+   使用 **[Git](https://git-scm.com/)** 进行版本控制。

+   具有高质量的代码，包括 **[单元测试](https://medium.com/data-science/debugging-made-easy-use-pytest-to-track-down-and-fix-python-code-ecbad62057b8?sk=c1f82fe59d96a39ce6b0ad66780a50e7)**、**[代码检查](https://medium.com/data-science/a-data-scientists-guide-to-improving-python-code-quality-21660ecea97d?sk=47ff68f9f933bc19473b0baf0b8b3d98)**、**[格式化](https://medium.com/data-science/a-data-scientists-guide-to-improving-python-code-quality-21660ecea97d?sk=47ff68f9f933bc19473b0baf0b8b3d98)** 和 **[严格的类型检查](https://medium.com/data-science/a-data-scientists-guide-to-python-typing-boosting-code-clarity-194371b4ef05?sk=3392534f0fdccbf893be642cac7c9152)**。

+   云平台如 **[AWS](https://aws.amazon.com/)**、**[GCP](https://cloud.google.com/)** 和 **[Azure](https://azure.microsoft.com/en-gb)**。

+   像这样的工作流程编排工具：**[Airflow](https://airflow.apache.org/)**、**[Argo](https://argoproj.github.io/workflows/)** 和 **[Metaflow](https://metaflow.org/)**。

+   以及实验跟踪工具，如 **[MLflow](https://mlflow.org/)** 或 **[Weights & Biases](https://wandb.ai/site/)**。

虽然看起来很多，但你不需要做这个列表上的每一件事。

主要的事情是开始学习这些内容，并在过程中学习；不要试图一次性学习所有内容；那只是拖延。

## 记录和沟通

最后，也是可能最重要的部分是记录你的学习过程。

***仅凭技术技能是无法获得工作的。***

沟通是作为机器学习工程师或数据科学家必须具备的最重要技能之一，尤其是在你晋升的时候。

通过以下方式展示你的项目：

+   将你的项目添加到 GitHub，并拥有一个良好的文档化的 README。

+   包括设置和使用说明，以便用户可以探索和与你的项目互动。

+   写一篇博客文章解释你的项目以及你是如何做到的。

+   在领英、推特、Reddit、Discord、YouTube 或任何可能对此感兴趣的人的地方分享它。

你分享的工作越多，你向潜在雇主和合作伙伴展示的就越明显。

* * *

实际上，创建一个坚实的项目组合并不难；这只需要持续的工作和耐心，而大多数人都不愿意这样做。

没有能让你快速被雇佣的项目；能让你被雇佣的是花时间构建一些个人、高质量且新颖的东西。

那就是秘密。

## 另一件事！

我提供一对一的辅导通话，我们可以聊任何你需要的事情——无论是项目、职业建议，还是只是确定你的下一步。我在这里帮助你前进！

[**与 Egor Howell 的一对一辅导通话**]

*职业指导、工作建议、项目帮助、简历审查* topmate.io](https://topmate.io/egorhowell/1203300)

## 与我联系

+   [**YouTube**](https://www.youtube.com/@egorhowell)

+   [**领英**](https://www.linkedin.com/in/egorhowell/)

+   [**Instagram**](https://www.instagram.com/egorhowell/)

+   [**网站**](https://egorhowell.com/)
