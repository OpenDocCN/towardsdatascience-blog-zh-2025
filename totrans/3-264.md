# 分支拓展：适用于机器学习协作的 4 种 Git 工作流程

> 原文：[`towardsdatascience.com/branching-out-4-git-workflows-for-collaborating-on-ml/`](https://towardsdatascience.com/branching-out-4-git-workflows-for-collaborating-on-ml/)

自从完成硕士学位已经超过 15 年了，但我仍然被管理我的*R*脚本的痛苦和挫败感所困扰。作为一个（正在恢复中的）完美主义者，我按照日期非常系统地给每个脚本命名（想想：`ancova_DDMMYYYY.r`）。一个我确信比`_v1`、`_v2`、`_final`及其竞争对手更好的系统。对吧？

问题在于，每次我想调整我的模型输入或审查先前的模型版本时，我不得不在脚本海洋中游泳。

快进几年，几种编程语言，以及职业生涯的滑雪滑行之后，我清楚地看到，我独自与代码版本控制斗争的经历是一个幸运的觉醒。

虽然我设法克服了那些早期的挑战（伴随着一些令人尴尬的时刻！），但现在我认识到，大多数开发，尤其是在敏捷工作方式下，都依赖于强大的版本控制系统。跟踪变更、回滚到先前版本以及在协作代码库中确保可重复性不能是事后才考虑的事情。实际上，这是一个必需品。

当我们使用版本控制工作流程，通常在 Git 中，我们为开发并部署更可靠、更高品质的数据和 AI 解决方案打下基础。

## **在我们开始之前**

如果你已经使用版本控制并且正在考虑为你的团队制定不同的工作流程，欢迎！你来到了正确的地点。

如果你刚开始使用 Git 或者只在使用个人项目时使用过它，我建议你回顾一些 Git 的入门原则。在跳入团队工作流程之前，你需要更多的背景知识。GitHub 提供了几个 Git 和 GitHub 教程的链接[这里](https://docs.github.com/en/get-started/start-your-journey/git-and-github-learning-resources)。而且[这篇](https://towardsdatascience.com/a-simple-git-workflow-for-github-beginners-and-everyone-else-87e39b50ee08)入门文章介绍了如何创建仓库和添加文件等基础知识。

## **开发团队以不同的方式工作**

### 但一个普遍的特征是依赖版本控制。

Git 作为一个版本控制系统非常灵活，它允许开发者以很大的自由度来管理他们的代码。不过，如果你不小心，灵活性如果没有得到有效管理，就会留下混乱的空间。建立 Git 工作流程可以指导你的团队开发，使你更一致、更有效地使用 Git。把它看作是团队在 Git 高速公路和旁路上导航的共同路线图。

通过定义何时创建分支、如何合并更改以及为什么审查代码，我们创造了一个共同的理解，并培养出更可靠的团队开发方式。这意味着每个团队都有机会创建适合其特定组织结构、用例、技术堆栈和需求的 Git 工作流程。团队使用 Git 的方式可能和开发团队一样多。极致的灵活性。

你可能会觉得这个想法很自由。你和你的团队有自由设计一个适合你们的工作流程！

但如果这听起来令人生畏，不必担心。有几种既定的协议可以作为团队工作流程的起点。

## **让 Git 成为你的朋友**

版本控制以多种方式有用，但我在我的团队中反复看到的益处主要集中在几个基本类别上。我们在这里关注工作流程，所以不会深入探讨，但 Git 和 GitHub 的核心前提和优势值得强调。

**（几乎）任何操作都是可逆的。** 这意味着版本控制系统让您可以自由地发挥创意和犯错误。回滚任何令人遗憾的代码更改就像`git revert`一样简单。就像一个好邻居一样，Git 命令就在那里。

**简化代码协作。** 一旦你开始使用它，Git 就能真正促进团队间的无缝协作。工作可以并行进行，而不会干扰到任何人的代码，所有的代码更改都记录在提交快照中。这意味着团队中的任何人都可以查看其他人正在做什么以及他们是如何做的，因为更改都记录在 Git 历史中。协作变得简单易行。

**在功能分支中隔离探索性工作。** 如何知道哪种模型最适合您特定的业务问题？在最近的一个收入案例中，可能是时间序列模型，也许是基于树的算法，或者是卷积神经网络。甚至可能是贝叶斯方法。如果没有 Git 为我们团队提供的并行分支能力，尝试不同的方法可能会导致代码库陷入混乱。

**内置的审查流程（大幅提高代码质量）。** 通过使用 GitHub 的拉取请求系统进行代码同行评审，我看到一个又一个团队在利用集体知识编写更干净、更快、更模块化的代码方面取得了进步。由于代码审查有助于团队成员识别和解决错误、设计缺陷和可维护性，最终导致代码质量更高。

**可重现性。** 指的是对代码库的每次更改都记录在 Git 历史中。这使得跟踪更改、回滚到先前版本和重现过去的实验变得极其容易。我无法低估其在调试、代码维护和确保任何实验发现可靠性的重要性。

## **针对不同类型工作的不同工作流程**

### **特性分支工作流程：标准代表**

这是开发团队中最常见的 Git 工作流程。它在流行度方面很难被取代，而且有很好的理由。在特性分支工作流程中，每个新的功能或代码的改进都在其自己的专用分支中开发，与主代码库分开。

分支工作流程为每个开发者提供了一个隔离的工作空间（一个分支）——他们自己的项目完整副本。这允许团队中的每个人都能专注于独立的工作，不受项目其他部分发生的事情的影响。他们可以做出代码变更并忘记上游开发，独立工作直到他们准备好分享他们的代码。

到那时，他们可以利用 GitHub 的拉取请求（PR）功能来促进代码审查，并与团队协作，确保更改在合并到代码库之前得到评估和批准。

这种方法对敏捷开发团队以及需要频繁代码变更的复杂项目团队特别有益。

特性分支工作流程可能看起来是这样的：

```py
# In your terminal:

$ git switch <new_branch_name> # Creates and switches onto a new branch
$ git push -u origin <new_branch_name> # For first push only. Creates new working branch on the remote repository

# Create and activate your virtual environment. Pip install any required packages.

$ python3 -m venv <new_venv_name>
$ source new_venv_name/bin/activate
$ pip install requirements.txt (or <packages>)

# Make changes to your code in feature branch
# Regularly stage and commit your code changes, and push to remote. For example:

$ git add <file> # Stages the file to prepare repo snapshot for commit
$ git commit -m "<Your descriptive message>" # Records file snapshots into your version history
$ git push # Sends local commits to the remote repository; to your working branch

# Raise Pull Request (PR) on repo's webpage. Request reviewer(s) in PR.
# After PR is approved and merged to `main`, delete working branch.
```

### **集中式工作流程：Git 入门**

我认为这种方法是一种入门级工作流程。我的意思是，`main`主分支是唯一一个变更进入仓库的点。一个单一的`main`分支被用于所有开发和所有变更都提交到这个分支，忽略分支的存在（我们不是一直忽略软件功能吗？）。

这不是你会在高速开发团队或持续交付团队中找到的方法。所以你可能想知道——集中式 Git 工作流程是否真的有合理的理由？

想到了两个用例。

首先，集中式 Git 工作流程可以简化非常小团队的初始探索。当重点是快速原型设计和冲突风险最小——就像项目的早期阶段——集中式工作流程可以很方便。

其次，使用集中式 Git 工作流程可以是一个将团队迁移到版本控制的好方法，因为它不需要除了`main`以外的任何分支。但请谨慎使用，因为事情可能会迅速变得一团糟。随着代码库的增长或更多人参与，代码冲突和意外覆盖的风险会更大。

否则，集中式 Git 工作流程通常不推荐用于持续开发，尤其是在团队环境中。

集中式工作流程可能看起来是这样的：

```py
# In your terminal:

$ git checkout <main> # Switches onto `main` branch

# Create and activate your virtual environment. Pip install any required packages.

$ python3 -m venv <new_venv_name>
$ source new_venv_name/bin/activate
$ pip install requirements.txt (or <packages>)

# Make changes to code
# Regularly stage and commit your code changes, and push to remote. For example:

$ git add <file> # Stages the file to prepare repo snapshot for commit
$ git commit -m "<Your descriptive message>" # Records file snapshots into your version history
$ git push # Sends local commits to the remote repository; to whichever branch you're working on. In this case, the `main` branch
```

## **ML 工作流程：分支实验**

与传统软件开发团队相比，数据科学家和 MLOps 团队有一个相对独特的用例。机器学习和 AI 项目的开发本质上是实验性的。因此，从 Git 工作流程的角度来看，协议需要灵活以适应频繁迭代和复杂的分支策略。你可能还需要跟踪代码之外的内容，如实验结果、数据或模型工件。

功能分支与实验分支相结合可能是最受欢迎的方法。

这种方法从熟悉的功能分支工作流程开始。然后在功能分支内，为特定的实验创建子分支。想想：“experiment_hyperparam_tuning”或“experiment_xgboost”。这种工作流程提供了足够的粒度和灵活性，以跟踪单个实验。并且与标准功能分支一样，这可以隔离开发，允许探索实验方法，而不会影响主代码库或其他开发者的工作。

但**注意**：我说它很受欢迎，但这并不意味着分支实验工作流程易于管理。如果事情变得过于复杂，所有这些都可能变成一团糟的意大利面式分支。这种工作流程涉及频繁的分支和合并，这在快速实验面前可能感觉像是不必要的开销。

分支实验工作流程可能看起来像这样：

```py
# In your terminal:

$ git checkout <feature_branch> # Move onto a feature branch ready for ML experiments
$ git switch <experiment_branch> # Creates and switches onto a new branch for experiments

# Create and activate your virtual environment. Pip install any required packages.
# Make changes to your code in feature branch.
# Continue as in Feature Branching workflow.
```

## **可重复的 ML 工作流程**

将像[MLflow](https://mlflow.org/docs/latest/getting-started/index.html)这样的工具集成到功能分支工作流程或分支实验工作流程中，提供了额外的可能性。可重复性是 ML 项目的一个关键关注点，这也是为什么存在像 MLflow 这样的工具。它有助于管理整个机器学习生命周期。

对于我们的工作流程，MLflow 通过启用实验跟踪、在注册表中记录模型运行和比较不同模型规格的性能来增强我们的能力。

对于分支实验工作流程，MLflow 的集成可能看起来像这样：

```py
# In your terminal:

$ git checkout <feature_branch> # Move onto a feature branch ready for ML experiments
$ git switch <experiment_branch> # Creates and switches onto a new branch for experiments

# Create and activate your virtual environment. Pip install any required packages.
# Initialise MLflow within your Python script.
# Make changes to branch. As you experiment with different hyperparameters or model architectures, create new experiment branches and log the results with MLflow.
# Regularly stage and commit your code changes and MLflow experiment logs. For example:

$ git add <file> <file> <file> # Stages the file to prepare repo snapshot for commit
$ git commit -m "<Your descriptive message>" # Records file snapshots into your version history
$ git push # Sends local commits to the remote repository; to your working branch

# Use the MLflow UI or API to compare the performance of different experiments within your feature branch. You may want to select the best-performing model based on the logged metrics.
# Merge experimental branch(es) into the parent feature branch. For example:

$ git checkout <feature_branch> # Switch back onto the parent feature branch
$ git merge <experiment_branch> # Merge experiment branch into the parent feature branch

# Raise Pull Request (PR) to merge it into `main` once the feature branch work is completed. Request reviewers. Delete merged branches.
# Deploy if applicable. If the model is ready for deployment, use the logged model artifact from MLflow to deploy it to a production environment.
```

# **要点**

我上面分享的 Git 工作流程应该为你的团队提供一个良好的起点，以简化协作开发，并帮助他们构建高质量的数据和 AI 解决方案。它们不是僵化的模板，而是可适应的框架。尝试实验不同的工作流程。然后调整它们，以制定最适合你需求的方法。

+   **Git 工作流程简化**：另一种选择过于可怕，过于混乱，过于缓慢，无法持续。它正在阻碍你。

+   **你的团队很重要**：理想的流程将根据你团队的大小、结构和项目复杂性而有所不同。

+   **项目需求**：项目的具体需求，如发布频率和 ML 实验水平，也将影响你对工作流程的选择。

最终，最适合任何数据或 MLOps 开发团队的 Git 工作流程是满足该团队特定需求和开发过程的那个。
