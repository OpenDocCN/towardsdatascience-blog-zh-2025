# Polars 与 Pandas — 独立速度比较

> 原文：[`towardsdatascience.com/polars-vs-pandas-an-independent-speed-comparison/`](https://towardsdatascience.com/polars-vs-pandas-an-independent-speed-comparison/)

## 概述

1.  [简介 — 目的和原因](https://medium.com/@ebbeberge/6b89b2d71aea#a6cb)

1.  [数据集、任务和设置](https://medium.com/@ebbeberge/6b89b2d71aea#969a)

1.  [结果](https://medium.com/@ebbeberge/6b89b2d71aea#f09e)

1.  [结论](https://medium.com/@ebbeberge/6b89b2d71aea#ca94)

1.  [总结](https://medium.com/@ebbeberge/6b89b2d71aea#c801)

## 简介 — 目的和原因

处理大量数据时，速度很重要。如果你在云数据仓库或类似环境中处理数据，那么你的数据摄取和处理的执行速度会影响以下方面：

+   **云成本：** 这可能是最大的因素。在大多数计费模型中，计算时间越多，成本就越高。在基于预分配资源一定量的其他计费中，如果你的数据摄取和处理速度更快，你可以选择更低的服务级别。

+   **数据时效性：** 如果您有一个需要 5 分钟处理数据的实时流，那么当用户通过例如 Power BI 报告查看数据时，至少会有 5 分钟的延迟。在某些情况下，这种差异可能很大。即使是批处理作业，数据时效性也很重要。如果您每小时运行一个批处理作业，如果它只需要 2 分钟而不是 20 分钟，那就好多了。

+   **反馈循环：** 如果您的批处理作业只需一分钟就能运行，那么您将获得非常快速的反馈循环。这可能会使您的工作更加愉快。此外，它还使您能够更快地发现逻辑错误。

如您从标题中可能已经理解的那样，我将提供两个 Python 库 Polars 和 Pandas 之间的速度比较。如果您之前对 Pandas 和 Polars 有所了解，那么您知道 Polars 是（相对）新加入的成员，并宣称比 Pandas 快得多。您可能也知道 Polars 是用 Rust 实现的，这是许多其他现代 Python 工具的趋势，如 [uv](https://github.com/astral-sh/uv) 和 [Ruff](https://github.com/astral-sh/ruff)。

我想要在 Polars 和 Pandas 之间进行速度比较测试有两个明显的理由：

### 原因 1 — 调查主张

Polars 在其网站上宣称：*与 pandas 相比，它（Polars）可以实现超过 30 倍的性能提升*。

正如你所见，你可以通过他们提供的[链接到基准测试](https://github.com/pola-rs/polars-benchmark/tree/main)来查看。他们公开速度测试的做法值得赞扬。但如果你正在为自己的工具和竞争对手的工具编写比较测试，那么可能会有轻微的利益冲突。我并不是说他们故意夸大 Polars 的速度，而是他们可能无意识地选择了有利于比较的结果。

**因此，进行速度比较测试的第一个原因很简单，就是看看这能否支持 Polars 提出的声明。**

### 原因 2 — 更细粒度

在 Polars 和 Pandas 之间进行速度比较测试的另一个原因是让它稍微更透明地展示性能提升可能出现在哪里。

如果你在这两个库方面都是专家，这可能已经很明显了。然而，Polars 和 Pandas 之间的速度测试主要对那些考虑更换工具的人感兴趣。在这种情况下，你可能还没有太多地尝试 Polars，因为你不确定它是否值得。

**因此，进行速度比较测试的第二个原因很简单，就是看看速度提升出现在哪里。**

我想要在不同的任务上测试这两个库，这些任务既包括数据摄取也包括数据处理。我还想考虑既小又大的数据集。我将坚持在数据工程中的常见任务，而不是那些很少使用的晦涩任务。

### 我不会做的事情

+   **我不会提供 Pandas 或 Polars 的教程**。如果你想学习 Pandas 或 Polars，那么从它们的文档开始是一个好地方。

+   **我不会涵盖其他常见的数据处理库**。这可能会让 PySpark 的粉丝感到失望，但拥有分布式计算模型使得比较变得有些困难。你可能会发现，在非常容易并行化的任务上，PySpark 比 Polars 更快，但在需要将所有数据保持在内存中以提高传输速度的其他任务上则较慢。

+   **我不会提供完整的可重复性**。因为这是一个谦逊的博客文章，所以我只会解释我使用的数据集、任务和系统设置。我不会提供一个包含数据集的完整运行环境，并将一切整理得井井有条。这不是一个精确的科学实验，而是一个只关心粗略估计的指南。

最后，在我们开始之前，我想说我喜欢 Polars 和 Pandas 这两个工具。显然，我没有从他们那里获得任何财务或其他补偿，也没有其他动机，只是对他们的性能感到好奇 ☺️

## 数据集、任务和设置

让我们先描述一下我将要考虑的数据集，库将要执行的任务，以及我将运行它们的系统设置。

### 数据集

在大多数公司，你都需要处理小数据和（相对）大数据集。在我看来，一个好的数据处理工具可以应对光谱的两端。小数据集挑战任务的启动时间，而大数据集则挑战可扩展性。我将考虑两个数据集，它们都可以在 Kaggle 上找到：

+   **关于 CSV 格式的较小数据集：** CSV 文件无处不在，这并不是秘密！它们通常相当小，来自 Excel 文件或数据库转储。还有什么比[经典的鸢尾花数据集](https://www.kaggle.com/datasets/uciml/iris)（许可协议为[CC0 1.0 Universal License](https://creativecommons.org/publicdomain/zero/1.0/))更好的例子，它有 5 列和 150 行。我在 Kaggle 上链接的鸢尾花版本有 6 列，但经典的版本没有运行索引列。所以如果你想得到与我完全相同的数据库，请删除这个列。从任何角度想象，鸢尾花数据集都是小数据。

+   **关于 Parquet 格式的庞大数据集：** Parquet 格式对于大数据来说非常实用，因为它具有内置的按列压缩（以及许多其他优点）。我将使用代表金融交易的[Transaction 数据集](https://www.kaggle.com/datasets/ismetsemedov/transactions)（许可协议为[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)）作为示例。该数据集有 24 列和 7483766 行。在 Kaggle 上找到的 CSV 格式接近 3GB。我使用了 Pandas 和 Pyarrow 将其转换为 Parquet 文件。由于 Parquet 文件格式的压缩，最终结果仅为 905MB。这处于人们所说的“大数据”的低端，但对我们来说已经足够了。

### 任务

我将在五个不同的任务上进行速度比较。前两个是 I/O 任务，而后三个是数据处理中的常见任务。具体来说，任务包括：

1.  **读取数据：** 我将使用两个库的相应方法`read_csv()`和`read_parquet()`读取两个文件。我不会使用任何可选参数，因为我想要比较它们的默认行为。

1.  **写入数据：** 我将使用 Pandas 的`to_csv()`和`to_parquet()`方法以及 Polars 的`write_csv()`和`write_parquet()`方法将两个文件都写回相同的副本作为新文件。我不会使用任何可选参数，因为我想要比较它们的默认行为。

1.  **计算数值表达式：** 对于鸢尾花数据集，我将计算表达式`SepalLengthCm ** 2 + SepalWidthCm`作为 DataFrame 副本中的新列。对于交易数据集，我将简单地计算表达式`(amount + 10) ** 2`作为 DataFrame 副本中的新列。我将使用标准的[在 Pandas 中转换列](https://pandas.pydata.org/docs/getting_started/intro_tutorials/05_add_columns.html)的方式，而在 Polars 中，我将使用标准的函数`all()`、`col()`和`alias()`来实现等效的转换。

1.  **过滤器：**对于鸢尾花数据集，我将选择符合条件 `SepalLengthCm >= 5.0` 和 `SepalWidthCm <= 4.0` 的行。对于交易数据集，我将选择符合分类条件 `merchant_category == 'Restaurant'` 的行。我将使用每个库中基于布尔表达式的标准过滤方法。在 pandas 中，这可以通过 `df_new = df[df['col'] < 5]` 这样的语法实现，而在 Polars 中，这可以通过 `filter()` 函数和 `col()` 函数类似地实现。我将使用 `&` 操作符来组合鸢尾花数据集的两个数值条件。

1.  **分组：**对于鸢尾花数据集，我将按 `Species` 列进行分组，并计算每个物种的四个列 `SepalLengthCm`、`SepalWidthCm`、`PetalLengthCm` 和 `PetalWidthCm` 的平均值。对于交易数据集，我将按 `merchant_category` 列进行分组，并计算 `merchant_category` 中每个类别的实例数量。当然，我将使用 Pandas 中的 `groupby()` 函数和 Polars 中的 `group_by()` 函数以明显的方式完成。

### **设置**

+   **系统设置：**我使用 16GB RAM 和一个拥有 6 个核心（通过超线程提供 12 个逻辑核心）的 Intel Core i5–10400F CPU 在本地运行所有任务。所以这绝对不是最先进的，但对于简单的基准测试来说已经足够好了。

+   **Python：**我正在运行 Python 3.12。这不是最新的稳定版本（最新的稳定版本是 Python 3.13），但我认为这是一个好事。通常，云数据仓库中支持的最新 Python 版本会落后一两个版本。

+   **Polars & Pandas：**我正在使用 Polars 版本 1.21 和 Pandas 2.2.3。这两个版本都是两个包的最新稳定发布版。

+   **时间测试：**我正在使用 Python 的标准 timeit 模块，并对 10 次运行的结果进行中位数计算。

尤其有趣的是，Polars 将如何通过多线程利用 12 个逻辑核心。有方法可以让 Pandas 利用多个处理器，但我想在不进行任何外部修改的情况下比较 Polars 和 Pandas。毕竟，这可能是它们在世界上大多数公司运行的方式。

## 结果

在这里，我将记录每个五个任务的结果，并做一些简要的评论。在下一节中，我将尝试将主要观点总结成结论，并指出 Polars 在这次比较中的一个缺点：

### 任务 1 — 读取数据

**读取任务的中位数运行时间（10 次运行）：**

```py
# Iris Dataset
Pandas: 0.79 milliseconds
Polars: 0.31 milliseconds

# Transactions Dataset
Pandas: 14.14 seconds
Polars: 1.25 seconds
```

对于读取 Iris 数据集，Polars 比 Pandas 快大约 2.5 倍。对于交易数据集，差异更加明显，Polars 比 Pandas 快 11 倍。我们可以看到，Polars 在读取小型和大型文件方面都比 Pandas 快得多。性能差异随着文件大小的增加而增长。

### 任务 2— **写入数据**

**写入任务的中位数运行时间（10 次运行）：**

```py
# Iris Dataset
Pandas: 1.06 milliseconds
Polars: 0.60 milliseconds

# Transactions Dataset
Pandas: 20.55 seconds
Polars: 10.39 seconds
```

对于写入 iris 数据集，Polars 比 Pandas 快约 75%。对于事务数据集，Polars 比 Pandas 快约 2 倍。再次看到 Polars 比 Pandas 快，但这里的差异小于读取文件时的差异。尽管如此，性能上的近 2 倍差异仍然是一个巨大的差异。

### 任务 3 — **计算数值表达式**

**计算数值表达式任务 10 次运行的中位运行时间如下**：

```py
# Iris Dataset
Pandas: 0.35 milliseconds
Polars: 0.15 milliseconds

# Transactions Dataset
Pandas: 54.58 milliseconds
Polars: 14.92 milliseconds
```

对于计算数值表达式，Polars 在 iris 数据集上的速度比 Pandas 快约 2.5 倍，在事务数据集上快约 3.5 倍。这是一个相当大的差异。需要注意的是，即使在大型数据集事务中，两个库计算数值表达式的速度都很快。

### 任务 4 — **过滤器**

**过滤任务 10 次运行的中位运行时间如下**：

```py
# Iris Dataset
Pandas: 0.40 milliseconds
Polars: 0.15 milliseconds

# Transactions Dataset
Pandas: 0.70 seconds
Polars: 0.07 seconds
```

对于过滤器，Polars 在 iris 数据集上快 2.6 倍，在事务数据集上快 10 倍。这可能是对我来说最令人惊讶的改进，因为我怀疑过滤任务的性能提升不会这么巨大。

### 任务 5 — 分组

**分组任务 10 次运行的中位运行时间如下**：

```py
# Iris Dataset
Pandas: 0.54 milliseconds
Polars: 0.18 milliseconds

# Transactions Dataset
Pandas: 334 milliseconds 
Polars: 126 milliseconds
```

对于分组任务，在 iris 数据集的情况下，Polars 的速度提高了 3 倍。对于事务数据集，Polars 比 Pandas 快了 2.6 倍。

## 结论

在强调以下每个点之前，我想指出，在整个比较过程中，Polars 处于某种不公平的位置。在实践中，通常会在一次之后执行多个数据转换。为此，Polars 有[懒加载 API](https://docs.pola.rs/user-guide/concepts/lazy-api/)，在计算之前优化这一点。由于我已经考虑了单次摄取和转换，Polars 的这个优势被隐藏了。这在实际情况下会有多大的改进还不清楚，但它可能会使性能差异更大。

### 数据摄取

**Polars 在读取和写入数据方面都显著快于 Pandas**。在读取数据时，差异最大，对于事务数据集，性能差异达到了 11 倍。在所有测量中，Polars 的表现都显著优于 Pandas。

### 数据处理

**Polars 在常见数据处理任务上比 Pandas 快得多**。这种差异在过滤器任务中最为明显，但至少可以预期在整体性能上会有 2-3 倍的区别。

### 最终结论

**Polars 在所有任务中，无论是小数据还是大数据，都始终比 Pandas 快**。改进非常显著，从 2 倍到惊人的 11 倍不等。当涉及到读取大型 parquet 文件或执行过滤语句时，Polars 在 Pandas 之前飞跃。

然而…**正如 Polars 的基准测试所暗示的，Polars 的性能远未达到比 Pandas 快 30 倍的水平**。我认为我所展示的任务是在现实硬件基础设施上执行的标准任务。因此，我认为我的结论为我们提供了质疑 Polars 提出的关于你可以期望的改进的声明的空间。

尽管如此，我坚信 Polars 的性能比 Pandas 快得多。使用 Polars 并不比使用 Pandas 复杂。**因此，对于你的下一个数据工程项目，其中数据适合内存，我强烈建议你选择 Polars 而不是 Pandas**。

## 总结

![图片](img/cf2306efd90aba9f040fd7c802d2dba1.png)

图片由 [Spencer Bergen](https://unsplash.com/@spencerbergen?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral) 提供

我希望这篇博客文章能给你关于 Polars 和 Pandas 速度差异的不同视角。如果你在 Polars 和 Pandas 的性能差异方面有与我不同的体验，请留言评论。

如果你对 AI、数据科学或数据工程感兴趣，请关注我或在 [LinkedIn](https://www.linkedin.com/in/eirik-berge/) 上与我联系。

**喜欢我的写作？** 查看我的一些其他帖子：

+   [作为数据科学家成功所需的软技能](https://medium.com/towards-data-science/the-soft-skills-you-need-to-succeed-as-a-data-scientist-ceac760230d3)

+   [如何作为数据科学家编写高质量的 Python 代码](https://towardsdatascience.com/how-to-write-high-quality-python-as-a-data-scientist-cde99f582675)

+   [用漂亮的类型提示让你的 Python 代码现代化](https://towardsdatascience.com/modernize-your-sinful-python-code-with-beautiful-type-hints-4e72e98f6bf1)

+   [在 Python 中可视化缺失值竟然如此简单](https://towardsdatascience.com/visualizing-missing-values-in-python-is-shockingly-easy-56ed5bc2e7ea)

+   [使用 PyOD 在 Python 中介绍异常/离群值检测 🔥](https://towardsdatascience.com/introducing-anomaly-outlier-detection-in-python-with-pyod-40afcccee9ff)
