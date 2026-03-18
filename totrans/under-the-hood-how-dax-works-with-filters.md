# 内部机制：DAX 如何与过滤器协同工作

> 原文：[`towardsdatascience.com/under-the-hood-how-dax-works-with-filters/`](https://towardsdatascience.com/under-the-hood-how-dax-works-with-filters/)

## <mdspan datatext="el1759192734018" class="mdspan-comment">介绍</mdspan>

让我们从一张简单的表格开始：

![](img/e2f890567aa074b8fe027f73bce7ae89.png)

图 1 – 从简单的表格开始（图由作者绘制）

矩阵视觉中的每一行都显示了每个月的总在线销售额。

到目前为止，一切顺利。

解释是，我们看到按月份过滤的在线销售额总额。

但这并不是全部真相。

让我们看看数据模型：

![](img/2ef0c4115645224fb02b9ebd61194b6a.png)

图 2 – 包含日期表和事实表的数据库模型部分（图由作者绘制）

当你仔细观察时，你会发现两个日期列之间建立了关系。

它与月份列没有关系。

当我们采取这条路线时，上述解释并不完全准确。

完整的解释应该是：每一行显示了通过日期表过滤的在线销售额总额。日期表的行按月份分组。每一行显示了每个月所有天的总销售额。

当我们意识到这个细节时，我们就更接近于理解 DAX 的一般原理，特别是时间智能函数。

让我们更进一步。

## YTD 和基本查询

现在，让我们添加一个 YTD 度量来检查会发生什么：

![](img/ef401daee616456dc55ea63620b03787.png)

图 3 – YTD 度量及其结果与之前相同（图由作者绘制）

这个度量没有什么特别之处，结果也容易理解。

现在，让我们看看[DATESYTD()](https://dax.guide/datesytd/)函数究竟做了什么。

[dax.guide](https://dax.guide/)的解释说：“返回一年中在过滤器上下文中可见的最后一天之前的**日期集”]。

这究竟意味着什么？

要深入了解这个问题，让我们首先编写一个 DAX 查询，以获取 6 月 2024 年的日期列表，如上图所示的可视化中所示：

```py
DEFINE
    VAR YearFilter = TREATAS({ 2024 }, 'Date'[Year])
    VAR MonthFilter = TREATAS({ 6 }, 'Date'[Month])

EVALUATE
    SUMMARIZECOLUMNS('Date'[Date]
                        ,YearFilter
                        ,MonthFilter
                        )
```

结果是 6 月的 30 天列表：

![](img/1f24580d5068c7ab9f3e0c1e32f3ca0c.png)

图 4 – 获取 2024 年 6 月所有天的基本查询（图由作者绘制）

这是应用于上述矩阵中 6 月 2024 年行的过滤器。

当我们将 DATESYTD()函数应用于结果时，会出现什么结果？

这里是查询：

```py
DEFINE
    VAR YearFilter = TREATAS({ 2024 }, 'Date'[Year])
    VAR MonthFilter = TREATAS({ 6 }, 'Date'[Month])

    VAR BasisDates = CALCULATETABLE(
                            SUMMARIZECOLUMNS('Date'[Date]
                                        ,YearFilter
                                        ,MonthFilter
                                        )
                                    )

    VAR YTDDates = DATESYTD(TREATAS(BasisDates, 'Date'[Date])
                                    )

EVALUATE
    YTDDates
```

这里是结果：

![](img/7c2c298543ea3305361ce48083c8344e.png)

图 5 – 从 2024 年 1 月 1 日开始到 6 月最后一天的日期列表（图由作者绘制）

这是一个包含 182 行的列表，包含从年初到 2024 年 6 月最后一天的所有日期。

这就是 YTD 的定义。

当我们查看以下度量：

```py
Online Sales (YTD) =
VAR YTDDates = DATESYTD('Date'[Date])

RETURN
    CALCULATE([Sum Online Sales]
                ,YTDDates
                )
```

我们意识到变量 YTDDates“仅仅”是一个日期列表，作为 CALCULATE()函数的过滤器应用。

这是所有时间智能函数的关键。

## 回退一年 – 一些例子

应用另一个函数到结果上会发生什么？

例如，[SAMEPERIODLASTYEAR()](https://dax.guide/sameperiodlastyear/)？

为了回答这个问题，我使用了以下 DAX 查询：

```py
DEFINE
    VAR YearFilter = TREATAS({ 2024 }, 'Date'[Year])
    VAR MonthFilter = TREATAS({ 6 }, 'Date'[Month])

    VAR BasisDates = CALCULATETABLE(
                            SUMMARIZECOLUMNS('Date'[Date]
                                        ,YearFilter
                                        ,MonthFilter
                                        )
                                    )

    VAR YTDDates = DATESYTD(TREATAS(BasisDates, 'Date'[Date])
                                    )

    VAR YTDDatesPY = SAMEPERIODLASTYEAR(YTDDates)

EVALUATE
    YTDDatesPY
```

我故意将 SAMEPERIODLASTYEAR()的调用与 DATESYTD()分开，以便更容易阅读。本来可以将 DATESYTD()嵌套到 SAMEPERIODLASTYEAR()中。

这次，我们有 181 行，因为 2024 年是闰年。

日期被回退了一年：

![](img/14665aab9576eeaa02b0ceefdefb4fcc.png)

图 6 – 应用 SAMEPERIODLASTYEAR()后的查询结果（作者制图）

所以，再次强调，当我们将时间智能函数应用于度量值时，该函数，例如 DATESYTD()，返回一个日期列表。

请注意：在应用过滤器时，**任何**现有的日期表过滤器都会被移除。

## 自定义逻辑

现在，让我们将这个知识应用到自定义时间智能逻辑中。

首先，让我们稍微改变一下年份和月份的过滤器：

```py
DEFINE
    VAR YearMonthFilter = TREATAS({ 202406  }, 'Date'[MonthKey])

EVALUATE
    SUMMARIZECOLUMNS('Date'[Date]
                        , YearMonthFilter
                        )
```

这个查询的结果与本文开头相同。

这次，我在[MonthKey]列上设置了一个数值过滤器。

我如何回到上一年？

如果你从数学的角度思考，它只是减去 100：

202406 – 100 = 202306

让我们试试：

![](img/000239f4257cd4ac844e1cafe1bfa444.png)

图 7 – 从[MonthKey]列减去 100 后的查询结果（作者制图）

你也可以用其他数字格式来做这件事。

例如，如果你取一个财政年度，比如这样：2425（对于 24/25 财年）

你可以推断出 101 来得到上一年度的财政年：2425 – 101 = 2324

自定义时间智能逻辑的另一个例子是滚动平均，其中对于每一天，我们计算过去 10 天的平均值：

![](img/9581617524a2a5879289fe983c07a078.png)

图 8 – 上十天移动平均度量的代码和结果（作者制图）

因为变量 DateRange 的内容再次是一个日期列表，我可以将 SAMEPERIODLASTYEAR()函数应用到它上面，并得到我需要的结果：

```py
DEFINE
    VAR YearFilter = TREATAS({ 2024 }, 'Date'[Year])
    VAR MonthFilter = TREATAS({ 6 }, 'Date'[Month])

    // 1\. Get the first and last Date for the current Filter Context
    VAR MaxDate = CALCULATE(MAX( 'Date'[Date] )
                            ,YearFilter
                            ,MonthFilter
                            )

    VAR MinDate =
        CALCULATE(
            DATEADD( 'Date'[Date], - 10, DAY )
            ,'Date'[Date] = MaxDate
            )

    // 2\. Generate the Date range needed for the Moving average (Four months)
    VAR  DateRange =
     CALCULATETABLE(
        DATESBETWEEN( 'Date'[Date]
            ,MinDate
            ,MaxDate
        )
    )

EVALUATE
    SAMEPERIODLASTYEAR( DateRange )
```

这就是结果：

![](img/3b07b9ed45bb5cf132e8f4270fff9713.png)

图 9 – 上一年度移动平均的结果（作者制图）

这个逻辑返回 11 行，因为它包括了月份的最后一天。根据所需的结果，我们需要调整计算日期列表（应用于度量值的过滤器）的第一天和最后一天的方式。

当然，这和我上面展示的是重复的。然而，它表明同样的方法可以应用于各种场景。

一旦你理解了这个，你与时间智能函数和其他接受值表作为输入的函数的工作将变得更容易理解和掌握。

## 结论

当我使用 DAX Studio 进行查询时，你可以在 Power BI Desktop 中的 DAX 查询工具中使用相同的查询。

我故意使用这些查询来演示我们在 DAX 中始终与表一起工作，即使我们可能没有意识到这一点。

但这是一个重要的细节，有助于我们理解 DAX。

尽管这里展示的一些 DAX 代码可能随着 Power BI 中新[基于日历的时间智能](https://powerbi.microsoft.com/en-us/blog/calendar-based-time-intelligence-time-intelligence-tailored-preview/)功能的推出而变得过时，但这里解释的原则仍然有效。函数，如 DATESYTD() 或 SAMEPERIODLASTYEAR()，仍然存在并且以与之前相同的方式工作。目前，这一侧不会有任何变化，因为这里描述的概念仍然有效。

## 参考文献

就像我之前的文章一样，我使用了 Contoso 示例数据集。你可以从微软[这里](https://www.microsoft.com/en-us/download/details.aspx?id=18279)免费下载 ContosoRetailDW 数据集。

根据该文档中的描述[在此](https://github.com/microsoft/Power-BI-Embedded-Contoso-Sales-Demo)，Contoso 数据可以在 MIT 许可证下自由使用。我将数据集更改以将数据转移到当代日期。
