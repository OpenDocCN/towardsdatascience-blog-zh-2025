# Python 中的现代 DataFrame：使用 Polars 和 DuckDB 的动手教程

> 原文：[`towardsdatascience.com/modern-dataframes-in-python-a-hands-on-tutorial-with-polars-and-duckdb/`](https://towardsdatascience.com/modern-dataframes-in-python-a-hands-on-tutorial-with-polars-and-duckdb/)

如果你曾经使用 Python 进行数据处理，你可能经历过等待 Pandas 操作完成几分钟的挫败感。

起初，一切看起来都很正常，但随着你的数据集增长和工作流程变得更加复杂，你的笔记本电脑突然感觉像是在准备起飞。

几个月前，我参与了一个分析超过 300 万行数据的电子商务交易项目。

这是一次相当有趣的经历，但大多数时候，我都在看着通常只需几秒钟就能完成的简单 groupby 操作突然拖到几分钟。

到那时，我意识到 Pandas 是很棒的工具，但并不总是足够。

本文探讨了 Pandas 的现代替代方案，包括 Polars 和 DuckDB，并探讨了它们如何简化并改进大数据集的处理。

为了清晰起见，在我们开始之前，让我坦白几点。

这篇文章并不是深入探讨 Rust 内存管理，也不是宣布 Pandas 已过时。

而不是深入探讨，这是一个实用的、动手的指南。你将看到真实示例、个人经验和关于工作流程的实用见解，这些可以为你节省时间和精力。

* * *

## 为什么 Pandas 会感觉慢

在我参与电子商务项目的时候，我记得处理过超过两个吉字节大小的 CSV 文件，Pandas 中的每个过滤或聚合通常需要几分钟才能完成。

在那段时间里，我会盯着屏幕，希望能一边喝咖啡一边看几集电视剧，而代码在运行。

我遇到的主要痛点是速度、内存和工作流程复杂性。

我们都知道大型 CSV 文件会消耗大量的 RAM，有时甚至超过我的笔记本电脑能够舒适处理的能力。再加上，链式多个转换也使得代码更难维护，执行速度更慢。

Polars 和 DuckDB 以不同的方式解决这些挑战。

Polars，用 Rust 编写，使用多线程执行来高效处理大数据集。

另一方面，DuckDB 是为分析而设计的，它执行 SQL 查询而不需要你将所有内容加载到内存中。

基本上，它们各自都有其独特的超级能力。Polars 是速度之王，而 DuckDB 则有点像是内存魔术师。

而最好的部分？它们都无缝集成到 Python 中，让你可以在不重写整个工作流程的情况下增强你的工作流程。

## 设置你的环境

在我们开始编码之前，确保你的环境已经准备好。为了保持一致性，我使用了 Pandas 2.2.0、Polars 0.20.0 和 DuckDB 1.9.0。

固定版本可以让你在遵循教程或分享代码时省去很多麻烦。

```py
pip install pandas==2.2.0 polars==0.20.0 duckdb==1.9.0
```

在 Python 中，导入库：

```py
import pandas as pd
import polars as pl
import duckdb
import warnings
warnings.filterwarnings("ignore") 
```

例如，我将使用一个包含订单 ID、产品 ID、地区、国家、收入和日期等列的电子商务销售数据集。您可以从[Kaggle](https://www.kaggle.com/datasets)下载类似的集，或生成合成数据。

## 加载数据

高效地加载数据为你的整个工作流程奠定了基调。我记得有一个项目，CSV 文件有近 500 万行。

Pandas 处理了这个问题，但加载时间很长，测试过程中的重复加载非常痛苦。

那是你希望你的笔记本电脑有一个“快进”按钮的时刻之一。

转向 Polars 和 DuckDB 完全改善了所有事情，突然间，我几乎可以瞬间访问和操作数据，这实际上使测试和迭代过程变得更加愉快。

使用 Pandas：

```py
df_pd = pd.read_csv("sales.csv")
print(df_pd.head(3))
```

使用 Polars：

```py
df_pl = pl.read_csv("sales.csv")
print(df_pl.head(3))
```

使用 DuckDB：

```py
con = duckdb.connect()
df_duck = con.execute("SELECT * FROM 'sales.csv'").df()
print(df_duck.head(3))
```

DuckDB 可以直接查询 CSV 文件，而无需将整个数据集加载到内存中，这使得处理大型文件变得容易得多。

## 过滤数据

这里的问题是，当处理数百万行数据时，Pandas 的过滤可能会很慢。我曾经需要分析一个庞大的销售数据集中的欧洲交易。Pandas 花费了数分钟，这减缓了我的分析。

使用 Pandas：

```py
filtered_pd = df_pd[df_pd.region == "Europe"]
```

Polars 更快，并且可以高效地处理多个过滤器：

```py
filtered_pl = df_pl.filter(pl.col("region") == "Europe")
```

DuckDB 使用 SQL 语法：

```py
filtered_duck = con.execute("""
    SELECT *
    FROM 'sales.csv'
    WHERE region = 'Europe'
""").df()
```

现在，你可以在几秒钟内而不是几分钟内过滤大型数据集，这样你就有更多时间专注于真正重要的见解。

## 快速聚合大型数据集

聚合通常是 Pandas 开始感觉缓慢的地方。想象一下为营销报告计算每个国家的总收入。

在 Pandas 中：

```py
agg_pd = df_pd.groupby("country")["revenue"].sum().reset_index()
```

在 Polars 中：

```py
agg_pl = df_pl.groupby("country").agg(pl.col("revenue").sum()) 
```

在 DuckDB 中：

```py
agg_duck = con.execute("""
    SELECT country, SUM(revenue) AS total_revenue
    FROM 'sales.csv'
    GROUP BY country
""").df()
```

我记得在一个包含 1000 万行数据的集上运行这个聚合操作。在 Pandas 中，这几乎花了半小时。Polars 在不到一分钟内完成了相同的操作。

减轻压力的感觉几乎就像完成了一场马拉松，意识到你的腿还能工作。

## 规模化数据集的连接

将数据集合并是那些听起来很简单，直到你真正陷入数据泥潭的事情之一。

在实际项目中，你的数据通常存在于多个来源，因此你必须使用共享列，如客户 ID，将它们结合起来。

我是在一个需要将数百万个客户订单与同样庞大的人口统计数据集相结合的项目中艰难地学到这一点的。

每个文件本身就足够大了，但合并它们感觉就像是在你的笔记本电脑恳求饶命的同时，试图把两个拼图碎片拼在一起。

Pandas 花费了如此长的时间，以至于我开始像人们计时微波爆米花完成所需时间一样计时连接操作。

揭秘：爆米花每次都赢了。

Polars 和 DuckDB 给了我一条出路。

使用 Pandas：

```py
merged_pd = df_pd.merge(pop_df_pd, on="country", how="left")
```

Polars：

```py
merged_pl = df_pl.join(pop_df_pl, on="country", how="left")
```

DuckDB：

```py
merged_duck = con.execute("""
    SELECT *
    FROM 'sales.csv' s
    LEFT JOIN 'pop.csv' p
    USING (country)
""").df()
```

[连接](https://urbizedge.com/understanding-sql-joins/)大型数据集，这些操作曾经让你的工作流程陷入停滞，现在运行得既顺畅又高效。

## Polars 中的懒加载

在我的数据科学之旅早期，我没有意识到运行逐行转换会浪费多少时间。

Polars 采取了不同的方法。

它使用一种称为“懒加载”的技术，本质上是在你完成定义所有转换之前，不会执行任何操作。

它检查整个管道，确定最有效的路径，并同步执行所有操作。

这就像有一个朋友在你给出完整订单后再去厨房，而不是一个一个指令地执行并来回跑的人。

这篇 [TDS 文章](https://towardsdatascience.com/understanding-lazy-evaluation-in-polars-b85ccb864d0c/) 深入解释了懒加载。

下面是这个流程的示意图：

Pandas:

```py
df = df[df["amount"] > 100]
df = df.groupby("segment").agg({"amount": "mean"})
df = df.sort_values("amount")
```

Polars 懒模式：

```py
import polars as pl

df_lazy = (
    pl.scan_csv("sales.csv")
      .filter(pl.col("amount") > 100)
      .groupby("segment")
      .agg(pl.col("amount").mean())
      .sort("amount")
)

result = df_lazy.collect() 
```

我第一次使用懒模式时，没有看到即时结果感觉有点奇怪。但一旦我运行了最后的 `.collect()`，速度差异就很明显了。

懒加载不会神奇地解决每个性能问题，但它带来了 Pandas 没有设计到的效率级别。

* * *

## 结论与收获

与大型数据集打交道不必感觉像是在与你的工具搏斗。

使用 Polars 和 DuckDB 让我意识到问题并不总是数据。有时，问题是我用来处理它的工具。

如果你从这个教程中学到了一点，让它成为这一点：你不必放弃 Pandas，但当你的数据集开始超出其限制时，你可以寻求更好的东西。

Polars 不仅提供速度，还提供更智能的执行，然后 DuckDB 让你查询大文件就像它们很小一样。它们一起使处理大型数据集感觉更加可控和省力。

如果你想要深入了解本教程中探讨的思想，[Polars](https://docs.pola.rs/) 和 [DuckDB](https://duckdb.org/docs/stable/) 的官方文档是很好的起点。
