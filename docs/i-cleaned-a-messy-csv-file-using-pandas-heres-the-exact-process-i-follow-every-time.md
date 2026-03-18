# 我使用 Pandas 清理了一个混乱的 CSV 文件。以下是每次我遵循的精确过程。

> 原文：[`towardsdatascience.com/i-cleaned-a-messy-csv-file-using-pandas-heres-the-exact-process-i-follow-every-time/`](https://towardsdatascience.com/i-cleaned-a-messy-csv-file-using-pandas-heres-the-exact-process-i-follow-every-time/)

<mdspan datatext="el1764184178327" class="mdspan-comment">通常，数据并不</mdspan> 完美。你将遇到很多数据不一致性。空值、负值、字符串不一致等。如果你在数据分析工作流程的早期没有处理这些问题，那么查询和分析你的数据将会很痛苦。

现在，我之前使用 SQL 和 Excel 进行过数据清洗，但不是用 Python。因此，为了学习 Pandas（Python 的数据分析库之一），我将尝试进行一些数据清洗。

在这篇文章中，我将与你分享一个可重复的、适合初学者的数据清洗工作流程。到文章结束时，你应该对使用 Python 进行数据清洗和分析非常有信心。

## 我们将工作的数据集

我将使用一个包含典型现实世界错误的合成、混乱的人力资源数据集进行工作（不一致的日期、混合的数据类型、复合列）。这个数据集来自 Kaggle，旨在练习数据清洗、转换、探索性分析和预处理，用于数据可视化和机器学习。

该数据集包含超过 1,000 行和 13 列，包括员工信息，如姓名、部门-地区组合、联系详情、状态、薪资和绩效评分。它包括以下示例：

+   重复项

+   缺失值

+   日期格式不一致

+   错误条目（例如，非数值薪资值）

+   复合列（例如，“Department_Region”如“Cloud Tech-Texas”可以分割）

它包含以下列：

+   Employee_ID: 唯一的合成 ID（例如，EMP1001）

+   First_Name, Last_Name: 随机生成的个人姓名

+   Name: 全名（可能与 first/last 重复）

+   Age: 包含缺失值

+   Department_Region: 复合列（例如，“HR-Florida”）

+   Status: 员工状态（Active, Inactive, Pending）

+   Join_Date: 格式不一致（YYYY/MM/DD）

+   Salary: 包含无效条目（例如，“N/A”）

+   Email, Phone: 合成联系信息

+   Performance_Score: 性能评级分类

+   Remote_Work: 布尔标志（True/False）

您可以在此处访问数据集 [here](https://www.kaggle.com/datasets/desolution01/messy-employee-dataset) 并对其进行操作

该数据集是完全合成的。它不包含任何真实个人的数据，可以安全用于公共、学术或商业项目。

该数据集在 CC0 1.0 通用许可下属于公共领域。您可以自由使用、修改和分发，不受限制。

## 清洗工作流程概述

我将工作的数据清洗工作流程包括 5 个简单阶段。

1.  加载

1.  检查

1.  清洗

1.  审查

1.  导出

让我们更深入地了解这些每个阶段。

## 第一步—加载 CSV（并处理第一个隐藏问题）

在加载数据集之前有一些事情需要考虑。然而，这是一个可选步骤，我们可能不会在我们的数据集中遇到这些问题中的大多数。但了解这些事情没有坏处。以下是加载时需要考虑的一些关键事项。

### 编码问题（`utf-8`，`latin-1`）

编码定义了字符如何在文件中以字节的形式存储。Python 和 Pandas 通常默认为 UTF-8，它可以处理全球大多数现代文本和特殊字符。然而，如果文件是在较旧的系统或非英语环境中创建的，它可能使用不同的编码，最常见的是 Latin-1

所以如果你尝试用 UTF-8 读取 Latin-1 文件，Pandas 将遇到它不识别为有效 UTF-8 序列的字节。当你尝试读取有编码问题的 CSV 时，你通常会看到 UnicodeDecodeError。

如果默认加载失败，你可以尝试指定不同的编码：

```py
# First attempt (the default)
try:
df = pd.read_csv(‘messy_data.csv’)
except UnicodeDecodeError:
# Second attempt with a common alternative
df = pd.read_csv(‘messy_data.csv’, encoding=’latin-1')
```

### 错误的分隔符

CSV 代表“逗号分隔值”，但在现实中，许多文件使用其他字符作为分隔符，如分号（在欧洲很常见）、制表符，甚至是管道（|）。Pandas 通常默认为逗号（,）。

因此，如果你的文件使用分号（;）作为分隔符，但你用默认的逗号分隔符加载它，Pandas 会将整行视为一个单独的列。结果将是一个包含整行数据的单列 DataFrame，这使得无法进行操作。

解决方案很简单。你可以尝试检查原始文件（在 VS Code 或 Notepad++ 等文本编辑器中打开它）以查看什么字符分隔值。然后，将那个字符传递给 sep 参数，如下所示

```py
# If the file uses semicolons
df = pd.read_csv('messy_data.csv', sep=';')

# If the file uses tabs (TSV)
df = pd.read_csv('messy_data.csv', sep='\t')
```

### 导入错误的列

有时候，Pandas 会根据前几行猜测列的数据类型，但后续行包含意外的数据（例如，文本混合到以数字开始的列中）。

例如，Pandas 可能正确地将 0.1、0.2、0.3 识别为浮点数，但如果第 100 行包含值 N/A，Pandas 可能会强制整个列变为对象（字符串）类型以适应混合值。这很糟糕，因为直到清理掉这些错误值，你将失去在该列上执行快速、矢量化数值操作的能力。

为了解决这个问题，我使用 dtype 参数告诉 Pandas 列应该显式地是什么数据类型。这防止了静默类型转换。

```py
df = pd.read_csv(‘messy_data.csv’, dtype={‘price’: float, ‘quantity’: ‘Int64’})
```

### 读取前几行

你可以通过在加载过程中直接检查前几行来节省时间，使用 `nrows` 参数。这很好，尤其是在处理大型数据集时，因为它允许你在不加载整个 10 GB 文件的情况下测试编码和分隔符。

```py
# Load only the first 50 rows to confirm encoding and delimiter
temp_df = pd.read_csv('large_messy_data.csv', nrows=50)
print(temp_df.head())
```

一旦你确认了参数是正确的，你就可以加载完整文件。

让我们加载 Employee 数据集。我不期望在这里看到任何问题。

```py
import pandas as pd
df = pd.read_csv(‘Messy_Employee_dataset.csv’)
df
```

**输出**:

```py
1020 rows × 12 columns
```

现在我们可以继续到**第二步：检查**

## 第二步—检查数据集

我将这个阶段视为法医审计。我在寻找隐藏在表面下的混乱证据。如果我急于进行这一步，我保证自己会在后续的分析中遇到很多痛苦和错误。我在编写任何转换代码之前，总是运行这四个关键的检查。

以下方法为我提供了 1,020 名员工记录的完整健康报告：

### `1. df.head() 和 df.tail()`: 理解边界

我总是从视觉检查开始。我使用`df.head()`和`df.tail()`来查看前五行和后五行。这是我快速检查所有列是否对齐以及数据是否直观合理的快速检查。

**我的发现：**

当我运行`df.head()`时，我注意到我的`员工 ID`位于一个列中，并且 DataFrame 正在使用默认的数值索引**（0, 1, 2, ...）**。

虽然我知道我可以将“员工 ID”设置为索引，但到目前为止，我会保留它。我目前寻找的更大的视觉风险是数据在错误列中错位或名称中明显的首尾空格，这可能会在以后造成麻烦。

### `2. df.info()`: 检查数据类型问题和缺失值

这是最重要的方法。它告诉我列名、数据类型（`Dtype`）以及非空值的准确数量。

**关于 1,020 行数据的发现：**

+   **缺失年龄**：我的总条目数是**1,020**，但`年龄`列只有**809**个非空值。这是一大块缺失数据，我必须决定如何处理——是插补它，还是删除这些行？

+   **缺失薪资**：`薪资`列有**996**个非空值，这只是一个小的差距，但我必须解决。

+   **ID 类型检查**：`员工 ID`被正确地列为`object`（字符串）。这并不正确。ID 是标识符，不是要平均的数字，使用字符串类型可以防止 Pandas 意外地去除前导零。

### 3. 数据完整性检查：重复项和唯一计数

在检查数据类型之后，我需要知道是否有重复记录以及我的分类数据的一致性如何。

+   **检查重复项**：我运行了`df.duplicated().sum()`并得到了**0**的结果。这太完美了！这意味着我没有重复的行在我的数据集中造成混乱。

+   **检查唯一值（`df.nunique()`）**：我使用这个来了解每个列内的多样性。分类列中的低计数是可以接受的，但我寻找那些应该是唯一的但不是的列，或者那些有**太多**唯一值的列，这表明可能存在错误。

+   **“员工 ID”** 有 1,020 个唯一记录。这太完美了。这意味着所有记录都是唯一的。

+   **“姓名/姓氏字段有八个唯一记录。”** 这有点奇怪。这证实了数据集的合成性质。我的分析不会因为大量不同的名字而偏颇，因为我将它们视为标准字符串。

+   **部门 _ 地区**有 36 条唯一记录。地区/部门有 36 个唯一值表明存在大量拼写错误。我将在下一步检查此列的拼写变体（例如，“HR”与“人力资源”）。

+   **电子邮件（64 条唯一记录）。在 1020 名员工中，只有 64 条唯一电子邮件表明许多员工共享相同的占位符电子邮件。我将将其标记为排除在分析之外，因为它对识别个人没有用。**

+   **电话（1020 条唯一记录）。这是完美的，因为它证实电话号码是唯一的标识符。

+   **年龄/绩效评分/状态/远程工作**（2-4 条唯一记录）。对于分类或有序数据，这些低计数是预期的，这意味着它们可以编码。

### `4. df.describe()`: 检测异常和不合理值

我使用`df.describe()`来获取所有数值列的统计摘要。这是真正不可能的值——“红旗”——立即出现的地方。我主要关注`min`和`max`行。

**我的发现：**

我立即注意到了一个在**预期**的电话号码列中的问题，Pandas 错误地将其转换为数值类型。

```py
Mean
-4.942253 * 10⁹
Min
-9.994973 * 10⁹
Max
-3.896086 * 10⁶
25%
-7.341992e * 10⁹
50%
4.943997 * 10⁹
75%
-2.520391e * 10⁹
```

结果表明，所有电话号码的值都是巨大的负数！这证实了两件事：

Pandas 错误地将此列推断为数值类型，尽管电话号码是字符串。

文本中必须有 Pandas 无法解释的字符（例如，括号、破折号或国家代码）。我需要将其转换为对象类型并彻底清理。

### 5. `df.isnull().sum()`: 缺失数据的量化

虽然`df.info()`给出了非空值的计数，但`df.isnull().sum()`给出了空值的总数，这是一种更干净的方式来量化我的下一步。

**我的发现：**

+   `年龄`有**211**个空值（1020 - 809 = 211），并且

+   `薪水`有**24**个空值（1020 - 996 = 24）。这个精确的计数为**步骤 3**奠定了基础。

这个检查过程是我的安全网。如果我错过了**负电话号码**，任何涉及数值数据的分析步骤都会失败，或者更糟糕的是，会无预警地产生偏差的结果。

通过确定需要将`电话号码`视为字符串以及`年龄`中的显著缺失值**现在**，我有一个具体的清理列表。这防止了运行时错误，并且至关重要，确保我的最终分析基于合理、非损坏的数据。

## 第 3 步——标准化列名、纠正数据类型和处理缺失值

在手头有一系列缺陷（缺失年龄、缺失薪水、糟糕的负电话号码和混乱的分类数据）的情况下，我进入重头戏。我将这一步骤分为三个子阶段：确保一致性、修复损坏和填补空白。

### 1. 标准化列名和设置索引（一致性规则）

在进行任何严肃的数据操作之前，我对列名执行严格的检查。为什么？因为不小心输入 `df['Employee ID ']` 而不是 `df['employee_id']` 是一个无声的、令人沮丧的错误。一旦名称变得干净，我就设置索引。

我的黄金规则是 **snake_case 和全小写，ID 列应该是索引**。

我使用一个简单的命令来删除空白字符，将空格替换为下划线，并将所有内容转换为小写。

```py
# The Standardization Command
df.columns = df.columns.str.lower().str.replace(' ', '_').str.strip()
# Before: ['Employee_ID', 'First_Name', 'Phone']
# After: ['employee_id', 'first_name', 'phone']
```

现在我们已经标准化了列，我可以继续将 employee_id 设置为索引。

```py
# Set the Employee ID as the DataFrame Index
# This is crucial for efficient lookups and clean merges later.
df.set_index('employee_id', inplace=True)

# Let’s review it real quick
print(df.index)
```

**输出：**

```py
Index(['EMP1000', 'EMP1001', 'EMP1002', 'EMP1003', 'EMP1004', 'EMP1005',
'EMP1006', 'EMP1007', 'EMP1008', 'EMP1009',
...
'EMP2010', 'EMP2011', 'EMP2012', 'EMP2013', 'EMP2014', 'EMP2015',
'EMP2016', 'EMP2017', 'EMP2018', 'EMP2019'],
dtype='object', name='employee_id', length=1020)
```

完美，一切就绪。

### 2. 修复数据类型和损坏（处理负电话号码）

我的 `df.describe()` 检查揭示了最紧迫的结构缺陷：`Phone` 列被导入为垃圾数值类型。由于电话号码是 *标识符*（不是数量），它们必须是字符串。

在这个阶段，我将整个列转换为字符串类型，这将把所有那些负的科学记数法数字转换为可读文本（尽管仍然充满了非数字字符）。我将实际的文本清理（删除括号、破折号等）留给专门的标准化步骤（第 4 步）。

```py
# Fix the Phone dtype immediately
# Note: The column name is now 'phone' due to standardization in 3.1
df['phone'] = df['phone'].astype(str)
```

### 3. 处理缺失值（年龄和薪资差距）

最后，我解决 `df.info()` 揭示的差距：**211** 个缺失的 `Age` 值和 **24** 个缺失的 `Salary` 值（总共 **1,020 行**）。我的策略完全取决于列的角色和缺失数据的量级：

+   **薪资（24 个缺失值）**：在这种情况下，删除或丢弃所有缺失值将是最佳策略。薪资是财务分析的一个关键指标。插补它可能会歪曲结论。由于只有一小部分（2.3%）缺失，我选择删除不完整的记录。

+   **年龄（211 个缺失值）**。在这里，填充缺失值是最佳策略。年龄通常是预测建模（例如，离职率）的一个特征。丢弃 20% 的数据代价太大。我将使用 **中位数年龄** 来填充缺失值，以避免使用平均值导致分布偏斜。

我使用两个独立的命令执行此策略：

```py
# 1\. Removal: Drop rows missing the critical 'salary' data
df.dropna(subset=['salary'], inplace=True)

# 2\. Imputation: Fill missing 'age' with the median
median_age = df['age'].median()
df['age'].fillna(median_age, inplace=True)
```

在这些命令之后，我会再次运行 `df.info()` 或 `isnull().sum()` 以确认 `salary` 和 `age` 的非空计数现在反映了干净的数据集。

```py
# Rechecking the null counts for salary and age
df[‘salary’].isnull().sum())
df[‘age’].isnull().sum())
```

**输出：**

```py
np.int64(0)
```

到目前为止一切顺利！

通过解决这里的结构和缺失数据问题，后续步骤可以完全专注于值标准化，例如 `department_region` 中的混乱的 **36 个唯一值**——我们将在下一阶段解决。

## 第 4 步—值标准化：使数据一致

我的 DataFrame 现在具有正确的结构，但内部值仍然很脏。这一步是关于一致性。如果“IT”、“i.t”和“Info. Tech”都代表同一个部门，我需要将它们强制转换为单个、干净的值（“IT”）。这可以防止在分组、过滤和基于类别的任何统计分析中出错。

### 1. 清理损坏的字符串数据（电话号码修复）

记得第 2 步中损坏的`phone`列吗？它目前是一堆负的科学记数法数字，我们在第 3 步中将其转换为字符串。现在，是提取实际数字的时候了。

因此，我将移除所有非数字字符（破折号、括号、点等），并将结果转换为干净、统一的格式。正则表达式（`.str.replace()`）非常适合这个任务。我使用`\D`来匹配任何非数字字符，并将其替换为空字符串。

```py
# The phone column is currently a string like '-9.994973e+09'
# We use regex to remove everything that isn't a digit
df['phone'] = df['phone'].str.replace(r'\D', '', regex=True)

# We can also truncate or format the resulting string if needed
# For example, keeping only the last 10 digits:
df['phone'] = df['phone'].str.slice(-10)
print(df['phone'])
```

**输出：**

```py
employee_id
EMP1000 1651623197
EMP1001 1898471390
EMP1002 5596363211
EMP1003 3476490784
EMP1004 1586734256
...
EMP2014 2470739200
EMP2016 2508261122
EMP2017 1261632487
EMP2018 8995729892
EMP2019 7629745492
Name: phone, Length: 996, dtype: object
```

现在看起来好多了。清理包含噪声的标识符（如带有前导字符的 ID 或带有扩展的邮编）始终是一个好习惯。

### 2. 分离和标准化分类数据（修复 36 个区域）

我的`df.nunique()`检查在`department_region`列中发现了**36 个唯一值**。当我审查该列中的所有唯一值时，输出显示它们都整齐地结构化为`department-region`（例如，devops-california，finance-texas，cloud tech-new york）。

我认为解决这个问题的方法之一是将这个单独的列拆分为两个专用列。我将在连字符（`-`）处拆分该列，并将部分分配给新列：`department`和`region`。

```py
# 1\. Split the combined column into two new, clean columns
df[['department', 'region']] = df['department_region'].str.split('-', expand=True)
Next, I’ll drop the department_region column since it’s pretty much useless now
# 2\. Drop the redundant combined column
df.drop('department_region', axis=1, inplace=True)
Let’s review our new columns
print(df[[‘department’, ‘region’]])
```

**输出：**

```py
department region
employee_id
EMP1000 devops california
EMP1001 finance texas
EMP1002 admin nevada
EMP1003 admin nevada
EMP1004 cloud tech florida
... ... ...
EMP2014 finance nevada
EMP2016 cloud tech texas
EMP2017 finance new york
EMP2018 hr florida
EMP2019 devops illinois

[996 rows x 2 columns]
```

在拆分后，新的`department`列只有**6 个唯一值**（例如，‘devops’，‘finance’，‘admin’等）。这是一个好消息。这些值已经标准化，并准备好进行分析！我认为我们总是可以将所有类似的部门映射到单个类别。但我会跳过这一点。我不想在这篇文章中过于深入。

### 3. 转换日期列（修复 Join_Date）

`Join_Date`列通常以字符串（`object`）类型读取，这使得时间序列分析变得不可能。这意味着我们必须将其转换为合适的 Pandas `datetime`对象。

`pd.to_datetime()`是核心函数。我经常使用`errors='coerce'`作为安全网；如果 Pandas 无法解析日期，它将那个值转换为`NaT`（Not a Time），这是一个干净的空值，防止整个操作崩溃。

```py
# Convert the join_date column to datetime objects
df['join_date'] = pd.to_datetime(df['join_date'], errors='coerce')
```

日期的转换使得强大的时间序列分析成为可能，例如计算平均员工任期或按年份识别离职率。

在这一步之后，数据集中的每个值都是干净的、统一的，并且格式正确。分类列（如`department`和`region`）已准备好进行分组和可视化，数值列（如`salary`和`age`）已准备好进行统计分析。数据集正式准备好进行分析。

## 第 5 步—最终质量检查和导出

在关闭笔记本之前，我总是进行最后一次审计，以确保一切完美，然后导出数据，以便我可以在以后进行分析。

### 最终数据质量检查

这很快。我重新运行了两个最重要的检查方法，以确认所有我的清理命令实际上都起作用了：

+   `df.info()`**:** 我确认关键列（`年龄`, `薪水`）中**没有更多缺失值**，并且数据类型正确（`电话`是字符串类型，`加入日期`是日期时间类型）。

+   `df.describe()`**:** 我确保统计摘要显示的是合理的数字。现在应该从输出中移除`电话`列（因为它是一个字符串），而`年龄`和`薪水`应该有合理的最小值和最大值。

如果这些检查都通过了，我知道数据是可靠的。

### 导出清洗后的数据集

最后一步是保存这个清洗后的数据版本。我通常将其保存为新的 CSV 文件，以保持原始杂乱文件完好无损以供参考。如果我不想将`员工 ID`（现在是索引）保存为单独的列，我会使用`index=False`，如果我想将索引保存为新 CSV 文件中的第一列，我会使用`index=True`。

```py
# Exporting the clean DataFrame to a new CSV file
# We use index=True to keep our primary key (employee_id) in the exported file
df.to_csv('cleaned_employee_data.csv', index=True)
```

通过导出具有清晰、新文件名的文件（例如，`_clean.csv`），你正式标记了清洗阶段的结束，并为项目的下一阶段提供了一个干净的起点。

## 结论

说实话，我过去常常被杂乱的数据集压倒。缺失值、奇怪的数据类型、神秘的列——感觉就像面对白纸综合症。

但这个结构化、可重复的工作流程彻底改变了这一切。通过专注于**加载、检查、清洗、审查和导出**，我们立即建立了秩序：标准化列名，将`员工 ID`设置为索引，并使用智能策略进行插补和拆分杂乱的列。

现在，我可以直接跳到有趣的分析部分，而无需不断质疑我的结果。如果你在初始数据清洗步骤中遇到困难，可以尝试这个工作流程。我很乐意听听你的进展。如果你想玩转这个数据集，可以[从这里](https://www.kaggle.com/datasets/desolution01/messy-employee-dataset)下载。

想要建立联系吗？在这些平台上随意打招呼

[LinkedIn](https://www.linkedin.com/in/ibrahim-salami-059863228/?lipi=urn%3Ali%3Apage%3Ad_flagship3_feed%3BxTBPgk2fTXSRGVGhY8QDdQ%3D%3D)

[Twitter](https://x.com/IbbySalam)

[YouTube](https://www.youtube.com/@ibbysalam)

[Medium](https://medium.com/@ibbysalam)
