# 处理非结构化数据的先进 SQL 技术

> 原文：[`towardsdatascience.com/advanced-sql-techniques-for-unstructured-data-handling-832f3c7c43b9/`](https://towardsdatascience.com/advanced-sql-techniques-for-unstructured-data-handling-832f3c7c43b9/)

![照片由 Etienne Girardet 在 Unsplash 拍摄](img/a5312868d3b94a9a2e3a7b55f87750d5.png)

照片由[Etienne Girardet](https://unsplash.com/@etiennegirardet?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)拍摄

理想的数据分析数据集类似于 Table_1：

![Table_1（作者模拟数据）](img/38c0a07502f6bfb852d26b7f163dfb21.png)

Table_1（作者模拟数据）

然而，我们在现实中遇到的数据集大多类似于 Table_2：

![Table_2：客户支持日志（作者模拟数据）](img/d4ce74a5414c2ff101c6bcde356f4bd8.png)

Table_2：客户支持日志（作者模拟数据）

这两个表格之间的主要区别在于数据是否以行和列的形式组织良好，并且仅以数字或文本形式呈现。由于这些差异，Table_1 中的数据被称为**结构化数据**，而 Table_2 中的数据被归类为**非结构化数据**。

**非结构化数据**指的是没有预定义的结构或格式的信息。它在关系型数据库中难以存储和管理。但它通常包含有价值的信息，这些信息对于生成数据洞察、训练机器学习模型或执行自然语言处理（NLP）非常有用。

在这篇文章中，我将介绍 7 种用于处理非结构化数据的先进 SQL 技术。虽然我们在 SQL 中称这些技术为“先进”，但实际上它们构成了数据解析或文本挖掘的基础。

* * *

## JSON 解析

JSON 数据是“JavaScript 对象表示法”数据的简称。它是一种基于文本的格式，用于在 Web 开发中在服务器和 Web 应用程序之间交换信息。由于它的易用性、平台独立性和灵活性等优势，JSON 数据被广泛使用。但由于其复杂的结构和数据存储的嵌套级别不同，我们无法直接分析和从 JSON 数据中获得洞察。

```py
SELECT JSON_VALUE(json_column, '$.key') AS extracted_value
FROM table_name;
```

在 Table_2 中，`customer_data`列是 JSON 数据，可以转换为两个列`name`和`age`。

```py
SELECT JSON_VALUE(customer_data, '$.name') AS name, JSON_VALUE(customer_data, '$.age') AS age
FROM support_logs;
```

![JSON 解析的输出（作者截图）](img/4c64138e6058f31cbdb335a4c93e8e32.png)

JSON 解析的输出（作者截图）

* * *

## 正则表达式

正则表达式（regex）是一系列用于定义搜索模式的字符。正则表达式允许开发人员通过在文本中匹配特定模式来在数据库中查找和操作复杂的字符串数据。SQL 中正则表达式的语法如下：

```py
SELECT column_name
FROM table_name
WHERE column_name REGEXP 'pattern';
```

正则表达式广泛用于灵活搜索、数据验证和数据提取。一个典型的用例是从文本中**提取电子邮件**。

```py
SELECT column_name
FROM users
WHERE column_name REGEXP '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+.[a-zA-Z]{2,}';
```

有时人们容易混淆 `REGEXP` 操作符和 `LIKE` 操作符。这两个操作符都用于 SQL 中在字符串内执行模式匹配。但 `LIKE` 只支持基本通配符，而 `REGEXP` 允许使用正则表达式语法进行复杂模式，由于其更健壮的语法，它为高级模式匹配场景提供了更大的灵活性。同时，正则表达式需要更多了解原始数据结构，这可能具有挑战性。

* * *

## 键值对解析

当使用 JSON 解析时，格式相对标准化。如果字符串数据具有更复杂的结构，但键和值由冒号、分号或等号等分隔符分隔，JSON 解析可能不足以进行数据提取。相反，我们应该使用 **键值对解析**。

**键值对解析**是将存储在以“键”与其对应“值”配对格式的数据中提取和分离数据的过程。这种方法通过字符串函数或正则表达式（regex）实现。

常用的字符串函数，如 `SUBSTRING`、`SUBSTRING_INDEX`、`POSITION` 和 `REPLACE`，通常用于根据分隔符提取键和值部分。以下是一个使用 `SUBSTRING_INDEX` 函数来配对键值对的示例。

```py
SELECT SUBSTRING_INDEX(SUBSTRING_INDEX(column_name, 'key=', -1), ';', 1) AS value
FROM table_name;
```

* * *

## 使用窗口函数进行文本分析

窗口函数在查询的指定行集（称为“窗口”）上执行计算，并返回与当前行相关的单个值。您可以参考我的另一篇文章 ***‘在技术行业中取得成功的最有用的高级 SQL 技巧’***，以深入了解窗口函数的语法、优势和用例。

> [**在技术行业中取得成功的最有用的高级 SQL 技巧**](https://towardsdatascience.com/the-most-useful-advanced-sql-techniques-to-succeed-in-the-tech-industry-0f0690e8386c)

除了在分区之间计算指标的能力之外，窗口函数还可以用于基于文本的分析，这一点很少有人意识到。

窗口函数在基于文本的分析中的应用非常广泛。例如，如果我们想按长度对文本进行排序，我们可以使用以下语法：

```py
SELECT column_name, RANK() OVER (ORDER BY LENGTH(column_name) DESC) AS rank
FROM table_name;
```

* * *

## 数据标记化

数据标记化是一种安全技术，它将敏感数据，如信用卡号码或社会保险号码，替换为随机的标记。SQL 本身并不直接支持标记化，但它可以通过以下方式与标记化数据协同工作：

+   **查找表**：一个映射表将标记与其原始值关联起来。

+   **加密或哈希函数**：虽然这不是真正的标记化，但这些方法可以混淆数据。

数据标记化通常不被视为清理非结构化数据集的典型方法。但它是文本挖掘中一个重要的技术，尤其是在越来越多的

数据分词通常不被视为清理非结构化数据集的方法。然而，它是文本挖掘的重要技术，尤其是在数据隐私越来越受到侵犯，数据安全成为数据使用中的严重风险时。

* * *

## COALESCE()

COALESCE()是一个函数，它从一系列表达式中返回第一个非空值。它在处理不完整或不一致的数据时非常有用，这在非结构化数据集中非常常见。COALESCE()函数的语法是：

```py
SELECT COALESCE(column1, column2, 'default_value') AS result
FROM table_name;
```

COALESCE()广泛用于替换 null 值，选择第一个可用的值或回退逻辑。

* * *

## CAST()

CAST()将数据从一种类型转换为另一种类型。语法是：

```py
SELECT CAST(column_name AS target_type) AS converted_value
FROM table_name;
```

当使用 CAST()函数时，我们必须谨慎，尤其是在处理包含缺失值（null）的数据时。

```py
SELECT 
    CAST((JSON_VALUE(customer_data, '$.age') AS INT) AS customer_age
FROM 
    support_logs;
```

上述代码将返回错误。这是因为`customer_age`列在解析后包含‘null’，您不能将‘无’转换为具体的整数或字符串等。

```py
SELECT 
    CAST(JSON_VALUE(customer_data, '$.age') AS UNSIGNED) AS customer_age
FROM 
    support_logs;
```

* * *

## 处理非结构化数据的 SQL 示例

让我们重新审视 Table_2：客户支持日志表，并使用上述技术将非结构化数据转换为结构化数据表，以便进行分析。

![Table_2：客户支持日志（作者模拟数据）](img/d4ce74a5414c2ff101c6bcde356f4bd8.png)

Table_2：客户支持日志（作者模拟数据）

下面是我们旨在完成的任务：

1.  从`customer_data`列中提取客户姓名和年龄。

1.  处理`issue_description`列中的缺失值。

1.  从`extra_info`列中提取每个票证的优先级和状态，以帮助 IT 团队优先处理工作负载并跟踪每个票证的状态。

1.  提取用于进一步分析的解决时间（以小时为单位）。

1.  根据问题描述的长度对票证进行排名。

1.  对电话号码进行分词以保护客户隐私。

```py
SELECT 
    ticket_id,
    customer_id,

    -- Extract customer names and ages
    JSON_VALUE(customer_data, '$.name') AS customer_name,
    CAST(JSON_VALUE(customer_data, '$.age') AS UNSIGNED) AS customer_age,

    -- Handle missing issue descriptions
    COALESCE(issue_description, 'No issue reported') AS issue_description,

    -- Extract ticket priority and status
    SUBSTRING_INDEX(REGEXP_SUBSTR(extra_info, 'priority=[^;]+'), '=', -1) AS priority,
    SUBSTRING_INDEX(SUBSTRING_INDEX(extra_info, 'status=', -1), ';', 1) AS status,

    -- Extract hours of ticket resolution
    CAST(REGEXP_REPLACE(NULLIF(resolution_time, 'N/A'), '[⁰-9]', '') AS UNSIGNED) AS resolution_time_hours,

    -- Rank tickets by lengths
    RANK() OVER (ORDER BY LENGTH(issue_description) DESC) AS issue_length_rank,

    -- Tokenize phone number
    CONCAT('TOKEN-', RIGHT(MD5(phone_number), 8)) AS tokenized_phone_number
FROM support_logs;
```

我们可以将原始的非结构化数据转换为没有敏感信息的结构化表，如下所示：

![文本挖掘输出（作者截图）](img/998bde3b3f70e8920550834e9d0fbf6f.png)

文本挖掘输出（作者截图）

* * *

## 结论

SQL 不仅是一种强大的数据检索和操作工具，而且其处理来自各种来源（如日志、电子邮件、网站、社交媒体和移动应用）的文本的语法也非常强大。

非结构化数据可能令人困惑且难以解释，但通过利用 SQL 的相关功能，我们可以提取高度有价值的见解，并将数据科学项目的成功推向新的高度。

感谢阅读！如果您觉得这篇文章有帮助，请给它一些点赞！关注我并通过电子邮件订阅，以便在发布新文章时收到通知。我的目标是帮助数据分析师和数据科学家，无论您是初学者还是有经验的，提高您的技术技能并在职业生涯中取得更大的成功。
