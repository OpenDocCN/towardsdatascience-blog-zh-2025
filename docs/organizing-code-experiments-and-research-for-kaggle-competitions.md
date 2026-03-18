# 为 Kaggle 比赛**组织代码、实验和研究**

> 原文：[`towardsdatascience.com/organizing-code-experiments-and-research-for-kaggle-competitions/`](https://towardsdatascience.com/organizing-code-experiments-and-research-for-kaggle-competitions/)
> 
> <mdspan datatext="el1762998801858" class="mdspan-comment">告诉我</mdspan>，我就忘记了。教我，我就记住了。参与其中，我就学会了。

<mdspan datatext="el1762999364682" class="mdspan-comment">这句古老的谚语依然</mdspan>适用，*实践出真知*是掌握新技能最富有教育意义的途径之一。在数据科学和机器学习的领域，参加比赛是获得实际操作经验和提升技能能力最有效的方法之一。

[Kaggle](https://www.kaggle.com/)是世界最大的数据科学社区，其比赛在业界享有极高的声誉。世界上许多领先的机器学习会议（例如，[NeurIPS](https://neurips.cc/))、组织（例如，[Google](https://ai.googleblog.com/))和大学（例如，[Stanford](https://stanford.edu/))都在 Kaggle 上举办比赛。

特色的 Kaggle 比赛向在私有排行榜上表现优异的选手颁发奖牌。最近，我参加了我的第一个颁发奖牌的 Kaggle 比赛，我有幸获得了**银牌**。这是[NeurIPS – Ariel 数据挑战 2025](https://www.kaggle.com/competitions/ariel-data-challenge-2025)。我并不打算在这里分享我的解决方案。如果你感兴趣，你可以在这里查看我的[solution here](https://ibrahimhabib.me/projects/ariel-2025)。

在参加之前，我没有意识到 Kaggle 除了测试机器学习技能之外还测试了哪些方面。

Kaggle 考验一个人的编码和软件工程技能。它强调正确组织代码库以快速迭代和尝试新想法的能力。它还考验了以清晰方式跟踪实验和结果的能力。

成为[NeurIPS 2025 比赛赛道](https://nips.cc/virtual/2025/events/Competition)的一部分，这是一个研究会议，也考验了快速有效地研究并学习新领域的能力。

总的来说，这次比赛让我受益匪浅，除了机器学习之外，还教会了我许多其他课程。

这篇文章的目的是与你分享一些这些非机器学习课程。它们都围绕一个原则：**组织**，**组织**，**组织**。

首先，我将说服你为什么清晰的代码结构和流程组织不是**浪费时间**或**锦上添花**，而是参加 Kaggle 比赛特别是任何成功的数据科学项目所必需的。然后，我将与你分享我在代码结构和实验过程中使用的一些技术和学到的经验教训。

我想以谦逊的态度开始。我绝不是这个领域的专家。我还在旅程的起点。我只希望一些读者能从中找到一些有益的教训，并从我的错误中学习。如果您有任何其他建议或建议，我敦促您分享它们，这样我们都可以一起学习。

## 1 科学黄金法则：组织

众所周知，自然科学家喜欢详细记录他们的工作和研究过程。不明确的步骤可能会导致（并将会导致）错误的结论和理解。不可重复的工作是科学的祸害。对于我们数据科学家来说，为什么应该有所不同呢？

### 1.1 但速度很重要！

常见的反驳论点是数据科学的性质是**快节奏**和**迭代的**。一般来说，实验成本低且快速；此外，世界上有谁喜欢写文档而不是编码和构建模型呢？

尽管我对此观点表示同情，我也喜欢快速的结果，但我担心这种心态是短视的。记住，任何数据科学项目的最终目标要么是提供准确、数据支持的、可重复的见解，要么是构建可靠和可重复的模型。如果快速的工作妥协了最终目标，那么它就毫无价值。

我解决这个困境的方法是将组织中的**平凡**部分尽可能地简化、快速和痛苦最小化。我们不应该寻求完全删除组织过程，而应该修复其缺陷，使其尽可能高效和富有成效。

### 1.2 缺乏组织的成本

想象一下这个场景。对于你的每一个实验，你都有一个 Kaggle 上的单独笔记本，它从加载数据和预处理到训练模型、评估它，最后提交它，样样都做。到目前为止，你已经运行了数十个实验。你发现了一个小错误，这个错误在你所有的实验中使用的数据加载函数中。修复它将是一场噩梦，因为你将不得不逐个检查你的笔记本，修复错误，确保没有引入新的错误，然后重新运行所有实验以获取更新的结果。所有这些本可以通过清晰的代码结构和可重用、模块化的代码来避免。

Drivendata (2022) 提到了一个无组织数据科学项目成本的绝佳例子。它提到了一个失败的数据科学项目，该项目耗时数月才完成，耗资数百万美元。失败归结于项目早期发现的一个错误结论。数据清洗中的代码错误污染了数据，导致了错误的见解。如果团队更好地跟踪数据来源和转换，他们就会更早地捕捉到这个错误，从而节省资金。

如果从这个章节中可以学到一点，那就是**组织不仅仅是锦上添花，而是数据科学项目的必要组成部分**。没有清晰的代码结构和流程组织，我们注定会犯错误、浪费时间，并产生不可复制的成果。

### 1.3 要跟踪和组织什么？

我认为有三个主要方面值得跟踪：

1.  代码库

1.  实验结果和配置

1.  研究和学习

## 2 代码库

最后，代码是任何数据科学项目的骨架。因此，我们可以从软件工程师那里学到一些东西。

### 2.1 仓库结构

只要你认真思考你的代码库结构，你就在做得很棒。

没有一个普遍认同的结构（也永远不会）。因此，本节非常主观和有偏见。我将讨论我喜欢并使用的通用结构。

我喜欢使用广泛流行的 [Cookiecutter Data Science](https://cookiecutter-data-science.drivendata.org/) (ccds) 模板来初始化我的工作。当你使用 ccds 初始化一个项目时，它会创建一个具有以下结构的文件夹。^(1)

```py
├── LICENSE            <- Open-source license if one is chosen
├── Makefile           <- Makefile with convenience commands like `make data` or `make train`
├── README.md          <- The top-level README for developers using this project.
├── data
│   ├── external       <- Data from third party sources.
│   ├── interim        <- Intermediate data that has been transformed.
│   ├── processed      <- The final, canonical data sets for modeling.
│   └── raw            <- The original, immutable data dump.
│
├── docs               <- A default mkdocs project; see www.mkdocs.org for details
│
├── models             <- Trained and serialized models, model predictions, or model summaries
│
├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
│                         the creator's initials, and a short `-` delimited description, e.g.
│                         `1.0-jqp-initial-data-exploration`.
│
├── pyproject.toml     <- Project configuration file with package metadata for 
│                         {{ cookiecutter.module_name }} and configuration for tools like black
│
├── references         <- Data dictionaries, manuals, and all other explanatory materials.
│
├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures        <- Generated graphics and figures to be used in reporting
│
├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
│                         generated with `pip freeze > requirements.txt`
│
├── setup.cfg          <- Configuration file for flake8
│
└── {{ cookiecutter.module_name }}   <- Source code for use in this project.
    │
    ├── __init__.py             <- Makes {{ cookiecutter.module_name }} a Python module
    │
    ├── config.py               <- Store useful variables and configuration
    │
    ├── dataset.py              <- Scripts to download or generate data
    │
    ├── features.py             <- Code to create features for modeling
    │
    ├── modeling                
    │   ├── __init__.py 
    │   ├── predict.py          <- Code to run model inference with trained models          
    │   └── train.py            <- Code to train models
    │
    └── plots.py                <- Code to create visualizations 
```

#### 2.1.1 环境管理

当你使用 ccds 时，你会被提示选择一个环境管理器。我个人更喜欢 [uv](https://docs.astral.sh/uv/#learn-more) 由 [Astral](https://astral.sh/) 提供。它记录了 `pyproject.toml` 文件中使用的所有包，并允许我们通过简单地使用 `uv sync` 来重新创建相同的环境。

在底层，`uv` 使用 venv。我发现使用 `uv` 比直接管理虚拟环境要简单得多，因为管理和读取 `pyproject.toml` 比读取 `requirements.txt` 要简单得多。

此外，我发现 `uv` 比使用 `conda` 更简单。`uv` 是专门为 Python 构建的，而 `conda` 则更加通用。

#### 2.1.2 生成的模块

该模板的大部分内容是 `{ cookiecutter.module_name }` 目录。在这个目录中，你定义了一个 Python 包，该包应包含你代码的所有重要部分（例如，预处理函数、模型定义、推理函数等）。

我发现这个包的使用非常有帮助，在第 2.3 节中，我将讨论这里应该放置什么内容，以及在 Jupyter 笔记本中应该放置什么内容。

#### 2.1.3 保持灵活性

不要认为这种结构是完美或完整的。你不必使用 `ccds` 提供的所有内容，如果项目需要，你可以（并且应该）对其进行修改。`ccds` 为你提供了一个很好的起点，以便根据你确切的项目需求和需求进行调整。

### 2.2 版本控制

Git 已经成为任何涉及代码的项目绝对必要的工具。它允许我们跟踪更改、回滚到早期版本，并且通过 GitHub 与团队成员协作。

当你使用 Git 时，你基本上是在访问一台可以修复你代码中引入的任何错误的时光机。如今，Git 的使用是不可或缺的。

### 2.3 三种代码类型

在数据科学社区中，何时使用 Python 脚本和何时使用 Jupyter 笔记本是一个长期争论的话题。在这里，我提出我对这个问题的立场。

我喜欢将所有代码分别存放在三个目录之一中：

1.  模块

1.  脚本

1.  笔记本

#### 2.3.1 模块

该模块应包含你创建的所有重要函数和类。

它的使用帮助我们最小化冗余，并为所有在数据上发生的重要操作创建单一的真实来源。

在数据科学项目中，一些操作将在所有的训练和推理工作流程中重复，例如从文件中读取数据、转换数据和模型定义。在所有的笔记本或脚本中重复所有这些函数是困难的，并且非常无聊。使用模块允许我们一次性编写代码，然后将其导入到任何地方。

此外，这有助于减少错误和错误。当模块中发现错误时，你只需在模块中修复一次，它就会自动在所有导入它的脚本和笔记本中修复。

#### 2.3.2 脚本

脚本目录包含.py 文件。这些文件是项目生成输出的唯一来源。它们是与我们的模块和代码交互的接口。

这些文件的两个主要用途是训练和推理。所有使用的模型都应该通过运行其中一个脚本创建，所有在 Kaggle 上的提交都应该使用这样的文件。

这些脚本的使用有助于使我们的结果可重现。为了重现一个旧的结果（例如，训练相同的模型），只需要克隆相同版本的 repo 并运行用于生成旧结果的脚本^(2)。

由于脚本是从命令行界面（CLI）运行的，使用库来管理 CLI 参数可以简化代码。我喜欢为没有太多配置选项的简单脚本使用[typer](https://typer.tiangolo.com/)，而对于复杂的脚本则使用[hydra](https://hydra.cc/docs/intro/)（我将在稍后更深入地讨论 hydra）。

#### 2.3.3 笔记本

Jupyter 笔记本因其提供的快速反馈循环而非常适合探索和原型设计。

在许多情况下，我会从笔记本中开始编写代码，以快速测试并找出所有错误。然后，我才会将其转移到模块中。

然而，笔记本不应该用来创建最终结果。它们难以重现和跟踪更改。因此，始终使用脚本创建最终输出。

## 3 在 Kaggle 上运行代码库

使用前一小节中讨论的结构，我们需要遵循以下步骤在 Kaggle 上运行我们的代码：

1.  克隆仓库

1.  安装所需包

1.  运行其中一个脚本

由于 Kaggle 为我们提供了一个 Jupyter Notebook 界面来运行我们的代码，并且大多数 Kaggle 比赛都对互联网访问有限制，因此提交并不像在本地机器上运行脚本那样简单。以下，我将讨论如何在 Kaggle 上执行上述每个步骤。

### 3.1 克隆仓库

首先，由于网络限制，我们无法在提交笔记本中直接从 GitHub 克隆我们的仓库。然而，Kaggle 允许我们将其他 Kaggle 笔记本的输出导入到当前笔记本中。因此，解决方案是创建一个单独的 Kaggle 笔记本，用于克隆我们的仓库并安装所需的软件包。然后，将这个笔记本的输出导入到提交笔记本中。

很可能您将使用私有仓库。在 Kaggle 上克隆私有仓库最简单的方法是使用个人访问令牌（PAT）。您可以通过遵循[这个指南](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)在 GitHub 上创建 PAT。一个很好的做法是为 Kaggle 创建一个具有最小所需权限的 PAT。

在克隆笔记本中，您可以使用以下代码来克隆您的仓库：

```py
from kaggle_secrets import UserSecretsClient
user_secrets = UserSecretsClient()
github_token = user_secrets.get_secret("GITHUB_TOKEN")
user = "YOUR_GITHUB_USERNAME"
CLONE_URL = f"https://oauth2:{github_token}@github.com/{user}/YOUR_REPO_NAME.git"
get_ipython().system(f"git clone {CLONE_URL}")
```

此代码将您的仓库下载到当前笔记本的工作目录中。它假设您已经将 PAT 存储在名为`GITHUB_TOKEN`的 Kaggle 秘密中。在运行它之前，请确保在笔记本设置中激活此秘密。

### 3.2 安装所需软件包

在克隆笔记本中，您也可以安装所需的软件包。如果您使用的是`uv`，您可以通过运行以下命令来构建自定义模块、安装它以及安装其依赖项：^(3)。

```py
cd ariel-2025 && uv build
```

这将在您的模块的`dist/`目录中创建一个 wheel 文件。然后您可以通过运行以下命令在自定义目录中安装它及其所有依赖项：^(4)。

```py
pip install /path/to/wheel/file --target /path/to/custom/dir
```

请确保将`/path/to/wheel/file`和`/path/to/custom/dir`替换为实际路径。`/path/to/wheel/file`将是`REPO_NAME/dist/`目录中`.whl`文件的路径。`/path/to/custom/dir`可以是您喜欢的任何目录。请记住自定义目录路径，因为后续笔记本将依赖于它来导入您的模块和项目依赖项。

我喜欢在单个笔记本中同时下载仓库和安装软件包。我将这个笔记本命名为与仓库相同的名称，以简化后续的导入。

### 3.3 运行脚本之一

在任何后续笔记本中要做的第一件事是导入包含克隆仓库和已安装软件包的笔记本。当您这样做时，Kaggle 会将导入笔记本中`/kaggle/working/`目录的内容存储到名为`/kaggle/input/REPO_NAME/`的目录中，其中`REPO_NAME`是仓库的名称^(5)。

许多时候，您的脚本将创建相对于其位置的输出（例如，提交文件）。默认情况下，您的代码将位于/kaggle/input/REPO_NAME/，该目录为只读。因此，您需要将仓库的内容复制到`/kaggle/working/`，这是当前工作目录，并且是可读写的。虽然这可能是不必要的，但这是一个无害的好习惯，可以防止愚蠢的问题。

```py
cp -r /kaggle/input/REPO_NAME/REPO_NAME/ /kaggle/working/
```

如果您直接从`/kaggle/working/scripts/`运行您的脚本，您将遇到导入错误，因为 Python 找不到已安装的包和您的模块。这可以通过更新`PYTHONPATH`环境变量来轻松解决。我使用以下命令更新它然后运行我的脚本：

```py
! export PYTHONPATH=/kaggle/input/REPO_NAME/custom_dir:$PYTHONPATH && cd /kaggle/working/REPO_NAME/scripts && python your_script.py --arg1 val1 --arg2 val2
```

我通常为了简便，将运行脚本的任何笔记本命名为脚本名称。此外，当我在 Kaggle 上重新运行笔记本时，我会用当前 Git 提交的哈希值来命名版本，以便跟踪使用了哪个版本的代码来生成结果。^(6)。

### 3.4 收集所有内容

最后，需要两个笔记本：

1.  克隆笔记本：克隆仓库并安装所需的包。

1.  脚本笔记本：运行其中一个脚本。

您可能需要在流程中添加更多的脚本笔记本。例如，您可能有一个用于训练的笔记本和一个用于推理的笔记本。这些笔记本中的每一个都将遵循上面讨论的脚本笔记本的结构。

当一个步骤运行时间较长且很少变化时，将流程中的每个步骤（例如，数据预处理、训练、推理）分离到自己的笔记本中是有用的。例如，在 Ariel 数据挑战中，我的预处理步骤运行了超过七个小时。如果我把所有东西都放在一个笔记本中，每次尝试新想法时我都要等待七个小时。此外，Kaggle 核的时限制将使得在一个笔记本中运行整个流程变得不可能。

然后，每个笔记本将导入前一个笔记本的输出并运行自己的步骤，并从这里开始构建。一个好的建议是将任何数据文件或模型的路径作为脚本的参数，这样您在 Kaggle 或其他环境中运行时可以轻松更改它们。

当您更新代码时，重新运行克隆笔记本以更新 Kaggle 上的代码。然后，仅重新运行必要的脚本笔记本以生成新的结果。

### 3.5 所有这些努力值得吗？

绝对是的！

我知道指定的流程在启动项目时会增加一些开销。然而，从长远来看，这将为您节省更多的时间和精力。您将能够在本地上编写所有代码，并在 Kaggle 上运行相同的代码。

当您创建新的模型时，您只需复制其中一个脚本笔记本并更改脚本即可。您的本地和 Kaggle 代码之间不会出现冲突。您将能够使用 Git 跟踪所有更改。您只需简单地检查出相应的 Git 提交并在 Kaggle 上重新运行必要的笔记本，就可以重现任何旧的结果。

此外，您将能够在任何您喜欢的机器上进行开发。所有东西都集中存储在 GitHub 上。您可以从本地机器工作。如果您需要更多计算能力，您可以从云虚拟机工作。如果您想在 Kaggle 上训练，您也可以这样做。您的所有代码和环境在所有地方都是相同的。

为如此大的便利付出如此小的代价是值得的。一旦管道设置完成，你就可以将其抛诸脑后，专注于真正重要的事情：研究和构建模型！

## 4 记录学习和研究

当你深入研究一个新领域时，你将花费大量时间进行研究和阅读论文。很容易在阅读到的所有信息中迷失方向，你可能会忘记某个想法或概念是在哪里遇到的。因此，管理和组织你的学习非常重要。

### 4.1 阅读跟踪

拉杰普尔卡(2023) 建议列出你阅读的所有论文和文章。这让你可以快速概述你阅读的内容，并在需要时参考它。

拉杰普尔卡教授还建议为每篇论文标注一、二或三星。一星论文是不相关的论文，但在阅读之前你可能不知道这一点。二星论文是相关的。三星论文是非常相关的。这让你可以快速过滤之后的阅读内容。

你还应该在每篇你阅读的论文上做笔记。这些笔记应该关注论文如何与你项目相关。它们应该简短以便于回顾，但要有足够的细节来把握主要思想。在论文列表中，你应该为每篇论文链接阅读笔记，以便于访问。

我还喜欢在论文本身上做笔记，例如高亮。如果你使用 PDF 阅读器或电子墨水设备，你应该存储论文的标注版本以供将来参考，并在你的笔记中链接它。如果你更喜欢在纸上阅读，你可以扫描标注版本并将其数字化存储。

### 4.2 工具

对于大多数文档，我喜欢使用 Google Docs，因为它让我可以从任何地方访问我的笔记。此外，你可以在 Google Docs 中使用 Markdown 编写，这是我最喜欢的写作格式（我正在用它来写这篇文章）。

[Zotero](https://www.zotero.org/) 是一个管理研究论文的出色工具。它在存储和组织论文方面做得很好。你可以为每个项目创建一个收藏夹，并将所有相关论文存储在那里。使用浏览器扩展导入论文非常简单，并且以 BibTeX 格式导出引用非常直接。

## 5 实验跟踪

在数据科学项目中，你经常会运行许多实验并尝试许多想法。再次强调，很容易在这片混乱中迷失方向。

我们已经通过正确地结构化我们的代码库并使用脚本来运行我们的实验取得了很大的进步。尽管如此，我仍想讨论两个软件工具，它们可以帮助我们做得更好。

### 5.1 Wandb

[Weights and Biases (wandb)](https://wandb.ai/site/)，发音为“w-and-b”（代表权重和偏差）或“wand-b”（代表像魔杖一样神奇）或“wan-db”（代表像数据库一样），是一个跟踪实验的出色工具。它允许我们在一个中心位置运行多个实验并保存它们的配置和结果。

![](img/011773b7360d77113eb7342518dcc128.png "在赢得 Kaggle 竞赛奖牌的过程中学到的经验和技巧")

图 1：Adrish Dey 的 [使用 Hydra 配置 W&B 项目](https://wandb.ai/adrishd/hydra-example/reports/Configuring-W-B-Projects-with-Hydra--VmlldzoxNTA2MzQw) 文章中的 Wandb 仪表板图片

Wandb 为我们提供了一个仪表板，用于比较不同实验的结果、使用的超参数和训练曲线。它还跟踪系统指标，如 GPU 和 CPU 使用情况。

Wandb 还集成了 [Hugging Face](https://huggingface.co/) 库，使得在使用 [transformers](https://huggingface.co/docs/transformers/en/index) 时跟踪实验变得容易。

一旦开始使用多个实验，wandb 就成为了一个不可或缺的工具。

### 5.2 Hydra

[Hydra](https://hydra.cc/) 是 Meta 开发的一个工具，简化了配置管理。它允许你将所有配置定义在 YAML 文件中，并可以从 CLI 轻松覆盖它们。

这是一个非常灵活的工具，适用于多种用例。[本指南](https://hydra.cc/docs/patterns/configuring_experiments/) 讨论了如何使用 Hydra 进行实验配置。

## 6 端到端流程

![](img/3bc0200fc81669ed052cd811b6d73f3c.png)

图 2：作者使用 Mermaid.js 创建的端到端组织 Kaggle 竞赛流程

图 2 总结了本文中讨论的流程。首先，我们研究想法并记录我们的学习成果。然后，我们在本地机器上的 Jupyter Notebooks 中对这些想法进行实验。一旦我们有一个可行的想法，我们就将代码重构到我们的模块中，并创建脚本来运行实验。我们在 Kaggle 上运行新的实验。最后，我们跟踪新实验的结果。

由于一切都被仔细跟踪，我们能够预测我们的不足，并迅速回到研究或开发阶段进行修复。

## 7 结论

无序是数据科学项目中所有邪恶的根源。如果我们想要产生可靠且可重复的工作，我们必须在我们的流程中追求组织和清晰。Kaggle 竞赛也不例外。

在本文中，我们讨论了一种组织代码库的技术、跟踪研究和学习的小贴士以及跟踪实验的工具。图 2 总结了所提出的技术。

希望这篇文章对你有所帮助。如果你有任何其他建议或意见，请在下面的评论部分分享。

祝你在下一场比赛中好运！

### 7.1 参考文献

Drivendata. (2022). *The 10 Rules of Reliable Data Science*.

Rajpurkar, P. (2023). *Harvard CS197: AI Research Experiences*。[`www.cs197.seas.harvard.edu`](https://www.cs197.seas.harvard.edu/)

脚注

1.  文章发布时，文件夹结构是从 [ccds 网站](https://cookiecutter-data-science.drivendata.org/) 复制的。

1.  假设所有随机数生成器都使用了种子。

1.  当在 Jupyter Notebook 中的代码单元运行任何 bash 命令时，你需要在命令前加上 `!` 前缀。

1.  当在 Kaggle 中的代码单元运行任何 bash 命令时，假设当前工作目录是 `/kaggle/working/`。任何之前的 `cd` 命令都将被忽略。你必须在所有命令中指定从 `/kaggle/working/` 的路径，或者在你的命令前加上 `cd PATH &&` 以在运行剩余命令之前更改目录。

1.  如果你忘记了路径，你可以从笔记本右侧边栏的笔记本标签中复制它。

1.  注意，在重新运行脚本笔记本之前，你必须重新运行克隆笔记本以更新 Kaggle 上的代码。不要忘记在重新运行脚本笔记本之前，在克隆笔记本中“检查更新”。
