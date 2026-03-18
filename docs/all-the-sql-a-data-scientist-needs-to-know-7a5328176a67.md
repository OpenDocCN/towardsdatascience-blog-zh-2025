# 数据科学家所需了解的所有 SQL

> 原文：[`towardsdatascience.com/all-the-sql-a-data-scientist-needs-to-know-7a5328176a67/`](https://towardsdatascience.com/all-the-sql-a-data-scientist-needs-to-know-7a5328176a67/)

![使用 Grok 2 人工生成的图像](img/27e94cccc84bd4bc235a1817d6d7c433.png)

使用 Grok 2 人工生成的图像。

## 简介

在我看来，SQL 是数据专业人士应该具备的最重要技能之一。无论你是数据分析师、数据科学家还是软件开发人员，你很可能会每天使用 SQL 与数据库进行交互。

从数据科学家的角度来看，你不需要成为 SQL 专家。能够使用 SQL 提取、操作和分析数据应该足以完成大多数数据科学家的任务。你经常会发现，你只使用 SQL 在将数据加载到 Jupyter Notebook 之前，然后使用 [Pandas](https://pandas.pydata.org/pandas-docs/stable/index.html) 实施一些探索性数据分析（EDA）。

本文的目的是讨论 SQL 语法的基础，讨论 SQL 最佳实践，以及可供你练习 SQL 技能的资源。

## 什么是 SQL？

SQL 是一种为管理和操作关系数据库而创建的特定领域语言。SQL 已被数据科学家以及大多数数据专业人士广泛采用，成为与数据库交互时的首选语言。

SQL 的缩写代表：

+   **结构化**：数据以有组织的状态存储，与未结构化数据（例如音频、视频、文本）不同。

+   **查询**：用户如何与数据库交流，通过编写 SQL 查询提取他们所需的信息。

+   **语言**：SQL 是一种编程语言，设计得非常用户友好且易于阅读，与一些传统编程语言不同。

SQL 有许多不同的[风味](https://www.nobledesktop.com/learn/sql/what-is-sql-and-how-do-we-use-it)，比较不同风味的主要区别在于它们是付费服务还是免费服务。多年来，已经发布了几个 SQL 的开源版本，其中最受欢迎的是 [MySQL](https://www.mysql.com/) 和 [PostgreSQL](https://www.postgresql.org/)。

根据我的经验，[Transact-SQL](https://en.wikipedia.org/wiki/Transact-SQL)（通过 MS SQL Server）、[GoogleSQL](https://cloud.google.com/bigquery/docs/introduction-sql#:~:text=BigQuery%20supports%20the%20GoogleSQL%20dialect,the%20broadest%20range%20of%20functionality.)（通过 BigQuery）和 PostgreSQL 是最受欢迎的。如果我从零开始，我会专注于 Transact-SQL（通过 MS SQL Server），因为大多数教程都涵盖了这种 SQL 风味。

*有关 SQL 的更多信息，请参阅[这里](https://en.wikipedia.org/wiki/SQL)。*

## SQL 基础知识

一些职业，例如数据工程师和数据库管理员（DBAs），需要具备高级的 SQL 知识，但对于数据科学家来说并非如此。随着经验的积累，你会发现编写 SQL 脚本变得相当重复，大多数时候你只是在复制之前的脚本并进行一些小的修改。

大多数数据科学家都会使用 SQL 在导入 Python 环境之前执行基本的数据转换。我将为你提供作为数据科学家日常 90%的 SQL 相关任务所需的所有基本命令。

### 选择数据

最重要的 SQL 命令是`SELECT`，这个命令允许你定义你想要从查询中指定的表中选择的列。

```py
select
    order_date,
    product_sku,
    order_quantity
from
    my_store.ecommerce.orders
```

列可以单独声明，或者你可以使用星号（*）表示你想要选择该表中所有列。

上面的查询将选择`my_store.ecommerce.orders`表中的所有行，无论是否存在重复行。为了避免这种情况，你可以使用`DISTINCT`命令来仅返回唯一的行。

```py
select distinct
    order_date,
    product_sku,
    order_quantity
from
    my_store.ecommerce.orders
```

### 数据工程

有时候你的表中没有你想要的列，但可能包含创建该列所需的基础数据。使用类似`CASE`命令的东西，你可以在 SQL 查询中创建自己的特征。

```py
select distinct
    order_date,
    product_sku,
    order_quantity,
    case when order_quantity >= 5 then "High"
         when order_quantity between 3 and 5 then "Medium"
         else "Low" end as order_quantity_status
from
    my_store.ecommerce.orders
```

在上面的查询中，我们根据`order_quantity`列中的值创建了`order_quantity_status`列。`CASE`命令充当`IF-ELSE`语句，类似于你可能在其他编程语言中遇到的东西。

> 注意：除了使用`CASE`来创建新特征之外，还有许多替代方法。关于这些方法的更多信息，请参阅本文底部的学习资源。

### 分组和排序数据

这些子句非常直观，`GROUP BY`子句用于聚合列，而`ORDER BY`子句用于在输出中排序列。

```py
select
    order_date,
    count(distinct product_sku) as distinct_product_count
from
    my_store.ecommerce.orders
group by
    order_date
order by
    count(distinct product_sku) desc
```

在上面的查询中，我们按`order_date`分组，并计算每天售出的独特产品数量。在计算完这个聚合后，我们按新创建的`distinct_product_count`列降序返回输出。

### 过滤数据

遇到规模达到数 TB 的数据库表并不罕见。为了降低处理成本和时间，在查询中包含过滤条件是必不可少的。

```py
select
    order_date,
    product_sku,
    order_quantity
from
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
```

在查询中包含`WHERE`子句可以使你利用分区和/或索引的优势。通过减少查询需要处理的数据量，你的查询将以极低的成本运行得更快。你的数据工程师和数据库管理员会感谢你的！

不仅`WHERE`子句可以用于过滤日期，它还可以应用于你表中的任何列。例如，如果我们只想包括 SKU100、SKU123 和 SKU420，并且只想看到这些产品的订单数量少于 3 的订单，我们可以使用以下查询：

```py
select
    order_date,
    product_sku,
    order_quantity
from
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
    and product_sku in ("SKU100", "SKU123", "SKU420")
    and order_quantity < 3
```

> 注意：也要花时间查看`HAVING`子句，这是使用聚合列值进行过滤的另一种方法。以下查询通过仅返回每日总和大于或等于 100 的订单日期和订单总数来演示这一点。

```py
select
    order_date,
    sum(order_quantity) as total_orders
from
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
group by
    order_date
having
    sum(order_quantity) >= 100
```

### 数据连接

最广泛采用的数据库设计模式是[星型模式](https://www.databricks.com/glossary/star-schema)，它使用事实表和维度表。事实表包含定量数据，如指标和度量，而维度表提供更多描述性信息，为事实表中的信息提供进一步上下文。

作为数据科学家，你的责任是确定所需数据所在的表。其次，你必须执行正确的连接来合并这些表。

```py
select
    o.order_date,
    o.product_sku,
    o.order_quantity,
    p.product_name,
    p.product_weight
from
    my_store.ecommerce.orders o
inner join
    my_store.ecommerce.product_details p
on
    o.product_sku = p.product_sku
where
    o.order_date >= "2024-12-01"
```

在上述查询中，我们正在对`product_sku`列执行`INNER JOIN`。`INNER JOIN`将返回所有成功在`product_details`表中识别出`product_sku`的订单行。

注意分配给每个表的别名非常重要，多个表具有相同的列名并不罕见。通过使用别名，你可以明确指出你正在引用的特定列。

> 注意：确保你花时间研究替代连接，例如`LEFT JOIN`、`RIGHT JOIN`和`FULL OUTER JOIN`。对于视觉学习者，可以查看关于 SQL 连接的[这个链接](https://www.w3schools.com/sql/sql_join.asp)。

### 数据聚合

在使用 SQL 时，你应该非常熟悉聚合列。你将最频繁使用的常见命令是`COUNT()`、`SUM()`、`MIN()`、`MAX()`和`AVG()`。

```py
select
    count(product_sku) as product_count,
    sum(order_quantity) as total_orders,
    min(order_quantity) as minimum_orders,
    max(order_quantity) as maximum_orders,
    avg(order_quantity) as average_orders
from
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
```

这些聚合函数用于从你的数据中生成描述性统计。虽然这可以使用 Python 完成，但我发现使用 SQL 来完成这项任务更有效率，尤其是在即时回答利益相关者问题时。

## 下一步

在掌握基础知识之后，你应该扩展你的知识并专注于中级 SQL。在我日常工作中经常出现的一些常见过程包括[常用表达式表（CTEs）](https://www.atlassian.com/data/sql/using-common-table-expressions#:~:text=A%20Common%20Table%20Expression%20(CTE,focus%20on%20non%2Drecurrsive%20CTEs)和[窗口函数](https://mode.com/sql-tutorial/sql-window-functions)。

### CTEs

由于我大多数 SQL 都是通过 GCP 上的 [BigQuery](https://cloud.google.com/bigquery?hl=en) 来使用的，我在几乎所有的查询中都使用 CTE。CTE 允许你创建可以稍后作为更广泛、更大 SQL 查询一部分的临时表。

```py
with total_product_orders_daily as 
(
  select
    order_date,
    product_sku,
    sum(order_quantity) as total_orders
  from  
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
)

select
    tpod.order_date,
    tpod.product_sku,
    p.product_name,
    tpod.total_orders
from
    total_product_orders_daily tpod
inner join
    my_store.ecommerce.product_details p
on
    tpod.product_sku = p.product_sku
```

上面的查询首先创建了一个 CTE，计算 `total_orders`，然后将其与 `total_product_orders_daily` 表连接到 `my_store.ecommerce.product_details` 表。此外，请注意，`WHERE` 子句尽可能早地在 CTE 中出现，你应该始终尽可能早地减少你正在处理的数据量。

### 窗口函数

窗口函数对与当前行相关的一组行执行计算，每一行保持一个独立的身份。例如，如果你想对你的数据进行排名或识别重复记录，你可以通过实现窗口函数来完成这个操作。

```py
select
    order_date,
    product_sku,
    order_quantity,
    rank() over (partition by order_date, product_sku order by order_quantity desc) as daily_sku_order_rank
from
    my_store.ecommerce.orders
where
    order_date >= "2024-12-01"
```

上面的查询创建了一个名为 `daily_sku_order_rank` 的列，该列按 `order_date` 对每个 `product_sku` 进行降序排名。

要使用窗口函数来识别和删除重复记录，你可以使用以下代码：

```py
with base_table as 
(
  select
    order_date,
    product_sku,
    order_quantity,
    row_number() over (partition by order_date, product_sku) as daily_sku_row_num
  from
    my_store.ecommerce.orders
  where
    order_date >= "2024-12-01"
)

select
    order_date,
    product_sku,
    order_quantity,
from
    base_table
where
  daily_sku_order_rank = 1
```

对于 `daily_sku_order_rank` 大于 1（重复记录）的情况，这些记录将在 CTE 执行并生成输出时被删除。

> 注意：在执行窗口函数时，如 `_` `DENSE_RANK` `_`，还有更多可用的函数，更多信息请在此处查看。

## SQL 最佳实践

与其他编程脚本类似，在编写 SQL 脚本时，你应该始终考虑其他人可能会重用你的代码。为了使这个过程更容易，最好遵循一些 SQL 最佳实践。一些突出的最佳实践包括：

1.  **使用有意义的命名约定：** 更好的做法是使用更长且更具描述性的列/表命名约定。

1.  **代码格式化：** 在整个脚本中保持一致的缩进。关于文本的大小写没有正确或错误之分，选择一个并坚持下去。

1.  **避免选择所有列：** 选择你想要包含在输出中的特定列，从表中选择时不要使用星号。

1.  **索引列：** 在 `WHERE`、`JOIN` 或 `ORDER BY` 子句中频繁使用的列应该被索引，因此可以优化查询性能。

1.  **函数的使用位置：** 与你应该索引列的位置类似，你也应该不在 `WHERE`、`JOIN` 或 `ORDER BY` 子句中使用任何函数（例如 `CAST()`、`LEN()`）。这也适用于通配符。

> 注意：除了上述提到的之外，还有更多的 SQL 最佳实践，这有时可能取决于你使用的 SQL 类型。我鼓励你向公司内部咨询，看看是否已经建立了任何内部 SQL 最佳实践，你可以将其应用于你的工作中。

## SQL 实践资源

当你在专业环境中使用真实数据时，你的 SQL 开发将始终以更快的速度进步。对于尚未找到第一份工作的有志于成为数据科学家的人来说，有许多在线替代方案可以帮助他们保持和提升 SQL 技能。

我发现的一些学习 SQL 的顶级资源包括：

> [**W3Schools.com**](https://www.w3schools.com/sql/default.asp)
> 
> [**Solve SQL Code Challenges**](https://www.hackerrank.com/domains/sql)
> 
> [**Master Coding for Data Science**](https://www.stratascratch.com/)
> 
> [**SQLZoo**](https://sqlzoo.net/wiki/SQL_Tutorial)
> 
> [**SQLBolt – Learn SQL – Introduction to SQL**](https://sqlbolt.com/)
> 
> [**DataLemur – Ace the SQL & Data Science Interview**](https://datalemur.com/)

个人而言，我认为**StrataScratch**是最好的选择，因为它允许你选择不同的 SQL 版本，拥有丰富的题目选择，并且界面友好（类似于[LeetCode](https://leetcode.com/))）。

对于更理论性的学习，我会选择**W3Schools**。我在刚开始学习 SQL 时就开始阅读这个资源，它总是存在于我的书签中，以便我在需要时刷新特定主题的记忆。

我建议的一点是不要花太多时间试图找到正确的资源，选择一个，然后开始解决挑战。从基础任务开始，逐步提升，保持耐心和学习的连贯性。在你被认为面试准备就绪之前，你不需要完成所有困难挑战，随着你的进步，你的信心将会增长。

> 注意：这些资源中的一些是免费的，而其他一些有免费层，但其中一些内容位于付费墙后。

## 最后的想法

所有数据科学家都应该至少具备 SQL 的基础知识。不幸的是，SQL 在学术层面没有得到应有的认可，这通常导致毕业生在尝试获得第一份数据科学家职位时缺乏技能。

这种语言并不是最吸引人的，与学习 Python 相比，它通常被描述为相当无聊。只有当你开始在一个专业环境中工作时，你才会理解 SQL 在你的职业生涯中有多么重要。

不仅网上有大量的免费资源可以教你，而且还有一群工程师和科学家在线讨论 SQL 的最佳实践。

抽出时间学习 SQL，在职业生涯早期掌握这项技能无疑会使你在竞争中脱颖而出。

* * *

***免责声明：我与此文中讨论的任何公司、软件或产品都没有任何关联。此外，除非另有说明，本文中包含的所有图片均为作者所有。*

* * *

*如果你喜欢阅读这篇文章，请关注我在 Medium、[X](https://twitter.com/marccodess)和[GitHub](https://github.com/marccodess)上的账号，以获取更多关于数据科学、人工智能和工程的相关内容。*

*祝您学习愉快！🚀*
